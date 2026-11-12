# Agam Computing Ecosystem — Test Plan

| Field | Value |
|---|---|
| Status | **Draft v0.1, revision 2 — NOT APPROVED.** Owner's provisional decisions of 2026-11-12 incorporated. |
| Date | 2026-11-12 (revision 2) |
| Codex owns | `agam-os/tests/**`, `agam-lang/tests/**`, `agam-dev/verify/**`, `agam-dev/reports/codex/**` |
| Paths | Relative to the repository root |
| Related | `AOS_KERNEL_SPEC.md`, `AGAMLANG_SPEC.md`, `ROADMAP.md` |

No test described here exists yet, and none has been run. The only things executed so far are the architect's scratch-sandbox checks recorded in `DECISIONS.md` §8 and `AOS_KERNEL_SPEC.md` §12.5 and §15. Timeouts and counts in this document are budgets chosen by the architect, not measurements.

---

## 1. Principles

1. **Tests are written from the specification, not from the implementation.** Codex should be able to write every test in this plan before the corresponding code exists.
2. **A test that cannot fail is not a test.** For each new test, Codex records one deliberate failure (for example, a wrong expected value) to show the test detects it.
3. **Only explicit success counts.** For kernel tests, **host** exit status 33 plus the expected serial lines is success. Host status 0, a timeout, or missing output is a failure. The guest exit value (`0x10`) and the host exit status (33) are different numbers; see `AOS_KERNEL_SPEC.md` §12.5.
4. **Everything is re-run on the integration branch.** Results from an agent's own worktree are useful, but only the gate run on `main` decides acceptance.
5. **Skips are visible.** A test that cannot run (missing optional tool) reports `SKIP` with a reason; it never reports `PASS`.
6. **Required before stretch.** Tests marked *stretch* (O-09) are attempted only after every required test of the milestone passes. They are reported honestly (`TEST-VERIFIED`, `FAILED` or `SKIPPED`) but do not block acceptance. A stretch failure that reveals a violation of a normative requirement is filed as a finding for the architect and owner to triage.

---

## 2. Evidence protocol (P-15)

### 2.1 Status vocabulary

Every claim about code in a report, commit message or document uses exactly one of:

| Status | Meaning | Minimum evidence |
|---|---|---|
| `UNVERIFIED` | Written but not built or run | none |
| `BUILD-VERIFIED` | Compiles/links (or imports) without warnings | command, exit status, tool versions, commit |
| `TEST-VERIFIED` | The named tests passed | command, exit status, result lines, tool versions, commit, artefact path |
| `FAILED` | Ran and failed | same as above, plus the failing output |
| `SKIPPED` | Could not run | reason |

Words such as "works", "passes", "tested", "verified", "confirmed" MUST NOT be used without one of these statuses and its evidence. Performance numbers MUST NOT be stated at all unless a measurement command and its raw output are included.

### 2.2 Evidence block (copy into reports)

````
### Evidence: <what is being claimed>
- Status: TEST-VERIFIED
- Commit: <full hash> (branch <name>, working tree clean: yes/no)
- Environment: <output of `sh agam-dev/verify/check_env.sh --brief`>
- Command: `<exact command>`
- Exit status: <n>
- Date/time (UTC): <timestamp>
- Artefacts: agam-os/build/test-results/<run id>/
- Output excerpt:
```
<relevant lines, unedited; use "[...]" for omissions>
```
````

### 2.3 Where reports live

| File | Writer | Content |
|---|---|---|
| `agam-dev/reports/claude/<milestone>.md` | Claude | What was implemented, per-file status, evidence blocks, open issues, resolution of V-items |
| `agam-dev/reports/claude/<milestone>-review-response.md` | Claude | Response to each review finding |
| `agam-dev/reports/claude/proposals/<nnn>-<slug>.md` | Claude | Requested spec changes, or objections to a test |
| `agam-dev/reports/codex/<milestone>-evidence.md` | Codex | Evidence blocks for every test ID of the milestone, run on the integration branch |
| `agam-dev/reports/codex/reviews/<milestone>.md` | Codex | Review findings (§7) |
| `agam-dev/reports/codex/proposals/<nnn>-<slug>.md` | Codex | Requested spec changes |

---

## 3. Test levels

| Level | What | Where it runs | Tooling |
|---|---|---|---|
| Static | Build cleanliness, ELF structure, forbidden instructions, reproducibility | Host | `make`, `llvm-readelf`, `llvm-objdump`, `llvm-nm`, `sha256sum` |
| Boot | Normal kernel boots and logs correctly | QEMU | Harness |
| In-kernel | Test case compiled into a test kernel; result via exit device | QEMU | Harness + `agam-os/tests/cases/*.c` |
| Matrix | In-kernel suite across memory sizes and repeated runs | QEMU | Harness |
| Unit / golden | AgamLang lexer, parser, diagnostics | Host Python | `pytest` |
| Conformance | `.agam` programs with expected output or error | Host Python | `pytest` |
| Fusion equivalence | Same program in English, Hinglish, mixed | Host Python | `pytest` |
| Fuzz | Random and mutated input never crashes the interpreter | Host Python | `pytest`, seeded |
| Review | Human-readable inspection against the spec | — | §7 |

---

## 4. Harness contract (`agam-dev/verify/run.py`)

### 4.1 Command line

```
python3 agam-dev/verify/run.py aos [--suite k1|k2|k3|k4|k5]... [--all] [--mem SIZE] [--matrix] [--repeat N] [--only TEST_ID]
python3 agam-dev/verify/run.py agamlang [--suite l1|l2|l3|l4|l5]... [--all]
python3 agam-dev/verify/run.py all
```

- Prints one line per test: `PASS|FAIL|SKIP <test id> <name> (<seconds>s)[: <reason>]`, then a summary line `SUMMARY passed=<n> failed=<n> skipped=<n>`.
- Exit status 0 only if `failed == 0`; 1 if any test failed; 2 for a usage or environment error.
- Writes artefacts to `agam-os/build/test-results/<UTC timestamp>/<test id>/` and a machine-readable `summary.json` in the run directory.
- Depends only on the Python standard library for the `aos` suites. The `agamlang` suites invoke `pytest`.

### 4.2 Running a kernel test (`agam-os/tests/harness/qemu.py`)

1. Build: `make -C agam-os iso TEST=<case>` (or `make -C agam-os iso` for the normal kernel). A build failure is a test failure.
2. Launch QEMU with the reference command from `AOS_KERNEL_SPEC.md` §11.5, with serial redirected to `serial.log` and `-d int,cpu_reset -D qemu.log`. Memory size comes from `--mem` (default `256M`).
3. Enforce a timeout (default 20 s; 60 s for `pmm_exhaust`; 120 s for 4 GiB runs). On timeout, kill QEMU and fail.
4. Save `cmd.txt` (exact command), `serial.log`, `qemu.log`, `exit_status`.
5. Normalise `\r\n` to `\n` before matching.
6. Verdict: pass only if the **host** exit status equals the expected one (`HOST_STATUS_PASS = 33` by default; `HOST_STATUS_FAIL = 35` for tests that expect a panic) **and** every required pattern matches in order. The harness never compares a host status with a guest exit value.

For the normal (non-test) kernel there is no exit device call: the harness waits for the line `[aos] halt`, then terminates QEMU and checks the log.

### 4.3 Parsers (`agam-os/tests/harness/aosdump.py`)

Parse the boot log, exception dump and memory-map lines defined in `AOS_KERNEL_SPEC.md` §12 into Python dictionaries. A line that does not match the specified format is itself a test failure (the format is normative).

---

## 5. AOS test catalogue

Expected **host** exit status is 33 unless stated. "Dump" means the block of spec §12.3. Tests are *required* unless their name is marked **(stretch)**.

### K1

| ID | Name | Method | Pass condition |
|---|---|---|---|
| T-K1-01 | build_clean | `make -C agam-os distclean && make -C agam-os iso` | Exit 0; output contains no `warning:` |
| T-K1-02 | elf_header | `llvm-readelf -h` | Class ELF64; machine X86-64; type EXEC; entry ≥ `0xffffffff80000000` |
| T-K1-03 | elf_segments | `llvm-readelf -lW` | Exactly three LOAD segments with flags `R E`, `R`, `RW`; each virtual address 4 KiB-aligned; none both W and E |
| T-K1-04 | no_simd | `llvm-objdump -d` | No operand matching `%xmm`, `%ymm`, `%zmm`, `%st` |
| T-K1-05 | boot_banner | Normal ISO | Banner, `limine:`, `hhdm:`, `kernel:`, `init complete`, `halt` lines appear in order and match spec §12.1; `base revision 6`; HHDM offset ≥ `0xffff800000000000`; kernel virt ≥ `0xffffffff80000000` |
| T-K1-06 | smoke | Case `smoke` (empty test body) | `BEGIN smoke`, `RESULT PASS smoke` |
| T-K1-07 | panic_exit | Case calls `panic("test panic %d", 42)` | Status **35**; panic block with `reason: test panic 42` |
| T-K1-08 | smoke_stability | T-K1-06 ten times | Ten passes |
| T-K1-09 | exit_convention | Harness self-test on the owner's QEMU: (a) `smoke` gives host status 33; (b) `panic_exit` gives 35; (c) the normal kernel, which never calls the exit device, is reported as a timeout when the wait for `[aos] halt` is disabled; (d) a recorded host status of 0 is classified as failure | All four hold; QEMU version recorded (resolves V-05) |

### K2

| ID | Name | Method | Pass condition |
|---|---|---|---|
| T-K2-01 | gdt_log | Normal ISO | Line `[aos] gdt: loaded cs=0x0008 tr=0x0028` |
| T-K2-02 | gdt_selectors | Case reads `CS`, `TR` via `gdt_read_cs/tr`, and `DS`, `ES`, `SS` via inline assembly | `0x08`, `0x28`, `0x10`, `0x10`, `0x10` |
| T-K2-03 | kprintf_formats | Case prints a fixed list of format/argument pairs between marker lines | Output between markers equals the expected text byte for byte (covers every conversion, `l`/`ll`/`z`, zero padding, width, `%p`, NULL `%s`, `%%`, INT64 minimum, UINT64 maximum) |
| T-K2-04 | string_funcs | Case checks `memset`, `memcpy`, `memcmp`, `memmove` with overlapping ranges in both directions, `strlen` | All `KTEST_CHECK`s hold |
| T-K2-05 | reproducible_elf | Two clean builds | Equal SHA-256 of `agam-os/build/kernel/aos.elf` |
| T-K2-06 | symbol_order | `llvm-nm` | `__kernel_start ≤ __text_start < __text_end ≤ __rodata_start < __rodata_end ≤ __data_start < __data_end ≤ __bss_start ≤ __bss_end ≤ __kernel_end`; each `*_start` 4 KiB-aligned |

### K3

Faults are triggered with inline assembly so the compiler cannot optimise them away.

| ID | Name | Trigger | Pass condition |
|---|---|---|---|
| T-K3-01 | idt_log | Normal ISO | Line `[aos] idt: loaded 256 gates` |
| T-K3-02 | exc_bp_resume | `int3`, then continue | Breakpoint line with a `rip` inside the kernel text range; `RESULT PASS` |
| T-K3-03 | exc_de | `div` by a zero register | Dump `vector=0 name=#DE`; `FAULT-OK vector=0` |
| T-K3-04 | exc_ud | `ud2` | Dump `vector=6 name=#UD` |
| T-K3-05 | exc_gp | Read from non-canonical address `0x0000800000000000` | Dump `vector=13 name=#GP`, `error=0x0000000000000000` |
| T-K3-06 | exc_pf_read | Read from `0x00007ffffffff000` | Dump `vector=14`; `cr2=0x00007ffffffff000`; `pf: present=0 write=0 user=0 rsvd=0 exec=0` |
| T-K3-07 | exc_pf_write_rodata **(stretch)** | Write one byte at `__rodata_start` | Dump `vector=14`; `cr2` equals `__rodata_start` (from `llvm-nm`); `present=1 write=1` |
| T-K3-08 | exc_pf_exec_data **(stretch)** | Call a function pointer to a `ret` byte stored in `.data` | Dump `vector=14`; `present=1 exec=1`; `rip == cr2`; address inside `[__data_start, __data_end)` |
| T-K3-09 | exc_df_ist **(stretch)** | Load `rsp` with `0x00007ffffffff000`, then `int3` | Complete dump with `vector=8 name=#DF` and `error=0x0000000000000000`. Reaching a complete dump demonstrates the switch to IST1, since the interrupted stack is unusable. The dump's `rsp` and `rip` are recorded but not asserted (saved state after a double fault is not reliable). |
| T-K3-10 | exc_unexpected | `ud2` with no armed expectation | Status **35**; dump followed by panic block `reason: unhandled exception 6 (#UD)` |

Every dump in K3 is additionally checked for: all lines of the format present, `cs=0x0008`, `ss=0x0010`, `rip` inside kernel text (except T-K3-08 and T-K3-09).

### K4

Test cases receive memory facts from `pmm_get_stats()`; the host side cross-checks against the memory-map log.

| ID | Name | Method | Pass condition |
|---|---|---|---|
| T-K4-01 | memmap_log | Normal ISO | Entry count line matches number of entry lines; bases ascending; `usable` entries 4 KiB-aligned; sum of usable ≤ RAM and ≥ 75 % of RAM |
| T-K4-02 | pmm_stats | Case prints stats | Invariant 1 of spec §10.5; `low_reserved ≤ 256`; `bitmap_frames ≥ 1`; values equal those in the boot log |
| T-K4-03 | pmm_unique | 1 000 allocations, addresses printed | All distinct, aligned, ≥ `0x100000`, inside a logged usable entry, outside the logged bitmap range (checked on the host) |
| T-K4-04 | pmm_alloc_free | Allocate 1 000, free all | `free_frames` returns to the initial value; invariant holds at three checkpoints |
| T-K4-05 | pmm_contig | `pmm_alloc_contig(16)`, `(0)`, `(free_frames + 1)` | 16 adjacent frames; `PMM_INVALID`; `PMM_INVALID`; freeing restores the count |
| T-K4-06 | pmm_exhaust | Allocate until `PMM_INVALID`, then free all (frames linked through their own first word via the HHDM) | Allocation count equals initial `free_frames`; no panic; count restored |
| T-K4-07 | pmm_rw | 256 frames, pattern in first and last word | All patterns read back |
| T-K4-08 | pmm_double_free | Allocate, free, free | Status **35**; panic reason begins `pmm:` |
| T-K4-09 | pmm_free_invalid **(stretch)** | Four cases, each its own test kernel: `pmm_free_unaligned`, `pmm_free_low` (below 1 MiB), `pmm_free_beyond` (at `highest_frame * 4096`), `pmm_free_bitmap` (at `bitmap_base`) | Each: status **35**; panic reason begins `pmm:` |
| T-K4-10 | pmm_reuse | Allocate A, free A, allocate until A is returned again (bounded by `free_frames`) | A is eventually returned (freed frames are reusable) |

### K5

| ID | Name | Method | Pass condition |
|---|---|---|---|
| T-K5-01…04 | matrix_128M / 256M / 1G / 4G | All K1–K4 tests with `--mem` | All pass; at 4 GiB at least one usable entry has base ≥ `0x100000000` |
| T-K5-05 | stability | Full suite at 256 MiB, 20 repetitions | Zero failures |
| T-K5-06 | uefi_boot (optional) | `boot_banner` and `smoke` with OVMF firmware | Pass, or `SKIP` with reason if OVMF is not installed |
| T-K5-07 | ownership | `check_ownership.py` for both agents over the sprint | Exit 0 |

### Rules for in-kernel test cases (`agam-os/tests/cases/*.c`)

- Include only headers from `agam-os/kernel/include/aos/`. Do not include files from `agam-os/kernel/src/` or `agam-os/kernel/arch/`.
- Define `const char aos_test_name[]` and `void aos_test_main(void)`.
- Use `KTEST_CHECK` for assertions and `ktest_expect_fault` before a deliberate fault.
- Use inline assembly (not undefined C behaviour) to trigger faults.
- One deliberate fault per case; each case is one QEMU boot.
- Print host-checked data on lines beginning `@@AOS-DATA@@ ` so the parser can find them.

---

## 6. AgamLang test catalogue

### 6.1 Layout

```
agam-lang/tests/
├── test_keywords.py  test_lexer.py  test_parser.py
├── test_conformance.py  test_fusion.py  test_diagnostics.py
├── test_catalogue.py  test_fuzz.py  test_cli.py
├── ast_golden/<name>.agam + <name>.json
├── conformance/<area>/<name>.agam + .out [+ .in] [+ .err]
├── fusion/<name>/en.agam, hi.agam, mixed.agam, expected.out
└── diagnostics_golden/<name>.agam + <name>.en.txt + <name>.hi.txt
```

- `.out`: exact expected standard output. `.in`: standard input. `.err`: one line `<code> line <n>`; its presence means expected exit status 1 (otherwise 0).
- Golden files are created and changed only by Codex, and only from the spec.

### 6.2 Tests

| ID | Area | Pass condition (summary; details in `ROADMAP.md` L1–L5 criteria) |
|---|---|---|
| T-L1-01 | Alias table | Every alias × {lower, UPPER, Title} lexes to its canonical kind with correct `lang` and lexeme |
| T-L1-02 | Phrases | Each two-word alias merges; split across lines or separated by a comment does not merge; reserved components alone raise E0208 at parse |
| T-L1-03 | Indentation | Token sequences for nesting, blank/comment lines, bracket continuation, end-of-file dedents; E0103; E0104 |
| T-L1-04 | Literals | Numbers and text per spec §4.4–§4.5, including each error |
| T-L1-05 | Lexical errors | E0100–E0106 each with code, line, column |
| T-L1-06 | Fusion tokens | Canonical token output identical across each triple |
| T-L1-07 | Lexer fuzz | 500 seeded inputs raise only `AgamError` |
| T-L2-01 | Golden ASTs | One or more per grammar production |
| T-L2-02 | Precedence | ≥ 20 expressions, expected tree shape |
| T-L2-03 | Syntax errors | E0201–E0212 each with code, line, column |
| T-L2-04 | Fusion ASTs | AST JSON identical across each triple |
| T-L2-05 | Parser fuzz | 500 seeded mutations raise only `AgamError` |
| T-L3-01 | Operators | Every row of spec §7.3; unsupported combinations raise E0302 |
| T-L3-02 | Display forms | Every value type |
| T-L3-03 | Control flow | Conformance programs for if/elif/else, while, repeat, stop, skip |
| T-L3-04 | Runtime errors | E0302 (condition), E0401, E0404 |
| T-L3-05 | CLI | Exit statuses; stdout/stderr separation; `--version` |
| T-L4-01 | Functions | Arguments, return, recursion, closures, E0303, E0304 |
| T-L4-02 | Lists and text | Literals, indexing, assignment, E0305, E0306, for loops |
| T-L4-03 | Built-ins | Each built-in under both names, each documented error, `ask` with `.in` and E0405 |
| T-L4-04 | Limits | E0402 without a Python traceback; E0403 with `--max-steps` |
| T-L4-05 | Examples | Every file in `agam-lang/examples/` exits 0 |
| T-L5-01 | Catalogue completeness | Every code in both catalogues; every code triggered by some test |
| T-L5-02 | Rendered diagnostics | Golden text in English and Hinglish |
| T-L5-03 | Suggestions | 10 misspellings suggested; 5 unrelated words not suggested |
| T-L5-04 | Message language | `auto`, `en`, `hi` selection |
| T-L5-05 | Whole-program fuzz | 2 000 seeded cases; exit status only 0, 1 or 2; no traceback text on stderr |
| T-L5-06 | Fusion corpus | ≥ 15 triples covering every keyword; token, AST and output equivalence |

Fuzz tests use a fixed seed recorded in the test file; a failure message prints the seed and the offending input so it can be added as a regression case.

---

## 7. Review procedure (Codex reviews Claude's code)

For each milestone Codex reviews the diff of Claude's paths and files `agam-dev/reports/codex/reviews/<milestone>.md`:

| Check | Question |
|---|---|
| Spec conformance | Does each normative item of the cited spec sections have matching code? Any behaviour not in the spec? |
| Hardware structures | Are sizes and field offsets asserted? Do encodings match the spec tables? |
| Assembly | Register save/restore symmetric? Stack aligned at calls? Direction flag cleared? |
| Memory safety | Bounds on every array and bitmap access? Integer overflow in size calculations? |
| Failure paths | Does every "MUST panic" path exist? Are missing Limine responses handled? |
| Build | Flags equal the spec? No git, no timestamps, no network outside `fetch-limine.sh`? |
| Python (AgamLang) | No `eval`/`exec`; no bare `except`; every error path yields a `Diagnostic` |

Finding format: `ID`, severity, `file:line`, spec section, description, suggested fix.

| Severity | Meaning | Gate effect |
|---|---|---|
| High | Violates the spec, corrupts state, or makes a test unreliable | Blocks acceptance |
| Medium | Likely bug or fragile construct | Blocks acceptance |
| Low | Style, clarity, minor robustness | Does not block |

Claude answers each finding in `agam-dev/reports/claude/<milestone>-review-response.md` with `fixed (commit)`, `disputed (reason)` or `deferred (reason)`. Disputes go to the architect.

**Reviewing the tests.** If Claude believes a test contradicts the spec, Claude does not edit it; Claude files `agam-dev/reports/claude/proposals/<nnn>-<slug>.md`. The architect decides whether the spec, the test or the code changes. The architect also spot-checks at least three tests per milestone against the spec.

---

## 8. Merge gate (run in the `ecosystem` worktree for every milestone)

```
git merge --no-ff agent/claude
git merge --no-ff agent/codex
sh agam-dev/verify/check_env.sh
python3 agam-dev/verify/check_ownership.py --agent claude --base <previous milestone tag>
python3 agam-dev/verify/check_ownership.py --agent codex  --base <previous milestone tag>
make -C agam-os distclean
make -C agam-os test
```

The milestone is accepted only if every command exits 0, the reports of §2.3 exist, no High or Medium finding is open, and the owner agrees. The integration branch is then tagged.

Rules for the gate:

- **The owner runs or explicitly approves every command that changes `main`** (the two merges and the tag). Agents never merge into, commit to, or push `main`.
- `git` commands run with Git for Windows in `D:\Agam-Computing\ecosystem`. `sh`, `python3` and `make` commands run in WSL2 from `/mnt/d/Agam-Computing/ecosystem`. Git is never run from WSL (risk R-02).
- Before any commit or merge: `git worktree list` and `git branch --show-current` are run and their output is checked against the expected worktree and branch.
- Every commit is authored and committed as the owner's GitHub identity only, with no AI attribution of any kind (O-10; `CLAUDE.md` §4, `AGENTS.md` §4).

---

## 9. Known gaps (accepted for now)

- No real-hardware testing; no KVM; TCG only.
- No multi-processor, timing or interrupt-latency tests (nothing in Sprint 1 uses them).
- No kernel code coverage measurement.
- No continuous integration service until Q-11 is answered; the gate is run by hand.
- UEFI boot is optional (Q-03).
- Double-fault delivery, write-protection and no-execute enforcement, and invalid-free detection are covered only by stretch tests (O-09); if those are not run, the corresponding behaviour ships as `UNVERIFIED`.
- AgamLang performance is not tested and no performance claims are made.
