# Agam Computing Ecosystem — Decision Log

| Field | Value |
|---|---|
| Status | **Draft v0.1, revision 5 — NOT APPROVED.** A focused document-only correction after the third independent review (§8.6). The three partly resolved findings (AR-14, AR-27, AR-31) and AR-34…AR-39 are *addressed in draft — pending independent re-review* (§10.3). No finding is closed, no blocker is cleared, and no gate is open. |
| Date | 2026-11-13 (revision 5) |
| Rule | This file is the only place where a decision's status changes. Only the project owner can approve. |

## How to read this file

| Prefix | Meaning | Who can change it |
|---|---|---|
| **C-nn** | **Confirmed** — stated by the owner in the original brief | Owner |
| **O-nn** | **Owner decision, provisional** — binding for these drafts; the owner may revise it | Owner |
| **P-nn** | **Proposal** — the architect's recommendation, with its status | Owner approves, rejects or amends |
| **Q-nn** | **Open question** | Owner, or evidence |
| **B-nn** | **Blocker** — must be cleared before the gate it names opens (§9) | Cleared with evidence, by the party named |
| **AR-nn** | **Review finding** from an independent review (§10) | Closed only by independent re-review |

---

## 1. Gates — all closed

Revisions 3, 4 and 5 are document revisions only. Editing these documents under the owner's instructions authorises no implementation, no test code, no harness code and no checker code.

| Gate | Permits | Opens when |
|---|---|---|
| **G-BOOT** | Only the bootstrap work listed in `ROADMAP.md` §2.2: root metadata and the empty `agam-dev/gates/ACCEPTED` file, environment check, gate tools (ownership checker, inventory checker, manifest capture), runner scaffolding with self-tests, Limine pin files and fetch script, Makefile skeleton, reports. No kernel, interpreter, kernel-test or language-test file. | B-07 cleared by re-review; B-04 cleared; then the owner says so. **No evidence produced by the bootstrap itself is required**, so there is no cycle (AR-01). |
| **G-AOS** | Kernel code and kernel tests (K1–K5) | M0 accepted (this clears B-01); B-02 disposition recorded; B-05 cleared **for the AOS track**; B-08 cleared; the owner explicitly approves the AOS documents |
| **G-LANG** | Interpreter code and language tests (L1–L5) | M0 accepted; B-05 cleared **for the language track**; B-06, B-09 and B-10 cleared; the owner explicitly approves the AgamLang specification |

Opening one gate never opens another. **Committing is not approving.** A document revision committed on `review/architecture-v0.1`, and review reports, evidence or proposals committed on `agent/codex`, are records. Such a commit approves no document, closes no finding, clears no blocker, opens no gate and authorises no implementation, test, harness or checker code. **B-05 is judged per track (O-23).** For G-AOS it requires that no Critical or Major finding is open against the AOS track; for G-LANG, against the language track. A finding belongs to the track of the documents it concerns — process, AOS or language — as stated by the review that raises it and recorded in §10; a finding that concerns more than one track counts for each. Process findings act through B-07 and M0, which both tracks need. AOS may therefore proceed while AgamLang wording or language findings are pending, but nothing AOS-specific is relaxed: G-AOS still needs M0 accepted, the B-02 disposition, B-05 for its track, B-08 and the owner's explicit approval. Which findings block which gate: AR-01…04, AR-15, AR-18, AR-25…27, AR-34, AR-35, AR-37…39 → G-BOOT (through B-07); AR-05…07, AR-16, AR-17, AR-19, AR-21, AR-28, AR-33 → G-AOS (B-08); AR-08…13, AR-22, AR-29, AR-30 → G-LANG (B-09); AR-14, AR-31, AR-32, AR-36 → G-LANG (B-10).

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

### 3.3 Given on 2026-11-12/13 for revision 4, after the second independent review

| ID | Decision | Where applied |
|---|---|---|
| O-22 | **Revision 4 is a document-only correction pass.** Resolve the partly resolved findings and AR-25…AR-33; keep every gate closed; do not clear B-05; preserve the accepted Fusion decisions; where the review exposes an ambiguity that needs an owner decision, **present the options instead of inventing a policy**. | Options are in §11 (Q-17…Q-23). Where a definite text was needed for the re-review, the drafted option is marked provisional. |

### 3.4 Given on 2026-11-13: final decisions on the options of revision 4

These answer Q-17…Q-23. They change rules in the documents; they open no gate and clear no blocker.

| ID | Decision | Answers | Where applied |
|---|---|---|---|
| O-23 | **B-05 is scoped per track.** AOS may progress independently of pending AgamLang wording, but every AOS-specific gate condition and blocker stays enforced. | Q-18 | §1, §9; `ARCHITECTURE.md` §10; `ROADMAP.md` K1 and L1 prerequisites; the review brief |
| O-24 | A text literal over the limit is the lexical error E0108. The list literal limit is checked before any item is evaluated. | Q-19 | `AGAMLANG_SPEC.md` §4.5, §7.1; P-44 |
| O-25 | `MAX_NESTING` = 100 and `MAX_COMPARE` = 1,000,000 are kept. Exceeding a comparison limit must produce the specified diagnostic (E0408) and never silently return false. | Q-20 | `AGAMLANG_SPEC.md` §7.1, §7.5, Appendix B.3, C.11; P-45 |
| O-26 | Verification tools are read by **both** the architect and Claude Code. Independent review by a party other than the author remains mandatory; neither reading replaces the other. | Q-21 | `TEST_PLAN.md` RV-M0-01, §8.1, §8.2, §8.6; `ROADMAP.md` §2.2 (M0-C3), §2.3, R-28; `ARCHITECTURE.md` §6.4; `CLAUDE.md` §7; `AGENTS.md` §3 |
| O-27 | Trusted verification tools are stored outside all Git worktrees: `D:\Agam-Computing\.gate-tools\<BASE>\`. | Q-22 | `TEST_PLAN.md` §8.1; `ARCHITECTURE.md` §4 |
| O-28 | An exact syntactic nesting limit of 100 that counts nested blocks and grouping constructs. A binary operator in a flat expression is not another level. Diagnostic E0214. P-47 was revised to this before it was applied (§6.1). | Q-23 | `AGAMLANG_SPEC.md` §5.2, §5.3, §7.1, §7.10, §8.6, Appendix A, B.3, C.11, §12; `TEST_PLAN.md` T-L2-03, T-L3-01; `CLAUDE.md` §9 |
| O-29 | The existing Codex reports and the Claude output copies are preserved. Nothing is deleted or relocated. | Q-17 | Nothing was changed. How the gate rules are met is decided by O-31 (§3.6). |

### 3.5 Given on 2026-11-13 for revision 5, after the third independent review

| ID | Decision | Where applied |
|---|---|---|
| O-30 | **Revision 5 is a focused document-only correction pass.** Address the remainders of AR-14, AR-27 and AR-31 and the new findings AR-34…AR-39; preserve the AOS architecture and every approved owner decision unless a finding requires a change; keep every gate closed and B-05 uncleared; do not modify the reviews or their evidence. | §10.3. `AOS_KERNEL_SPEC.md` is byte-identical to revision 4. No owner decision O-01…O-29 was changed. |
| O-31 | **Housekeeping plan for the preserved material (answers Q-17).** The Codex reports, their evidence and the proposals are preserved and committed on `agent/codex`, after a check for sensitive content. The `Claude outputs` copies are preserved locally and are not deleted. | §3.6 (procedure); `TEST_PLAN.md` §4.9; `ROADMAP.md` §2.2 (M0-A1, M0-X6); `AGENTS.md` §3, §4; `CLAUDE.md` §3. Q-17 is closed. |

### 3.6 Housekeeping plan for the preserved material (O-31)

**Scope.** In `codex-worktree`, under `agam-dev/reports/codex/`: the review reports `ARCHITECTURE_REVIEW.md`, `ARCHITECTURE_REVIEW_R3.md` and `ARCHITECTURE_REVIEW_R4.md`, the three evidence folders, and proposals 001–003 — 51 files on 2026-11-13, of which 37 contain CRLF and 5 contain NUL bytes. Later review reports and evidence folders follow the same plan. In `claude-worktree`: the folder `Claude outputs`.

**Codex material — preserve, check, commit.**

1. **Attribute lines first.** The `-text` lines of `ROADMAP.md` §2.2 must be in `.gitattributes` on `agent/codex` before any of these files is staged. The owner commits them and brings that commit to `agent/codex`.
2. **Check for sensitive content, read-only.** Before staging, the files are searched for: credentials, tokens, keys and passwords; personal data other than the commit identity of O-18 — other e-mail addresses, account names, absolute paths that reveal a user name, machine names; and material copied from outside the project that may not be redistributed. The result goes to the owner, who decides file by file.
3. **Nothing is edited.** A file that must not be published is left untracked and stays cited by hash. It is never redacted or rewritten: a changed file no longer matches its cited SHA-256, and the reviews must stay as their author wrote them.
4. **Byte check.** For each file to be committed, `git check-attr text -- <file>` reports `unset`, and `git hash-object <file>` prints the same ID as `git hash-object --no-filters <file>`.
5. **Commit** on `agent/codex`, in `codex-worktree`, with Git for Windows, under the identity of O-18 and with no attribution text (`AGENTS.md` §4), and only when the owner asks for it. The commit contains files of the byte-exact set and nothing else.
6. **Afterwards** each report's SHA-256 still equals the value recorded in §8.5 and §8.6.

**`Claude outputs` — preserve locally.** The folder stays where it is, untracked. `.gitignore` names it (`ROADMAP.md` M0-A1), so that `claude-worktree` reports a clean status when a tip is submitted. It is never deleted, moved, edited or committed, and it must never be copied into the integration worktree, where the runner would reject it (`TEST_PLAN.md` §4.6).

**What this plan is not.** Committing review material is record-keeping. It approves no document, closes no finding, clears no blocker, opens no gate and authorises no implementation. Whether `agent/codex` is pushed is a separate decision of the owner (Q-15).

**Preliminary scan.** On 2026-11-13 the architect searched the 51 files read-only (388,204 bytes). No credential-like string was found. Found: the commit e-mail address of O-18; one e-mail address that comes from upstream Limine metadata; and the Windows user-profile path of the reviews' scratch directories, which contains an account name. Whether these may be published is the owner's decision. This scan does not replace step 2.

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
| P-19 | Gate procedure of `TEST_PLAN.md` §8: fixed baseline; raw manifests with modes and blob IDs; **checkers and inventory exported from the baseline to a location outside every worktree**; record-for-record merge verification; live-ref checks before capture, after the run and before promotion; verified record-commit delta | Proposed; implements O-17; amended for AR-25, AR-27 |
| P-41 | Bootstrap scope of `ROADMAP.md` §2.2, including runner scaffolding, gate tools and self-tests, with the M0 trust exception of `TEST_PLAN.md` §8.6 | Proposed; implements O-21; amended for AR-25 |
| P-42 | The required-test inventory is part of the architecture documents (`TEST_PLAN.md` Appendix A); the candidate catalogue and summary are judged against it by a trusted tool | Proposed (AR-26) |
| P-49 | Record verification with explicit content inputs: the judged directory and the old `ACCEPTED` content, each tied to a blob ID of the raw record; exact record file set; the archived summary judged again | Proposed (AR-34) |
| P-50 | Bytecode caches and build output are inputs: removed, and the interpreter settings fixed, by system tools before any candidate code runs; no cache is a permitted output | Proposed (AR-35) |
| P-52 | `agam-dev/gates/ACCEPTED` exists from the pre-agent baseline as an empty file; a missing file is an error | Proposed (AR-37); the alternative was to read a missing file as empty for the first M0 attempt only |
| P-53 | Two sets of tracked files — active text (LF) and byte-exact (`-text`: gate records and archived review material) — with the `.gitattributes` content of `ROADMAP.md` §2.2 | Proposed (AR-39) |

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
| P-43 | One ending guard for normal completion, panic and every arriving vector, checked before any output (`AOS_KERNEL_SPEC.md` §12.4) | Proposed (AR-28) |

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
| P-35 | Source metadata (`span`, `origin`, `lang_counts`) outside node equality and default JSON; E0900 tested by internal injection | Proposed (AR-14); required by O-21 |
| P-44 | Literal limits: E0107 for numeric literals only; an over-long text literal is the lexical error E0108; an over-long list literal is E0408 before any item is evaluated | Approved by the owner (O-24) |
| P-45 | Display and comparison limits are defined on values (`depth`, expanded `count`), not on a traversal; the cycle rule uses no limit | Approved by the owner (O-25) |
| P-46 | Diagnostics: per-row variant, span and argument matrix (Appendix B), worked examples (Appendix C), node-span rules, and a public API that takes plain Python values | Proposed (AR-31) |
| P-47 | Exact syntactic nesting limit: `MAX_SOURCE_NESTING` = 100, counting blocks and bracket pairs; operators add no level; parse error E0214 (`AGAMLANG_SPEC.md` §5.3) | Approved as revised, and applied (O-28); history in §6.1 |
| P-48 | Floor for long operator sequences: a statement with up to 1,000 operations must work; beyond that, "works or E0408" (`AGAMLANG_SPEC.md` §7.10) | **Provisional**; alternatives in Q-24 |
| P-51 | Parse errors fixed position by position (`AGAMLANG_SPEC.md` §5.4): E0203 for every `INDENT` that does not follow a header; E0201 after every complete header; Rule X tied to positions; a new English wording for E0203 | Proposed (AR-36). The matching Hinglish wording is a proposal for the owner (Q-07). |

### 6.1 P-47 — how the source nesting limit reached its approved form

The first form of P-47, recorded on 2026-11-13 as a proposed answer to Q-23, counted every binary operation, prefix operation, call and index as a nesting construct. Its consequence was that an unbracketed chain of more than 100 operators exceeded the limit. The owner rejected that part (O-28): the limit counts nested blocks and grouping constructs only, and a binary operator in a flat expression is not another level.

The approved form is normative in `AGAMLANG_SPEC.md` §5.3 and is not repeated here. In short: the level of a token is the number of enclosing blocks plus the number of enclosing bracket pairs; it may not exceed 100; E0214 is reported at the opening bracket, or at the first token of the first statement of the block, that would be the 101st level.

Because O-28 excludes operators from the count, E0214 does not bound the cost of a very long flat operator sequence. That is covered by a separate, provisional floor (P-48), on which no owner decision exists yet (Q-24).

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

"Expected" is the Ubuntu 24.04 package version on 2026-11-12. "Observed by Codex" is the second review's version inspection (evidence R3-E05, file `environment-r3.json`, which looked up each tool under its plain name only). "Owner-reported" is what the owner stated. None of these replaces the M0 record (T-M0-01, E-M0-04), which does not exist yet.

| Tool | Expected | Observed by Codex (R3-E05) | Owner-reported |
|---|---|---|---|
| Ubuntu under WSL2 | 24.04 LTS | 24.04.5 LTS, kernel `6.18.58.1-microsoft-standard-WSL2` | 24.04.5 LTS |
| Clang | 18.1.3 | `/usr/bin/clang`, 18.1.3 | 18.1.3; `clang-18` in PATH |
| LLD | 18.1.3 | `/usr/bin/ld.lld`, 18.1.3 | 18.1.3; `ld.lld-18` in PATH |
| LLVM tools | 18.1.3 | `llvm-objdump`, `llvm-readelf`, `llvm-nm` **not found** under those unversioned names | `llvm-objdump-18`, `llvm-readelf-18`, `llvm-nm-18`, `llvm-objcopy-18`, `llvm-ar-18` in PATH (2026-11-13) |
| GNU Make | 4.3 | 4.3 | 4.3 |
| QEMU | 8.2.2 | 8.2.2 (`1:8.2.2+ds-0ubuntu1.18`) | 8.2.2 |
| GDB | 15.1 | 15.1 | 15.1 |
| xorriso | 1.5.6 | 1.5.6 | — |
| Host C compiler (`cc`) | GCC 13 series | 13.3.0 | — |
| Python | 3.12.3 | `/usr/bin/python3`, 3.12.3 | — |
| curl, tar, xz | Any | curl 8.5.0, GNU tar 1.35, XZ Utils 5.4.5 | — |
| venv and pinned pytest | Chosen at M0 | Not established | — |
| OVMF (optional) | 2024.02 | Not established | — |
| Git | Git for Windows | 2.52.0.windows.1 | — |

**Tool names.** The two observations agree once names are considered: the LLVM binary utilities are installed under versioned names. The specifications therefore accept either form for every LLVM tool (`clang`/`clang-18`, `ld.lld`/`ld.lld-18`, `llvm-objdump`/`llvm-objdump-18`, and so on); `check_env.sh` resolves each one and the Makefile takes the resolved names as variables (`AOS_KERNEL_SPEC.md` §2, §11.1).

### 8.3 What the architect ran (scratch sandboxes, not the owner's WSL, not project code)

| Check | Result |
|---|---|
| Throwaway C file including the pinned `limine.h`, compiled with the C flag set, Clang 18.1.3 | Exit 0, zero warnings, no `xmm`/`ymm`/`st` operands. Not linked; no assembly. |
| Hand-assembled boot sectors in QEMU 8.2.2 with `isa-debug-exit` and `-no-reboot` | Guest `0x10` → host 33; `0x11` → 35; `0x00` → 1; no exit call → timeout; guest reset → 0 |
| Limine `CONFIG.md`, `USAGE.md`, protocol specification at the pinned commits | Option names, ISO command and entry-state statements match the kernel specification |
| Second download of `limine-binary.tar.xz` (2026-11-12, sandbox on the owner's computer) | 906,108 bytes; SHA-256 `939f670e…0103`; 16 entries, identical to Codex's listing |
| Bytecode-cache fixtures (2026-11-13, cloud sandbox; CPython **3.12.3** and 3.13.18; pytest 9.1.1 on 3.13.18 only — not the owner's WSL, not project code) | On both versions: a timestamp cache with matching size and modification time, and an unchecked hash-based cache with a *different* modification time, are executed in place of the source, with and without `-B`; a checked hash-based cache is not. With `-B -X pycache_prefix=<new empty directory>`, or with the two environment variables, the source is executed and nothing is written. With `-E` the variables are ignored and the stale caches run again; the command-line options still work with `-E`. A sourceless `.pyc` beside the sources is importable even with a prefix. `pytest`: a stale rewritten test cache makes a test whose source now fails exit 0, by default and with `-B` alone; with the prefix the test fails as its source says; a run leaves no file in the tree only when `-p no:cacheprovider` is also given. |
| Scratch Git repository (2026-11-13, cloud sandbox, Git 2.43.0 on Linux — not Git for Windows) with the `.gitattributes` content of `ROADMAP.md` §2.2, also with per-extension lines placed before the `-text` lines | `git check-attr text` reports `unset` for a file in an evidence folder, a nested raw file, a review report, proposal 003 and `ACCEPTED`, and not for proposal 004 or a document. CRLF and NUL files of the byte-exact set are stored unchanged (`git hash-object --no-filters` equals the index blob); active text files are normalised. `git ls-files --eol` shows `attr/-text` for the byte-exact files. An empty `ACCEPTED` has the blob `e69de29b…`; after appending `M0` and LF, the blob recomputed from the exported old content plus that line equals the new blob of the raw record; `git rev-list --parents -n 1` prints the commit and its single parent. The two `find` commands of `TEST_PLAN.md` §8.2 step 8a list exactly the caches and the build output outside `cache/`, and skip `.git`. |

### 8.4 Repository state on 2026-11-13, before revision 5 (read from the repository's own files; no git command was run by the architect)

| Item | Value |
|---|---|
| Remote | `origin` → `https://github.com/Agamsma/Agam-Computing` |
| `main` | `4b2edd6e7c67c0dd167aec3e2d7e2004c86cf48d` — tracks only `README.md` |
| `review/architecture-v0.1` | `cbf2cffdc6f34307c70adddcc887907e1ec092e0` — **revision 4**, committed by the owner (parent `fd2670d…`, revision 3); checked out in `D:/Agam-Computing/ecosystem` |
| `agent/codex` | `fd2670d7e9c664c8b1b3cd14dfee3c46d4a45299` (revision 3); checked out in `codex-worktree` |
| `agent/claude` | `4b2edd6…`; checked out in `claude-worktree`; does not contain the documents |
| Working tree of `ecosystem` before revision 5 | The nine documents were byte-identical to commit `cbf2cff` (blob IDs recomputed without Git). `README.md` and `.gitattributes` differ from their blobs by line endings only (CRLF in the working copy). No untracked file. |
| `.gitattributes` | New in `cbf2cff`, added by the owner: `* text=auto`, then `eol=lf` for `*.md`, `*.sh`, `*.py`, `*.c`, `*.h`, `*.S`, `*.asm`, `*.ld`, `Makefile`, `*.mk`. It has no `-text` line and no general `eol=lf`; `ROADMAP.md` §2.2 states what M0-A1 must change. The architect did not edit it. |
| Revision 5 | Uncommitted changes to eight documents in the `ecosystem` working folder; `AOS_KERNEL_SPEC.md` unchanged |
| Commit identity | `cbf2cff`, `fd2670d` and `349c48a` carry `Agam Sharma <rs3543450@gmail.com>` as author and committer (O-18), with no attribution text. `4b2edd6` used a different address (Q-14). |
| Untracked items | The three review reports, their three evidence folders and proposals 001–003 in `codex-worktree`; a folder `Claude outputs` with two older document copies in `claude-worktree`. All preserved (O-29); plan in §3.6 (O-31). |

### 8.5 Independent review record

| Item | First review | Second review |
|---|---|---|
| Subject | Draft v0.1 revision 2 | Draft v0.1 revision 3 |
| Reviewed commit | `349c48a46d49cfac725b687868173d9423930309` | `fd2670d7e9c664c8b1b3cd14dfee3c46d4a45299` |
| Report (in `codex-worktree`) | `agam-dev/reports/codex/ARCHITECTURE_REVIEW.md` | `agam-dev/reports/codex/ARCHITECTURE_REVIEW_R3.md` |
| Report SHA-256 | `b3f923ecaec34b9a2a318a37cd0ed3235627479618b4bee81ce5ad2cefeb86a9` | `a1b51c9a8fb2f08a05b8a52dd6c695ced74e937c33952e06304e7599629eaab7` (computed by the architect) |
| Supporting material | `architecture-review-evidence/`, proposal 001 | `architecture-review-r3-evidence/`, proposal 002 |
| Findings | 0 Critical, 14 Major, 8 Minor, 2 Informational | Of AR-01…24: 19 Resolved, 5 Partially Resolved (document level). New: 0 Critical, 7 Major, 2 Minor (AR-25…33) |
| Verdict | Not ready | Not ready for owner approval. Process: ready after fixes. AOS: ready after a focused fix and prerequisites. Language: not ready. |
| Tracking | Untracked; cited by reviewed commit and file hash; to be committed under O-31 (§3.6) | Untracked; cited the same way |

Neither report nor any evidence file was modified by revision 3 or revision 4. "Resolved" in the second review is a **document-level** disposition: it does not mean any implementation or required runtime test exists or passed.

### 8.6 Third independent review

| Item | Value |
|---|---|
| Subject | Draft v0.1 revision 4, commit `cbf2cffdc6f34307c70adddcc887907e1ec092e0` |
| Report (in `codex-worktree`) | `agam-dev/reports/codex/ARCHITECTURE_REVIEW_R4.md` |
| Report SHA-256 | `14f0fe9884b8a5fc17a078fac818886c76bba0129df664e7c34897b038bde220` (computed by the architect) |
| Supporting material | `architecture-review-r4-evidence/` (13 files), proposal 003 |
| Findings re-reviewed | 14: **11 Resolved, 3 Partially Resolved** (AR-14, AR-27, AR-31), at specification level. None of the 19 findings resolved earlier was re-opened. |
| New findings | 0 Critical, 3 Major (AR-34, AR-35 process; AR-36 language), 3 Minor (AR-37…39 process) |
| Verdicts | Whole architecture: not ready. Process: not ready. **AOS: ready for scoped owner consideration** — no AOS-specific Critical or Major finding. Language: not ready. |
| Reviewer environment | Git for Windows 2.52.0.windows.1 and Windows CPython 3.13.3; no WSL command; no project tool exists to run |
| Tracking | Untracked; cited by reviewed commit and file hash |

The report, its evidence and proposal 003 were read in full and were not modified. As before, "Resolved" is a **document-level** disposition: no implementation-level `T-`, `E-` or `RV-` item has been run by anyone.

---

## 9. Blockers

No blocker is cleared by this revision. The second review recommends keeping every one open.

| ID | Blocker | State after revision 4 | Cleared by | Gate |
|---|---|---|---|---|
| B-01 | Toolchain evidence on the owner's machine | Versions observed by Codex and reported by the owner (§8.2); LLVM utilities present under versioned names; venv, pytest and optional OVMF not established; no M0 record | M0 accepted (T-M0-01, E-M0-04) | G-AOS, G-LANG |
| B-02 | Independent verification of the Limine pin (condition of O-03) | Identity and pairing evidence exists and was re-audited by the second review (R3-E06). Signature and runtime checks are separate and not performed. | **The owner records the disposition** (E-M0-06) | G-AOS |
| B-03 | Git identity and attribution | Identity recorded (O-18) and seen on both document commits. Still to show: attribution switched off in each tool (E-M0-02). Q-14 open. | E-M0-02 | M0 acceptance |
| B-04 | Approved documents on `main` | `main` is still `4b2edd6`. Revision 4 is on `review/architecture-v0.1` (`cbf2cff`); revision 5 is uncommitted. | The owner advances `main` to the approved document commit | G-BOOT |
| B-05 | Independent architecture review with no Critical or Major finding open, **judged per track** (O-23) | Third review. **AOS track**: AR-05/28 and AR-17/33 assessed Resolved; no AOS-specific Critical or Major finding is open; the reviewer supports the owner clearing the review portion for AOS. **Language track**: AR-36 (Major) open, with the remainders of AR-14 and AR-31; addressed in revision 5. **Not cleared for either track; cannot be cleared by the architect.** Process findings act through B-07. | The owner, for the AOS track, on the third review; re-review of revision 5, then the owner, for the language track | G-AOS (AOS track); G-LANG (language track) |
| B-06 | Hinglish wording: keyword table, built-in names, message catalogue and Appendix B.2 words (Q-07), output words (Q-08), Devanagari (Q-09) | Complete proposals prepared; only structural parity was checked by the review; none approved | The owner's review | G-LANG |
| B-07 | Process contract (AR-01…04, AR-15, AR-18, AR-25…27, AR-34, AR-35, AR-37…39) | AR-02, AR-25 and AR-26 assessed Resolved at document level by the third review. The remainder of AR-27 is AR-34 and AR-35 (Major); AR-37…39 are Minor. All addressed in revision 5 — pending independent re-review. The housekeeping plan is decided (O-31) and not yet carried out. | Re-review | G-BOOT |
| B-08 | Kernel test and allocator contract (AR-05…07, AR-16, AR-17, AR-19, AR-21, AR-28, AR-33) | Every finding assessed Resolved at document level by the third review; the reviewer regards the review portion as resolved. Not cleared: Q-16 and the owner's approval of the AOS documents are outstanding, and all runtime evidence is later work. | The owner (Q-16, approval) | G-AOS |
| B-09 | Independent language oracle and closed semantics (AR-08…13, AR-22, AR-29, AR-30) | Every finding assessed Resolved at document level by the third review. The floor for long operator sequences remains provisional (Q-24). Not cleared. | The owner (Q-24) | G-LANG |
| B-10 | Diagnostic contract (AR-14, AR-31, AR-32, AR-36) | AR-32 assessed Resolved. AR-14 and AR-31 remain Partially Resolved through AR-36 (Major), addressed in revision 5 — pending independent re-review. The Hinglish half also needs B-06, now including the changed E0203 row. | Re-review; owner (B-06) | G-LANG |

---

## 10. Finding-resolution table (all independent reviews)

### 10.1 Findings of the first review (AR-01…AR-24), as resolved in revision 3

The second independent review gave each of these a **document-level** disposition, shown in the next table. The architect closes nothing: "Resolved" is the reviewer's assessment of the text, and every implementation-level item named in the last-but-one column of the detailed table remains `UNVERIFIED` until its gate opens and it is run.

| Disposition by the second review | Findings |
|---|---|
| Resolved (document level) | AR-01, AR-03, AR-04, AR-06, AR-07, AR-08, AR-09, AR-10, AR-12, AR-13, AR-15, AR-16, AR-18, AR-19, AR-20, AR-21, AR-22, AR-23, AR-24 |
| Partially Resolved — remainder carried into §10.2 | AR-02 (→ AR-25, AR-27), AR-05 (→ AR-28), AR-11 (→ AR-29), AR-14 (→ AR-31), AR-17 (→ AR-33) |

The detailed table below is the revision-3 record, kept as it was written. Where revision 4 renumbered or replaced a section it names — in particular the gate procedure of `TEST_PLAN.md` §8 and the checker's provenance — §10.2 gives the current text.

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

### 10.2 Findings open after the second review, as addressed in revision 4

Status of **every** row: *addressed in draft — pending independent re-review*. "Specification-level verification" is what a re-review can check now, on the documents. "Implementation-level verification" needs code that does not exist and a gate that is not open; it is listed so that nothing is mistaken for done (`TEST_PLAN.md` §2.5).

| Finding | Revised sections (revision 4) | Resolution | Specification-level verification (re-review, now) | Implementation-level verification (after gates) | Remaining owner decision |
|---|---|---|---|---|---|
| AR-02 remainder, **AR-25** (Major) Checker not actually pinned to the baseline | TEST §8.1, §8.2 steps 1, 3, 5, 9, §8.3, §8.6; ARCH §6.4; ROAD §2.2 (M0-X2), §2.3, §2.4, K1 criteria, R-26 | Checkers, capture helper and the inventory are exported byte-exactly from `BASE` to a read-only directory outside every worktree, with blob IDs re-checked; every checker call uses that copy. M0, which has no baseline tools, uses a complete by-hand check plus two independent readings of the new tools, by the architect and by Claude Code (O-26). A weakened-checker control follows at K1. | Walk §8.2: no checker call resolves inside a worktree; the export commands are valid and byte-exact; §8.6 covers modes, blobs, union and capture, not only paths | E-M0-07, E-M0-08, RV-M0-01; T-M0-05; E-K1-02 | None (O-26, O-27) |
| **AR-26** (Major) A required test omitted from the catalogue disappears | TEST §4.1, §4.4, §8.2 steps 7–8, §8.4, Appendix A; ROAD §2.2, §7 | The inventory of test IDs and classes is part of the architecture documents (Appendix A: 91 tests, 85 required). A trusted tool compares the candidate catalogue with it before the run and the summary with it after the run; omissions, duplicates, class changes, moved or unknown IDs fail. | Appendix A agrees with the §5–§6 tables; for each gate, §4.2 plus Appendix A gives one exact required set; try to construct an omission, demotion or substitution that §8.4 would accept | T-M0-04 (j), T-M0-07; every gate's step 7 and 8 | None |
| **AR-27** (Major) Content equality is not freshness; promotion checks incomplete | TEST §4.6, §8.1, §8.2 steps 0, 6, 8, 9, 10, §8.3 (`--verify-record`), §8.5, §8.7; ARCH §6.4; ROAD R-25 | Live refs and a clean integration worktree are checked with Git for Windows before capture, after the run and immediately before promotion. `gate.json` binds `tree.lst` as well as the manifests. Untracked inputs are forbidden outside named output locations. The record commit's delta is checked exactly; tested and promoted SHAs are both recorded. The text no longer claims that blob checks identify a commit or branch. | Check that each of the three points reads live refs; that the permitted-output list is exact; that the record delta rule admits nothing a test reads | T-M0-04 (f), (h), (i); T-M0-05 (`--verify-record`); every gate | None |
| AR-05 remainder, **AR-28** (Major) Faults during finalisation | AOS §7.2, §9.5, §12.4; TEST §4.7 item 5, T-K1-09, RV-K3-03; ROAD K3 criteria | One `ending` state, checked by every event — normal completion, panic, each arriving vector including #BP — before any output; if already set, halt silently. "Exactly one `RESULT`" is now promised only for a completed ending sequence; an interrupted one prints at most one. The harness requires exactly one `RESULT` line plus the expected host status. | Enumerate the event table of §12.4 against §9.5 and §7.2: every event and both states have one outcome; no path can print a second `RESULT` | T-K1-09 (synthetic logs), T-K1-07, T-K1-10, T-K3-10, T-K3-13…16; RV-K3-03 (the interrupted case cannot be provoked from test code) | None |
| AR-11 remainder, **AR-29** (Major) Literal limits lack one error and stage | LANG §4.5, §7.1, §7.3 rule 5, §8.6, Appendix A (E0108), B.3, C.11; TEST T-L1-04, T-L1-05, T-L4-04 | E0107 is numeric literals only. An over-long text literal is the lexical error E0108, decided while scanning, with first-problem order inside a literal. A list literal with too many items is E0408 when evaluated, before any item is evaluated. | The four-row code table of §7.1 leaves no literal or computed value without exactly one code; C.11 boundaries follow from the text | T-L1-04, T-L1-05, T-L4-04 | None (O-24) |
| **AR-30** (Major) Traversal budgets depend on the traversal | LANG §7.1 (`depth`, `count`), §7.5, §7.6, §7.8, §7.10, C.11; TEST T-L4-07, T-L4-08, RV-L4-01 | `MAX_VISITS` is removed. Depth and expanded element count are defined as properties of a value; display and comparison limits are stated on those properties and on the display length; identity decides `a == a` first. The cycle rule uses no limit, so "cycle" and "limit" can never compete. | Root depth, counting unit and shared sub-lists are unambiguous; the §7.5 example arithmetic (count 3,145,726) is right; no outcome depends on order, early exit or memoisation | T-L4-07, T-L4-08; RV-L4-01 | None (O-25) |
| AR-14 remainder, **AR-31** (Major) Golden authors still have to guess | LANG §2 (lines), §6.2, §6.3, §6.5, §8.1–§8.3, §8.5, §8.7, §10, Appendix A, **B**, **C**; TEST T-L2-01, T-L5-02, T-L5-07 | Appendix B fixes variant, span and every argument for each of the 53 rows, including user-function arity, E0100 offset with a byte-order mark, the E0106 span, E0204's expected phrase per parser position and the else-if naming fallback. Diagnostics without a span have a defined rendering. Node spans are defined. The public API takes plain Python values. `lang_counts` is excluded from equality. Appendix C gives eleven exact examples. | For each row of Appendix A there is exactly one row of B.3; every placeholder has a format in B.1; the C examples follow from the rules (carets, counts, spans); try to find a diagnostic a golden author would still have to guess | T-L2-01, T-L5-01…04, T-L5-07; RV-L5-01 | Hinglish words of Appendix A.2 and B.2 (O-20, B-06) |
| **AR-32** (Minor) Shallow-copy tip does not fix an indirect cycle | LANG Appendix A.1, A.2 (E0308) | The tip now asks for a value that does not contain the list, directly or inside another list. | The tip is true for the direct and the indirect case | T-L5-02 | Hinglish wording (O-20) |
| AR-17 remainder, **AR-33** (Minor) Remaining factual and default-field wording | AOS §3.1 (interrupt sources), §3.3 (last paragraph), §4.4 (Reserved/Bad row) | Masking is stated per delivery mode, as in the pinned protocol; the ownership row keeps the unconditional no-touch rule without asserting a fault; requests carry their header-defined identifiers and only other option or reserved fields are zero. | Compare the three passages with the protocol text at commit `6e85ddd…` | RV-K1-01 | None |

Also changed in revision 4, from remarks in the second review that were not findings: the successful `pmm_free_contig` now states the counter update (AOS §10.4); E0305 is split into "a list" and "a list or text" (LANG §8.6, Appendix A); LLVM tools may be found under versioned names (§8.2; AOS §2, §11.1); the source nesting limit is exact (LANG §5.3; O-28), with a provisional floor for long operator sequences (LANG §7.10; Q-24).

### 10.3 Findings open after the third review, as addressed in revision 5

The third independent review re-examined the fourteen findings of §10.2 and gave these **document-level** dispositions. The architect closes nothing.

| Disposition by the third review | Findings |
|---|---|
| Resolved (specification level) | AR-02, AR-05, AR-11, AR-17, AR-25, AR-26, AR-28, AR-29, AR-30, AR-32, AR-33 |
| Partially Resolved — remainder carried below | AR-14 (→ AR-36), AR-27 (→ AR-34, AR-35), AR-31 (→ AR-36) |

Status of **every** row below: *addressed in draft — pending independent re-review*. The two verification columns have the meaning of §10.2: the first is what a re-review can check on the documents now; the second needs code that does not exist and a gate that is not open.

| Finding | Revised sections (revision 5) | Resolution | Specification-level verification (re-review, now) | Implementation-level verification (after gates) | Remaining owner decision |
|---|---|---|---|---|---|
| AR-27 remainder, **AR-34** (Major) Record verification needs content, not only blob IDs | TEST §8.2 steps 8c, 8d, 9; §8.3 (record rules 1–4 and the paragraph on the tested and promoted commit); §8.6 item 5; T-M0-05; E-M0-09; ROAD R-30; ARCH §6.4 | `--verify-record` now receives the judged directory and the old `ACCEPTED` content. Each is tied to a blob ID of the raw record by recomputing the blob ID, without Git. The record file set is exact, including the architect's reading exactly when it is required. The new `ACCEPTED` is proved as old content plus one line, so earlier lines cannot change. The archived summary is judged again. The text no longer says that the record delta touches only unread files: `ACCEPTED` is named as the one read file, with the reason its single appended line is safe. | Every input of the record check is named in step 9 and tied to the raw record; rule 4 proves the append from the old content alone; no file can be archived that was not judged; a wrong prefix, a second line, a missing or substituted artefact each violate a stated rule | T-M0-05; E-M0-09 (M0, by hand); step 9 of every gate | None |
| AR-27 remainder, **AR-35** (Major) Permitted caches can supply executable unbound input | TEST §4.6 checks 3–6 and the two paragraphs after them; §8.1; §8.2 steps 8a–8c; T-M0-04 (i), (k), (l), (n); E-M0-10; §9; ROAD §2.2 (M0-A1), §4, R-29; ARCH §6.1, §6.4; CLAUDE §4, §8; AGENTS §3 | Bytecode caches are no longer a permitted output location. Before any candidate Python code is imported, the owner removes caches and leftover build output with system tools, switches bytecode writing off and points the cache prefix at a new empty directory outside the tree; after the run the same search must again find nothing. The runner repeats the checks as a second line and starts `pytest` without its cache plugin. The download cache is limited to pinned, re-verified files. | Walk step 8 for a planted cache of the runner, a harness module, a test and a language module: none can be read, written or left behind; the sourceless `.pyc` case is covered by the absence check; compare with Python's documented cache rules; the residual (a tool that removes the settings) is stated in TEST §9 | T-M0-04 (k), (l), (n); E-M0-10; steps 8a and 8c of every gate | None |
| AR-14 and AR-31 remainders, **AR-36** (Major) The diagnostic matrix omits parser positions | LANG §5.2, §5.4, §6.5 (reference), §8.6 (E0201, E0203), Appendix A (E0203, both languages), B.1 (Rules F and X), B.3 (E0201, E0203, E0204, E0205a), C.12–C.15; TEST T-L2-03, T-L5-02; CLAUDE §9; AGENTS §6 | §5.4 lists every parser position with the tokens accepted and the one diagnostic for any other token. An `INDENT` that does not follow a header is always E0203, at any depth, and never E0214. E0201 applies after every complete header, with "complete" defined for `else`, `repeat … times` and function definitions as well. Rule X is tied to the rows; of the structural tokens only `NEWLINE` can be the unexpected token. Both counterexamples of the review are worked exactly (C.12, C.13), with a third for a definition (C.14) and a table of which error wins (C.15). The English E0203 message was reworded because the old one was untrue inside a block. | For each row of §5.4 and each kind of token there is exactly one outcome; the lexer guarantees quoted there follow from §4.2; C.12–C.14 follow from the rendering rules character by character; C.15 follows from §5.2 and §5.4; look for a position or token still undecided | T-L2-03, T-L5-02; RV-L5-01 | Hinglish wording of the changed E0203 row (Q-07, B-06) |
| **AR-37** (Minor) The first `ACCEPTED` input has no initial state | TEST §4.2, §8.3 rule 4, T-M0-04 (m), E-M0-09; ROAD §2.2 (M0-A1); ARCH §6.1; DEC §1 | The owner creates `ACCEPTED` as an empty file in the pre-agent baseline. An empty file means no accepted milestone; a missing file is an error at every gate, the first included. The record commit therefore always modifies it and never creates it. | The file exists in every `BASE`; no rule reads a missing file as empty; no agent path contains it | T-M0-04 (m); E-M0-09 | None; the owner creates the file at M0-A1 |
| **AR-38** (Minor) An ignore rule cannot satisfy the binding rule | TEST §4.6 (check 3 and "Which worktree"), §8.2 step 0, T-M0-04 (i); DEC §3.6 (O-31) | The two rules are separated: a clean `git status` in an agent worktree when a tip is reported, and no unlisted file in the integration worktree during a gate run. An ignore rule can meet the first and never the second. The approved plan (O-31) uses each where it fits: the Codex material is committed and so becomes tracked and bound; `Claude outputs` is ignored in its own worktree and never enters the integration worktree. | No sentence lets an ignored file pass check 3; the plan of §3.6 is consistent with both rules and with O-29 | T-M0-04 (i) | None (O-31) |
| **AR-39** (Minor) LF acceptance needs an exception for archived and binary evidence | TEST §4.9, T-M0-03, E-M0-03; ROAD §2.2 (`.gitattributes`), R-03; AGENTS §3; DEC §3.6, §8.4 | Tracked files are in one of two sets. The byte-exact set — gate records and the archived review material — is marked `-text`, is never converted and keeps its cited hashes. The LF rules apply to the active text set only. Blob binding applies to both. The evidence may be committed only after the `-text` lines are in the branch. | The four patterns cover every archived file and nothing active; the last-match rule of `.gitattributes` gives `-text` to exactly those paths; T-M0-03 and E-M0-03 no longer demand LF of a binary manifest | T-M0-03; E-M0-03 | None (O-31); the owner commits the `.gitattributes` change |

Also corrected, from an editorial remark of the third review: `AGAMLANG_SPEC.md` §6.5 now points to Appendix C.10 for the node-span example.

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
| Q-24 | Long flat operator sequences. O-28 excludes operators from the nesting count, so E0214 does not bound them. **(a)** A statement with up to 1,000 operations must work; beyond that "works or E0408", not asserted by tests (drafted, provisional). **(b)** Sequences of any length must work. **(c)** An exact limit on the operations of one statement, with its own diagnostic. | (a) for v0.1 | B-09 (minor) |

Resolved: Q-01 (O-01), Q-02 (O-02), Q-05 (O-06), Q-06 (O-05), Q-13 (the owner created `review/architecture-v0.1`; what remains is B-04), Q-18 (O-23), Q-19 (O-24), Q-20 (O-25), Q-21 (O-26), Q-22 (O-27), Q-23 (O-28), Q-17 (O-29, O-31).

---

## 12. Change history

| Date | Change | By |
|---|---|---|
| 2026-11-12 | Draft v0.1 created | Architect |
| 2026-11-12 | Revision 2: owner decisions O-01…O-12; real layout and branches; version record; blockers | Architect |
| 2026-11-12 | Revision 3: response to the first independent review (AR-01…AR-24); owner decisions O-13…O-21. Committed by the owner as `fd2670d`. | Architect |
| 2026-11-13 | Revision 4: document-only correction pass after the second independent review (AR-25…AR-33 and the five partly resolved findings); options Q-17…Q-23 presented; uncommitted. | Architect |
| 2026-11-13 | Revision 4, final correction before commit: T-K5-01…04 defined as four rows; the owner's proposed answers to Q-17…Q-23 recorded as pending final approval (§3.4); P-47 written out (§6.1). No gate, blocker or normative rule changed. | Architect |
| 2026-11-13 | Revision 4, owner decisions applied: O-23…O-29, the final answers to Q-17…Q-23. B-05 judged per track; literal, display and comparison limits approved; two readings of verification tools; trusted-tools location fixed; P-47 revised and applied as §5.3 of the language specification with E0214; Q-24 opened. No gate opened, no blocker cleared. | Architect |
| 2026-11-13 | Revision 4 committed by the owner as `cbf2cff` on `review/architecture-v0.1`, together with a first `.gitattributes`. | Owner |
| 2026-11-13 | Revision 5: focused document-only correction after the third independent review (AR-34…AR-39 and the remainders of AR-14, AR-27, AR-31); proposals P-49…P-53; Q-17 restated; `AOS_KERNEL_SPEC.md` unchanged; uncommitted. No gate opened, no blocker cleared. | Architect |
| 2026-11-13 | Revision 5, final check before commit: cache policy stated precisely (three ways a cache is executed; options on the command line because `-E` and `-I` disable the variables); `.gitattributes` content annotated; O-31 recorded and Q-17 closed (§3.6); committing stated not to be approving (§1). | Architect |
