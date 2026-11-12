# AOS Kernel Specification — Sprint 1 scope

| Field | Value |
|---|---|
| Status | **Draft v0.1, revision 3 — NOT APPROVED.** Addresses review findings AR-05…AR-07, AR-16, AR-17, AR-19, AR-21, AR-23 (`DECISIONS.md` §10); all are *addressed in draft — pending independent re-review*. No kernel code may be written until the owner opens gate G-AOS. |
| Date | 2026-11-12 (revision 3) |
| Paths | **All paths in this document are relative to `agam-os/`** unless they begin with `../`. |
| Kernel style | Modular monolithic kernel; serial-first debugging (O-08) |
| Scope | Boot, serial console, GDT/TSS, IDT and CPU exceptions, physical memory manager, test hooks |
| Out of scope | Own page tables, heap, timer, hardware IRQs, SMP, user mode, framebuffer, ACPI, drivers |
| Audience | An engineer implementing or testing the kernel without access to any conversation |

Terms: **MUST / MUST NOT / SHOULD / MAY** are used in the RFC 2119 sense. "Normative" sections define interfaces that tests depend on; changing them requires a `DECISIONS.md` entry.

No AOS code exists. §15 separates what has been observed (by the architect in a scratch sandbox, and by Codex's independent review) from what has not been performed at all.

**Normative versus internal.** Interfaces that independent tests or the bootloader depend on are normative: the headers listed in §13.1, the serial formats of §12, the make targets of §11.1, hardware encodings, and the linker properties of §4.2. Everything else — file split, internal function names, helper signatures, algorithms — is the implementer's choice, and reviewers must not require a particular design.

---

## 1. Target

| Property | Value |
|---|---|
| Architecture | x86-64, long mode, 4-level paging |
| Executable format | ELF64, statically linked, non-relocatable |
| Load address | Higher half, base `0xFFFFFFFF80000000` |
| Boot protocol | Limine boot protocol, base revision 6 (P-01) |
| Boot media | Hybrid BIOS/UEFI ISO 9660 image |
| Reference machine | QEMU `q35`, CPU model `qemu64`, 1 CPU, TCG |
| Console | COM1 serial only |
| CPUs | Bootstrap processor only |

---

## 2. Toolchain and flags (P-04, P-05)

Expected versions are listed in `DECISIONS.md` §8.2; the versions actually installed are recorded at M0. Tools required in WSL2: `clang`, `ld.lld`, `llvm-objdump`, `llvm-readelf`, `llvm-nm`, GNU `make`, `xorriso`, `qemu-system-x86_64`, `gdb`, `sha256sum`, `tar` with xz support, `curl` **or** `wget`, `python3`, and a host C compiler reachable as `cc` (the Limine host utility's Makefile uses it). Git is not required in WSL and is never run there.

### 2.1 Compiler flags

Common to C and assembly (`COMMON`):

```
--target=x86_64-unknown-none-elf -m64 -march=x86-64
-g -Werror
-I kernel/include -I third_party/limine-protocol
-DAOS_VERSION="\"<contents of ../VERSION>\""
```

C only (`CFLAGS = COMMON +`):

```
-std=c17 -ffreestanding
-fno-stack-protector -fno-stack-check
-fno-pic -fno-pie -fno-lto
-mabi=sysv
-mno-80387 -mno-mmx -mno-sse -mno-sse2
-mno-red-zone -mcmodel=kernel
-fno-omit-frame-pointer
-O2
-Wall -Wextra -Wshadow -Wstrict-prototypes -Wmissing-prototypes
```

Assembly (`.S`, preprocessed) is built with `COMMON` only. The C-language and warning options above do not apply to it.

Why each group matters:

- `-ffreestanding`, no stack protector: there is no C library or runtime.
- No x87/MMX/SSE: Limine enters the kernel with `CR4` holding only `PAE`, so SSE instructions would raise #UD, and interrupt stubs do not save vector state.
- `-mno-red-zone`: interrupts use the current stack.
- `-mcmodel=kernel`: code is linked in the top 2 GiB of the address space.
- `-fno-omit-frame-pointer`: keeps stack walking possible in GDB.

Test kernels add `-DAOS_TEST=1`.

Status: `COMMON` plus the C-only set compiled a throwaway C file with Clang 18.1.3 (§15, V-01). The assembly set, and linking, have not been run (V-12).

### 2.2 Linker flags

```
ld.lld -m elf_x86_64 -nostdlib -static -z max-page-size=0x1000 --gc-sections -T kernel/linker.ld
```

Because of `--gc-sections`, Limine request objects MUST be marked `used` and kept by `KEEP()` in the linker script.

### 2.3 Functions the compiler may call implicitly

Clang may emit calls to `memcpy`, `memset`, `memmove` and `memcmp` even in freestanding mode. The kernel MUST provide all four (`kernel/src/string.c`) with standard C semantics.

### 2.4 Assembly

Assembly lives in `.S` files (preprocessed), AT&T syntax, assembled by Clang's integrated assembler. Every assembly file ends with `.section .note.GNU-stack,"",@progbits`.

---

## 3. Boot protocol contract

### 3.1 Facts taken from the Limine protocol specification (read 2026-11-12)

| Topic | Fact | Consequence for AOS |
|---|---|---|
| Base revisions | 0–6 specified; 0–5 deprecated | Request 6 |
| Support reporting | Bootloader zeroes the third word of the base-revision tag when the revision is supported | Check it first; halt if not supported |
| Request discovery | If start and end markers are present, the bootloader only honours requests between them (from revision 2). Markers are optional in the protocol. | **AOS always supplies both markers** and places every request between them |
| Entry `CS` / data segments | `CS=0x28`, others `0x30`, in a bootloader-owned GDT located in bootloader-reclaimable memory | Load own GDT early (§8) |
| IDT | Undefined; from revision 5 `IDTR` base and limit are 0 | Any exception before §9 triple-faults; keep early code minimal |
| Flags | `IF`, `DF` clear; from revision 5 `RFLAGS = 0x2` | Interrupts are off at entry and stay off in Sprint 1 |
| `CR0` | `PE`, `ET`, `WP`, `PG` set; all other bits clear from revision 5 | Write protection is on; rodata writes fault |
| `CR4` | `PAE` only (plus `LA57` only if requested) | No SSE; no 5-level paging |
| `EFER` | `LME`, `LMA`; `NX` only if the processor supports it | **AOS requires NX**: boot step 4 checks it and panics if it is absent (§5). The reference CPU model `qemu64` provides it. |
| Interrupt sources | Legacy PIC IRQs masked; I/O APIC redirection entries masked; local APIC enabled with spurious vector register `0x1FF` and LVT entries masked | Maskable hardware interrupts do not arrive in Sprint 1 because `IF=0` and sources are masked. **NMI, machine check and other non-maskable events can still occur**, so gates and stacks for vectors 2 and 18 exist; a handler for vector 255 exists too. |
| Stack | At least 64 KiB (or the requested size) in bootloader-reclaimable memory; a zero return address is pushed | Entry point can be a C function; MUST NOT return |
| General registers | All general-purpose registers other than `RIP` and `RSP` are zero at entry | — |
| Kernel mapping | Segments mapped at or above `0xFFFFFFFF80000000`; `PF_X` controls execute, `PF_W` controls write, read always allowed | Linker script must produce separate segments (§4.2) |
| Physical placement | Unspecified but contiguous; reported by the Executable Address feature | Never assume a physical load address |
| HHDM base | Chosen by the bootloader, may be randomised | Always read it from the response |
| HHDM coverage (revision 4+) | Usable, bootloader-reclaimable, executable and modules, framebuffer, reserved-mapped, ACPI reclaimable, ACPI NVS; mappings are rounded to whole pages | Plain Reserved and Bad memory are not mapped as such, but a byte of them **may** be reachable where it shares a rounded page with a mapped region. The kernel MUST NOT touch such memory whether or not an access would fault (§4.4). |
| HHDM permissions | HHDM mappings are supervisor read, write **and execute** | The kernel image is mapped W^X, but the same memory is also reachable, writable and executable, through the HHDM until the kernel builds its own page tables (K6). Known limitation, risk R-20. |
| Identity mapping | Dropped since base revision 1 | Nothing is mapped in the lower half for the kernel |
| Request scanning | Limine 12.9.6 scans the loaded image only up to the start of `.bss` | Requests must be initialised data placed before `.bss` (§4.2) |
| Memory map | Sorted by base; Usable and Bootloader-reclaimable entries are 4 KiB-aligned and do not overlap anything; other types may overlap and be unaligned | PMM uses only Usable entries |
| Low memory | From revision 3, the page at physical 0 may be reported as usable | PMM must never hand out frame 0 (§10.3) |
| Memory map type values | Usable 0, Reserved 1, ACPI reclaimable 2, ACPI NVS 3, Bad 4, Bootloader reclaimable 5, Executable and modules 6, Framebuffer 7, Reserved-mapped 8 | Use the header's macros, not these numbers |

### 3.2 Header

`third_party/limine-protocol/limine.h` is vendored unmodified from the `limine-protocol` repository at the commit recorded in `third_party/limine-protocol/PIN` (URL, commit hash, SHA-256 of the file, date). All magic numbers, request IDs and struct layouts MUST come from this header. Hand-copied constants are forbidden.

Proposed pin. Both the header hash and its pairing with the release were reproduced by Codex's independent review (evidence E02, E03); the owner's disposition of B-02 is pending:

| Item | Value |
|---|---|
| Repository | `https://github.com/limine-bootloader/limine-protocol` |
| Commit | `6e85ddd6f928b5b512015c4d47d1cfce2fd28079` (2026-10-27) |
| File | `include/limine.h`, SHA-256 `0aadf2633c85d8cb145344c968add158aeb1deb68af567d55ba8431c492b6d4c` |
| Why this commit | It is the commit that Limine 12.9.6's own `bootstrap` script pins for its `limine-protocol` dependency, so the bootloader release and this header belong together. |

Identifiers this specification relies on, all present in that file: `LIMINE_BASE_REVISION`, `LIMINE_BASE_REVISION_SUPPORTED`, `LIMINE_REQUESTS_START_MARKER`, `LIMINE_REQUESTS_END_MARKER`, the `LIMINE_*_REQUEST_ID` macros for bootloader info, HHDM, memory map, executable address, stack size and paging mode, `LIMINE_PAGING_MODE_X86_64_4LVL`, and the nine `LIMINE_MEMMAP_*` type macros.

### 3.3 Requests used in Sprint 1

| Request | Request revision and fields | Purpose | Response handling |
|---|---|---|---|
| Base revision tag (6), in `.limine_requests` | — | Protocol check | Not supported → panic `boot: limine base revision 6 not supported` |
| Requests start/end markers | — | Delimit the request area | — |
| Bootloader info | revision 0 | Log bootloader name and version | Missing → log `limine: unknown unknown`; continue |
| HHDM | revision 0 | Physical-to-virtual offset | Missing → panic `boot: no hhdm response` |
| Memory map | revision 0 | Input to the PMM | Missing, or `entry_count == 0` → panic `boot: no memory map` |
| Executable address | revision 0 | Kernel physical and virtual base (logged) | Missing → panic `boot: no executable address response` |
| Stack size | revision 0, `stack_size = 65536` | 64 KiB boot stack | Missing → continue (64 KiB is the protocol minimum) |
| Paging mode | **revision 1**, `mode = max_mode = min_mode = LIMINE_PAGING_MODE_X86_64_4LVL` | Guarantee 4-level paging | Missing, or reported mode is not 4-level → panic `boot: unsupported paging mode` |

Request revisions are per-feature numbers and are unrelated to the protocol base revision. Every request field not listed above is zero.

Each request object MUST be `static volatile`, 8-byte aligned, initialised, and carry `__attribute__((used, section(".limine_requests")))`. Markers use sections `.limine_requests_start` and `.limine_requests_end`.

All Limine-provided structures live in bootloader-reclaimable memory and stay valid for all of Sprint 1 (§10.3). How the rest of the kernel reaches the responses (accessor functions, globals) is internal.

---

## 4. Memory layout (P-07)

### 4.1 Virtual address space

| Range | Use | Sprint 1 state |
|---|---|---|
| `0x0000000000000000` – `0x00007FFFFFFFFFFF` | User space (future) | Treated as unmapped; the kernel MUST NOT dereference lower-half addresses |
| `0x0000800000000000` – `0xFFFF7FFFFFFFFFFF` | Non-canonical hole | An access raises #GP, or #SS when the reference goes through the stack segment (`RSP`- or `RBP`-based addressing) |
| `0xFFFF800000000000` – `0xFFFFFFFF7FFFFFFF` | Higher half below the kernel image. Contains the **HHDM** at the base Limine reports. | HHDM used read/write through `phys_to_virt()` |
| `0xFFFFFFFF80000000` – `0xFFFFFFFFFFFFFFFF` | Kernel image (text, rodata, data, bss) | Mapped by Limine from ELF program headers |

Reserved for later milestones (not used in Sprint 1; exact placement is fixed when the virtual memory manager is specified, after checking at run time that it cannot collide with the HHDM):

| Proposed range | Future use |
|---|---|
| `0xFFFFC00000000000` – `0xFFFFDFFFFFFFFFFF` | Kernel heap and dynamic kernel mappings |
| `0xFFFFE00000000000` – `0xFFFFEFFFFFFFFFFF` | Memory-mapped I/O and per-CPU areas |

Address translation helpers (`kernel/include/aos/mm.h`, normative):

```c
#define PAGE_SIZE 4096ULL
typedef uint64_t paddr_t;
void  mm_set_hhdm_offset(uint64_t offset);   /* called once during boot */
void *phys_to_virt(paddr_t p);               /* p + hhdm offset */
```

`phys_to_virt` is valid only for physical addresses inside HHDM-mapped memory-map types (§3.1).

### 4.2 Kernel image (`kernel/linker.ld`, normative properties)

- `ENTRY(kernel_entry)`; output format `elf64-x86-64`; location counter starts at `0xFFFFFFFF80000000`.
- Exactly three `PT_LOAD` program headers with explicit flags, each starting on a 4 KiB boundary, none overlapping a page of another:

| Segment | Flags | Sections, in order |
|---|---|---|
| text | R+X | `.text*` |
| rodata | R | `.rodata*` |
| data | R+W | `.limine_requests_start`, `.limine_requests`, `.limine_requests_end` (all `KEEP`), `.data*`, then `.bss*` and `COMMON` |

- No `PT_INTERP`, no `PT_DYNAMIC`, no relocation sections to apply at load time. ELF type is `EXEC`.
- No section is both writable and executable.
- Exported symbols and their exact meaning:

| Symbol | Meaning |
|---|---|
| `__kernel_start` | The link base, `0xFFFFFFFF80000000`; equals `__text_start` |
| `__text_start`, `__text_end` | First byte of the text segment (page-aligned); one past its last byte (not rounded) |
| `__rodata_start`, `__rodata_end` | Same, for rodata |
| `__data_start` | First byte of the R+W segment (page-aligned); the Limine request sections start here |
| `__data_end` | One past the last byte of initialised data; equals `__bss_start` |
| `__bss_start`, `__bss_end` | First byte of zero-initialised data; one past its last byte |
| `__kernel_end` | `__bss_end` rounded up to 4 KiB |

- The linker script MUST assert (`ASSERT`) that `__kernel_end <= 0xFFFFFFFFC0000000` (the whole image, not only its entry point, lies in the first 1 GiB above the link base, well inside the `-mcmodel=kernel` range) and that each `*_start` of a segment is 4 KiB-aligned.
- `/DISCARD/`: `.eh_frame*`, `.comment`, and notes the toolchain does not need.

### 4.3 Stacks

| Stack | Size | Location | Used for |
|---|---|---|---|
| Boot stack | 64 KiB | Bootloader-reclaimable memory (Limine) | All normal execution in Sprint 1 |
| IST1 | 16 KiB | `.bss`, 16-byte aligned | #DF (vector 8) |
| IST2 | 16 KiB | `.bss`, 16-byte aligned | NMI (vector 2) |
| IST3 | 16 KiB | `.bss`, 16-byte aligned | #MC (vector 18) |

There is no guard page under the boot stack in Sprint 1 (known limitation; fixed when the kernel owns its page tables).

### 4.4 Early memory ownership (Sprint 1, normative)

| Memory | Memory-map type | Provided by | Kernel may | Kernel MUST NOT |
|---|---|---|---|---|
| Kernel text, rodata, data, bss | Executable and modules | Limine, from the ELF | Execute, read, or read/write according to the segment | Give it to the PMM |
| GDT, TSS, IDT, IST stacks 1–3 | Inside kernel data/bss | Kernel | Use freely | Place them in bootloader memory |
| Boot stack (64 KiB) | Bootloader reclaimable | Limine | Run on it for all of Sprint 1 | Free or reuse it |
| Page tables | Bootloader reclaimable | Limine | Rely on them | Modify or free them |
| Limine's GDT | Bootloader reclaimable | Limine | Use it until `gdt_init()` | Rely on it afterwards |
| Limine responses, memory-map array, strings | Bootloader reclaimable | Limine | Read | Write or free |
| Usable memory at or above 1 MiB | Usable | Firmware | Hold the PMM bitmap; be allocated by the PMM | — |
| Usable memory below 1 MiB | Usable | Firmware | Nothing | Allocate or write |
| ACPI reclaimable, ACPI NVS, Reserved-mapped, Framebuffer | As named | Firmware / Limine | Nothing in Sprint 1 | Write |
| Reserved, Bad memory | As named | Firmware | Nothing | Touch (not mapped in the HHDM) |

Consequence: in Sprint 1 the kernel writes only to (a) its own data/bss segment, (b) the boot stack, (c) the PMM bitmap, and (d) frames obtained from the PMM. Any other write is a defect.

### 4.5 Consistency checklist (address, linker, Limine, ownership)

Reviewers check each row against the implementation; a mismatch between the two right-hand columns is a Major finding.

| # | Statement | Fixed in | Must agree with |
|---|---|---|---|
| 1 | Link base is `0xFFFFFFFF80000000` | `kernel/linker.ld` | `-mcmodel=kernel` (code must sit in the top 2 GiB); Limine rule "at or above `0xffffffff80000000`"; test T-K1-02 |
| 2 | The ELF is type `EXEC`, statically linked, with no dynamic section and no relocations to apply | §2 flags (`-fno-pic -fno-pie`, `-static`) | Limine: a non-relocatable executable must already request higher-half addresses; KASLR cannot apply |
| 3 | Exactly three page-aligned `PT_LOAD` segments: R+X, R, R+W; `max-page-size=0x1000` | `kernel/linker.ld`, §2.2 | Limine maps `PF_X`/`PF_W` as given; `CR0.WP` and `EFER.NX` are set at entry |
| 4 | Base revision tag and all requests lie between the markers, as initialised data, in the R+W segment, before `.bss` | `kernel/linker.ld`, `limine_requests.c` | Limine writes response pointers into them and scans only up to `.bss` |
| 5 | The kernel's physical address is never assumed | Executable address request | Limine: physical placement is unspecified |
| 6 | The HHDM offset is read at run time; `randomise_hhdm_base: no` exists only to keep logs repeatable | `mm.c`, `boot/limine.conf` | Limine may choose or randomise the base |
| 7 | 4-level paging is requested as minimum and maximum | Paging mode request | §4.1 assumes 48-bit canonical addresses |
| 8 | Stack size 64 KiB is requested; the stack is in bootloader-reclaimable memory | Stack size request | §10.3: the PMM never uses bootloader-reclaimable memory |
| 9 | The PMM uses Usable memory at or above 1 MiB only, and reaches its bitmap through the HHDM | `pmm.c` | Limine maps Usable memory into the HHDM (base revision ≥ 3) |
| 10 | Nothing the kernel needs after `gdt_init()` lives in Limine's GDT | `gdt.c` | §4.4 ownership table |

---

## 5. Boot sequence (normative order)

`kernel_entry(void)` is a C function with `__attribute__((noreturn))`. It runs these steps in order; each prints its log line (§12.1) only after it succeeds.

| Step | Action | Milestone |
|---|---|---|
| 1 | `serial_init()`; print banner | K1 |
| 2 | Check that base revision 6 is supported; else panic | K1 |
| 3 | Validate responses as in §3.3; log bootloader, HHDM offset, kernel bases; record the HHDM offset | K1 |
| 4 | Require NX: `CPUID.80000001H:EDX[20]` is 1 and `EFER.NXE` (MSR `0xC0000080`, bit 11) is 1; else panic `cpu: NX not available`. Log `[aos] cpu: nx enabled`. | K1 |
| 5 | `gdt_init()` | K2 |
| 6 | `idt_init()` | K3 |
| 7 | `pmm_init()` | K4 |
| 8 | Print `[aos] init complete` | K1 |
| 9 | Test kernel: print the `BEGIN` line, call `aos_test_main()`, then call `ktest_finish()` (§12.4). Normal kernel: print `[aos] halt`, then `hcf()`. | K1 |

`hcf()` ("halt and catch fire") is `for (;;) { cli; hlt; }`.

Until step 6 completes, any CPU exception causes a triple fault (machine reset). Steps 1–5 MUST therefore avoid anything that can fault, and QEMU is always run with `-no-reboot` so a reset is visible as an abnormal exit.

---

## 6. Serial console (`kernel/include/aos/serial.h`, normative)

```c
void serial_init(void);
void serial_putc(char c);
void serial_write(const char *s, size_t n);
```

- Device: 16550-compatible UART at I/O base `0x3F8` (COM1), 115200 baud, 8 data bits, no parity, 1 stop bit.
- Initialisation writes, in order: `0x3F9←0x00` (interrupts off), `0x3FB←0x80` (divisor latch), `0x3F8←0x01`, `0x3F9←0x00` (divisor 1), `0x3FB←0x03` (8N1), `0x3FA←0xC7` (FIFO on, cleared), `0x3FC←0x03` (DTR, RTS).
- Transmit is polled: wait for bit 5 of the line status register (`0x3FD`) for at most 100 000 polls, then write the byte; if the wait times out the byte is dropped (the kernel must not hang on a missing UART).
- `serial_putc('\n')` transmits `\r\n`.
- No receive path in Sprint 1.

Port I/O primitives (`kernel/arch/x86_64/io.h`): `outb`, `inb`, `outl`, `inl`, `io_wait` as `static inline` functions using inline assembly.

---

## 7. Kernel library

### 7.1 `kprintf` (`kernel/include/aos/kprintf.h`, normative)

```c
void kprintf(const char *fmt, ...) __attribute__((format(printf, 1, 2)));
void kvprintf(const char *fmt, va_list ap);
```

Supported: `%c`, `%s` (NULL prints `(null)`), `%d`/`%i`, `%u`, `%x`, `%X`, `%p` (prints `0x` and 16 lower-case hex digits), `%%`; length modifiers `l`, `ll`, `z`; flag `0`; decimal field width. No floating point, no precision, no `*`. An unsupported conversion is printed literally.

### 7.2 Panic and assertions (`kernel/include/aos/panic.h`, normative)

```c
_Noreturn void panic(const char *fmt, ...) __attribute__((format(printf, 1, 2)));
#define KASSERT(cond) /* on failure: panic("assertion failed: %s (%s:%d)", #cond, __FILE__, __LINE__) */
_Noreturn void hcf(void);
```

`panic` disables interrupts, prints the block in §12.2, then, in a test kernel, ends the test as failed (§12.4), or calls `hcf()` otherwise. A panic raised while a panic is already in progress goes straight to `hcf()`.

`KASSERT` is always enabled in Sprint 1.

### 7.3 Memory functions

`memcpy`, `memmove`, `memset`, `memcmp`, plus `strlen` (`kernel/include/aos/string.h`). Plain C, byte-wise is acceptable.

---

## 8. GDT and TSS (`kernel/include/aos/gdt.h`)

### 8.1 Layout (normative selectors)

| Index | Selector | Descriptor | Raw value |
|---|---|---|---|
| 0 | `0x00` | Null | `0x0000000000000000` |
| 1 | `0x08` | Kernel code, 64-bit, DPL 0 | `0x00AF9A000000FFFF` |
| 2 | `0x10` | Kernel data, DPL 0 | `0x00CF92000000FFFF` |
| 3 | `0x18` | User data, DPL 3 (unused in Sprint 1) | `0x00CFF2000000FFFF` |
| 4 | `0x20` | User code, 64-bit, DPL 3 (unused in Sprint 1) | `0x00AFFA000000FFFF` |
| 5–6 | `0x28` | 64-bit TSS (16-byte system descriptor, type `0x9`, present) | built at run time |

User data precedes user code so that a future `SYSCALL`/`SYSRET` setup can use this table unchanged.

### 8.2 TSS

A 104-byte packed structure: `reserved0` (u32), `rsp0`, `rsp1`, `rsp2` (u64), `reserved1` (u64), `ist[7]` (u64 each), `reserved2` (u64), `reserved3` (u16), `iopb_offset` (u16). Sprint 1 values: `rsp0–2 = 0`; `ist[0]`, `ist[1]`, `ist[2]` = tops of IST1, IST2, IST3; `iopb_offset = 104` (no I/O bitmap). A `_Static_assert` MUST check the size is 104. Every reserved field and every unused interrupt-stack entry (`ist[3]`…`ist[6]`) MUST be zero. The GDT stays in writable memory, because the processor updates the TSS descriptor's busy bit.

### 8.3 `gdt_init()`

1. Fill the TSS and build the TSS descriptor (limit 103, base = address of the TSS).
2. `lgdt`.
3. Reload `CS` with a far return to selector `0x08`.
4. Load `DS`, `ES`, `SS` with `0x10`; load `FS`, `GS` with `0`.
5. `ltr` with selector `0x28`.
6. Log `[aos] gdt: loaded cs=0x0008 tr=0x0028` using values read back with `mov %cs` and `str`.

Introspection helpers for tests (normative): `uint16_t gdt_read_cs(void); uint16_t gdt_read_tr(void);`

---

## 9. IDT and exception handling (P-08)

### 9.1 IDT

- 256 gates of 16 bytes (`IDTR` limit 4095). Every gate: present, DPL 0, 64-bit interrupt gate (type attribute `0x8E`), code selector `0x08`, reserved field zero.
- IST index: vector 8 → 1, vector 2 → 2, vector 18 → 3, all others → 0.
- `idt_init()` fills all gates, executes `lidt`, logs `[aos] idt: loaded 256 gates`. It does **not** enable interrupts.

### 9.2 Vectors 0–31

| Vec | Name | Error code | | Vec | Name | Error code |
|---|---|---|---|---|---|---|
| 0 | #DE divide error | no | | 16 | #MF x87 floating point | no |
| 1 | #DB debug | no | | 17 | #AC alignment check | yes |
| 2 | NMI | no | | 18 | #MC machine check | no |
| 3 | #BP breakpoint | no | | 19 | #XM SIMD floating point | no |
| 4 | #OF overflow | no | | 20 | #VE virtualisation | no |
| 5 | #BR bound range | no | | 21 | #CP control protection | yes |
| 6 | #UD invalid opcode | no | | 22–27 | reserved | no |
| 7 | #NM device not available | no | | 28 | #HV hypervisor injection | no |
| 8 | #DF double fault | yes (always 0) | | 29 | #VC VMM communication | yes |
| 9 | reserved (legacy) | no | | 30 | #SX security | yes |
| 10 | #TS invalid TSS | yes | | 31 | reserved | no |
| 11 | #NP segment not present | yes | | | | |
| 12 | #SS stack segment | yes | | | | |
| 13 | #GP general protection | yes | | | | |
| 14 | #PF page fault | yes | | | | |
| 15 | reserved | no | | | | |

### 9.3 Entry stubs (`kernel/arch/x86_64/isr_stubs.S`)

One stub per vector (generated with an assembler macro). For vectors without a CPU-pushed error code the stub pushes `0` first, so every frame has the same shape. Then: push the vector number, jump to the common stub, which pushes the 15 general registers, passes the frame pointer in `%rdi`, clears the direction flag (`cld`), calls `interrupt_dispatch`, pops the registers, discards vector and error code (`add $16, %rsp`), and executes `iretq`.

A table `isr_stub_table[256]` of stub addresses is exported for `idt_init()`.

### 9.4 Frame (normative; `kernel/include/aos/interrupt.h`)

```c
struct interrupt_frame {
    uint64_t r15, r14, r13, r12, r11, r10, r9, r8;
    uint64_t rbp, rdi, rsi, rdx, rcx, rbx, rax;
    uint64_t vector, error_code;
    uint64_t rip, cs, rflags, rsp, ss;     /* pushed by the CPU */
};                                         /* 22 * 8 = 176 bytes */
void interrupt_dispatch(struct interrupt_frame *frame);
```

`_Static_assert(sizeof(struct interrupt_frame) == 176)`.

Stack alignment argument (must hold; tested by review): in 64-bit mode the CPU aligns `RSP` to 16 bytes before pushing its five words. CPU frame (40) + error code (8) + vector (8) + 15 registers (120) = 176 = 11 × 16, so `RSP` is 16-byte aligned at the `call`, as the System V ABI requires.

### 9.5 Dispatch policy in Sprint 1 (normative)

`CR0`, `CR2`, `CR3`, `CR4` are read on entry to the handler, before anything is printed.

In a **test kernel with an armed expectation** (§12.4), the expectation takes precedence over the default action of **every** vector, including #BP:

| Vector arrives | Action |
|---|---|
| Equal to the armed vector | Print the dump; print `FAULT-OK`; disarm; `ktest_finish()` |
| Different from the armed vector | Print the dump; print `FAULT-WRONG`; mark the test failed; disarm; `ktest_finish()` |

Otherwise (normal kernel, or no expectation armed):

| Vector | Action |
|---|---|
| 3 (#BP) | Print `[aos] breakpoint at rip=0x<16 hex>` and return; execution continues after the `int3` with every register unchanged |
| Any other 0–31 | Print the dump (§12.3), then `panic("unhandled exception %lu (%s)")` |
| 32–255 | `panic("unexpected interrupt vector %lu")` |

A software `int n` does not push an error code, so tests raise real faults and never use `int` for an error-code vector.

### 9.6 Required and stretch verification (O-09)

- Required in Sprint 1: the IDT, all stubs, the dump, the breakpoint path, and the code that sets up IST1–IST3.
- **Stretch**: testing the double-fault path (corrupted stack → #DF on IST1) and the permission-fault tests (write to rodata, execute data). Until a stretch test has been run, the corresponding behaviour is reported as `UNVERIFIED`, and that is acceptable for the Sprint 1 tag.

---

## 10. Physical memory manager (P-09)

### 10.1 Interface (`kernel/include/aos/pmm.h`, normative)

```c
#define PMM_INVALID   ((paddr_t)0)
#define PMM_LOW_LIMIT 0x100000ULL          /* nothing below 1 MiB is ever managed */
#define PMM_MAX_PHYS  0x10000000000ULL     /* 1 TiB: provisional upper bound of managed memory */

struct pmm_stats {
    uint64_t usable_frames;    /* frames of Usable entries below PMM_MAX_PHYS, including low memory */
    uint64_t low_reserved;     /* of those, frames below PMM_LOW_LIMIT */
    uint64_t bitmap_frames;    /* frames occupied by the bitmap */
    uint64_t free_frames;      /* currently allocatable */
    uint64_t allocated_frames; /* currently handed out */
    uint64_t highest_frame;    /* one past the highest managed frame index */
    paddr_t  bitmap_base;      /* physical address of the first bitmap frame */
};

paddr_t pmm_alloc(void);
paddr_t pmm_alloc_contig(size_t count);
void    pmm_free(paddr_t frame);
void    pmm_free_contig(paddr_t first, size_t count);
void    pmm_get_stats(struct pmm_stats *out);
```

Initialisation (`pmm_init`) is called once from boot step 7. Its signature is internal.

### 10.2 Definitions

- A **frame** is 4 KiB; frame index `f` is physical address `f * 4096`.
- The **clipped range** of a Usable entry `[base, base + length)` is `[max(base, PMM_LOW_LIMIT), min(base + length, PMM_MAX_PHYS))`. It may be empty.
- `highest_frame` = (largest end of any non-empty clipped range) / 4096. It is at most 2^28.
- The **bitmap** has `bitmap_words = ceil(highest_frame / 64)` 64-bit words; `bitmap_bytes = bitmap_words * 8`; `bitmap_frames = ceil(bitmap_bytes / 4096)`. Bit `f` describes frame `f`: set = not allocatable, clear = free.
- **Padding bits** are bits with index ≥ `highest_frame` in the last word. They are always set and are never examined by allocation or free.
- A frame is **managed** if its index is below `highest_frame`, it lies inside a clipped range, and it is not one of the bitmap's own frames. Being managed never changes after initialisation.
- All arithmetic is 64-bit and checked: no step may rely on wrap-around, and a byte count is never formed by multiplying an unchecked frame count.

### 10.3 Initialisation algorithm

1. For every Usable entry: `base` and `length` must be multiples of 4096 and `base + length` must not overflow, else `panic("pmm: bad memory map entry %lu")`. Other entry types are only logged.
2. Compute the clipped ranges. If none is non-empty, `panic("pmm: no usable memory")`. If any Usable memory lies at or above `PMM_MAX_PHYS`, log `[aos] pmm: ignoring memory above 0x0000010000000000`.
3. Compute `highest_frame`, `bitmap_words`, `bitmap_bytes`, `bitmap_frames`.
4. Place the bitmap at the **start of the lowest-addressed clipped range** whose length is at least `bitmap_frames * 4096`. (A Usable entry that begins below 1 MiB is eligible through its clipped part.) If there is none, `panic("pmm: no region for bitmap")`.
5. Set every bit of every bitmap word.
6. Clear the bit of every frame inside a clipped range.
7. Set the bits of the bitmap's own frames.
8. Set the counters; the allocation hint starts at 0.
9. Log the lines of §12.1.

Policy fixed here:

- **Nothing below 1 MiB is ever allocated** (`PMM_INVALID` is 0, and low memory holds firmware structures).
- **Only Usable entries are used.** Bootloader-reclaimable memory holds the running stack, the page tables, the bootloader's GDT and all Limine responses, and is not reclaimed in Sprint 1.
- The protocol guarantees Usable entries do not overlap any other entry, so the kernel image needs no separate exclusion.

### 10.4 Operations

All panics raised here have a reason beginning `pmm: `.

- **`pmm_alloc`**: examines frame indexes `hint … highest_frame − 1`, then `0 … hint − 1`, and takes the first clear bit: sets it, updates the counters, sets the hint to the next index (wrapping to 0 at `highest_frame`). Returns `PMM_INVALID` if no bit is clear. Never panics on exhaustion. The frame's contents are unspecified.
- **`pmm_alloc_contig(count)`**: returns `PMM_INVALID` if `count == 0` or `count > highest_frame`, without scanning. Otherwise takes the lowest run of `count` consecutive clear bits within `[0, highest_frame)`, all or nothing. The hint is unchanged.
- **`pmm_free(frame)`**: panics unless the address is 4 KiB-aligned, its index is below `highest_frame`, the frame is managed, and its bit is set. Since a managed frame's bit is set only while it is allocated, this rejects double frees, frees of never-allocated frames, low memory, the bitmap, and unmanaged or reserved memory. Then clears the bit and updates the counters; the hint is unchanged.
- **`pmm_free_contig(first, count)`**:
  - `count == 0`: returns immediately, without examining `first`.
  - Otherwise `first` must be 4 KiB-aligned; with `f = first / 4096`, it must hold that `f < highest_frame` and `count <= highest_frame − f` (checked in this form, so nothing overflows).
  - Every frame `f … f + count − 1` must satisfy the conditions of `pmm_free`. **All of them are validated before any bit is changed.**
  - On any violation: panic, with the bitmap and counters unchanged. Otherwise all the bits are cleared.
  - The frames need not have been obtained by one `pmm_alloc_contig` call.
- **Limitation**: a stale address whose frame has since been allocated again cannot be told apart from a valid free.
- Not re-entrant and not thread-safe; Sprint 1 is single-CPU with interrupts disabled.

### 10.5 Invariants (tests and review check these)

1. `usable_frames == low_reserved + bitmap_frames + free_frames + allocated_frames` at all times.
2. Every address returned is 4 KiB-aligned, at or above `PMM_LOW_LIMIT`, below `highest_frame * 4096`, inside a Usable entry, and outside the bitmap.
3. No address is returned twice without an intervening free.
4. After freeing everything that was allocated, `free_frames` equals its value right after initialisation.
5. Every allocated frame is readable and writable through `phys_to_virt()`.
6. No operation reads or writes a bitmap bit with index ≥ `highest_frame`, other than step 5 of initialisation.

### 10.6 Required and stretch verification (O-09, O-19)

- Required: the positive and boundary tests of `TEST_PLAN.md` §5 (K4), including `pmm_free_contig` with zero count and with valid ranges.
- Required: the validation code for every invalid case above, and its independent review (review item RV-K4-01).
- **Stretch**: tests that deliberately pass invalid arguments and expect a panic, other than the plain double free.

---

## 11. Build system contract (normative; implemented by Claude, consumed by Codex)

### 11.1 Make targets

| Command | Result |
|---|---|
| `make` or `make kernel` | `build/kernel/aos.elf` |
| `make iso` | `build/aos.iso` (normal kernel) |
| `make iso TEST=<case>` | `build/test/<case>/aos.elf` and `build/test/<case>/aos.iso`, built with `-DAOS_TEST=1` and linked with `tests/cases/<case>.c` |
| `make run` | Boots `build/aos.iso` in QEMU, serial on the terminal |
| `make debug` | Same as `make run`, with `-gdb tcp:127.0.0.1:1234 -S` (QEMU waits for GDB on the loopback interface only) |
| `make test MILESTONE=<id>` | Runs `python3 ../agam-dev/verify/run.py gate <id>` and returns its exit status. `make test` without `MILESTONE` is an error. Not part of the M0 skeleton. |
| `make check-env` | Runs `sh ../agam-dev/verify/check_env.sh` |
| `make fetch` | Runs `tools/fetch-limine.sh` (download, verify, unpack, build the host utility) |
| `make print-flags` | Prints the exact compiler and linker flags |
| `make clean` | Removes `build/` except `build/cache/` |
| `make distclean` | Removes `build/` entirely |

Overridable variables: `CC` (default `clang`), `LD` (`ld.lld`), `QEMU` (`qemu-system-x86_64`), `QEMU_MEM` (`256M`), `TEST`.

The Makefile MUST NOT invoke `git` (P-06) and MUST fail with a clear message if a required tool is missing.

### 11.2 Limine artefacts

`tools/fetch-limine.sh` downloads the release tarball named in `third_party/limine/PIN` into `build/cache/`, verifies its SHA-256, unpacks it, and builds the `limine` host utility. Files needed: `limine-bios.sys`, `limine-bios-cd.bin`, `limine-uefi-cd.bin`, `BOOTX64.EFI`, and the `limine` host utility. A checksum mismatch is a hard error.

Proposed pin. The asset hash and inventory were reproduced by Codex's independent review (evidence E01); the owner's disposition of B-02 is pending:

| Item | Value |
|---|---|
| Release | Limine **12.9.6** (tag `v12.9.6`, tag commit `89a9512c623a4779cc73e3869383876d54264b28`, released 2026-10-27/28) |
| Asset | `limine-binary.tar.xz` |
| URL | `https://github.com/limine-bootloader/limine/releases/download/v12.9.6/limine-binary.tar.xz` |
| SHA-256 | `939f670e0831cc940453fd3a0ddfb2b324c0b4e3c5d0e741c190aa0d13df0103` — 906,108 bytes. Reproduced from independent downloads by Codex (review evidence E01) and by the architect on 2026-11-12. The release's detached signature has not been verified (V-13). |
| Base revision support | `#define SUPPORTED_BASE_REVISION 6` in `common/protos/limine.c` at that tag |
| Host utility | Shipped as portable C source inside the binary package; built with `make` in the unpacked directory |

### 11.3 ISO layout

```
iso_root/
├── boot/
│   ├── aos.elf
│   └── limine/
│       ├── limine.conf
│       ├── limine-bios.sys
│       ├── limine-bios-cd.bin
│       └── limine-uefi-cd.bin
└── EFI/BOOT/BOOTX64.EFI
```

Image creation (`tools/mkiso.sh`):

```
xorriso -as mkisofs -R -r -J -b boot/limine/limine-bios-cd.bin \
    -no-emul-boot -boot-load-size 4 -boot-info-table -hfsplus \
    -apm-block-size 2048 --efi-boot boot/limine/limine-uefi-cd.bin \
    -efi-boot-part --efi-boot-image --protective-msdos-label \
    <iso_root> -o <image.iso>
limine bios-install <image.iso>
```

### 11.4 `boot/limine.conf`

```
timeout: 0
serial: no

/AOS
    protocol: limine
    path: boot():/boot/aos.elf
    kaslr: no
    randomise_hhdm_base: no
```

These option names were read in `CONFIG.md` at tag `v12.9.6`: `timeout` and `serial` are global options; `protocol` is a per-entry option; `path`, `kaslr` and `randomise_hhdm_base` are Limine-protocol entry options. Global options must come before the first entry, as they do here. For a non-relocatable executable `kaslr` has no effect and `randomise_hhdm_base` already defaults to `no`; both are written out so the behaviour does not depend on defaults. Behaviour at boot is still to be verified in K1.

### 11.5 Reference QEMU command

```
qemu-system-x86_64 -M q35 -accel tcg -cpu qemu64 -smp 1 -m 256M \
    -cdrom build/aos.iso -boot d \
    -serial stdio -display none -no-reboot \
    -device isa-debug-exit,iobase=0xf4,iosize=0x04
```

The test harness adds `-d int,cpu_reset -D <log file>` and redirects serial to a file or pipe. `make debug` adds `-gdb tcp:127.0.0.1:1234 -S`. GDB session: `gdb build/kernel/aos.elf -ex "target remote :1234"`.

---

## 12. Serial output formats (normative — tests match these exactly)

All lines end with `\n` (transmitted as `\r\n`). Hex values use lower-case digits.

### 12.1 Boot log

```
[aos] AOS <version> booting
[aos] limine: <name> <version>, base revision 6
[aos] hhdm: offset=0x<16 hex>
[aos] kernel: phys=0x<16 hex> virt=0x<16 hex>
[aos] cpu: nx enabled
[aos] gdt: loaded cs=0x0008 tr=0x0028
[aos] idt: loaded 256 gates
[aos] memmap: <n> entries
[aos] memmap: [<i>] base=0x<16 hex> len=0x<16 hex> type=<name>
[aos] pmm: bitmap phys=0x<16 hex> bytes=<bitmap_bytes> frames=<bitmap_frames> covers=<highest_frame>
[aos] pmm: usable=<n> low_reserved=<n> bitmap=<n> free=<n>
[aos] init complete
[aos] halt
```

- `<name>` for memory types: `usable`, `reserved`, `acpi-reclaimable`, `acpi-nvs`, `bad`, `bootloader-reclaimable`, `executable`, `framebuffer`, `reserved-mapped`, or `unknown(<n>)`.
- If the bootloader-info response is missing, the second line is `[aos] limine: unknown unknown, base revision 6`.
- `bytes=` is `bitmap_bytes` exactly as defined in §10.2 (whole 64-bit words, **not** rounded to frames); `frames=` is `bitmap_frames`; `covers=` is `highest_frame`. The bitmap therefore occupies physical `[phys, phys + frames * 4096)`.
- Lines for a subsystem appear only from the milestone that introduces it.

### 12.2 Panic

```
*** AOS PANIC ***
reason: <message>
```

### 12.3 Exception dump

```
*** AOS EXCEPTION ***
vector=<decimal> name=<name> error=0x<16 hex>
rip=0x<16 hex> cs=0x<4 hex> rflags=0x<16 hex>
rsp=0x<16 hex> ss=0x<4 hex>
cr0=0x<16 hex> cr2=0x<16 hex> cr3=0x<16 hex> cr4=0x<16 hex>
rax=0x<16 hex> rbx=0x<16 hex> rcx=0x<16 hex> rdx=0x<16 hex>
rsi=0x<16 hex> rdi=0x<16 hex> rbp=0x<16 hex> r8=0x<16 hex>
r9=0x<16 hex> r10=0x<16 hex> r11=0x<16 hex> r12=0x<16 hex>
r13=0x<16 hex> r14=0x<16 hex> r15=0x<16 hex>
```

For vector 14 one more line follows: `pf: present=<0|1> write=<0|1> user=<0|1> rsvd=<0|1> exec=<0|1>` (error-code bits 0–4). `<name>` is the mnemonic from §9.2 (`#DE`, `NMI`, `#PF`, …) or `reserved`.

### 12.4 Test protocol (`kernel/include/aos/ktest.h`, normative; compiled only when `AOS_TEST` is defined)

```c
#define AOS_EXIT_PASS 0x10u            /* guest exit values; see §12.5 */
#define AOS_EXIT_FAIL 0x11u

extern const char aos_test_name[];     /* defined by the test case */
void aos_test_main(void);              /* defined by the test case */

void ktest_check(bool ok, const char *expr, const char *file, int line);
#define KTEST_CHECK(cond) ktest_check((cond), #cond, __FILE__, __LINE__)
void ktest_expect_fault(uint8_t vector);
_Noreturn void ktest_finish(void);
```

State: a **failed** flag, which once set is never cleared, and at most one **armed** expectation.

| Rule | Behaviour |
|---|---|
| Start | Boot step 9 prints `@@AOS-TEST@@ BEGIN <aos_test_name>`, calls `aos_test_main()`, and when that returns calls `ktest_finish()` |
| `KTEST_CHECK` fails | Prints `@@AOS-TEST@@ CHECK-FAILED <expr> (<file>:<line>)`; sets **failed**; execution continues |
| `ktest_expect_fault(v)` | `v` must be below 32, else `panic("ktest: invalid vector")`. If an expectation is already armed, `panic("ktest: expectation already armed")`. Otherwise arms `v`. Any vector 0–31 may be armed, including 3. |
| Armed vector arrives | Dump, then `@@AOS-TEST@@ FAULT-OK vector=<n>`; disarm; `ktest_finish()` |
| A different vector arrives while armed | Dump, then `@@AOS-TEST@@ FAULT-WRONG expected=<a> got=<n>`; set **failed**; disarm; `ktest_finish()` |
| A vector other than #BP arrives while nothing is armed | Dump, panic block; the panic ends the test as failed (next row) |
| `panic` in a test kernel | Panic block, then `@@AOS-TEST@@ RESULT FAIL <aos_test_name>`, then exit with `AOS_EXIT_FAIL` |
| `ktest_finish()` | **The single normal finalisation path.** If an expectation is still armed: print `@@AOS-TEST@@ FAULT-MISSING vector=<a>` and set **failed**. Then print `@@AOS-TEST@@ RESULT PASS <aos_test_name>` if **failed** is clear, otherwise `… RESULT FAIL …`; write `AOS_EXIT_PASS` or `AOS_EXIT_FAIL` to the exit device; `hcf()`. |

Consequences that tests rely on:

- A failed check is sticky: a later matching fault, or a normal return, still yields `RESULT FAIL` and host status 35.
- Every test kernel run that reaches finalisation prints exactly one `RESULT` line.
- If finalisation is entered a second time (for example a fault inside it), the kernel goes straight to `hcf()`.

How the kernel implements this internally (helper names, where the state lives) is not normative.

### 12.5 Exit codes: guest value versus host status (normative)

Two different numbers are involved and MUST NOT be confused:

- The **guest exit value** `v` is what the kernel writes (one 32-bit `outl`) to I/O port `0xF4`, the `isa-debug-exit` device.
- The **host exit status** is what the `qemu-system-x86_64` process returns to the shell or harness. QEMU computes it as `(v << 1) | 1`; on the Linux host used for builds and tests, the shell sees the low 8 bits of that value.

| Outcome | Guest exit value (kernel constant) | Host exit status (harness constant) |
|---|---|---|
| Test passed | `AOS_EXIT_PASS = 0x10` (16) | `HOST_STATUS_PASS = 33` |
| Test failed, or panic | `AOS_EXIT_FAIL = 0x11` (17) | `HOST_STATUS_FAIL = 35` |

Host statuses that never come from the kernel's exit call, all of which the harness treats as failure:

| Host status | Cause |
|---|---|
| 0 | The guest reset (for example a triple fault) while QEMU runs with `-no-reboot`, or the guest shut down normally. Status 0 is therefore not proof of a reset; the `cpu_reset` log distinguishes them. The exit device cannot produce 0, because its status is always odd. |
| 1 | QEMU's own error exit (bad option, missing image). Guest value 0 would also give 1, which is why 0 is never used as a guest value. |
| Timeout (harness kills QEMU) | The kernel hung, halted without calling the exit device, or the exit device is missing (the write then has no effect). |
| Anything else | Failure |

Rules:

- `AOS_EXIT_PASS` and `AOS_EXIT_FAIL` are defined once, in `kernel/include/aos/ktest.h`. `HOST_STATUS_PASS` and `HOST_STATUS_FAIL` are defined once, in the harness. Code and documents always say which of the two numbers they mean.
- Guest values MUST be below `0x80` so the host status is not truncated.
- **Only host status 33, together with the expected serial lines, means success.**
- After writing the exit value the kernel calls `hcf()`, in case the device is absent.
- An expected-panic test is a successful test of a failure path only when the host status is 35 **and** its panic and `RESULT FAIL` lines are present.

What has been observed so far (architect, scratch sandbox, 2026-11-12; QEMU 8.2.2, Ubuntu package `1:8.2.2+ds-0ubuntu1.18`; `-M q35 -accel tcg -cpu qemu64 -no-reboot -device isa-debug-exit,iobase=0xf4,iosize=0x04`; hand-assembled 16-bit boot sectors, **not** an AOS kernel):

| Guest action | Observed host status |
|---|---|
| 8-bit write of `0x10` to port `0xF4` | 33 |
| 32-bit write of `0x10` | 33 (five consecutive runs) |
| 8-bit and 32-bit write of `0x11` | 35 |
| 8-bit write of `0x00` | 1 |
| Write of `0x10` with no `isa-debug-exit` device | No exit; killed by timeout |
| `cli; hlt` loop | No exit; killed by timeout |
| Empty IDT loaded, then an interrupt raised (guest reset) | 0 |

This matches QEMU's source (`hw/misc/debugexit.c`). It MUST be repeated on the owner's machine with the installed QEMU, and with the real test kernel, in K1 (V-05).

---

## 13. Source tree (Sprint 1)

```
kernel/
├── linker.ld
├── include/aos/
│   ├── types.h  mm.h  serial.h  kprintf.h  panic.h  string.h
│   ├── gdt.h  interrupt.h  pmm.h  ktest.h
├── src/
│   ├── main.c                 kernel_entry and the boot sequence
│   ├── boot/limine_requests.c requests and accessors (no other file includes limine.h, except pmm.c for the memmap types)
│   ├── serial.c  kprintf.c  panic.c  string.c  mm.c
│   ├── pmm.c
│   └── ktest.c                compiled only for test kernels
└── arch/x86_64/
    ├── io.h
    ├── gdt.c  gdt_flush.S
    ├── idt.c  isr_stubs.S  exception.c
    └── cpu.h                  control-register and flag helpers
```

### 13.1 Test-facing headers (normative) and internal headers

The tree above is a suggested layout. Only these are fixed, because independent tests include or depend on them:

| Header | Normative content |
|---|---|
| `aos/ktest.h` | §12.4 |
| `aos/pmm.h` | §10.1 |
| `aos/mm.h` | `PAGE_SIZE`, `paddr_t`, `phys_to_virt()` |
| `aos/gdt.h` | `gdt_read_cs()`, `gdt_read_tr()` |
| `aos/kprintf.h`, `aos/panic.h`, `aos/string.h` | §7 |
| `aos/types.h` | Fixed-width and `bool` types for the headers above |

The interrupt-frame **layout** (§9.4), the descriptor encodings (§8, §9.1) and the linker symbols (§4.2) are normative as data, whatever header declares them. All other headers, source files and function names are internal.

---

## 14. Coding standard

- C17; no variable-length arrays; no floating point; fixed-width integer types for anything with a defined size.
- Every header has an include guard and compiles on its own.
- Internal functions are `static`; public names carry the subsystem prefix (`pmm_`, `serial_`, `gdt_`, `idt_`, `ktest_`).
- Hardware structures use `__attribute__((packed))` plus a `_Static_assert` on their size.
- Allowed extensions: the attributes `packed`, `noreturn`, `used`, `section`, `aligned`, `format`; inline assembly only under `kernel/arch/`.
- No `__DATE__`, `__TIME__`, or other inputs that make builds non-reproducible.
- Comments explain why, and cite the specification section a structure implements.
- Zero warnings (`-Werror`).

---

## 15. Verification status of this specification's assumptions

"Observed" below means the architect ran or read something in a scratch sandbox, not on the owner's machine and not with AOS code. Every item still needs evidence from the owner's environment in the named milestone.

| # | Item | State on 2026-11-12 | Verified in |
|---|---|---|---|
| V-01 | The C flag set in §2.1 is accepted by Clang and the kernel contains no x87, MMX, SSE or AVX instruction or register use | **Partly observed**: Clang 18.1.3 compiled a throwaway C file with the C flag set, zero warnings, and no `xmm`/`ymm`/`st` operands. The real kernel, and the wider check of T-K1-04, are untested. | K1 |
| V-02 | Identifier names in the vendored `limine.h` | **Observed**: all identifiers listed in §3.2 exist at the proposed commit, and the probe file using them compiled. | K1 (Codex re-checks) |
| V-03 | `limine.conf` option names | **Read** in `CONFIG.md` at tag `v12.9.6` (§11.4). Not yet booted. | K1 |
| V-04 | `xorriso` flags and `limine bios-install` | The command in §11.3 is identical to `USAGE.md` at tag `v12.9.6`. Not yet executed. | K1 |
| V-05 | Guest value `0x10` gives host status 33 | **Observed** with boot sectors on QEMU 8.2.2 (§12.5). Not yet on the owner's machine or with an AOS kernel. | K1 |
| V-06 | The lower half is unmapped at entry | The protocol states identity mapping was dropped in base revision 1. Not yet observed. | K3 (`exc_pf_read`) |
| V-07 | Limine honours segment permissions (rodata write and data execution fault) | The protocol states `PF_W`/`PF_X` are reflected. Not yet observed. | **Stretch**, K3 |
| V-08 | A corrupted stack pointer followed by an exception reaches the #DF handler on IST1 | Not observed. | **Stretch**, K3 |
| V-09 | Memory above 4 GiB appears as Usable entries with more than about 3 GiB of RAM | Not observed. | K5 |
| V-10 | The Limine release asset matches its SHA-256 and contains the files in §11.2; the header matches its SHA-256; the release pins the same header commit and supports base revision 6 | **Reproduced independently**: by Codex (review evidence E01, E02, E03: downloads, hashes, archive inventory, source inspection) and, for the archive, by a separate download by the architect. This is evidence of artifact identity and source pairing only. | Owner records the disposition of B-02 |
| V-11 | The whole toolchain is installed in WSL2 Ubuntu 24.04 at the versions in `DECISIONS.md` §8.2 | **Owner-reported** on 2026-11-12 for Clang, LLD, QEMU, GDB and Make. No command output has been filed yet; the other tools are unreported. | M0 (`check_env.sh` output; clears B-01) |
| V-12 | The assembly flag set of §2.1 assembles `.S` files, and the link line of §2.2 links the kernel | **Not performed.** | K1 |
| V-13 | The release's detached signature is valid | **Not performed.** Not required by any gate; recorded so that a matching hash is not mistaken for signature verification. | Optional |
| V-14 | The kernel boots under the pinned Limine release with base revision 6 (runtime compatibility) | **Not performed.** | K1 |

---

## 16. After Sprint 1 (outline only; each needs its own specification)

1. **K6 — Virtual memory manager:** own 4-level page tables, kernel remapped with exact permissions, HHDM rebuilt, guard pages under stacks, bootloader-reclaimable memory reclaimed.
2. **K7 — Kernel heap.**
3. **K8 — Interrupt controllers and timer:** ACPI MADT parsing, local APIC, I/O APIC, timer tick, `IF=1`.
4. **K9 — Framebuffer console.**
5. **K10 — Kernel threads and scheduler.**
6. **K11 — User mode and system calls** (the GDT in §8 is already laid out for this).
