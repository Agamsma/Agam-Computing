# Agam Computing Ecosystem — Test Plan

| Field | Value |
|---|---|
| Status | **Draft v0.1, revision 3 — NOT APPROVED.** Addresses review findings AR-02, AR-03, AR-04, AR-08, AR-10, AR-16, AR-19, AR-20 and the test side of AR-05…AR-14 (`DECISIONS.md` §10); all are *addressed in draft — pending independent re-review*. |
| Date | 2026-11-12 (revision 3) |
| Codex owns | `agam-os/tests/**`, `agam-lang/tests/**`, `agam-dev/verify/**`, `agam-dev/reports/codex/**` |
| Paths | Relative to the repository root |

No project test, harness or runner exists yet, and none has been run. What has been executed so far is listed in `DECISIONS.md` §8.3 and §8.5. Timeouts and counts here are budgets chosen by the architect, not measurements.

---

## 1. Principles

1. **Tests are written from the specification, not from the implementation**, and never take their expected values from implementation data (§6.1).
2. **A test that cannot fail is not a test.** For each new test, Codex records one deliberate failure.
3. **Only explicit success counts.** For kernel tests, host exit status 33 plus the expected serial lines is success (`AOS_KERNEL_SPEC.md` §12.5).
4. **Acceptance is decided on a gate branch**, by the procedure of §8, bound to exact commits.
5. **Required tests must run.** A required test that is skipped, missing or not selected fails acceptance (§4.4).
6. **Required before stretch** (O-09). Stretch results are reported honestly but never decide the exit status. A stretch failure is always triaged (§4.5).
7. **Build once, run many.** Repeats and matrices never rebuild what an earlier step of the same run built (§4.8).

---

## 2. Evidence protocol (P-15)

### 2.1 Status vocabulary

| Status | Meaning | Minimum evidence |
|---|---|---|
| `UNVERIFIED` | Written, or designed, but not built or run | none |
| `BUILD-VERIFIED` | Compiles/links (or imports) without warnings | command, exit status, tool versions, commit |
| `TEST-VERIFIED` | The named tests passed | command, exit status, result lines, tool versions, commit, artefact path |
| `FAILED` | Ran and failed | as above, plus the failing output |
| `SKIPPED` | Could not run | reason |

"Works", "passes", "tested", "verified" MUST NOT be used without one of these and its evidence. Source inspection and arithmetic are reported as such, never as runtime results. Performance numbers MUST NOT be stated without a measurement command and its raw output.

### 2.2 Evidence block

````
### Evidence: <what is being claimed>
- Status: TEST-VERIFIED
- Commit: <full SHA> (branch <name>; working tree clean: yes/no)
- Gate binding (if a gate run): BASE <sha>, TIP_C <sha>, TIP_X <sha>, GATE_HEAD <sha>
- Environment: <output of `sh agam-dev/verify/check_env.sh --brief`>
- Command: `<exact command>`
- Exit status: <n>
- Date/time (UTC): <timestamp>
- Artefacts: <path>
- Output excerpt:
```
<relevant lines, unedited; "[...]" marks omissions>
```
````

Evidence about an untracked file (for example a review report that has not been committed) cites the commit that was reviewed and the SHA-256 of the file.

### 2.3 Where reports live

| File | Writer | Content |
|---|---|---|
| `agam-dev/reports/claude/<milestone>.md` | Claude | What was implemented, per-item status, evidence blocks, open issues |
| `agam-dev/reports/claude/<milestone>-review-response.md` | Claude | Response to each review finding |
| `agam-dev/reports/claude/proposals/<nnn>-<slug>.md` | Claude | Requested spec changes, or objections to a test |
| `agam-dev/reports/codex/<milestone>-evidence.md` | Codex | Evidence for every test, manual-evidence and review ID of the milestone |
| `agam-dev/reports/codex/reviews/<milestone>.md` | Codex | Review findings (§7) |
| `agam-dev/reports/codex/proposals/<nnn>-<slug>.md` | Codex | Requested spec changes |
| `agam-dev/reports/codex/ARCHITECTURE_REVIEW.md` and its evidence folder | Codex | Architecture reviews (path chosen by the owner for the first review) |
| `agam-dev/gates/<milestone>/` | Owner | Gate record: manifests, runner summary, acceptance (§8) |

### 2.4 Identifiers and severities

| Prefix | Meaning |
|---|---|
| `T-<milestone>-nn` | Automated test in the runner's catalogue |
| `E-<milestone>-nn` | Manual evidence: a command run by hand, or an owner action, recorded with an evidence block |
| `RV-<milestone>-nn` | Mandatory review item, answered in Codex's review |

Severities: **Critical** and **Major** block acceptance; **Minor** should be fixed but does not block; **Informational** records a result. (These replace the High/Medium/Low names of earlier revisions: Critical and Major correspond to High and Medium, Minor to Low.)

---

## 3. Test levels

| Level | What | Where | Tooling |
|---|---|---|---|
| Tooling self-test | Runner accounting, ownership checker, environment | Host | Python standard library |
| Static | Build cleanliness, ELF structure, forbidden instructions, reproducibility | Host | `make`, LLVM tools, `sha256sum` |
| Boot (observe) | Normal kernel boots and logs correctly | QEMU | Harness |
| In-kernel | Test case compiled into a test kernel; result through the exit device | QEMU | Harness + `agam-os/tests/cases/*.c` |
| Matrix | Required in-kernel tests across memory sizes and repeats | QEMU | Harness |
| Unit / golden | AgamLang lexer, parser, diagnostics | Host Python | `pytest` |
| Conformance | `.agam` programs with expected output or error | Host Python | `pytest` |
| Fusion equivalence | Keyword substitutions; built-in alias behaviour | Host Python | `pytest` |
| Fuzz | Random and mutated input never produces a traceback or status 70 | Host Python | `pytest`, seeded |
| Manual evidence, review | `E-` and `RV-` items | — | §2.4, §7 |

---

## 4. Runner contract (`agam-dev/verify/run.py`)

The runner is Git-free and uses only the Python standard library. It is created in M0 with its accounting logic, the `m0` suite and its own self-tests; later suites are added in the milestone that introduces them.

### 4.1 Catalogue and classes

`agam-dev/verify/catalogue.json` is hand-written by Codex from §5 and §6 of this document. Every test has an ID, a suite, a **class** and a **mode**:

| Class | Meaning |
|---|---|
| `required` | Must be selected, must run, must pass |
| `stretch` | Run only with `--stretch`; reported separately; never changes the exit status |
| `optional` | Needs an optional tool (UEFI firmware); may be `SKIP`; never changes the exit status |

| Mode | Verdict rule |
|---|---|
| `host` | A host command or Python check |
| `exit` | Test kernel: expected host exit status and serial lines (§4.7) |
| `observe` | Normal kernel: wait for `[aos] halt`, terminate QEMU, check the log (§4.7). This is a full verdict, not a skip. |
| `pytest` | A `pytest` node; one runner result per catalogue ID |

### 4.2 Suites required at each gate (normative)

| Gate | Required suites |
|---|---|
| M0 | `m0` |
| K1 | `m0`, `k1` |
| K2 | `m0`, `k1`, `k2` |
| K3 | `m0`, `k1`, `k2`, `k3` |
| K4 | `m0`, `k1`…`k4` |
| K5 | `m0`, `k1`…`k5` |
| L1 | `m0`, `l1` |
| L2 | `m0`, `l1`, `l2` |
| L3 | `m0`, `l1`…`l3` |
| L4 | `m0`, `l1`…`l4` |
| L5 | `m0`, `l1`…`l5` |

**Cross-track regression:** the gate also runs the suites of every milestone listed in `agam-dev/gates/ACCEPTED` (one milestone ID per line, maintained by the owner). So once L2 is accepted, a K-gate also runs `l1` and `l2`, and the reverse. Suites of milestones that are neither the target nor accepted are **outside** the run: they are not selected and not reported as skipped.

### 4.3 Commands

```
python3 agam-dev/verify/run.py gate <MILESTONE> [--stretch] [--mem SIZE] [--matrix] [--repeat N]
python3 agam-dev/verify/run.py dev <suite>... [--only TEST_ID] [--stretch]
```

- `gate` selects every required test of §4.2 and requires the binding files of §4.6. It is the only command whose result counts for acceptance.
- `dev` is for an agent's own worktree. Its summary starts with `DEV RUN — not acceptance evidence`.
- `make -C agam-os test MILESTONE=<id>` is a convenience that calls `gate`.
- `--matrix` and `--repeat` exist only from K5 (suite `k5` uses them internally).

### 4.4 Result accounting and exit status (normative)

One line per test: `PASS|FAIL|SKIP|MISSING <class> <test id> <name> (<seconds>s)[: <reason>]`. Then:

```
REQUIRED selected=<n> passed=<n> failed=<n> skipped=<n> missing=<n>
STRETCH  selected=<n> passed=<n> failed=<n> skipped=<n>
OPTIONAL selected=<n> passed=<n> failed=<n> skipped=<n>
```

- `MISSING`: the catalogue lists a required test for a selected suite, but no implementation is registered for it.
- **Exit status 0 only if** `REQUIRED selected > 0` **and** `passed == selected` (so `failed == skipped == missing == 0`).
- Exit status 1 if any required test failed, was skipped or is missing, or if no required test was selected.
- Exit status 2 for a usage or environment error, including a failed binding check (§4.6).
- Stretch and optional results never change the exit status.
- Artefacts go to `agam-os/build/test-results/<UTC timestamp>/`, with a `summary.json` holding every line above, the binding, and `check_env.sh --brief`.

### 4.5 Stretch triage

Every stretch `FAIL` produces a finding in Codex's review. The architect classifies it before acceptance: test defect; behaviour outside the normative text; or **violation of a normative requirement**, which becomes a blocking Major finding. Acceptance needs every stretch failure classified. A stretch test that was not run is recorded as `SKIPPED`, and the behaviour it covers stays `UNVERIFIED`.

### 4.6 Binding results to commits; stale results (normative)

A `gate` run reads `.gate/<MILESTONE>/gate.json` and `.gate/<MILESTONE>/tree.lst`, produced on Windows in §8:

- `gate.json` holds `BASE`, `TIP_C`, `TIP_X`, `GATE_HEAD` (full SHAs) and the SHA-256 of each manifest file.
- `tree.lst` is the byte-exact output of `git ls-tree -r -z --full-tree <GATE_HEAD>`.

Before running any test, the runner recomputes, without Git, the blob ID of every listed file from the working tree (SHA-1 of `blob <size>\0<content>`), and checks that every mode is `100644`. Any difference, missing file or extra tracked-looking difference is exit status 2. This catches an edited tree, a wrong branch, and CRLF conversion.

`summary.json` repeats the four SHAs. **A gate result is valid only for exactly those SHAs and that tool environment.** It is void if `main` no longer equals `BASE`, if either agent tip has a different SHA, if the merge was redone, if the tree fails the check, or if `check_env.sh --brief` differs from the recorded one.

### 4.7 Running kernel tests (`agam-os/tests/harness/qemu.py`, from K1)

1. Build with `make -C agam-os iso TEST=<case>` (or `iso` for the normal kernel). A build failure is a test failure.
2. Launch QEMU with the reference command of `AOS_KERNEL_SPEC.md` §11.5, serial to `serial.log`, plus `-d int,cpu_reset -D qemu.log`. Memory from `--mem` (default `256M`).
3. Timeout budget: 20 s by default; 60 s for `pmm_exhaust`; 120 s for 4 GiB runs. On timeout, kill QEMU and fail.
4. Save `cmd.txt`, `serial.log`, `qemu.log`, `exit_status`. Normalise `\r\n` to `\n`.
5. `exit` mode: pass only if the **host** exit status equals the expected one (33; 35 for tests that expect a failure result) and every required pattern matches in order.
6. `observe` mode: pass only if the required lines appear in order ending with `[aos] halt` within the budget; the harness then terminates QEMU. The exit device is not involved, and host status 33 is not expected.

### 4.8 Build once

Within one runner invocation each image is built at most once and cached by case name; `--repeat` and `--matrix` reuse it. Tests that rebuild from clean (`T-K1-01`, `T-K2-05`) run first, in a separate build directory, and never inside a repeat or matrix loop.

### 4.9 Parsers

`agam-os/tests/harness/aosdump.py` parses the formats of `AOS_KERNEL_SPEC.md` §12. A line that deviates from a normative format is itself a failure.

---

## 5. AOS test catalogue

Class is `required` unless marked **stretch** or **optional**. Expected host status is 33 for `exit` tests unless stated.

### M0 (suite `m0`)

| ID | Name | Method | Pass condition |
|---|---|---|---|
| T-M0-01 | env_tools | `sh agam-dev/verify/check_env.sh` | Exit 0; every tool of `AOS_KERNEL_SPEC.md` §2 reported with path and version; the script runs no `git` |
| T-M0-02 | pin_fetch | `sh agam-os/tools/fetch-limine.sh` twice | Asset and header SHA-256 equal the PIN files; second run reports already verified; the files of spec §11.2 exist |
| T-M0-03 | tree_integrity | Binding check of §4.6, plus a scan of every listed file | Blob IDs and modes match; no carriage-return byte in any tracked text file |
| T-M0-04 | runner_selftest | The runner executed on synthetic catalogues | (a) all pass → 0; (b) a required `SKIP` → 1; (c) a required `MISSING` → 1; (d) empty required selection → 1; (e) a stretch `FAIL` → 0 and reported; (f) tree differs from `tree.lst` → 2; (g) a suite outside §4.2's set is not selected |
| T-M0-05 | ownership_selftest | The checker of §8.3 on synthetic manifests | Accepts valid manifests; rejects a foreign path, a rename into a foreign path (seen as delete plus add), a mode other than `100644`, a symlink or submodule entry, a type change, overlapping manifests, and a merge manifest that differs from the union in path, blob or mode |
| T-M0-06 | make_skeleton | `make -C agam-os print-flags` | Exit 0; prints every flag of spec §2.1 and §2.2 |

| ID | Manual evidence |
|---|---|
| E-M0-01 | Corrupting the cached Limine asset makes `fetch-limine.sh` exit non-zero |
| E-M0-02 | Git for Windows: `git config user.name`, `user.email` and `git log --format="%an <%ae> / %cn <%ce>%n%B"` over the bootstrap commits show only the identity of `DECISIONS.md` O-18 and no attribution text |
| E-M0-03 | Git for Windows: `git ls-files --eol` shows `i/lf w/lf` for every file in all three worktrees |
| E-M0-04 | The versions printed by T-M0-01 compared with `DECISIONS.md` §8.2; differences accepted by the owner |
| E-M0-05 | Git for Windows: `git worktree list` and `git branch --show-current` in each worktree |
| E-M0-06 | The owner's recorded disposition of the Limine pin evidence (B-02) |
| E-M0-07 | For the M0 gate only: the owner's by-hand comparison of each agent's changed paths with the bootstrap scope (`ROADMAP.md` §2), because no accepted checker exists yet |

### K1 (suite `k1`)

| ID | Name | Mode | Pass condition |
|---|---|---|---|
| T-K1-01 | build_clean | host | `make -C agam-os distclean iso` exits 0 with no `warning:` |
| T-K1-02 | elf_header | host | ELF64, X86-64, type EXEC, entry ≥ `0xffffffff80000000` |
| T-K1-03 | elf_segments | host | Exactly three LOAD segments, flags `R E`, `R`, `RW`, each page-aligned, none W+E; no `INTERP` or `DYNAMIC` header; no `.rela*`/`.rel*` section to apply |
| T-K1-04 | no_fp_simd | host | In `llvm-objdump -d` output of the kernel: no operand naming `%xmm`, `%ymm`, `%zmm`, `%mm0`–`%mm7`, `%st` or `%k0`–`%k7`; and no x87, MMX, SSE/AVX, or state-save mnemonic (`f…` x87 forms, `emms`, `fxsave`/`fxrstor`, `xsave…`/`xrstor…`, `ldmxcsr`/`stmxcsr`, VEX/EVEX-encoded forms). Codex derives the exact pattern from the installed tool's real output and records it. |
| T-K1-05 | boot_banner | observe | Banner, `limine:`, `hhdm:`, `kernel:`, `cpu: nx enabled`, `init complete`, `halt` in order and in the formats of spec §12.1; HHDM offset ≥ `0xffff800000000000`; kernel virt ≥ `0xffffffff80000000` |
| T-K1-06 | smoke | exit | `BEGIN smoke`, `RESULT PASS smoke` |
| T-K1-07 | panic_exit | exit, host 35 | Panic block with `reason: test panic 42`, then `RESULT FAIL panic_exit` |
| T-K1-08 | smoke_stability | exit | T-K1-06 ten times from one built image |
| T-K1-09 | exit_convention | host | On the owner's QEMU: `smoke` gives host 33; `panic_exit` gives 35; the normal kernel run in `exit` mode is reported as a timeout failure; a recorded host status 0 is classified as failure |
| T-K1-10 | kt_check_fail_return | exit, host 35 | A failed `KTEST_CHECK` followed by a normal return: `CHECK-FAILED`, then `RESULT FAIL` |
| T-K1-11 | requests_layout | host | From `llvm-nm`/`llvm-readelf`: start marker, requests and end marker lie in that order inside the R+W segment, below `__bss_start`; symbols ordered and aligned as in spec §4.2; `__kernel_end ≤ 0xffffffffc0000000` |

| ID | Manual evidence / review |
|---|---|
| E-K1-01 | Resolution of V-01…V-05, V-12, V-14 with evidence blocks |
| RV-K1-01 | Linker script and flags match spec §2 and §4.2; nothing calls git; nothing but `fetch-limine.sh` uses the network |

### K2 (suite `k2`)

| ID | Name | Mode | Pass condition |
|---|---|---|---|
| T-K2-01 | gdt_log | observe | `[aos] gdt: loaded cs=0x0008 tr=0x0028` |
| T-K2-02 | gdt_selectors | exit | `CS=0x08`, `TR=0x28`, `DS=ES=SS=0x10` |
| T-K2-03 | kprintf_formats | exit | Output between markers equals the expected text byte for byte, for every conversion of spec §7.1 |
| T-K2-04 | string_funcs | exit | `memset`, `memcpy` (non-overlapping only), `memcmp`, `strlen`, and `memmove` with overlap in both directions |
| T-K2-05 | reproducible_elf | host | Two clean builds give the same SHA-256 of `aos.elf` |

| ID | Review |
|---|---|
| RV-K2-01 | GDT descriptor values, TSS layout and size, zeroed reserved fields and unused interrupt-stack entries, `ltr` selector, against spec §8 |

### K3 (suite `k3`)

Faults are raised with inline assembly. Every dump is checked for all lines of spec §12.3, `cs=0x0008`, `ss=0x0010`.

| ID | Name | Mode | Pass condition |
|---|---|---|---|
| T-K3-01 | idt_log | observe | `[aos] idt: loaded 256 gates` |
| T-K3-02 | exc_bp_resume | exit | `int3` with nothing armed: breakpoint line, execution continues, `RESULT PASS` |
| T-K3-03 | exc_de | exit | Armed 0; `div` by zero; dump `vector=0 name=#DE`; `FAULT-OK vector=0` |
| T-K3-04 | exc_ud | exit | Armed 6; `ud2`; dump `vector=6 name=#UD` |
| T-K3-05 | exc_gp | exit | Armed 13; read through a non-stack register from `0x0000800000000000`; dump `vector=13`, `error=0x0000000000000000` |
| T-K3-06 | exc_pf_read | exit | Armed 14; read `0x00007ffffffff000`; `cr2` equal to it; `pf: present=0 write=0 user=0 rsvd=0 exec=0` |
| T-K3-07 | exc_pf_write_rodata **(stretch)** | exit | Write at `__rodata_start`: `vector=14`, `present=1 write=1`, `cr2` equal to `__rodata_start` |
| T-K3-08 | exc_pf_exec_data **(stretch)** | exit | Call into `.data`: `vector=14`, `present=1 exec=1`, `rip == cr2` |
| T-K3-09 | exc_df_ist **(stretch)** | exit | Invalid `rsp` then `int3`: complete dump with `vector=8`, `error=0` |
| T-K3-10 | exc_unexpected | exit, host 35 | `ud2` with nothing armed: dump, panic block `reason: unhandled exception 6 (#UD)`, `RESULT FAIL` |
| T-K3-11 | exc_regs | exit | Fifteen distinct constants loaded into the general registers (all but `rsp`), then an armed `ud2`: the dump shows exactly those values |
| T-K3-12 | exc_bp_regs | exit | Constants loaded, direction flag set, `int3`, then every register and `RFLAGS.DF` compared after resumption: all unchanged |
| T-K3-13 | kt_check_fail_then_fault | exit, host 35 | Failed check, then an armed fault that arrives: `FAULT-OK`, then `RESULT FAIL` |
| T-K3-14 | kt_fault_missing | exit, host 35 | Armed 6, test returns: `FAULT-MISSING vector=6`, `RESULT FAIL` |
| T-K3-15 | kt_fault_wrong | exit, host 35 | Armed 13, `ud2`: `FAULT-WRONG expected=13 got=6`, `RESULT FAIL` |
| T-K3-16 | kt_bp_expected | exit | Armed 3, `int3`: `FAULT-OK vector=3`, `RESULT PASS` |

| ID | Review |
|---|---|
| RV-K3-01 | Stub push/pop symmetry, stack alignment argument of spec §9.4 checked against the assembly, `cld` before the C call, error-code vector list |
| RV-K3-02 | IDT gate encoding, interrupt-stack indexes, TSS interrupt-stack pointers — mandatory because their runtime test is stretch |

### K4 (suite `k4`)

| ID | Name | Mode | Pass condition |
|---|---|---|---|
| T-K4-01 | memmap_log | observe | Entry count matches; bases ascending; `usable` entries page-aligned; total usable ≤ RAM and ≥ 75 % of RAM (sanity bound for the reference machine) |
| T-K4-02 | pmm_stats | exit | Invariant 1; `low_reserved ≤ 256`; values equal the boot log; `bytes == 8 * ceil(covers / 64)` and `frames == ceil(bytes / 4096)` |
| T-K4-03 | pmm_unique | exit | 1,000 allocations: distinct, aligned, ≥ 1 MiB, inside a logged usable entry, outside `[bitmap phys, phys + frames * 4096)`, below `covers * 4096` (checked on the host) |
| T-K4-04 | pmm_alloc_free | exit | Allocate 1,000, free all: `free_frames` restored; invariant holds at three checkpoints |
| T-K4-05 | pmm_contig | exit | At the reference configuration `pmm_alloc_contig(16)` returns 16 adjacent frames; `(0)`, `(highest_frame + 1)` and `(free_frames + 1)` return `PMM_INVALID`; counts restored after freeing |
| T-K4-06 | pmm_exhaust | exit | Allocations until `PMM_INVALID` equal the initial `free_frames`; no panic; count restored |
| T-K4-07 | pmm_rw | exit | Patterns in the first and last word of 256 frames read back |
| T-K4-08 | pmm_double_free | exit, host 35 | Allocate, free, free: panic reason begins `pmm:` |
| T-K4-09 | pmm_free_invalid **(stretch)** | exit, host 35 | Four kernels: unaligned; below 1 MiB; at `highest_frame * 4096`; at `bitmap_base` |
| T-K4-10 | pmm_reuse | exit | A freed frame is eventually returned again (exercises hint wrap-around) |
| T-K4-11 | pmm_free_contig_valid | exit | `pmm_free_contig(x, 0)` changes nothing, for a valid and an invalid `x`; a run obtained by one contiguous allocation is freed in one call; frames obtained by separate single allocations that happen to be adjacent are freed in one call; the highest managed frame can be allocated and freed; invariant holds throughout |
| T-K4-12 | pmm_free_contig_invalid **(stretch)** | exit, host 35 | Three kernels: range containing a free frame; range crossing `highest_frame`; `count` so large that `first + count * 4096` would overflow |

| ID | Review |
|---|---|
| RV-K4-01 | Validation code exists for every invalid case of spec §10.4, validates before mutating, and uses the non-overflowing comparisons — mandatory because T-K4-09 and T-K4-12 are stretch |
| RV-K4-02 | Bitmap arithmetic, clipped-range placement, padding bits, and scan bounds against spec §10.2–§10.5 (invariant 6) |

### K5 (suite `k5`)

| ID | Name | Pass condition |
|---|---|---|
| T-K5-01…04 | matrix_128M / 256M / 1G / 4G | Every required `exit`/`observe` test of `k1`–`k4` at each size; at 4 GiB at least one usable entry has base ≥ `0x100000000` |
| T-K5-05 | stability | Every required `exit`/`observe` test of `k1`–`k4` at 256 MiB, 20 repetitions from the same built images, zero failures |
| T-K5-06 | uefi_boot **(optional)** | `boot_banner` and `smoke` with OVMF; `SKIP` with reason if absent |

| ID | Manual evidence |
|---|---|
| E-K5-01 | Recorded GDB session (`agam-dev/reports/claude/gdb-session.md`) |
| E-K5-02 | Status of every V-item; V-07, V-08 and V-13 may be `UNVERIFIED` |
| E-K5-03 | Measured durations of the K1–K5 gate runs, recorded for future budgets |

### Rules for in-kernel test cases (`agam-os/tests/cases/*.c`)

- Include only the test-facing headers of `AOS_KERNEL_SPEC.md` §13.1.
- Define `aos_test_name` and `aos_test_main`; never call the exit device directly.
- Raise faults with inline assembly, never with undefined C behaviour, and never with `int n` for an error-code vector.
- One deliberate fault per case; each case is one boot.
- Print host-checked data on lines beginning `@@AOS-DATA@@ `.

---

## 6. AgamLang test catalogue

### 6.1 Layout and independent expectations

```
agam-lang/tests/
├── requirements-dev.txt                      pinned pytest (M0)
├── fixtures/keywords_expected.json           hand-written from AGAMLANG_SPEC §3.2 and §3.3
├── fixtures/codes_expected.json              hand-written from §8.6
├── fixtures/messages_expected.json           hand-written from Appendix A
├── test_*.py
├── ast_golden/<name>.agam + <name>.json
├── conformance/<area>/<name>.agam + .out [+ .in] [+ .err]
├── fusion/<name>/en.agam, hi.agam, mixed.agam, expected.out
├── builtin_alias/<name>/en.agam, hi.agam, expected.out
└── diagnostics_golden/<name>.agam + <name>.en.txt + <name>.hi.txt
```

- Every fixture and golden file is derived **by hand from the specification**. Tests assert that the implementation's tables equal the fixtures; they never read `keywords.py` or the message catalogues to learn what to expect.
- `.err` is one line: `<code> line <n>`; its presence means expected exit status 1.
- Hinglish fixtures follow the proposed wording and change only after an approved spec change (O-20).

### 6.2 Tests

| ID | Area | Pass condition |
|---|---|---|
| T-L1-01 | Alias table | The runtime table equals `keywords_expected.json` exactly: 52 words, 5 phrases, 6 reserved components, with kind and language. Every alias in lower, UPPER, Title and alternating case lexes to its kind with the right `lang`, `lexeme` and `parts`. |
| T-L1-02 | Phrases and composition | Each phrase merges; does not merge across a line break, comment or punctuation, including inside brackets. Every else-if form of spec §3.3, including `warna if`, `else agar`, `nahi to agar`, gives one `KW_ELIF` with the lexeme, span, parts and `lang` (`mixed` where applicable) of §3.4. A lone component gives `RESERVED_WORD` and no lexer error. |
| T-L1-03 | Indentation | `INDENT`/`DEDENT` sequences; E0103; E0104; the mixed tab/space example of §4.2 |
| T-L1-04 | Literals | Numbers and text per §4.4–§4.5, including `1e3`, leading zeros, the 1,000-digit boundary (accepted) and 1,001 digits (E0107), decimal overflow (E0107) |
| T-L1-05 | Lexical errors | E0100–E0107 with code, line, column |
| T-L1-06 | Fusion tokens | `tokens --canonical` identical across each keyword-substitution triple; `IDENT` lines carry the name |
| T-L1-07 | Lexer fuzz | 500 seeded inputs raise only `AgamError` |
| T-L2-01 | Golden ASTs | At least one per grammar production; default JSON contains no span or origin |
| T-L2-02 | Precedence | ≥ 20 expressions |
| T-L2-03 | Syntax errors | E0201–E0213 with code, line, column; E0208 for each case (a)–(c) of §5.2 and raised by the parser; E0210 for `stop` in a function nested in a loop; error precedence of §5.2 |
| T-L2-04 | Fusion ASTs | Default AST JSON identical across each keyword-substitution triple |
| T-L2-05 | Parser fuzz | 500 seeded mutations raise only `AgamError` |
| T-L3-01 | Operators | Every row of §7.4 for the types available (numbers, text, booleans, nothing); unsupported combinations give the stated E0302 variant |
| T-L3-02 | Display forms | Every available value type |
| T-L3-03 | Control flow | if / else-if / else, while, repeat, stop, skip |
| T-L3-04 | Runtime errors | E0302c, E0401, E0404 |
| T-L3-05 | CLI | Exit statuses; stdout/stderr separation; `--version`; `--messages` on every command; `--max-steps` validation |
| T-L3-06 | Numeric closure | E0406 (negative base, fractional exponent); E0401 for `0 ** -1`; E0407 for decimal overflow, whole-number overflow at the 1,000-digit boundary, and conversion of an oversized whole number; `10 ** 999` accepted and `10 ** 1000` rejected; whole/decimal comparison exact |
| T-L4-01 | Functions | Arguments, return, recursion, the closure example of §7.9, E0303, E0304, E0213 |
| T-L4-02 | Lists and text | Literals, indexing, stores, E0305, E0306a–c, E0302d, for loops, snapshot iteration |
| T-L4-03 | Built-ins | Each built-in under both names; each error of §7.11; `number()` grammar accept/reject table; `ask` with `.in` and E0405 |
| T-L4-04 | Limits | E0402 without a traceback; E0403; `range` and concatenation at exactly 1,000,000 (accepted) and one more (E0408); display over the limit (E0408) |
| T-L4-05 | Examples | Every file in `agam-lang/examples/` exits 0 |
| T-L4-06 | Built-in alias behaviour | Each `builtin_alias` pair gives identical output and status; `length == lambai` is `true` |
| T-L4-07 | No cycles | E0308 for `append(a, a)`, `a[0] = a`, an indirect cycle through a nested list, and an augmented indexed store; after each error the list's display form is unchanged; sharing without a cycle is accepted |
| T-L4-08 | Equality and display | §7.5 table including `[true] == [1]` false and function identity; text escaping inside lists |
| T-L4-09 | Evaluation order | Programs whose functions record their call order: binary operands, arguments, `a[f()] = g()` (g first), `a[f()] += g()` (f first, once), `show`, short-circuit, error precedence of §7.3 rule 10 |
| T-L5-01 | Catalogue completeness | The implementation's codes equal `codes_expected.json`; both catalogues equal `messages_expected.json`; every code E01xx–E04xx is raised by at least one program test |
| T-L5-02 | Rendered diagnostics | Golden text, English and Hinglish, covering header, tab expansion, caret position and width, zero-width spans, end of file, E0100 without excerpt, keyword quoted as typed (including a mixed phrase) |
| T-L5-03 | Suggestions | 10 misspellings suggested; 5 unrelated words not; tie-break and case-only cases of §8.4 |
| T-L5-04 | Message language | `auto` by part counts, including a mixed phrase, a lexical error part-way through a file, and E0100 |
| T-L5-05 | Whole-program fuzz | 2,000 seeded cases; exit status only 0, 1 or 2; no traceback text; never 70 |
| T-L5-06 | Fusion corpus | ≥ 15 keyword-substitution triples covering every keyword, phrase and else-if form: token, AST and output identity |
| T-L5-07 | Internal error | Calling `agamlang.cli.main` in-process with a raising `_fault`: E0900 rendering, exit status 70, no traceback. No command-line or environment route to this exists (checked by review RV-L5-01). |

| ID | Review |
|---|---|
| RV-L1-01 | The parser and interpreter never read keyword spelling, language, `origin` or `lang_counts` to make a decision |
| RV-L4-01 | Size and cycle checks happen before allocation or mutation (§7.1, §7.8) |
| RV-L5-01 | No public switch triggers E0900; no bare `except`; no `eval`/`exec` |

### 6.3 Fuzz bounds

Each fuzz input is at most 2,000 code points, runs with `max_steps = 10000`, and the whole fuzz test runs in a subprocess with a wall-clock budget (600 s initially; a budget, not a measurement). A failure prints the seed and the input.

---

## 7. Review procedure

For each milestone Codex reviews the diff of Claude's paths and files `agam-dev/reports/codex/reviews/<milestone>.md`, answering every `RV-` item of that milestone and checking:

| Check | Question |
|---|---|
| Spec conformance | Does every normative item have matching code? Is there behaviour the spec does not describe? |
| Hardware structures | Sizes and offsets asserted? Encodings match the spec tables? Reserved fields zero? |
| Assembly | Save/restore symmetric? Stack aligned at calls? Direction flag cleared? |
| Memory safety | Bounds on every array and bitmap access? Overflow-free size arithmetic? |
| Failure paths | Every "MUST panic" path present, validating before mutating? |
| Build | Flags equal the spec? No git, no timestamps, network only in `fetch-limine.sh`? |
| Python | No `eval`/`exec`; no bare `except`; every error path yields a `Diagnostic` |
| Scope | Nothing normative was decided by the implementation; nothing internal is demanded by a test |

Finding format: ID, severity (§2.4), `file:line` or document section, spec section, description, suggested fix. Claude answers each finding with `fixed (commit)`, `disputed (reason)` or `deferred (reason)`; disputes go to the architect.

If Claude believes a test contradicts the spec, Claude files a proposal and does not edit the test. The architect spot-checks at least three tests per milestone against the spec.

---

## 8. Integration gate (normative; O-17)

### 8.1 Roles and terms

- All `git` commands are run by the **owner**, with Git for Windows, in `D:\Agam-Computing\ecosystem`. Agents never run them. `sh`, `python3` and `make` run in WSL2.
- **`BASE`**: the full SHA of `main` when the milestone starts. It is recorded then and does not change during the milestone. If `main` has to change, the milestone restarts from the new `BASE`: both agents merge it and every earlier manifest and result is void.
- **`TIP_C`, `TIP_X`**: the exact full SHAs of `agent/claude` and `agent/codex` submitted for the gate. Each agent's worktree must be clean (`git status --porcelain=v1 --untracked-files=all` prints nothing) when it reports its tip.
- **`gate/<milestone>`**: a temporary branch created from `BASE`. `main` is never used for trial merges.
- **Checker**: `agam-dev/verify/check_ownership.py` **as it exists at `BASE`**, so a tip cannot weaken the check applied to itself. For the M0 gate no accepted checker exists; E-M0-07 replaces it and T-M0-05 exercises the new checker.

### 8.2 Steps

```
# 1. Preconditions (Git for Windows)
git rev-parse main                                   # must print BASE
git merge-base --is-ancestor <BASE> <TIP_C>          # exit 0
git merge-base --is-ancestor <BASE> <TIP_X>          # exit 0

# 2. Per-agent manifests: every change from BASE to the tip, with modes and blob IDs
git diff --raw -z --no-abbrev --no-renames --output=.gate/<M>/claude.raw <BASE> <TIP_C>
git diff --raw -z --no-abbrev --no-renames --output=.gate/<M>/codex.raw  <BASE> <TIP_X>

# 3. Ownership and form check of each manifest (checker from BASE; Git-free)
python agam-dev/verify/check_ownership.py --agent claude --manifest .gate/<M>/claude.raw
python agam-dev/verify/check_ownership.py --agent codex  --manifest .gate/<M>/codex.raw

# 4. Trial merge on the gate branch
git switch -c gate/<M> <BASE>
git merge --no-ff -m "gate <M>: merge agent/claude" <TIP_C>
git merge --no-ff -m "gate <M>: merge agent/codex"  <TIP_X>
git rev-parse HEAD                                   # GATE_HEAD

# 5. Verify the merge result against the two approved manifests
git diff --raw -z --no-abbrev --no-renames --output=.gate/<M>/merged.raw <BASE> <GATE_HEAD>
python agam-dev/verify/check_ownership.py --verify-merge .gate/<M>/claude.raw .gate/<M>/codex.raw .gate/<M>/merged.raw

# 6. Binding files for the runner: tree.lst (byte-exact ls-tree) and gate.json
git ls-tree -r -z --full-tree <GATE_HEAD>            # captured byte-exactly into .gate/<M>/tree.lst

# 7. WSL2, in /mnt/d/Agam-Computing/ecosystem
python3 agam-dev/verify/run.py gate <M>

# 8. After acceptance (owner only)
#    add the gate record under agam-dev/gates/<M>/ and the line in agam-dev/gates/ACCEPTED, commit on gate/<M>
git switch main
git merge --ff-only gate/<M>
git tag <tag>
git branch -d gate/<M>
```

`.gate/` is untracked and ignored. A helper that captures the manifests and writes `gate.json` byte-exactly (`agam-dev/verify/gate_manifest.ps1`, read-only git commands only) is an M0 deliverable; until it is accepted the owner runs the commands above by hand.

### 8.3 What the checker verifies (Git-free)

For each agent manifest:

- every record's path belongs to that agent's set in `ARCHITECTURE.md` §6.1 — for additions, modifications **and deletions**;
- because rename detection is off, a rename appears as a deletion plus an addition, and **both** paths must belong to the agent;
- status is `A`, `M` or `D` only (a type change fails);
- the new mode of every added or modified file is `100644`; symbolic links (`120000`) and submodules (`160000`) fail;
- the two agent manifests have no path in common.

For the merge (`--verify-merge`):

- the set of records in `merged.raw` equals the union of the two agent manifests **record for record**: same path, same status, same old and new mode, same old and new blob ID. Blob IDs identify contents, so this verifies contents, modes, additions and deletions, not merely path names;
- any extra, missing or different record fails. A merge conflict in step 4 also fails the gate.

### 8.4 Failure and stale results

- On any failure: `git merge --abort` if needed, `git switch main`, `git branch -D gate/<M>`. `main` is untouched. The owning agent fixes its branch, which produces a new tip, and the gate restarts from step 1 with new manifests.
- Manifests, binding files and runner results are valid only for the exact `BASE`, `TIP_C`, `TIP_X` and `GATE_HEAD` they name (§4.6). Nothing is carried over from an earlier attempt.

### 8.5 Acceptance

The milestone is accepted only when: steps 1–7 succeeded for one set of SHAs; the runner exited 0; all `E-` and `RV-` items of the milestone are filed; no Critical or Major finding is open; every stretch failure is classified (§4.5); and the owner says so. The record commit of step 8 may add files only under `agam-dev/gates/`. **Only the owner advances `main`**, by fast-forward, and only the owner tags and pushes.

Every commit, including the merge and record commits, uses the identity of O-18 and carries no attribution text.

---

## 9. Known gaps (accepted for now)

- No real-hardware testing; no KVM; TCG only.
- No multi-processor, timing or interrupt-latency tests.
- No kernel code-coverage measurement.
- No continuous integration service (Q-11); the gate is run by hand.
- UEFI boot is optional (Q-03).
- Double-fault delivery, write-protection and no-execute enforcement, and invalid-argument panics of the allocator are covered only by stretch tests (O-09, O-19) plus mandatory review; without those runs the behaviour ships as `UNVERIFIED`.
- The release signature of the Limine asset is not verified (V-13).
- AgamLang performance is not tested, and its limits are provisional policy, not measured guarantees (O-14).
