# Agam Computing Ecosystem — Roadmap

| Field | Value |
|---|---|
| Status | **Draft v0.1, revision 3 — NOT APPROVED.** Addresses review findings AR-01, AR-03, AR-04, AR-15, AR-19, AR-20 (`DECISIONS.md` §10); all are *addressed in draft — pending independent re-review*. No gate is open. |
| Date | 2026-11-12 (revision 3) |
| Paths | Relative to the repository root |
| Branches | `main`, `agent/claude`, `agent/codex`, `review/architecture-v0.1` (read from the repository's own files on 2026-11-12), plus temporary `gate/<milestone>` branches (O-17) |
| Test IDs | Defined once, in `TEST_PLAN.md` §5–§6. This document names them; it does not restate their pass conditions. |

Nothing below has been run. `git` commands are run by the owner with Git for Windows; `sh`, `make` and `python3` run in WSL2 Ubuntu 24.04 from `/mnt/d/Agam-Computing/<worktree>`.

---

## 1. Timeline

| When | Milestone | Outcome | Gate that must be open |
|---|---|---|---|
| Day 0 | **M0** Bootstrap | Environment evidence, pins, runner and checker with self-tests | G-BOOT |
| Day 1 | **K1** Boot to kernel entry | Serial banner, automated pass/fail exit | G-AOS |
| Day 2 | **K2** GDT, TSS, build quality | Own descriptor tables; reproducible build | G-AOS |
| Day 3 | **K3** IDT and CPU exceptions | Register dump on every exception; test finalisation proven | G-AOS |
| Day 4 | **K4** Physical memory manager | Frame allocator from the memory map | G-AOS |
| Day 5 | **K5** Matrix and closure | Tag `aos-v0.1.0-sprint1` | G-AOS |
| Days 6–10 (proposed) | **L1–L5** AgamLang v0.1 | Tag `agamlang-v0.1.0` | G-LANG |
| Later | K6+, L6+, D1+ | §6 | New specifications |

**Five days is a planning target, not a commitment** (AR-20). Nothing has been built or timed on the owner's machine. After K1 is accepted the owner and architect compare the measured build, boot and gate durations (E-K5-03 starts collecting them at K1) with the plan and either confirm or re-plan K2–K5. Required verification is never reduced to keep a date. The AgamLang track shares no files with AOS and may run in parallel if the owner prefers.

Task IDs: `<milestone>-C<n>` Claude Code, `<milestone>-X<n>` Codex, `<milestone>-A<n>` architect, `<milestone>-O<n>` owner.

---

## 2. M0 — Bootstrap (Day 0)

### 2.1 What the bootstrap gate permits

G-BOOT is opened by the owner's explicit statement. **It needs no evidence produced by M0 itself**, which removes the earlier circular dependency (AR-01). It permits exactly the work listed in §2.2 and nothing else. In particular it does **not** permit any file under `agam-os/kernel/`, `agam-os/boot/`, `agam-os/tests/`, `agam-lang/src/`, `agam-lang/examples/`, or any file under `agam-lang/tests/` other than `requirements-dev.txt`.

Completing M0 is a prerequisite of G-AOS and G-LANG; it opens neither.

| | |
|---|---|
| **Prerequisites** | Revision 3 re-reviewed with no Critical or Major finding open against the process sections (blocker B-07); the owner has put the approved documents on `main` (B-04); the owner opens G-BOOT. |
| **Purpose** | Replace assumptions about the machine with recorded evidence, and create the verification tools that later gates rely on. |

### 2.2 Deliverables (the complete permitted scope)

| Task | Who | Paths and content |
|---|---|---|
| M0-O1 | Owner | WSL2 packages: `sudo apt install build-essential clang lld llvm make xorriso qemu-system-x86 gdb curl xz-utils python3 python3-venv python3-pip ovmf`. `build-essential` provides the `cc` that the Limine host utility needs; `curl` (or `wget`) and `xz-utils` are needed by the fetch script. |
| M0-O2 | Owner | Python environment for later language tests, outside the repository: `python3 -m venv ~/.venvs/agam`, then `~/.venvs/agam/bin/pip install -r agam-lang/tests/requirements-dev.txt` |
| M0-O3 | Owner | Confirm the Git identity of O-18 with Git for Windows; switch off automatic attribution in each AI tool. No configuration is changed by an agent. |
| M0-A1 | Architect, committed by the owner **before any agent work** | `.gitattributes` (`* text=auto eol=lf`), `.gitignore` (`agam-os/build/`, `.gate/`, `__pycache__/`, `.pytest_cache/`), `VERSION`, `README.md`. Each worktree's files are then re-written from the index so they have LF endings (E-M0-03 checks the result). |
| M0-X1 | Codex | `agam-dev/verify/check_env.sh`: prints name, path and version of every tool of `AOS_KERNEL_SPEC.md` §2; accepts `curl` or `wget`; exits non-zero if a required tool is missing; `--brief` prints one line; **never runs git**. The Git for Windows version is reported separately by the owner (E-M0-05). |
| M0-X2 | Codex | `agam-dev/verify/check_ownership.py` (Git-free manifest checker, `TEST_PLAN.md` §8.3) and `agam-dev/verify/gate_manifest.ps1` (captures the manifests and binding files byte-exactly using read-only git commands) |
| M0-X3 | Codex | **Runner scaffolding**: `agam-dev/verify/run.py` with catalogue loading, selection by gate, the result accounting and exit rules of `TEST_PLAN.md` §4.4, the binding check of §4.6, the `m0` suite, and its self-tests; `agam-dev/verify/catalogue.json` with the `m0` entries. No QEMU harness and no kernel or language test. |
| M0-X4 | Codex | `agam-lang/tests/requirements-dev.txt` with one exact `pytest` version; `agam-dev/verify/README.md` |
| M0-X5 | Codex | `agam-dev/reports/codex/M0-evidence.md` |
| M0-C1 | Claude | `agam-os/third_party/limine/PIN`, `agam-os/third_party/limine-protocol/PIN` and the vendored `limine.h`, with the values of `DECISIONS.md` §8.1; `agam-os/tools/fetch-limine.sh` (the only script that uses the network; builds the host utility with `cc`) |
| M0-C2 | Claude | `agam-os/Makefile` skeleton with `check-env`, `print-flags`, `fetch`, `clean`, `distclean` only — no `test` target yet |
| M0-C3 | Claude | `agam-dev/reports/claude/M0.md` |

### 2.3 Verification

The gate procedure of `TEST_PLAN.md` §8 with `<M>` = `M0`; in WSL2 `python3 agam-dev/verify/run.py gate M0`. Because no accepted checker exists yet, the owner compares each agent's changed paths with §2.2 by hand (E-M0-07).

### 2.4 Acceptance criteria

1. `run.py gate M0` exits 0: required tests T-M0-01…06 all pass, none skipped or missing.
2. Manual evidence E-M0-01…07 is filed.
3. The tool versions (E-M0-04) match `DECISIONS.md` §8.2, or each difference is accepted by the owner. This clears B-01.
4. The owner has recorded a disposition for the Limine pin evidence (E-M0-06; B-02).
5. No file outside §2.2 was added or changed.

**Risks**: R-01, R-02, R-03, R-04, R-21, R-22, R-26.

---

## 3. Sprint 1 — AOS (target: Days 1–5)

Verification for every K-milestone is the gate procedure of `TEST_PLAN.md` §8, with `python3 agam-dev/verify/run.py gate <milestone>` in step 7. The suites that run are fixed by `TEST_PLAN.md` §4.2.

### K1 — Boot to kernel entry (Day 1)

| | |
|---|---|
| **Prerequisites** | M0 accepted; G-AOS open (needs B-02 disposition, B-05 and B-08 cleared, and the owner's explicit approval of the AOS documents). |
| **Spec sections** | `AOS_KERNEL_SPEC.md` §2–§7, §11, §12.1, §12.2, §12.4, §12.5 |

| Task | Who | Paths |
|---|---|---|
| K1-C1 | Claude | `agam-os/Makefile` (all targets of spec §11.1), `agam-os/tools/mkiso.sh`, `agam-os/boot/limine.conf` |
| K1-C2 | Claude | `agam-os/kernel/linker.ld`; Limine requests; entry and boot steps 1–4, 8, 9; serial, kprintf, panic, memory functions; test hooks without fault expectations. File names under `agam-os/kernel/` are the implementer's choice except the headers of spec §13.1. |
| K1-C3 | Claude | `agam-dev/reports/claude/K1.md` |
| K1-X1 | Codex | `agam-os/tests/harness/qemu.py`; `k1` catalogue entries and runner registration |
| K1-X2 | Codex | `agam-os/tests/cases/smoke.c`, `panic_exit.c`, `kt_check_fail_return.c`; `agam-os/tests/test_k1.py` |
| K1-X3 | Codex | `agam-dev/reports/codex/K1-evidence.md`, `agam-dev/reports/codex/reviews/K1.md` |

**Acceptance criteria**

1. `run.py gate K1` exits 0: T-K1-01…11 and the `m0` suite pass, none skipped or missing.
2. E-K1-01 and RV-K1-01 are filed; no Critical or Major finding is open.
3. Measured build, boot and gate durations are recorded, and the owner confirms or re-plans K2–K5.

**Risks**: R-04, R-05, R-07. K1 carries the most unknowns (first full link, fetch utility, hybrid ISO and boot together); K2 is deliberately light so K1 can overrun.

### K2 — GDT, TSS and build quality (Day 2)

| | |
|---|---|
| **Prerequisites** | K1 accepted. |
| **Spec sections** | §8, §4.3, §14 |

| Task | Who | Paths |
|---|---|---|
| K2-C1 | Claude | GDT, TSS and segment reload (boot step 5) under `agam-os/kernel/`; `aos/gdt.h` |
| K2-C2 | Claude | K1 review fixes; `agam-dev/reports/claude/K2.md` |
| K2-X1 | Codex | `agam-os/tests/cases/gdt_selectors.c`, `kprintf_formats.c`, `string_funcs.c`; `agam-os/tests/test_k2.py`; `k2` catalogue entries |
| K2-X2 | Codex | Evidence and review reports for K2 |

**Acceptance criteria**

1. `run.py gate K2` exits 0: T-K2-01…05 plus the K1 and M0 suites.
2. RV-K2-01 is filed; no Critical or Major finding is open.

**Risks**: R-06.

### K3 — IDT and CPU exception handling (Day 3)

| | |
|---|---|
| **Prerequisites** | K2 accepted. |
| **Spec sections** | §9, §12.3, §12.4 |

| Task | Who | Paths |
|---|---|---|
| K3-C1 | Claude | Interrupt stubs, IDT, exception dump and dispatch (boot step 6) under `agam-os/kernel/` |
| K3-C2 | Claude | Fault expectations and the single finalisation path of spec §12.4; `agam-dev/reports/claude/K3.md` |
| K3-X1 | Codex | Required cases in `agam-os/tests/cases/`: `exc_bp_resume.c`, `exc_de.c`, `exc_ud.c`, `exc_gp.c`, `exc_pf_read.c`, `exc_unexpected.c`, `exc_regs.c`, `exc_bp_regs.c`, `kt_check_fail_then_fault.c`, `kt_fault_missing.c`, `kt_fault_wrong.c`, `kt_bp_expected.c` |
| K3-X2 | Codex | `agam-os/tests/test_k3.py`; dump parser `agam-os/tests/harness/aosdump.py`; `k3` catalogue entries |
| K3-X3 | Codex | **Stretch** cases: `exc_pf_write_rodata.c`, `exc_pf_exec_data.c`, `exc_df_ist.c` |
| K3-X4 | Codex | Evidence and review reports for K3 |

**Acceptance criteria**

1. `run.py gate K3` exits 0: required tests T-K3-01…06 and T-K3-10…16, plus earlier suites.
2. RV-K3-01 and RV-K3-02 are filed; no Critical or Major finding is open.
3. Stretch tests T-K3-07…09 are reported with their real status; any stretch failure is classified (`TEST_PLAN.md` §4.5).

**Risks**: R-06, R-05, R-09. K3 is the highest correctness risk of the sprint.

### K4 — Physical memory manager (Day 4)

| | |
|---|---|
| **Prerequisites** | K3 accepted. |
| **Spec sections** | §10, §12.1 |

| Task | Who | Paths |
|---|---|---|
| K4-C1 | Claude | Physical memory manager and memory-map logging (boot step 7) under `agam-os/kernel/`; `aos/pmm.h` |
| K4-C2 | Claude | `agam-dev/reports/claude/K4.md` |
| K4-X1 | Codex | Required cases: `pmm_stats.c`, `pmm_unique.c`, `pmm_alloc_free.c`, `pmm_contig.c`, `pmm_exhaust.c`, `pmm_rw.c`, `pmm_double_free.c`, `pmm_reuse.c`, `pmm_free_contig_valid.c` |
| K4-X2 | Codex | `agam-os/tests/test_k4.py`; memory-map parser; `k4` catalogue entries |
| K4-X3 | Codex | **Stretch** cases (O-19): four invalid single-frame frees and three invalid contiguous frees |
| K4-X4 | Codex | Evidence and review reports for K4 |

**Acceptance criteria**

1. `run.py gate K4` exits 0: required tests T-K4-01…08, T-K4-10, T-K4-11, plus earlier suites.
2. RV-K4-01 (validation code for every invalid case, reviewed) and RV-K4-02 are filed; no Critical or Major finding is open.
3. Stretch tests T-K4-09 and T-K4-12 are reported with their real status and classified if they fail.

**Risks**: R-08, R-09.

### K5 — Matrix and closure (Day 5)

| | |
|---|---|
| **Prerequisites** | K4 accepted. |

| Task | Who | Paths |
|---|---|---|
| K5-C1 | Claude | Fixes for open Critical and Major findings; `agam-dev/reports/claude/K5.md`; `agam-dev/reports/claude/gdb-session.md` |
| K5-X1 | Codex | `agam-os/tests/test_matrix.py`; `k5` catalogue entries; optional UEFI test |
| K5-X2 | Codex | `agam-dev/reports/codex/sprint1-final.md` |
| K5-A1 | Architect | Record what was learned in `DECISIONS.md`; propose the K6 specification |

**Acceptance criteria**

1. `run.py gate K5` exits 0: T-K5-01…05 (every **required** K1–K4 kernel test at four memory sizes, and 20 repetitions at 256 MiB, all from images built once), plus earlier suites.
2. E-K5-01…03 are filed. Every V-item has a recorded status; V-07, V-08 and V-13 may remain `UNVERIFIED`.
3. No Critical or Major finding is open; every stretch failure is classified.
4. The owner accepts the sprint and tags `aos-v0.1.0-sprint1` on `main`.

**Risks**: R-10, R-18.

---

## 4. Sprint 2 — AgamLang v0.1 (proposed Days 6–10)

Verification for every L-milestone is the gate procedure with `run.py gate <milestone>`. Language tests run with `PYTHONPATH=agam-lang/src` inside the environment of M0-O2.

| | L1 | L2 | L3 | L4 | L5 |
|---|---|---|---|---|---|
| **Content** | Lexer, Fusion normaliser, `tokens` | Parser, AST, `ast`, `check` | Core interpreter, `run` | Functions, lists, built-ins, limits | Diagnostics, catalogues, closure |
| **Spec sections** | §2–§4, §8 (lexical), §9 | §5, §6 | §7.1–§7.7, §7.10 without lists and functions | §7.8, §7.9, §7.11 | §8, Appendix A |
| **Prerequisites** | M0 accepted; G-LANG open (B-05, B-06, B-09, B-10 cleared; owner approval) | L1 accepted | L2 accepted | L3 accepted | L4 accepted |
| **Claude** | `agam-lang/pyproject.toml`; lexer, keyword table, source and token modules, lexical diagnostics, CLI `tokens` | Parser, nodes, CLI `ast`/`check` | Values, interpreter core, CLI `run` | Built-ins, lists, functions, limits; `agam-lang/examples/` | Both message catalogues, suggestions, message-language selection, internal-error path |
| **Codex** | `fixtures/keywords_expected.json`; lexer, phrase and composition tests; fusion triples; `l1` catalogue entries | Golden ASTs, precedence, syntax-error and fusion-AST tests | Core conformance, numeric-closure and CLI tests | Function, list, built-in, cycle, limit, equality, evaluation-order and built-in-alias tests | `codes_expected.json`, `messages_expected.json`, rendered-diagnostic goldens, suggestion, language, fuzz and injection tests |
| **Required tests** | T-L1-01…07 | T-L2-01…05 | T-L3-01…06 | T-L4-01…09 | T-L5-01…07 |
| **Reviews** | RV-L1-01 | — | — | RV-L4-01 | RV-L5-01 |

**Acceptance criteria (each L-milestone)**

1. `run.py gate <milestone>` exits 0 for the required tests above plus earlier `l` suites, `m0`, and the suites of accepted K-milestones.
2. The milestone's review items are filed; no Critical or Major finding is open.
3. L5 only: the owner accepts and tags `agamlang-v0.1.0`.

Independent expectations: Codex's fixtures and golden files are written by hand from the specification (`TEST_PLAN.md` §6.1). Hinglish expectations follow the proposed wording and are regenerated only after an approved specification change.

**Risks**: R-13, R-14, R-17, R-19, R-24.

---

## 5. Agent task summary

| Milestone | Claude Code (implements) | Codex (tests, reviews) |
|---|---|---|
| M0 | Limine pins, fetch script, Makefile skeleton | Environment check, manifest checker, runner scaffolding with self-tests |
| K1 | Build system, linker script, boot, serial, kprintf, panic, test hooks | QEMU harness, three cases, static checks, review |
| K2 | GDT, TSS | Three cases, reproducibility check, review |
| K3 | IDT, stubs, dump, expectations and finalisation | Twelve required cases (three stretch), dump parser, review |
| K4 | Physical memory manager | Nine required cases (seven stretch kernels), parser, review |
| K5 | Review fixes, GDB session record | Matrix, stability, final report |
| L1–L5 | Interpreter per spec | Independent fixtures; unit, golden, conformance, fusion, fuzz, injection tests; reviews |

---

## 6. Later milestones (outline; each needs an approved specification first)

| ID | Content | Depends on |
|---|---|---|
| K6 | Own page tables, guard pages, reclaim bootloader memory; removes the HHDM alias weakness (R-20) | Sprint 1 |
| K7 | Kernel heap | K6 |
| K8 | ACPI MADT, local APIC, I/O APIC, timer, interrupts enabled | K6 |
| K9 | Framebuffer console | K6 |
| K10 | Kernel threads, scheduler | K7, K8 |
| K11 | User mode, system calls | K10 |
| L6 | REPL, text interpolation, dictionaries | L5 |
| L7 | `agam translate` | L5 |
| L8 | Bytecode VM; later native back end | L6 |
| D1 | `agamdev` CLI | Sprint 1, L5 |
| X1 | AgamLang on AOS | K11, L8 |

---

## 7. Definition of done (every milestone)

1. All deliverables exist at the listed paths, and nothing outside the milestone's scope changed.
2. The gate procedure of `TEST_PLAN.md` §8 succeeded for one recorded set of `BASE`, `TIP_C`, `TIP_X` and `GATE_HEAD`, including manifest and merge verification.
3. The runner exited 0 for that gate: every required test ran and passed; none was skipped or missing.
4. All `E-` and `RV-` items of the milestone are filed with evidence blocks.
5. No Critical or Major finding is open; every stretch failure is classified.
6. Every commit carries the identity of O-18 and no attribution text.
7. The owner accepted the milestone and advanced `main` by fast-forward.

---

## 8. Risk register

Likelihood and impact are the architect's judgement (L, M, H), not measurements.

| ID | Risk | L | I | Mitigation | Detected by |
|---|---|---|---|---|---|
| R-01 | Host environment incomplete or different from the record | M | H | M0; versions recorded in every report | T-M0-01, E-M0-04 |
| R-02 | Git run from WSL sees the Windows worktrees as prunable and could delete their registrations | H | H | Git only with Git for Windows; no tool in WSL calls git | E-M0-05; rules |
| R-03 | CRLF line endings. Files are currently checked out with CRLF in the agent worktrees. CRLF **can** break shell scripts (for example the interpreter line and line continuations), Makefile recipes and byte-exact golden files; not every script fails, which makes the failures hard to spot. | H | M | `.gitattributes` is the first bootstrap change; the binding check hashes file contents | T-M0-03, E-M0-03 |
| R-04 | Limine release or protocol details differ at run time from what was read | M | H | Pin by hash; identity and pairing independently reproduced; boot compatibility checked at K1 (V-14) | K1 |
| R-05 | Faults before the IDT exists appear only as a reset | H | M | Serial first; `-no-reboot`; `cpu_reset` log; host status 0 is a failure | Harness |
| R-06 | Descriptor, frame or alignment errors | M | H | Static assertions; review items RV-K2-01, RV-K3-01; register-value tests | K2, K3 |
| R-07 | Compiler emits floating-point or vector instructions, or uses the red zone | L | H | Flags; widened disassembly check T-K1-04 | K1 |
| R-08 | Allocator hands out memory in use, or frees what it should not | M | H | Clipped ranges, bounded scans, validate-before-mutate; RV-K4-01, RV-K4-02 | K4 |
| R-09 | Access through the HHDM to memory the kernel does not own | M | M | Ownership table; the kernel never relies on a fault to stop such an access | Review |
| R-10 | QEMU/TCG differs from real hardware | H | L | Out of scope; recorded | — |
| R-11 | Both agents modify the same file | M | H | Disjoint ownership; manifest checker; merge verified record by record | Gate |
| R-12 | A report claims success without evidence | M | H | Evidence format; acceptance only from a bound gate run | Gate |
| R-13 | Hinglish keywords or messages read unnaturally | M | M | Wording stays a proposal until the owner approves (O-20, B-06) | Owner review |
| R-14 | Indentation confuses beginners | H | M | Dedicated error codes and messages | L1, L5 |
| R-15 | Scope creep toward non-goals | M | H | C-10; roadmap changes need approval | Architect |
| R-16 | Python version differences | L | M | Minimum version declared and checked | T-M0-01 |
| R-17 | Specification ambiguity makes tests and implementation disagree | M | M | Spec is the arbiter; proposals; independent re-review | Reviews |
| R-18 | UEFI firmware unavailable | M | L | Optional test | K5 |
| R-19 | Tests take expectations from the implementation | M | H | Hand-written fixtures from the spec; rule in `AGENTS.md` | Reviews |
| R-20 | The HHDM alias makes kernel memory writable and executable under Limine's page tables; stacks have no guard pages; allocated frames are not zeroed | H | L (no untrusted code in Sprint 1) | Recorded limitation; K6 | Review |
| R-21 | AI attribution or a wrong identity in a commit | M | M | O-18 rules; `git log` check at every gate | E-M0-02; gate |
| R-22 | Building on `/mnt/d` from WSL is slow or behaves differently | M | M | Measured at K1; fallback is moving the repository into the WSL file system | K1 |
| R-23 | Empty project folders are not stored by Git | H | L | They appear with their first committed files | — |
| R-24 | AgamLang limits are policy, not a sandbox: one operation can still be costly, and the step limit is optional | M | M | Checks before allocation; untrusted programs are out of scope for v0.1 | RV-L4-01 |
| R-25 | A gate result is reused after a tip, base or tool changed | M | H | Results are bound to exact SHAs and tool versions and are void otherwise | `TEST_PLAN.md` §4.6 |
| R-26 | A branch under test weakens the tool that checks it | M | H | The ownership checker is taken from `BASE`; for M0 the owner checks paths by hand | `TEST_PLAN.md` §8.1 |
| R-27 | The debugger port is reachable from the network | L | M | `make debug` binds to `127.0.0.1` only | RV-K1-01 |

---

## 9. Blockers

The authoritative list is `DECISIONS.md` §9. Nothing in this roadmap may start until the gate named in §1 for it is open.
