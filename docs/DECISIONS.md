# Agam Computing Ecosystem — Decision Log

| Field | Value |
|---|---|
| Status | **Draft v0.1, revision 3 — NOT APPROVED.** Prepared in response to the independent architecture review (§8.5). Every review finding is *addressed in draft — pending independent re-review* (§10). No gate is open. |
| Date | 2026-11-12 (revision 3) |
| Rule | This file is the only place where a decision's status changes. Only the project owner can approve. |

## How to read this file

| Prefix | Meaning | Who can change it |
|---|---|---|
| **C-nn** | **Confirmed** — stated by the owner in the original brief | Owner |
| **O-nn** | **Owner decision, provisional** — binding for these drafts; the owner may revise it | Owner |
| **P-nn** | **Proposal** — the architect's recommendation, with its status | Owner approves, rejects or amends |
| **Q-nn** | **Open question** | Owner, or evidence |
| **B-nn** | **Blocker** — must be cleared before the gate it names opens (§9) | Cleared with evidence, by the party named |
| **AR-nn** | **Review finding** from the independent review (§10) | Closed only by independent re-review |

---

## 1. Gates — all closed

Revision 3 is a document revision only. Editing these documents, including under the owner's instruction of 2026-11-12, authorises no implementation, no test code and no harness code.

| Gate | Permits | Opens when |
|---|---|---|
| **G-BOOT** | Only the bootstrap work listed in `ROADMAP.md` §2.2: root metadata, environment check, manifest checker, runner scaffolding with self-tests, Limine pin files and fetch script, Makefile skeleton, reports. No kernel, interpreter, kernel-test or language-test file. | B-07 cleared by re-review; B-04 cleared; then the owner says so. **No evidence produced by the bootstrap itself is required**, so there is no cycle (AR-01). |
| **G-AOS** | Kernel code and kernel tests (K1–K5) | M0 accepted (this clears B-01); B-02 disposition recorded; B-05 and B-08 cleared; the owner explicitly approves the AOS documents |
| **G-LANG** | Interpreter code and language tests (L1–L5) | M0 accepted; B-05, B-06, B-09 and B-10 cleared; the owner explicitly approves the AgamLang specification |

Opening one gate never opens another. Which findings block which gate: AR-01…04, AR-15, AR-18 → G-BOOT (through B-07); AR-05…07, AR-16, AR-17, AR-19, AR-21 → G-AOS (B-08); AR-08…13, AR-22 → G-LANG (B-09); AR-14 → G-LANG (B-10).

---

## 2. Confirmed decisions (original brief)

| ID | Decision |
|---|---|
| C-01 | Three projects in one repository: AOS, AgamLang, Agam Developer OS |
| C-02 | AOS kernel is original, x86-64, written initially in C17 and assembly |
| C-03 | Bootloader is Limine |
| C-04 | Toolchain is Clang/LLD |
| C-05 | Emulator is QEMU; debugger is GDB |
| C-06 | Three working directories: `ecosystem` (integration), `claude-worktree` (Claude Code), `codex-worktree` (Codex) |
| C-07 | First AgamLang implementation is a Python-based interpreter |
| C-08 | Fusion supports English, Hinglish and mixed syntax in one `.agam` file, through canonical tokens, not separate interpreters |
| C-09 | All architecture documents are Draft v0.1 until the owner explicitly approves them |
| C-10 | Priority is a minimal, reproducible AOS kernel; networking, GUI, AI integration and native AgamLang execution are out of scope |
| C-11 | No implementation before the architecture is reviewed and approved |
| C-12 | No invented benchmarks; no untested claims; no assumption that tools are installed |
| C-13 | Claude Code and Codex work independently without modifying the same files |
| C-14 | Architectural decisions are preserved in Markdown tracked by Git |
| C-15 | First sprint: bootable kernel, CPU exception handling, physical memory management, planned as five days |

---

## 3. Owner decisions, provisional

### 3.1 Given on 2026-11-12 after the architecture summary

| ID | Decision |
|---|---|
| O-01 | Build and test in WSL2 Ubuntu 24.04. Git operations stay on Windows. |
| O-02 | Use the existing branch names: `main`, `agent/claude`, `agent/codex`. |
| O-03 | Limine base revision 6, on condition that the pinned release and header compatibility are independently verified (B-02). |
| O-04 | Hardware interrupts stay disabled during Sprint 1; CPU exception handling only. |
| O-05 | AgamLang uses Python-style indentation, with no `end` or `khatam`. |
| O-06 | 0-based list indexing. |
| O-07 | Fusion is the default; English, Hinglish and mixed keywords work in the same source file. |
| O-08 | AOS is a modular monolithic kernel with serial-first debugging. |
| O-09 | Double-fault testing and advanced fault injection are stretch goals. |
| O-10 | No AI tool may appear as author, committer, co-author or contributor, and no AI attribution text may appear in commits, tags or pull requests. (Identity values: see O-18.) |
| O-11 | `main` is not modified without the owner's approval; branch and worktree are verified before any commit. |
| O-12 | Root `D:/Agam-Computing` with worktrees `ecosystem`, `claude-worktree`, `codex-worktree`; project folders `agam-os/`, `agam-lang/`, `agam-dev/`, `docs/`. |

### 3.2 Given on 2026-11-12 for revision 3, after the independent review

| ID | Decision | Where applied |
|---|---|---|
| O-13 | **Lists reject direct and indirect cycles** on every mutation path, including `append` and indexed assignment. An invalid mutation leaves the list unchanged. | `AGAMLANG_SPEC.md` §7.8 |
| O-14 | **Provisional v0.1 limits**: 1,000 decimal digits for whole-number magnitude; 1,000,000 elements or code points per collection or text. These are policy, not measured safety guarantees. Scope, boundaries and diagnostics are defined exactly, and checks happen before potentially expensive allocation or computation. | `AGAMLANG_SPEC.md` §4.4, §7.1, §7.4 |
| O-15 | **Python-compatible evaluation order**: expressions and arguments left to right; ordinary assignment evaluates the right-hand side before resolving the target; augmented assignment evaluates its target once, before the right-hand side. | `AGAMLANG_SPEC.md` §7.3 |
| O-16 | **Fusion accepts `warna if`, `else agar` and `nahi to agar`.** Phrase handling stays in the lexer/normaliser, with exact spans, original spelling and language metadata for mixed phrases. The parser stays independent of keyword spelling and language. | `AGAMLANG_SPEC.md` §3.3, §3.4 |
| O-17 | **Integration uses temporary `gate/<milestone>` branches.** Only the owner advances `main`, after successful verification and acceptance. The procedure fixes a baseline and exact agent-tip SHAs, invalidates stale results, and verifies contents, file modes, additions, deletions and renames. | `TEST_PLAN.md` §4.6, §8 |
| O-18 | **Commit identity**: author and committer name `Agam Sharma`, email `rs3543450@gmail.com` — the values currently configured in the repository. The GitHub username `Agamsma` is the account that owns the repository; it is **not** the commit name. No agent changes Git configuration. The no-attribution rule of O-10 stays. | `CLAUDE.md` §4, `AGENTS.md` §4 |
| O-19 | **Allocator invalid-range fault-injection tests are stretch** (under O-09). The validation code and its independent review are mandatory. Positive allocator tests are required. | `AOS_KERNEL_SPEC.md` §10.6; `TEST_PLAN.md` §5 K4 |
| O-20 | **Hinglish wording is pending the owner's review.** The complete proposed keyword and message material is prepared, and none of it is marked approved. | `AGAMLANG_SPEC.md` §3.2, Appendix A.2; B-06 |
| O-21 | Additional requirements for revision 3: milestone-specific gate commands and suites; required skips, missing required tests and empty required selections fail acceptance; runner self-tests and the scaffolding they need belong to M0; expected keyword fixtures are independent and derived from the specification; diagnostic source metadata is preserved without changing AST equality or default AST JSON; E0900 is tested by controlled internal injection, not a public switch; internal implementation details stay non-normative unless interoperability or independent testing needs them. | `TEST_PLAN.md` §4, §6.1; `ROADMAP.md` §2; `AGAMLANG_SPEC.md` §6.3, §8.7, §10; `AOS_KERNEL_SPEC.md` §13.1 |

---

## 4. Proposals — environment and process

| ID | Proposal | Status |
|---|---|---|
| P-03 | WSL2 Ubuntu 24.04 for build and test; Git for Windows for version control | Provisionally accepted (O-01) |
| P-06 | GNU Make; neither the build nor any WSL tool runs `git`; version from the root `VERSION` file | Proposed |
| P-10 | Path ownership: one writer per path (`ARCHITECTURE.md` §6.1), now including `agam-dev/gates/**` for the owner | Proposed; checker provenance amended (AR-02) |
| P-11 | Repository layout of `ARCHITECTURE.md` §4 | Proposed |
| P-15 | Evidence protocol, status vocabulary, test/evidence/review identifiers and severity names (`TEST_PLAN.md` §2) | Proposed; amended (AR-04, AR-19) |
| P-16 | `.gitattributes` forces LF; it is the first bootstrap change | Proposed |
| P-17 | Documents reach the agents through `main`, advanced by the owner once approved | Proposed; the owner has meanwhile committed revision 2 on `review/architecture-v0.1` |
| P-19 | Gate procedure of `TEST_PLAN.md` §8: fixed baseline, raw manifests with modes and blob IDs, checker taken from the baseline, record-for-record merge verification, results bound to SHAs | Proposed; implements O-17 |
| P-41 | Bootstrap scope of `ROADMAP.md` §2.2, including runner scaffolding and self-tests | Proposed; implements O-21 |

## 5. Proposals — AOS

| ID | Proposal | Status |
|---|---|---|
| P-01 | Limine boot protocol, base revision 6 | Provisionally accepted, conditional (O-03); evidence in §8.1 |
| P-02 | Pin Limine 12.9.6 and `limine-protocol` commit `6e85ddd…` | Provisionally accepted, conditional (O-03); evidence in §8.1 |
| P-04 | Compiler and linker flags of `AOS_KERNEL_SPEC.md` §2, now split into common, C-only and assembly sets | Proposed; C set observed to compile, assembly and link not run |
| P-05 | Assembly in `.S` files, Clang integrated assembler, AT&T syntax | Proposed |
| P-07 | Kernel image at `0xFFFFFFFF80000000`, non-relocatable, image limited to 1 GiB; HHDM base read at run time; 4-level paging; Limine's page tables kept in Sprint 1 | Proposed |
| P-08 | Sprint 1 interrupt handling: own GDT/TSS/IDT, vectors 0–31, interrupts disabled | Provisionally accepted (O-04) |
| P-09 | Bitmap frame allocator; Usable memory from 1 MiB up to a provisional 1 TiB bound; clipped ranges; validate-before-mutate; no reclaim of bootloader memory | Proposed; contract completed (AR-06, AR-07); see Q-16 |
| P-13 | Serial is the only console in Sprint 1 | Provisionally accepted (O-08) |
| P-14 | Compile-time test kernels ending through `isa-debug-exit`; one finalisation path; guest values `0x10`/`0x11`, host statuses 33/35 | Proposed; finalisation contract completed (AR-05) |
| P-18 | Modular monolithic kernel; only test-facing headers are normative | Provisionally accepted (O-08) |
| P-36 | NX is required: the kernel checks for it at boot and panics without it | Proposed (AR-17) |

## 6. Proposals — AgamLang

| ID | Proposal | Status |
|---|---|---|
| P-20 | CPython ≥ 3.11, standard library only at run time | Proposed |
| P-21 | Hand-written lexer and recursive-descent parser | Proposed |
| P-22 | One runtime alias table; canonical tokens; phrases and else-if composition in the normaliser | Provisionally accepted (O-07, O-16) |
| P-23 | Every alias word and phrase component is reserved in every file | Provisionally accepted (O-07) |
| P-24 | Keywords are ASCII case-insensitive; identifiers are case-sensitive | Proposed |
| P-25 | Python-style indentation blocks | Provisionally accepted (O-05) |
| P-26 | Keyword-first grammar in v0.1 | Proposed |
| P-27 | `pytest`, pinned, as a development dependency | Proposed |
| P-28 | Strict semantics: boolean conditions, no implicit text/number mixing, `/` gives a decimal, exclusive `range`, 0-based lists | 0-based accepted (O-06); rest proposed |
| P-29 | Diagnostics with stable codes, exact message catalogue, span rules and deterministic suggestions | Proposed; contract completed (AR-14); Hinglish wording pending (O-20) |
| P-30 | v0.1 language surface of `AGAMLANG_SPEC.md` | Proposed |
| P-31 | Numbers are finite reals; non-real and overflowing results are errors | Proposed (AR-11); limits per O-14 |
| P-32 | Strict recursive equality; function identity; no cyclic lists | Proposed (AR-12); cycles per O-13 |
| P-33 | Evaluation order and function-boundary rules of `AGAMLANG_SPEC.md` §7.3 and §5.2 | Proposed (AR-13); order per O-15 |
| P-34 | Independent expected fixtures written from the specification | Proposed (AR-08); required by O-21 |
| P-35 | Source metadata (`span`, `origin`) outside node equality and default JSON; E0900 tested by internal injection | Proposed (AR-14); required by O-21 |

## 7. Proposals — Developer OS

| ID | Proposal | Status |
|---|---|---|
| P-40 | Developer OS in stages D0–D3; for now `agam-dev/` holds verification tools, agent reports and gate records | Proposed |

---

## 8. Version and evidence record

### 8.1 Pinned external inputs

Evidence of **artifact identity and source pairing** is kept apart from checks that have **not been performed**.

| Input | Exact value | Identity / pairing evidence |
|---|---|---|
| Limine release | 12.9.6 — tag `v12.9.6`; annotated tag object `54727fd1c0d58f54e615fc6e1e72e38f4b635455`; commit `89a9512c623a4779cc73e3869383876d54264b28`; published 2026-10-27T23:16:30Z | Tag cloned by the architect; tag-to-commit resolution recorded by Codex (E01, `tag.json`) |
| Binary asset | `limine-binary.tar.xz`, 906,108 bytes, SHA-256 `939f670e0831cc940453fd3a0ddfb2b324c0b4e3c5d0e741c190aa0d13df0103` | Downloaded and hashed by Codex (E01, `downloads.json`); downloaded and hashed again by the architect on 2026-11-12 from a sandbox on the owner's computer: same size, same digest, same 16-entry listing |
| Asset contents | `limine-bios.sys`, `limine-bios-cd.bin`, `limine-uefi-cd.bin`, `BOOTX64.EFI`, `limine.c`, `Makefile` present | Codex E01 (`archive-list.txt`); architect's listing |
| Alternative asset | `limine-binary.tar.gz`, 2,143,398 bytes, SHA-256 `b38aabb042b0c0f9f562b7741be3c1204838d7bbacfa78bb411c674bd5b984ee` | Codex (`additional-artifacts.json`) |
| Protocol header | `limine-protocol` commit `6e85ddd6f928b5b512015c4d47d1cfce2fd28079`; `include/limine.h`, 17,709 bytes, SHA-256 `0aadf2633c85d8cb145344c968add158aeb1deb68af567d55ba8431c492b6d4c` | Hash computed by the architect from a clone and by Codex from a download (E02) |
| Release ↔ header pairing | The release's `bootstrap` pins commit `6e85ddd…`; `common/protos/limine.c` defines `SUPPORTED_BASE_REVISION 6` | Read by the architect; searched by Codex (E02, `source-inspection.txt`) |
| Identifiers | All 20 identifiers and the fields the kernel specification uses exist in the header | Architect's compile probe; Codex E03 |
| Base revision requested | 6 | O-03 |

| Check **not performed** | Status | Needed for |
|---|---|---|
| Detached signature of the release asset | Not performed (V-13) | Nothing; recorded so the hash is not mistaken for it |
| Building the Limine host utility from the asset | Not performed | M0 (T-M0-02) |
| Assembling `.S` files and linking with the specified flags | Not performed (V-12) | K1 |
| Booting a kernel with this release and base revision 6 | Not performed (V-14) | K1 |

Disposition of B-02 is the owner's to record (§9).

### 8.2 Toolchain (WSL2 Ubuntu 24.04 LTS)

"Expected" is the package version offered by the Ubuntu 24.04 archive on 2026-11-12. "Owner-reported" is what the owner stated on 2026-11-12; no command output has been filed yet, so M0 still records it (T-M0-01, E-M0-04).

| Tool | Package | Expected | Owner-reported | Evidence filed |
|---|---|---|---|---|
| Ubuntu under WSL2 | — | 24.04 LTS | 24.04.5 LTS, installed and running | None yet |
| Clang | `clang` → `clang-18` | 18.1.3 | 18.1.3 | None yet |
| LLD | `lld` → `lld-18` | 18.1.3 | 18.1.3 | None yet |
| LLVM tools | `llvm` → `llvm-18` | 18.1.3 | Not reported | — |
| GNU Make | `make` | 4.3 | 4.3 | None yet |
| QEMU | `qemu-system-x86` | 8.2.2 | 8.2.2 | None yet |
| GDB | `gdb` | 15.1 | 15.1 | None yet |
| xorriso | `xorriso` | 1.5.6 | Not reported | — |
| Host C compiler (`cc`) | `build-essential` | GCC 13 series | Not reported | — |
| curl or wget; xz | `curl`, `xz-utils` | Any | Not reported | — |
| Python | `python3`, `python3-venv` | 3.12.3 | Not reported | — |
| pytest | pinned in `requirements-dev.txt` | Exact version chosen at M0 | — | — |
| OVMF (optional) | `ovmf` | 2024.02 | Not reported | — |
| Git | Git for Windows | — | — | 2.52.0.windows.1 (Codex E00, `environment.json`) |

### 8.3 What the architect ran (scratch sandboxes, not the owner's WSL, not project code)

| Check | Result |
|---|---|
| Throwaway C file including the pinned `limine.h`, compiled with the C flag set, Clang 18.1.3 | Exit 0, zero warnings, no `xmm`/`ymm`/`st` operands. Not linked; no assembly. |
| Hand-assembled boot sectors in QEMU 8.2.2 with `isa-debug-exit` and `-no-reboot` | Guest `0x10` → host 33; `0x11` → 35; `0x00` → 1; no exit call → timeout; guest reset → 0 |
| Limine `CONFIG.md`, `USAGE.md`, protocol specification at the pinned commits | Option names, ISO command and entry-state statements match the kernel specification |
| Second download of `limine-binary.tar.xz` (2026-11-12, sandbox on the owner's computer) | 906,108 bytes; SHA-256 `939f670e…0103`; 16 entries, identical to Codex's listing |

### 8.4 Repository state on 2026-11-12 (read from the repository's own files; no git command was run for revision 3)

| Item | Value |
|---|---|
| Remote | `origin` → `https://github.com/Agamsma/Agam-Computing` |
| `main` | `4b2edd6e7c67c0dd167aec3e2d7e2004c86cf48d` — tracks only `README.md` |
| `review/architecture-v0.1` | `349c48a46d49cfac725b687868173d9423930309` — adds the nine revision-2 documents; checked out in `D:/Agam-Computing/ecosystem` |
| `agent/codex` | `349c48a…` (same commit); checked out in `codex-worktree` |
| `agent/claude` | `4b2edd6…`; checked out in `claude-worktree`; does not contain the documents |
| Revision 3 | Uncommitted changes in the `ecosystem` working folder on `review/architecture-v0.1` |
| Line endings | Committed with LF. Codex's evidence shows the agent worktrees checked out with CRLF (`i/lf w/crlf`), because no `.gitattributes` exists yet (R-03). |
| Commit identity | Commit `349c48a` and the repository configuration use `Agam Sharma <rs3543450@gmail.com>` (O-18). Commit `4b2edd6` used a different address (Q-14). |
| Untracked items | Codex's review report, evidence and proposal in `codex-worktree`; a folder `Claude outputs` with two older document copies in `claude-worktree` (Q-17) |

### 8.5 Independent review record

| Item | Value |
|---|---|
| Review | Architecture review of Draft v0.1 revision 2 by Codex, 2026-11-12 |
| Reviewed commit | `349c48a46d49cfac725b687868173d9423930309` |
| Report | `agam-dev/reports/codex/ARCHITECTURE_REVIEW.md` in `codex-worktree`; SHA-256 `b3f923ecaec34b9a2a318a37cd0ed3235627479618b4bee81ce5ad2cefeb86a9` (recorded by Codex and recomputed by the architect) |
| Supporting material | `agam-dev/reports/codex/architecture-review-evidence/` and `agam-dev/reports/codex/proposals/001-architecture-contract-clarifications.md` |
| Tracking | Untracked at the time of revision 3. It is cited by reviewed commit and file hash; committing it is the owner's decision (Q-17). It was not modified by revision 3. |
| Verdict | **Not ready for implementation approval**: 0 Critical, 14 Major, 8 Minor, 2 Informational |
| Confirmed by the review | Limine archive and header hashes reproduced; release/header pairing; GDT, TSS, IDT and interrupt-frame arithmetic; exit-value formula from QEMU source. None of these is a build, boot or runtime result. |

---

## 9. Blockers

No blocker is cleared by this revision.

| ID | Blocker | State after revision 3 | Cleared by | Gate |
|---|---|---|---|---|
| B-01 | Toolchain evidence on the owner's machine | Owner reports the main tools at the expected versions; no command output filed; several tools unreported | M0 accepted (T-M0-01, E-M0-04) | G-AOS, G-LANG |
| B-02 | Independent verification of the Limine pin (condition of O-03) | Identity and pairing evidence exists (§8.1): Codex E01–E03, plus the architect's separate archive download. Signature and runtime checks are separate and not performed. | **The owner records the disposition** (E-M0-06) | G-AOS |
| B-03 | Git identity and attribution | Identity recorded (O-18). Still to show: attribution switched off in each tool and absent from history (E-M0-02). Q-14 open. | E-M0-02 | G-BOOT acceptance (M0) |
| B-04 | Approved documents on `main` | Revision 2 is on `review/architecture-v0.1` and `agent/codex`, not on `main` or `agent/claude`. Revision 3 is uncommitted. | The owner advances `main` to the approved document commit | G-BOOT |
| B-05 | Independent architecture review with no Critical or Major finding open | First review: not ready. Revision 3 addresses all findings in draft. **Not cleared; cannot be cleared by the architect.** | Codex's re-review of revision 3, then the owner | G-AOS, G-LANG |
| B-06 | Hinglish wording: keyword table, built-in names, message catalogue (Q-07), output words (Q-08), Devanagari (Q-09) | Complete proposals prepared; none approved | The owner's review | G-LANG |
| B-07 | Process contract: bootstrap gate, ownership provenance, milestone suites, skip rules (AR-01…04, AR-15, AR-18) | Addressed in draft — pending independent re-review | Re-review | G-BOOT |
| B-08 | Kernel test and allocator contract (AR-05…07, AR-16, AR-17, AR-19, AR-21) | Addressed in draft — pending independent re-review | Re-review | G-AOS |
| B-09 | Independent language oracle and closed semantics (AR-08…13, AR-22) | Addressed in draft — pending independent re-review | Re-review | G-LANG |
| B-10 | Diagnostic contract (AR-14) | Addressed in draft — pending independent re-review; Hinglish half also needs B-06 | Re-review | G-LANG |

---

## 10. Finding-resolution table (independent review of revision 2)

Status of **every** row: *addressed in draft — pending independent re-review*. No finding is closed.

| Finding | Revised sections | Proposed resolution | Required verification | Remaining owner decision |
|---|---|---|---|---|
| AR-01 (Major) Bootstrap gate is circular | DEC §1, §9; ROAD §2.1–§2.4; CLAUDE §7; AGENTS §9 | G-ENV replaced by G-BOOT, which needs no evidence from the work it permits and permits only an exhaustive bootstrap list. M0 acceptance, not gate opening, supplies the environment evidence and is a prerequisite of G-AOS and G-LANG. | Re-review confirms no gate depends on its own output and no implementation path is in the bootstrap scope | Opening G-BOOT |
| AR-02 (Major) Ownership check lacks a safe backend and provenance | ARCH §6.1, §6.4; TEST §4.6, §8; ROAD §2.2 (M0-X2), §7; CLAUDE §4; AGENTS §4 | Git for Windows writes raw, NUL-delimited manifests (`--raw --no-abbrev --no-renames`) from a fixed `BASE` to each exact tip; a Git-free checker, taken from `BASE`, checks ownership, modes and record-for-record equality of the merge with the two manifests on a `gate/<milestone>` branch. Worktrees must be clean. Author identity is not used to partition changes. | T-M0-05 self-tests; E-M0-07 for the first gate; re-review of §8 | None (O-17) |
| AR-03 (Major) Universal gate cannot accept early milestones | TEST §4.2–§4.4; AOS §11.1; ROAD §2–§4 | Each gate has a fixed required suite set; `run.py gate <milestone>` selects only it, plus suites of accepted milestones of the other track. `make test` needs `MILESTONE`. Future suites are outside the run. | T-M0-04 (g); re-review | None |
| AR-04 (Major) Stretch and skip rules conflict | TEST §1, §4.1, §4.4, §4.5, §5 K5; ROAD K3–K5 criteria, §7; AOS §9.6, §10.6, §15 | Tests have a class. The runner reports required, stretch and optional totals separately and fails on any required failure, skip, missing test or empty selection. Observe mode is a defined verdict. Stretch V-items may stay `UNVERIFIED`; every stretch failure is classified. | T-M0-04 (b)–(e); re-review | None |
| AR-05 (Major) Test finalisation and fault expectations incomplete | AOS §5 step 9, §9.5, §12.4; TEST §5 (T-K1-07, T-K1-10, T-K3-13…16) | One finalisation path, `ktest_finish()`; sticky failure; pending expectation fails; an armed expectation takes precedence for every vector including #BP; wrong fault reported; panic ends the test as failed with a `RESULT` line. | Four negative-control tests and `kt_bp_expected`, once G-AOS opens; re-review | None |
| AR-06 (Major) `pmm_free_contig` has no contract | AOS §10.4, §10.6; TEST §5 (T-K4-05, T-K4-11, T-K4-12, RV-K4-01) | Zero count is a no-op; alignment and range checked without overflow; every frame validated before any change; invalid request panics unchanged. | T-K4-11 (required); RV-K4-01 (mandatory review); T-K4-12 (stretch) | None (O-19) |
| AR-07 (Major) Bitmap eligibility and bounds underspecified | AOS §10.2–§10.5, §12.1; TEST §5 (T-K4-02, T-K4-03, RV-K4-02) | Clipped ranges decide eligibility; checked arithmetic; 1 TiB provisional bound; empty-map panic; padding bits always set; scans bounded by `highest_frame`; hint wrap defined; log fields defined exactly. | T-K4-02, T-K4-03, T-K4-10; RV-K4-02 | Q-16: accept the 1 TiB bound |
| AR-08 (Major) Alias tests trust the implementation | LANG §3.1, §3.2, §10; ARCH §2; AGENTS §6; TEST §6.1, T-L1-01 | One runtime table; Codex hand-writes an independent fixture from the specification (52 words, 5 phrases, 6 components) and asserts exact equality. | T-L1-01; re-review | None (O-21) |
| AR-09 (Major) E0208 assigned to two stages | LANG §3.1 step 3, §4.1, §5.2, §8.6; TEST T-L1-02, T-L2-03 | The lexer emits `RESERVED_WORD` and never raises; the parser raises E0208 in three defined situations. | T-L1-02, T-L2-03 | None |
| AR-10 (Major) Canonical export and Fusion equality | LANG §4.1, §6.4, §9; ROAD §4; TEST T-L1-06, T-L2-04, T-L4-06, T-L5-06 | `IDENT` exports its spelling. Token and AST identity is required only for keyword substitution; built-in aliases are tested for equal behaviour and identity. Names are never canonicalised. | T-L1-06, T-L2-04, T-L4-06, T-L5-06 | None |
| AR-11 (Major) Numeric semantics not closed | LANG §4.4, §7.1, §7.4, §7.11, §8.6; TEST T-L1-04, T-L3-06, T-L4-03, T-L4-04 | Finite reals only; E0406, E0407, E0107; exact `number()` grammar; limits with boundaries; checks before expensive work; error precedence; no host exception escapes. | T-L3-06, T-L4-03, T-L4-04, T-L5-05; RV-L4-01 | Limits are provisional (O-14) |
| AR-12 (Major) Cycles, equality, display undefined | LANG §7.5, §7.6, §7.8; TEST T-L4-07, T-L4-08 | Cycles rejected on every store with the list unchanged (E0308); strict recursive equality; function identity; escaped text in lists; nesting and visit limits. | T-L4-07, T-L4-08 | None (O-13) |
| AR-13 (Major) Evaluation order and function boundaries missing | LANG §5.2, §7.2, §7.3, §7.9; TEST T-L2-03, T-L4-01, T-L4-09 | Python-compatible order stated rule by rule; loop context reset at function definitions; duplicate parameters E0213; fixed closure example. | T-L4-09, T-L2-03, T-L4-01 | None (O-15) |
| AR-14 (Major) Diagnostic contract cannot support exact goldens | LANG §3.4, §6.3, §8.1–§8.7, §9, §10, Appendix A; ROAD §4; TEST §6.1, T-L5-01…04, T-L5-07 | Complete message catalogue in both languages; span table; deterministic suggestions; language counts carried as data from the lexer; `origin` metadata outside equality and default JSON; `--messages` on every command; E0900 through an in-process test parameter only. | T-L5-01…04, T-L5-07; RV-L5-01 | **Hinglish wording review (O-20, B-06)** |
| AR-15 (Minor) Bootstrap prerequisites incomplete | ROAD §2.2; AOS §2; DEC §8.2 | Complete package list including `build-essential`, `curl`, `xz-utils`, venv; pinned pytest; Git version reported from Windows only. | T-M0-01, E-M0-04 | None |
| AR-16 (Minor) Forbidden-instruction test too narrow | TEST T-K1-04; AOS §15 V-01 | Check widened to MMX, x87 mnemonics, mask registers and state-save instructions, with the pattern derived from the real tool output. | T-K1-04 | None |
| AR-17 (Minor) Hardware statements too absolute | AOS §3.1, §4.1, §5 step 4, §12.5 | Statements qualified; NX made a checked boot requirement; non-canonical stack references noted; host status 0 not unique to reset. | Re-review; T-K1-05 (`nx enabled`) | None |
| AR-18 (Minor) Repository facts and review references stale | DEC §8.4, §8.5, §9; ARCH §9; ROAD header; brief; CLAUDE §4; AGENTS §4 | Actual snapshot, branches, report path and severity mapping recorded; identity recorded exactly; documents no longer hard-code the GitHub username as the commit name. | Re-review | Q-14, Q-17 |
| AR-19 (Minor) Traceability and oracles need tightening | TEST §2.4, §5; AOS §4.2, §12.1; ROAD §2–§4 | `E-` and `RV-` identifiers for manual evidence and reviews; one 20 s budget; symbol meanings; bitmap log fields; register-value and restore tests; `memcpy` overlap not tested; request-layout test. | Re-review of the traceability | None |
| AR-20 (Minor) Five days is unmeasured | ROAD §1, K1 criterion 3; TEST §4.8, §6.3, E-K5-03 | Five days stated as a target with a decision point after K1; images built once per run; fuzz bounded; durations recorded, not asserted. | E-K5-03 | Confirm or re-plan after K1 |
| AR-21 (Minor) Handoff details | AOS introduction, §2.1, §3.3, §4.2, §8.2, §9.1, §13.1, §15 | Request revisions and response handling; symbol meanings and image limit; zeroed reserved fields; separate C and assembly flags; only test-facing headers normative. | Re-review; V-12 at K1 | None |
| AR-22 (Minor) Beginner-facing choices | LANG §3.2, §3.3, §4.2–§4.4, §5.2, §7.11, §9, §12 | Else-if composition accepts mixed forms; tab/space rule with example; identifier normalisation; `1e3`; trailing commas; `--max-steps` values; built-in argument errors. | T-L1-02, T-L1-03, T-L1-04, T-L3-05, T-L4-03 | None (O-16) |
| AR-23 (Informational) Limited isolation | ARCH §8.2; AOS §11.1, §11.5; LANG §7.10, §12; ROAD R-20, R-24, R-27 | Limitations recorded; debugger bound to loopback; no security claim for untrusted code. | Re-review | None |
| AR-24 (Informational) Retain what the review supports | DEC §8.1, §8.5; AOS §15 | Evidence recorded and kept separate from unperformed checks; runtime tasks retained. | Re-review | B-02 disposition |

Abbreviations: DEC `DECISIONS.md`, ARCH `ARCHITECTURE.md`, ROAD `ROADMAP.md`, AOS `AOS_KERNEL_SPEC.md`, LANG `AGAMLANG_SPEC.md`, TEST `TEST_PLAN.md`.

---

## 11. Open questions

| ID | Question | Architect's suggestion | Blocks |
|---|---|---|---|
| Q-03 | BIOS boot as the single required boot path, UEFI optional? | Yes | G-AOS (minor) |
| Q-04 | Largest guest RAM in the matrix (4 GiB proposed)? | 4 GiB | K5 |
| Q-07 | Hinglish wording: keyword table, built-in names (`pucho`, `sankhya`, `shabd`, `lambai`, `jodo`, `ginti`, `prakar`), and every message of Appendix A.2 | Review word by word; natural-order forms deferred | B-06 |
| Q-08 | Should `show` print `true`/`false`/`nothing` always, or follow the file's language? | Fixed words in v0.1 | B-06 |
| Q-09 | Plan for Devanagari keywords? | Not in v0.1 | B-06 |
| Q-10 | Licence | — | Later |
| Q-11 | Continuous integration | — | Later |
| Q-12 | Real-hardware target | — | Later |
| Q-14 | Commit `4b2edd6` on `main` used a different email address. Leave it, or rewrite it (needs a force-push)? | Leave it | B-03 |
| Q-15 | Should `agent/*` and `gate/*` branches be pushed to GitHub? | Keep them local | Later |
| Q-16 | Accept 1 TiB as the provisional upper bound of managed physical memory? | Yes | B-08 |
| Q-17 | Commit Codex's review report and evidence on `agent/codex`? Remove the stray `Claude outputs` folder from `claude-worktree`? | Commit the report; remove the folder | — |

Resolved: Q-01 (O-01), Q-02 (O-02), Q-05 (O-06), Q-06 (O-05), Q-13 (the owner created `review/architecture-v0.1`; what remains is B-04).

---

## 12. Change history

| Date | Change | By |
|---|---|---|
| 2026-11-12 | Draft v0.1 created | Architect |
| 2026-11-12 | Revision 2: owner decisions O-01…O-12; real layout and branches; version record; blockers | Architect |
| 2026-11-12 | Revision 3: response to the independent review (AR-01…AR-24); owner decisions O-13…O-21; gates, gate procedure, runner rules, allocator and test-finalisation contracts, language semantics and diagnostics. Prepared under the owner's document-editing authorisation; uncommitted. | Architect |
