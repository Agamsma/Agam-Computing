# Agam Computing Ecosystem — Roadmap

| Field | Value |
|---|---|
| Status | **Draft v0.1, revision 2 — NOT APPROVED.** Owner's provisional decisions of 2026-11-12 incorporated. No milestone may start until the owner opens its gate in `DECISIONS.md` §1. |
| Date | 2026-11-12 (revision 2) |
| Paths | Relative to the repository root (`agam-os/`, `agam-lang/`, `agam-dev/`, `docs/`) |
| Branches | `main` (ecosystem), `agent/claude` (claude-worktree), `agent/codex` (codex-worktree) — read from the repository on 2026-11-12 |
| Related | `AOS_KERNEL_SPEC.md` (what to build), `AGAMLANG_SPEC.md` (what to build), `TEST_PLAN.md` (how it is verified) |

Every verification command below is a **proposed** command. None has been run. `git` commands are run with Git for Windows; `sh`, `make` and `python3` commands are run in WSL2 Ubuntu 24.04 from `/mnt/d/Agam-Computing/<worktree>` (O-01). Items marked **stretch** (O-09) are attempted only after the required items of the milestone pass and never block acceptance. A milestone is accepted only when its commands have been run on the integration branch and the evidence is filed (`TEST_PLAN.md` §2).

---

## 1. Timeline

| When | Milestone | Outcome | Gate |
|---|---|---|---|
| Day 0 | **M0** Environment and repository bootstrap | Tools verified, layout and pins in place | G-ENV |
| Day 1 | **K1** Boot to kernel entry | Serial banner, clean automated exit | G-AOS |
| Day 2 | **K2** GDT, TSS, kernel library hardening | Own descriptor tables; build quality gates | G-AOS |
| Day 3 | **K3** IDT and CPU exceptions | Every exception produces a register dump | G-AOS |
| Day 4 | **K4** Physical memory manager | Frame allocator from the Limine memory map | G-AOS |
| Day 5 | **K5** Hardening, matrix, review closure | Tagged `aos-v0.1.0-sprint1` | G-AOS |
| Days 6–10 (proposed) | **L1–L5** AgamLang v0.1 | Tagged `agamlang-v0.1.0` | G-LANG |
| Later | K6+ (virtual memory onward), L6+, D1+ | See §6 | new specs |

Days are working days counted from the owner's approval. Sprint 1 (K1–K5) is the five-day sprint requested in the brief. The AgamLang track shares no files with AOS, so it may instead run in parallel with Sprint 1 if the owner prefers; the default is sequential so that attention stays on the kernel.

Task IDs: `<milestone>-C<n>` is a Claude Code task, `<milestone>-X<n>` is a Codex task. Within a milestone, C and X tasks touch disjoint paths and can run at the same time.

---

## 2. M0 — Environment and repository bootstrap (Day 0)

| | |
|---|---|
| **Prerequisites** | Architecture reviewed by Codex; owner opens gate G-ENV; documents committed in the way the owner chooses (blocker B-04). |
| **Purpose** | Replace every assumption about the machine with recorded evidence. |

**Deliverables**

| Task | Owner | Paths |
|---|---|---|
| M0-O1 | Owner | In WSL2 Ubuntu 24.04: `sudo apt install clang lld llvm make xorriso qemu-system-x86 gdb python3 python3-venv ovmf`. In Git for Windows, inside the repository: set `user.name` and `user.email` to the owner's GitHub identity (O-10). |
| M0-A1 | Architect | `.gitattributes`, `.gitignore`, `VERSION`, `README.md`, `docs/*`, `CLAUDE.md`, `AGENTS.md` prepared in the `ecosystem` working folder. **Committed to `main` only by the owner or with the owner's explicit approval.** |
| M0-X1 | Codex | `agam-dev/verify/check_env.sh` — prints name, path and version of every tool in `AOS_KERNEL_SPEC.md` §2 plus `python3` and `git`; exits non-zero if any is missing; prints OS and kernel version; `--brief` prints a one-line summary for evidence blocks |
| M0-X2 | Codex | `agam-dev/verify/check_ownership.py` — given `--agent claude\|codex` and `--base <ref>`, lists changed paths and exits non-zero if any path is outside that agent's ownership (`ARCHITECTURE.md` §6) |
| M0-X3 | Codex | `agam-lang/tests/requirements-dev.txt` (pinned `pytest`), `agam-dev/verify/README.md` |
| M0-X4 | Codex | **Independent pin verification (blocker B-02)**: download the proposed Limine asset and header without using Claude's script, compute both SHA-256 values, list the archive's contents, confirm `SUPPORTED_BASE_REVISION` and the header identifiers, and file `agam-dev/reports/codex/M0-limine-pin.md` |
| M0-C1 | Claude | `agam-os/third_party/limine/PIN` and `agam-os/third_party/limine-protocol/PIN` with the values proposed in `DECISIONS.md` §8 (Limine 12.9.6; `limine-protocol` commit `6e85ddd…`), the vendored `limine.h`, `agam-os/tools/fetch-limine.sh` |
| M0-C2 | Claude | `agam-os/Makefile` skeleton with `check-env`, `print-flags`, `clean`, `distclean` only |
| M0-C3 | Claude | `agam-dev/reports/claude/M0-environment.md` — output of `check_env.sh`, the chosen Limine version with the source of the checksum |

**Verification commands**

```
# WSL2
sh agam-dev/verify/check_env.sh
sh agam-os/tools/fetch-limine.sh                 # second run must report "already verified"
python3 agam-dev/verify/check_ownership.py --agent claude --base main
python3 agam-dev/verify/check_ownership.py --agent codex  --base main

# Git for Windows, in each worktree
git worktree list
git branch --show-current
git status --porcelain
git config user.name
git config user.email
git log --format="%an <%ae> | %cn <%ce>%n%b" main..HEAD
```

**Acceptance criteria**

1. `check_env.sh` exits 0 and its full output is filed in both agents' reports.
2. `fetch-limine.sh` verifies the tarball hash; deliberately corrupting the cached tarball makes it exit non-zero (evidence required).
3. Both ownership checks exit 0 on the M0 branches; a deliberately wrong path makes the check exit non-zero (evidence required).
4. A file committed with LF line endings is still LF after checkout in every worktree (`git ls-files --eol` shows `i/lf w/lf`).
5. `git status` works in all three worktrees with Git for Windows, and `git worktree list` shows `main`, `agent/claude` and `agent/codex` in the expected folders.
6. Every tool version printed by `check_env.sh` is compared with `DECISIONS.md` §8; each difference is recorded and accepted by the owner, or the tool is changed.
7. Codex's independent pin report (M0-X4) agrees with the PIN files on both SHA-256 values. A disagreement stops the sprint.
8. In every worktree, `git config user.name` and `user.email` show the owner's GitHub identity, and no commit on any branch has an author, committer, `Co-authored-by` trailer or message text naming an AI tool.
9. A build run from `/mnt/d/…` in WSL2 completes (`make -C agam-os print-flags`), showing the Windows-hosted working folder is usable from WSL.

**Risks**: R-01, R-02, R-03, R-04, R-21, R-22.

---

## 3. Sprint 1 — AOS (Days 1–5)

### K1 — Boot to kernel entry (Day 1)

| | |
|---|---|
| **Prerequisites** | M0 accepted; gate G-AOS approved. |
| **Spec sections** | `AOS_KERNEL_SPEC.md` §2–§7, §11, §12.1, §12.2, §12.4 |

**Deliverables**

| Task | Owner | Paths |
|---|---|---|
| K1-C1 | Claude | `agam-os/Makefile` (all targets of spec §11.1), `agam-os/tools/mkiso.sh`, `agam-os/boot/limine.conf` |
| K1-C2 | Claude | `agam-os/kernel/linker.ld`, `agam-os/kernel/src/boot/limine_requests.c`, `agam-os/kernel/src/main.c` |
| K1-C3 | Claude | `agam-os/kernel/src/serial.c`, `kprintf.c`, `panic.c`, `string.c`, `mm.c`, `ktest.c`, `agam-os/kernel/arch/x86_64/io.h`, `cpu.h`, headers under `agam-os/kernel/include/aos/` |
| K1-C4 | Claude | `agam-dev/reports/claude/K1.md` — resolution of V-01…V-05 with evidence |
| K1-X1 | Codex | `agam-dev/verify/run.py`, `agam-os/tests/harness/qemu.py` (QEMU runner: timeout, serial capture, exit status, log files) |
| K1-X2 | Codex | `agam-os/tests/cases/smoke.c`, `agam-os/tests/cases/panic_exit.c`, `agam-os/tests/test_k1.py` (tests T-K1-01…09 in `TEST_PLAN.md`) |
| K1-X3 | Codex | `agam-dev/reports/codex/K1-evidence.md`, `agam-dev/reports/codex/reviews/K1.md` |

**Verification commands**

```
make -C agam-os distclean && make -C agam-os iso
llvm-readelf -hlW agam-os/build/kernel/aos.elf
make -C agam-os iso TEST=smoke
python3 agam-dev/verify/run.py aos --suite k1
```

**Acceptance criteria**

1. `make -C agam-os iso` exits 0 from a clean tree with zero warnings.
2. The ELF is `ELF64`, machine `X86-64`, type `EXEC`, entry point ≥ `0xffffffff80000000`, with exactly three `LOAD` segments flagged `R E`, `R`, `RW`.
3. Disassembly contains no `xmm`, `ymm` or x87 register operands.
4. Booting the normal ISO prints, in order, the banner, `limine:`, `hhdm:`, `kernel:`, `init complete`, `halt` lines exactly as formatted in spec §12.1, within 10 seconds.
5. The `smoke` test kernel ends with `@@AOS-TEST@@ RESULT PASS smoke` and **host** exit status 33 (guest exit value `0x10`; spec §12.5).
6. The `panic_exit` test kernel prints the panic block and QEMU returns **host** exit status 35 (guest exit value `0x11`).
7. Ten consecutive runs of criterion 5 all pass (no flakiness).
8. The exit convention is confirmed on the owner's QEMU (T-K1-09): a kernel that never calls the exit device is reported as a timeout, and host status 0 is classified as failure.

**Risks**: R-04 (Limine option or macro names differ from this draft), R-05 (silent triple fault before serial works), R-07.
**Schedule note**: K1 carries the most unknowns. K2 is deliberately light so that K1 can overrun into Day 2 without moving the sprint.

### K2 — GDT, TSS and build quality gates (Day 2)

| | |
|---|---|
| **Prerequisites** | K1 accepted. |
| **Spec sections** | §8, §4.3, §14 |

**Deliverables**

| Task | Owner | Paths |
|---|---|---|
| K2-C1 | Claude | `agam-os/kernel/arch/x86_64/gdt.c`, `gdt_flush.S`, `agam-os/kernel/include/aos/gdt.h`; boot step 4 in `main.c` |
| K2-C2 | Claude | Close K1 review findings; `agam-dev/reports/claude/K2.md` |
| K2-X1 | Codex | `agam-os/tests/cases/gdt_selectors.c`, `kprintf_formats.c`, `string_funcs.c`; `agam-os/tests/test_k2.py` (T-K2-01…04) |
| K2-X2 | Codex | `agam-os/tests/test_build.py` — reproducible-ELF check (T-K2-05), symbol-order check (T-K2-06) |
| K2-X3 | Codex | `agam-dev/reports/codex/K2-evidence.md`, `agam-dev/reports/codex/reviews/K2.md` |

**Verification commands**

```
make -C agam-os distclean && make -C agam-os iso && sha256sum agam-os/build/kernel/aos.elf
make -C agam-os clean && make -C agam-os iso && sha256sum agam-os/build/kernel/aos.elf      # must match
python3 agam-dev/verify/run.py aos --suite k1 --suite k2
```

**Acceptance criteria**

1. Boot log contains `[aos] gdt: loaded cs=0x0008 tr=0x0028`.
2. `gdt_selectors` passes: `CS == 0x08`, `TR == 0x28`, `DS == ES == SS == 0x10`.
3. `kprintf_formats` output matches the expected text byte for byte for every conversion in spec §7.1.
4. Two clean builds of the same tree produce byte-identical `aos.elf` (equal SHA-256).
5. All K1 tests still pass.

**Risks**: R-06 (wrong descriptor encoding causes #GP with no IDT yet, seen only as a reset — diagnose with the QEMU `-d int` log).

### K3 — IDT and CPU exception handling (Day 3)

| | |
|---|---|
| **Prerequisites** | K2 accepted. |
| **Spec sections** | §9, §12.3, §12.4 |

**Deliverables**

| Task | Owner | Paths |
|---|---|---|
| K3-C1 | Claude | `agam-os/kernel/arch/x86_64/isr_stubs.S`, `idt.c`, `exception.c`, `agam-os/kernel/include/aos/interrupt.h`; boot step 5 |
| K3-C2 | Claude | `ktest_expect_fault` support in `agam-os/kernel/src/ktest.c`; `agam-dev/reports/claude/K3.md` (including V-06; V-07 and V-08 only if the stretch tests are run) |
| K3-X1 | Codex | Required cases: `agam-os/tests/cases/exc_bp_resume.c`, `exc_de.c`, `exc_ud.c`, `exc_gp.c`, `exc_pf_read.c`, `exc_unexpected.c` |
| K3-X4 | Codex | **Stretch** cases: `exc_pf_write_rodata.c`, `exc_pf_exec_data.c`, `exc_df_ist.c` |
| K3-X2 | Codex | `agam-os/tests/test_k3.py` (required: T-K3-01…06 and T-K3-10; stretch: T-K3-07…09), dump parser in `agam-os/tests/harness/aosdump.py` |
| K3-X3 | Codex | `agam-dev/reports/codex/K3-evidence.md`, `agam-dev/reports/codex/reviews/K3.md` (must include an independent check of the stack-alignment argument in spec §9.4 against the assembly) |

**Verification commands**

```
python3 agam-dev/verify/run.py aos --suite k3
python3 agam-dev/verify/run.py aos --suite k1 --suite k2 --suite k3
```

**Acceptance criteria**

Required:

1. Boot log contains `[aos] idt: loaded 256 gates`.
2. `exc_bp_resume`: an `int3` prints the breakpoint line and execution continues to a PASS.
3. Each required faulting case ends with a dump whose `vector=` and `name=` match: `exc_de` → 0 `#DE`; `exc_ud` → 6 `#UD`; `exc_gp` → 13 `#GP`; `exc_pf_read` → 14 `#PF`.
4. `exc_pf_read`: `cr2` equals the faulting address used by the test; `pf: present=0 write=0`.
5. `exc_unexpected` (a fault with no armed expectation) returns host exit status 35 and prints the panic block.
6. In every dump, `cs=0x0008` and `ss=0x0010`, and all 15 general registers appear.
7. All earlier suites still pass.
8. The code that fills the TSS interrupt-stack entries and the IDT's interrupt-stack indexes exists and has been reviewed against spec §8.2 and §9.1, even though its test is stretch.

Stretch (O-09; reported, never blocking):

- S1. `exc_pf_write_rodata`: `pf: present=1 write=1`; `cr2` lies inside `[__rodata_start, __rodata_end)`.
- S2. `exc_pf_exec_data`: `pf: present=1 exec=1`; `rip == cr2`.
- S3. `exc_df_ist`: a complete dump with `vector=8` and `error=0x0000000000000000`. A complete dump is only possible if the handler ran on IST1, because the interrupted stack pointer is deliberately invalid.

**Risks**: R-06 (frame layout or alignment errors), R-05, R-09.

### K4 — Physical memory manager (Day 4)

| | |
|---|---|
| **Prerequisites** | K3 accepted (so PMM bugs produce dumps, not resets). |
| **Spec sections** | §10, §12.1 |

**Deliverables**

| Task | Owner | Paths |
|---|---|---|
| K4-C1 | Claude | `agam-os/kernel/src/pmm.c`, `agam-os/kernel/include/aos/pmm.h`; memory-map logging; boot step 6 |
| K4-C2 | Claude | `agam-dev/reports/claude/K4.md` |
| K4-X1 | Codex | `agam-os/tests/cases/pmm_stats.c`, `pmm_alloc_free.c`, `pmm_unique.c`, `pmm_contig.c`, `pmm_exhaust.c`, `pmm_rw.c`, `pmm_reuse.c`, `pmm_double_free.c` |
| K4-X4 | Codex | **Stretch** cases (invalid-free injection): `pmm_free_unaligned.c`, `pmm_free_low.c`, `pmm_free_beyond.c`, `pmm_free_bitmap.c` |
| K4-X2 | Codex | `agam-os/tests/test_k4.py` (T-K4-01…10), memory-map log parser in `agam-os/tests/harness/aosdump.py` |
| K4-X3 | Codex | `agam-dev/reports/codex/K4-evidence.md`, `agam-dev/reports/codex/reviews/K4.md` |

**Verification commands**

```
python3 agam-dev/verify/run.py aos --suite k4
python3 agam-dev/verify/run.py aos --all
```

**Acceptance criteria** (at the reference 256 MiB configuration)

1. Boot log lists every memory-map entry; entries are in ascending base order; the sum of `usable` lengths is ≤ configured RAM and ≥ 75 % of it (sanity bound, not a specification of Limine).
2. Invariant 1 of spec §10.5 holds after initialisation and after every test.
3. `pmm_unique`: 1 000 consecutive allocations return 1 000 distinct, 4 KiB-aligned addresses ≥ `0x100000`, each inside a `usable` entry from the log and outside the bitmap range from the log.
4. `pmm_alloc_free`: after freeing all of them, `free_frames` equals its initial value.
5. `pmm_exhaust`: allocating until `PMM_INVALID` yields exactly the initial `free_frames` allocations, with no panic; after freeing all, the count is restored.
6. `pmm_contig`: an allocation of 16 frames returns 16 adjacent frames; `pmm_alloc_contig(0)` returns `PMM_INVALID`; a request larger than total free memory returns `PMM_INVALID`.
7. `pmm_rw`: a distinct 64-bit pattern written to the first and last word of each of 256 allocated frames reads back correctly.
8. `pmm_double_free` ends in a panic whose reason starts with `pmm:` and host exit status 35. *Stretch*: the four invalid-free cases (unaligned, below 1 MiB, beyond the bitmap's coverage, inside the bitmap) do the same; the checks themselves are required code and are covered by review either way.
9. All earlier suites still pass.

**Risks**: R-08, R-09.

### K5 — Hardening, test matrix and review closure (Day 5)

| | |
|---|---|
| **Prerequisites** | K4 accepted. |

**Deliverables**

| Task | Owner | Paths |
|---|---|---|
| K5-C1 | Claude | Fixes for all open review findings of severity High or Medium; `agam-dev/reports/claude/K5.md` |
| K5-C2 | Claude | `agam-dev/reports/claude/gdb-session.md` — a recorded GDB session: break at `kernel_entry`, `info registers`, break in `interrupt_dispatch` during `exc_bp_resume`, backtrace |
| K5-X1 | Codex | `agam-os/tests/test_matrix.py` — full suite at 128 MiB, 256 MiB, 1 GiB, 4 GiB (T-K5-01…04); 20-run stability loop of the full suite at 256 MiB (T-K5-05) |
| K5-X2 | Codex | Optional UEFI run if OVMF is present (T-K5-06; recorded as SKIPPED with reason otherwise) |
| K5-X3 | Codex | `agam-dev/reports/codex/sprint1-final.md` — complete evidence table and review sign-off |
| K5-A1 | Architect | Update `DECISIONS.md` with anything learned; propose the K6 specification |

**Verification commands**

```
make -C agam-os distclean && make -C agam-os test
python3 agam-dev/verify/run.py aos --all --matrix
python3 agam-dev/verify/run.py aos --all --repeat 20
python3 agam-dev/verify/check_ownership.py --agent claude --base <sprint start tag>
python3 agam-dev/verify/check_ownership.py --agent codex  --base <sprint start tag>
```

**Acceptance criteria**

1. The full suite passes at all four memory sizes; at 4 GiB the log shows at least one `usable` entry with base ≥ `0x100000000`.
2. 20 consecutive full-suite runs pass with zero failures.
3. No open review finding of severity High or Medium.
4. Ownership checks pass for both agents over the whole sprint.
5. Every item V-01…V-09 has recorded evidence or a documented spec change.
6. The owner accepts the sprint; the integration branch is tagged `aos-v0.1.0-sprint1`.

**Risks**: R-10, R-18.

---

## 4. Sprint 2 — AgamLang v0.1 (proposed Days 6–10)

Common verification entry point: `python3 agam-dev/verify/run.py agamlang --suite <id>`. All commands run from the repository root with `PYTHONPATH=agam-lang/src`, or after `pip install -e agam-lang`.

### L1 — Lexer and Fusion keywords

| | |
|---|---|
| **Prerequisites** | Gate G-LANG approved; M0 accepted; Python ≥ 3.11 confirmed by `check_env.sh`. |
| **Spec sections** | `AGAMLANG_SPEC.md` §2–§4, §8.1–§8.2 (lexical errors), §9 (`tokens`) |

| Task | Owner | Paths |
|---|---|---|
| L1-C1 | Claude | `agam-lang/pyproject.toml`, `src/agamlang/{__init__,__main__,cli,source,tokens,keywords,lexer,diagnostics}.py`, `messages/en.py`, `messages/hi.py` (lexical codes) |
| L1-X1 | Codex | `agam-lang/tests/test_lexer.py`, `test_keywords.py`, `agam-lang/tests/fusion/` (triples of `.agam` files: `en`, `hi`, `mixed`) |

**Verification**: `python3 agam-dev/verify/run.py agamlang --suite l1`

**Acceptance criteria**

1. Every alias in spec §3.2, in lower, upper and mixed case, lexes to its canonical kind (generated test, one case per alias and case variant).
2. For every fusion triple, `tokens --canonical` output is identical across the three files.
3. Indentation tests: correct `INDENT`/`DEDENT` sequences for nested blocks, blank lines, comment lines, and bracket continuation; E0103 and E0104 raised with the right line and column.
4. Each of E0100–E0106 has at least one test asserting code, line and column.
5. Every keyword token carries the user's exact lexeme and the correct `lang`.
6. No input from a 500-case random byte/character fuzz run raises anything other than `AgamError` (seeded, reproducible).

**Risks**: R-13, R-14.

### L2 — Parser and AST

| | |
|---|---|
| **Prerequisites** | L1 accepted. |
| **Spec sections** | §5, §6, §8 (E02xx), §9 (`ast`, `check`) |

| Task | Owner | Paths |
|---|---|---|
| L2-C1 | Claude | `src/agamlang/nodes.py`, `parser.py`; `ast` and `check` commands; E02xx messages |
| L2-X1 | Codex | `agam-lang/tests/test_parser.py`, `agam-lang/tests/ast_golden/` (program + expected JSON), `test_fusion_ast.py` |

**Verification**: `python3 agam-dev/verify/run.py agamlang --suite l2`

**Acceptance criteria**

1. Each grammar production in spec §5 is covered by at least one golden AST test.
2. Operator precedence and associativity: a table-driven test of at least 20 expressions compares the AST shape with the expected one (for example `-2 ** 2`, `1 + 2 * 3`, `not a and b or c`).
3. For every fusion triple, `ast` output is identical across the three files.
4. Each of E0201–E0212 has at least one test asserting code, line and column.
5. Parser fuzz: 500 seeded random token-level mutations of valid programs raise only `AgamError`.

**Risks**: R-17.

### L3 — Interpreter core

| | |
|---|---|
| **Prerequisites** | L2 accepted. |
| **Spec sections** | §7.1–§7.5 (without functions, lists, for loops), §9 (`run`) |

| Task | Owner | Paths |
|---|---|---|
| L3-C1 | Claude | `src/agamlang/values.py`, `interpreter.py`; `run` command; E03xx/E04xx messages used so far |
| L3-X1 | Codex | `agam-lang/tests/conformance/core/` (`.agam` + `.out` + optional `.in` and `.err`), `test_conformance.py` |

**Verification**: `python3 agam-dev/verify/run.py agamlang --suite l3`

**Acceptance criteria**

1. Every row of the operator table in spec §7.3 has a passing test, and every unsupported type combination tested raises E0302.
2. `show` display forms match spec §7.4 exactly for every value type available so far.
3. `if`/`elif`/`else`, `while`, `repeat`, `stop`, `skip` each have conformance programs with expected output.
4. Non-boolean conditions raise E0302; bad repeat counts raise E0404; division by zero raises E0401.
5. Exit statuses match spec §9; standard output and standard error are separated as specified.

### L4 — Functions, lists, built-ins

| | |
|---|---|
| **Prerequisites** | L3 accepted. |
| **Spec sections** | §7.2, §7.4 (functions, for), §7.5, §7.6 |

| Task | Owner | Paths |
|---|---|---|
| L4-C1 | Claude | `src/agamlang/stdlib.py`; functions, closures, lists, indexing, for loops, limits in `interpreter.py`; `agam-lang/examples/*.agam` |
| L4-X1 | Codex | `agam-lang/tests/conformance/functions/`, `lists/`, `builtins/`, `limits/` |

**Verification**: `python3 agam-dev/verify/run.py agamlang --suite l4`

**Acceptance criteria**

1. Every built-in in spec §7.6 is tested under both its English and Hinglish name, including each documented error.
2. Recursion: `factorial(20)` and `fib(15)` produce correct values; unbounded recursion raises E0402 and never a Python traceback.
3. Closures: a counter-maker example behaves per the lexical scoping rule.
4. `--max-steps 1000` on an infinite loop raises E0403.
5. Indexing errors raise E0306 with the valid range in the message.
6. Every file in `agam-lang/examples/` runs with exit status 0.

### L5 — Diagnostics quality and conformance closure

| | |
|---|---|
| **Prerequisites** | L4 accepted. |
| **Spec sections** | §8 in full |

| Task | Owner | Paths |
|---|---|---|
| L5-C1 | Claude | Complete both message catalogues, "did you mean", message-language selection; fix review findings |
| L5-X1 | Codex | `agam-lang/tests/test_diagnostics.py` (golden rendered errors in both languages), `test_catalogue.py`, `test_fuzz.py`, `agam-dev/reports/codex/agamlang-v0.1-final.md` |

**Verification**: `python3 agam-dev/verify/run.py agamlang --all`

**Acceptance criteria**

1. Every code in spec §8.4 exists in both catalogues, and every code has at least one triggering test.
2. Golden-file tests of the rendered diagnostic (header, source line, caret position, message, tip) pass in English and Hinglish for at least one error per stage.
3. "Did you mean" suggests the right word for at least 10 listed misspellings (5 keywords, 5 variable names) and stays silent for 5 unrelated words.
4. `--messages auto` picks Hinglish for a majority-Hinglish file and English otherwise.
5. Fuzz: 2 000 seeded cases (random text, mutated valid programs) produce only exit statuses 0, 1 or 2 — never 70, never a traceback.
6. The complete fusion corpus (at least 15 triples covering every keyword) passes token, AST and output equivalence.
7. Owner accepts; tag `agamlang-v0.1.0`.

**Risks**: R-13 (owner's review of Hinglish wording may change message text late — golden files are regenerated only by Codex after an approved spec change).

---

## 5. Agent task summary

| Milestone | Claude Code (implements) | Codex (tests, reviews) |
|---|---|---|
| M0 | Limine pins, fetch script, Makefile skeleton | Environment check, ownership checker, test scaffolding, independent Limine pin verification |
| K1 | Build system, linker script, boot, serial, kprintf, panic, ktest | QEMU harness, smoke and panic cases, K1 review |
| K2 | GDT/TSS | Selector, kprintf, string and build-quality tests, K2 review |
| K3 | IDT, stubs, exception dump | Six required exception cases (three more as stretch), dump parser, K3 review |
| K4 | PMM | Eight required PMM cases (four invalid-free cases as stretch), memory-map parser, K4 review |
| K5 | Review fixes, GDB session record | Memory matrix, stability loop, final evidence |
| L1–L5 | Interpreter modules per spec | Unit, golden, conformance, fusion, fuzz tests; reviews |

Rules common to all tasks are in `CLAUDE.md` and `AGENTS.md`.

---

## 6. Later milestones (outline; each requires an approved specification first)

| ID | Content | Depends on |
|---|---|---|
| K6 | Virtual memory manager: own page tables, guard pages, reclaim bootloader memory | Sprint 1 |
| K7 | Kernel heap | K6 |
| K8 | ACPI MADT, local APIC, I/O APIC, timer, interrupts enabled | K6 |
| K9 | Framebuffer console | K6 |
| K10 | Kernel threads, scheduler | K7, K8 |
| K11 | User mode, system calls | K10 |
| L6 | REPL, text interpolation, dictionaries (v0.2) | L5 |
| L7 | `agam translate` (re-emit a program in one language) | L5 |
| L8 | Bytecode VM; later native back end | L6 |
| D1 | `agamdev` CLI | Sprint 1, L5 |
| X1 | AgamLang running on AOS | K11, L8 |

---

## 7. Definition of done (every milestone)

1. All deliverables exist at the listed paths.
2. All verification commands were run **on the integration branch** after merging both agents' work, and exited 0.
3. Evidence reports are filed by both agents (`TEST_PLAN.md` §2).
4. Codex's review is filed; no High or Medium finding is open.
5. Both ownership checks pass.
6. Every commit of the milestone is authored and committed as the owner's GitHub identity, with no AI attribution anywhere (O-10).
7. Merges into `main`, tags and pushes were done by the owner or with the owner's explicit approval (O-11).
8. Stretch items are listed in the evidence report with their real status, including `SKIPPED`.
9. The owner has accepted the milestone.

---

## 8. Risk register

Likelihood and impact are the architect's judgement (L = low, M = medium, H = high), not measurements.

| ID | Risk | L | I | Mitigation | Detected by |
|---|---|---|---|---|---|
| R-01 | Host environment unknown; required tools may be missing or old | H | H | M0 exists to verify; nothing assumed installed; versions recorded in every report | `check_env.sh` |
| R-02 | Worktree `.git` links hold Windows absolute paths. Linux git cannot follow them, and from Linux the two agent worktrees are reported as **prunable** (observed 2026-11-12). Running `git worktree prune` or `git gc` from WSL could delete their registrations. | H | H | **Git is never run from WSL** (O-01); all git operations use Git for Windows; the build never calls git (P-06) | M0 criterion 5; `CLAUDE.md`/`AGENTS.md` §4 |
| R-03 | CRLF line endings break scripts, Makefiles and golden files | M | M | `.gitattributes` forcing LF (P-16) | M0 criterion 4 |
| R-04 | Limine release or protocol details differ from this draft (the project changes quickly) | M | H | Pin release and header by hash; verify names on first use (V-02…V-04); use only the non-deprecated base revision | K1 |
| R-05 | Faults before the IDT exists appear only as a reset | H | M | Serial first; `-no-reboot`; `-d int,cpu_reset` log kept for every run; status 0 treated as failure | Harness |
| R-06 | Descriptor encoding, interrupt frame layout or stack alignment errors | M | H | `_Static_assert` on every hardware structure; independent review of the alignment argument; nine exception tests | K2, K3 |
| R-07 | Compiler emits SSE or uses the red zone | L | H | Flags in spec §2; disassembly check in the build tests | K1, K2 |
| R-08 | PMM hands out memory that is in use (bitmap, low memory, bootloader structures) | M | H | Usable entries only; nothing below 1 MiB; bitmap self-reservation; address-range checks in tests | K4 |
| R-09 | Access through the HHDM to memory that is not mapped (Reserved, Bad) | M | M | `phys_to_virt` documented as valid only for mapped types; PMM touches Usable memory only | K3 dumps |
| R-10 | QEMU/TCG behaviour differs from real hardware | H | L | Real hardware is out of scope; recorded as a limitation | — |
| R-11 | Both agents modify the same file | M | H | Disjoint path ownership; automated ownership check in every gate | `check_ownership.py` |
| R-12 | A report claims success without having run anything | M | H | Mandatory evidence format; statuses `UNVERIFIED` / `BUILD-VERIFIED` / `TEST-VERIFIED`; integration-branch gate re-runs everything | Gate |
| R-13 | Hinglish keywords read unnaturally or collide with common names | M | M | Owner reviews the table (Q-07); table is data in one module; "did you mean" | L1, owner review |
| R-14 | Indentation rules confuse beginners | H | M | Dedicated error codes and messages; indentation was chosen by the owner (O-05) | L1, L5 |
| R-15 | Scope creep toward non-goals | M | H | Non-goals are confirmed (C-10); roadmap changes need approval | Architect |
| R-16 | Python version differences between machines | L | M | Minimum version declared and checked | `check_env.sh` |
| R-17 | Spec ambiguity makes tests and implementation disagree | H | M | Spec is the arbiter; disagreements become proposals under `agam-dev/reports/<agent>/proposals/`; architect resolves and records | Gate failures |
| R-18 | UEFI firmware (OVMF) unavailable | M | L | BIOS is the required path (Q-03); UEFI test may be SKIPPED with a recorded reason | K5 |
| R-19 | Tests are written to match the implementation instead of the spec | M | H | Codex writes tests from the spec before or without reading the implementation; review notes must cite spec sections | Reviews |
| R-20 | Under Limine's page tables the HHDM maps memory read/write/execute, so the kernel image's W^X protection can be bypassed through the HHDM alias | H | L (Sprint 1 has no untrusted code) | Recorded as a known limitation; removed when the kernel builds its own page tables (K6) | Review |
| R-21 | An agent adds AI attribution to a commit (author, committer, `Co-authored-by`, "Generated with" text), or commits with the wrong email | M | M | O-10 rule in `CLAUDE.md` and `AGENTS.md`; identity set in repository-local git config by the owner; `git log` check in M0 and in every gate; history is fixed before any push | M0 criterion 8; gate |
| R-22 | Building in WSL2 on the Windows-hosted folder (`/mnt/d/…`) is slow or behaves differently (file modes, case-insensitive names, CRLF) | M | M | LF enforced by `.gitattributes`; no two files differing only by case; `core.filemode=false` already set; M0 criterion 9; fallback is to move the repository into the WSL file system (needs worktrees recreated) | M0 |
| R-23 | Empty directories are not stored by Git: `agam-os/`, `agam-lang/`, `agam-dev/` and `docs/` currently exist only in the `ecosystem` working folder and are absent from both agent worktrees | H | L | They appear in the agent worktrees as soon as their first files are committed and merged; no placeholder files needed | M0 |

---

## 9. Critical blockers before implementation

The authoritative list is `DECISIONS.md` §9. In short, implementation cannot start until: the toolchain is installed and verified in WSL2 (B-01); Codex has independently verified the Limine pin (B-02); the Git identity is set and checked (B-03); the owner has decided how these documents are committed (B-04); and Codex's architecture review is complete with no High or Medium finding open (B-05). The interpreter additionally needs the Hinglish keyword table approved (B-06).
