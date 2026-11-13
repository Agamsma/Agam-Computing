# Agam Computing Ecosystem — Architecture

| Field | Value |
|---|---|
| Status | **Draft v0.1, revision 5 — NOT APPROVED.** A focused correction after the third independent review. This document changes for AR-34, AR-35 and AR-37…AR-39 (ownership table, integration flow, evidence record); those findings are *addressed in draft — pending independent re-review* (`DECISIONS.md` §10.3). Nothing here authorises implementation. |
| Date | 2026-11-13 (revision 5) |
| Owner | Project owner (approves); Architect (maintains) |
| Related | `DECISIONS.md`, `ROADMAP.md`, `AOS_KERNEL_SPEC.md`, `AGAMLANG_SPEC.md`, `TEST_PLAN.md`, `CODEX_REVIEW_BRIEF.md`, `../CLAUDE.md`, `../AGENTS.md` |

Every statement here is tracked by ID in `DECISIONS.md`:

- **C-nn — Confirmed** in the original brief.
- **O-nn — Owner decision, provisional** (2026-11-12).
- **P-nn — Proposal** by the architect, with its status.
- **Q-nn — Open question.** **B-nn — Blocker.** **AR-nn — Review finding.**

No AOS or AgamLang code exists. §9 lists what was read or run, and by whom.

---

## 1. Purpose and scope

| Project | Folder | What it is | Phase-1 goal | Priority |
|---|---|---|---|---|
| **Agam OS (AOS)** | `agam-os/` | Original x86-64 operating system with its own kernel | Boot in QEMU, handle CPU exceptions, manage physical memory | 1 |
| **AgamLang** | `agam-lang/` | Beginner-friendly language; English, Hinglish and mixed syntax (Fusion) | Python-hosted tree-walking interpreter with friendly diagnostics | 1 |
| **Agam Developer OS** | `agam-dev/` | AI-assisted development environment for the other two | A documented, enforceable workflow; shared verification tools | 2 (deferred) |

### Explicit non-goals for Phase 1 (C-10)

Networking, GUI, AI integration inside AOS, native AgamLang compilation, running AgamLang on AOS, SMP, user mode, file systems, hardware interrupts and timers (O-04), real-hardware support.

---

## 2. Guiding principles

1. **Minimal and reproducible beats featureful.**
2. **The spec is the arbiter.** Implementation and tests are both written from the spec, independently. **Tests never take their expected values from implementation data**: where the implementation has a table (keywords, messages, error codes), the tests compare it with an independent fixture written by hand from the specification.
3. **Evidence or it did not happen.** Source inspection and arithmetic are labelled as such and are never presented as build, boot or runtime results (`TEST_PLAN.md` §2).
4. **One writer per path** (§6.1), checked from recorded Git data, not from trust.
5. **Pinned inputs**, with identity evidence kept apart from checks that were not performed (`DECISIONS.md` §8.1).
6. **The owner controls `main` and the public history** (O-10, O-11, O-17, O-18).
7. **Required before stretch** (O-09, O-19). A required test that does not run is a failure, not a skip.
8. **Normative means "someone independent depends on it".** Interfaces that tests, the bootloader or users rely on are fixed; internal design is left to the implementer.

---

## 3. System context

```
                    ┌──────────────────────────────────────────────────────┐
                    │ Project owner: approves decisions; owns main, pushes │
                    └───────────────┬──────────────────────────────────────┘
                                    │
                    ┌───────────────▼──────────────────────────────────────┐
                    │ Architect: docs/*.md, CLAUDE.md, AGENTS.md           │
                    └───────┬──────────────────────────────┬───────────────┘
               specs + tasks│                              │specs + tasks
     ┌──────────────────────▼───────────┐   ┌──────────────▼────────────────────┐
     │ Claude Code — implements         │   │ Codex — tests and reviews         │
     │ folder  claude-worktree          │   │ folder  codex-worktree            │
     │ branch  agent/claude             │   │ branch  agent/codex               │
     │ writes  agam-os/ (not tests/)    │   │ writes  agam-os/tests/            │
     │         agam-lang/ (not tests/)  │   │         agam-lang/tests/          │
     │         agam-dev/reports/claude/ │   │         agam-dev/verify/          │
     │                                  │   │         agam-dev/reports/codex/   │
     └──────────────────────┬───────────┘   └──────────────┬────────────────────┘
                            │  merged by the owner (disjoint paths)
                    ┌───────▼──────────────────────────────▼───────────────┐
                    │ folder ecosystem — branch main — verification gate   │
                    └──────────────────────────────────────────────────────┘
```

All three folders are worktrees of one repository (`https://github.com/Agamsma/Agam-Computing`) under `D:/Agam-Computing` (O-12).

In Phase 1, AOS and AgamLang share no code, only the repository, the workflow and the evidence rules. The two tracks can therefore run in parallel.

---

## 4. Repository layout (P-11, built on the owner's folders, O-12)

```
/                                (repository root; same in all three worktrees)
├── README.md  CLAUDE.md  AGENTS.md  VERSION  .gitattributes  .gitignore      Architect
├── docs/                        Architect — architecture documents only
│   ├── ARCHITECTURE.md  DECISIONS.md  ROADMAP.md
│   ├── AOS_KERNEL_SPEC.md  AGAMLANG_SPEC.md  TEST_PLAN.md
│   └── CODEX_REVIEW_BRIEF.md
├── agam-os/                     AOS
│   ├── Makefile                 Claude — build entry points
│   ├── kernel/                  Claude — include/aos/, src/, arch/x86_64/, linker.ld
│   ├── boot/                    Claude — limine.conf
│   ├── third_party/             Claude — pinned inputs (Limine PIN, vendored limine.h)
│   ├── tools/                   Claude — fetch-limine.sh, mkiso.sh
│   ├── tests/                   Codex  — cases/ (in-kernel test cases), harness/, test_*.py
│   └── build/                   untracked — all generated output
├── agam-lang/                   AgamLang
│   ├── pyproject.toml           Claude
│   ├── src/agamlang/            Claude — the interpreter package
│   ├── examples/                Claude — sample .agam programs
│   └── tests/                   Codex  — unit, golden, conformance, fusion, fuzz
└── agam-dev/                    Developer OS (workflow tooling for now)
    ├── verify/                  Codex  — check_env.sh, check_ownership.py, check_inventory.py, gate_manifest.ps1, run.py, catalogue.json
    ├── reports/
    │   ├── claude/              Claude — task reports, evidence, proposals
    │   └── codex/               Codex  — evidence, reviews, proposals
    └── gates/                   Owner  — one record per accepted milestone, and the ACCEPTED list

.gate/                           untracked, ignored — manifests and binding files of a gate in progress
```

Outside every worktree (`D:\Agam-Computing\.gate-tools\<BASE>\`; O-27): the owner's read-only export of the gate tools and of `docs/TEST_PLAN.md` as they exist in the milestone's baseline (`TEST_PLAN.md` §8.1). It is not part of the repository.

Rules that make the layout work:

- **Build → test contract.** `agam-os/Makefile` (Claude) exposes `iso [TEST=<case>]`, `run`, `debug`, `clean`, `test`. The harness (Codex) calls `make -C agam-os iso …` and launches QEMU itself. Neither side edits the other's files; the contract is in `AOS_KERNEL_SPEC.md` §11 and `TEST_PLAN.md` §4.
- **In-kernel test cases belong to Codex.** `agam-os/tests/cases/<case>.c` is written against the public headers in `agam-os/kernel/include/aos/` and compiled into a test kernel by the Makefile.
- **The required-test inventory is a document.** `TEST_PLAN.md` Appendix A lists every test ID with its suite and class. Codex's `catalogue.json` must equal it; an agent cannot add, drop, move or reclassify a test by editing the catalogue (AR-26).
- **`docs/` has one writer.** Agents request spec changes through proposal files in their own `agam-dev/reports/<agent>/proposals/` folder.
- **Empty folders are not stored by Git** (R-23): the project folders appear in a worktree once their first files are committed and merged.
- **Only test-facing interfaces are fixed.** Inside `agam-os/kernel/` and `agam-lang/src/agamlang/` the file split is the implementer's choice, apart from the headers and modules the specifications name as normative.

---

## 5. Component architecture

### 5.1 AOS — modular monolithic kernel (O-08; detail in `AOS_KERNEL_SPEC.md`)

"Modular monolithic" means: one statically linked kernel image running in one address space at one privilege level, internally divided into modules that talk to each other only through the headers in `agam-os/kernel/include/aos/`. There are no loadable modules and no separate server processes. The module boundaries are what make independent testing and later refactoring possible.

```
 QEMU (q35, TCG) → firmware (SeaBIOS; OVMF optional) → Limine 12.9.6 → kernel entry
                                                                           │
   ┌───────────────────────────────────────────────────────────────────────▼──────┐
   │ boot     Limine requests, base-revision check, hand-off validation           │
   │ serial   COM1 driver — initialised first; every diagnostic goes here         │
   │ klib     kprintf, memory functions, panic                                    │
   │ gdt      kernel GDT + TSS with interrupt stacks                              │
   │ idt      256 gates, exception stubs, register dump                           │
   │ pmm      memory-map parsing, bitmap frame allocator                          │
   │ ktest    test hooks, compiled only into test kernels                         │
   └──────────────────────────────────────────────────────────────────────────────┘
   Later, in order: virtual memory → heap → interrupt controllers and timer → scheduler → user mode
```

Serial-first debugging (O-08) means the serial port is working before anything that can fail, all kernel output has a fixed, machine-checkable format, and tests judge the kernel only by its serial output and its exit status.

Key properties for Sprint 1: higher-half kernel at `0xFFFFFFFF80000000`; Limine's page tables are kept; hardware interrupts stay disabled and only CPU exceptions are handled (O-04); early memory ownership is fixed by a table (`AOS_KERNEL_SPEC.md` §4.4).

### 5.2 AgamLang (detail in `AGAMLANG_SPEC.md`)

```
 .agam source (UTF-8; English, Hinglish, or both — always, with no mode switch)
      │
      ▼
 Lexer ── words ──► Keyword normaliser (alias table: surface word/phrase → canonical KW_*)
      │                    ▲ single source of truth: keywords.py
      ▼
 Canonical token stream  (KW_IF is KW_IF whether the user wrote "if" or "agar")
      │
      ▼
 Parser (recursive descent) ──► AST (language-neutral; every node has a source span)
      │
      ▼
 Tree-walking interpreter ──► output
      │
      └──► Diagnostics: error code + span + message catalogue (English / Hinglish)
```

Fusion is the default and only mode (O-07): one lexer, one grammar, one AST, one interpreter. Phrases and mixed else-if forms such as `warna if` are resolved in the lexer's normaliser, which records the user's spelling, span and language for diagnostics; the parser never sees them (O-16). Blocks use Python-style indentation (O-05); lists are 0-based (O-06) and can never contain themselves (O-13); evaluation order follows Python (O-15); numbers and collections have provisional size limits (O-14).

### 5.3 Agam Developer OS (deferred; P-40)

| Stage | Content | When |
|---|---|---|
| D0 | The workflow: `CLAUDE.md`, `AGENTS.md`, path ownership, evidence reports; shared tools in `agam-dev/verify/` | Now |
| D1 | `agamdev` CLI (Python, standard library only) wrapping the existing commands | After AOS Sprint 1 and AgamLang L5 |
| D2 | Dashboard, debugging helpers, AgamLang playground | Later |
| D3 | Agent orchestration and AI-assisted tutoring | Later; needs its own architecture document |

---

## 6. Ownership, branches and Git rules

### 6.1 Path ownership (P-10)

| Path | Sole writer |
|---|---|
| `docs/**`, `CLAUDE.md`, `AGENTS.md`, `README.md`, `VERSION`, `.gitattributes`, `.gitignore` | Architect, committed by the owner |
| `agam-dev/gates/**` | Owner (gate records) |
| `agam-os/**` except `agam-os/tests/**` | Claude Code |
| `agam-lang/**` except `agam-lang/tests/**` | Claude Code |
| `agam-dev/reports/claude/**` | Claude Code |
| `agam-os/tests/**`, `agam-lang/tests/**` | Codex |
| `agam-dev/verify/**`, `agam-dev/reports/codex/**` | Codex |
| `agam-os/build/**`, `.gate/**` | Untracked and ignored outputs; this rule takes precedence over the broader `agam-os/**` rule |
| `__pycache__/`, `.pytest_cache/`, `*.pyc` | Ignored by Git, but **not outputs**: Python can execute them, so none may exist in the integration worktree during a gate run (`TEST_PLAN.md` §4.6) |

`agam-dev/gates/ACCEPTED` exists from the pre-agent baseline as an empty file, created by the owner (`ROADMAP.md` M0-A1). Gate records and archived review evidence are byte-exact files that Git never converts (`TEST_PLAN.md` §4.9).

The two agents' path sets are disjoint. Additions, modifications, deletions and both sides of a rename must all fall inside the writer's set. Every tracked file has mode `100644`; symbolic links and submodules are not allowed.

### 6.2 Branches (O-02, O-17)

| Folder | Branch | Rule |
|---|---|---|
| `D:/Agam-Computing/ecosystem` | `main` — at present `review/architecture-v0.1` is checked out here for the document review | `main` advances only by the owner's fast-forward after an accepted gate or an approved document commit |
| `D:/Agam-Computing/claude-worktree` | `agent/claude` | Starts each milestone with `git merge main` |
| `D:/Agam-Computing/codex-worktree` | `agent/codex` | Starts each milestone with `git merge main` |
| `ecosystem`, during a gate | `gate/<milestone>` | Temporary; created from the milestone's baseline; deleted after the gate |

### 6.3 Git rules (O-01, O-10, O-11, O-18)

- **Git runs only with Git for Windows.** No git command is run inside WSL, and no tool that runs in WSL calls git: from Linux the agent worktrees look prunable (R-02).
- **Before every commit**: `git worktree list`, `git branch --show-current` and `git status` are run and checked.
- **Commit identity** (author and committer): name `Agam Sharma`, email `rs3543450@gmail.com`, as configured in the repository. `Agamsma` is the GitHub username of the account that owns the repository; it is not the commit name. Nobody changes Git configuration on the owner's behalf.
- **No AI attribution anywhere**: no `Co-authored-by` trailers, no "generated with" lines, no tool or model names, no session links.
- **Agents never commit to, merge into, tag or push `main` or a `gate/*` branch, and never push at all.**

### 6.4 Milestone flow and integration (O-17; procedure in `TEST_PLAN.md` §8)

```
 milestone start: BASE = main (recorded)       agents merge BASE into their branches
        │
        ├─► Codex writes tests from the spec        ─► tip TIP_X (exact SHA, clean worktree)
        └─► Claude implements from the spec         ─► tip TIP_C (exact SHA, clean worktree)
        │
 owner, Git for Windows:
        live check: main, agent/claude, agent/codex equal BASE, TIP_C, TIP_X; worktree clean
        TOOLS: gate tools and TEST_PLAN.md exported from BASE to a read-only folder
               outside every worktree; blob IDs re-checked           (M0: by-hand check instead)
        raw manifests BASE→TIP_C and BASE→TIP_X (paths, statuses, modes, blob IDs)
        ownership check of each manifest                              — tool from TOOLS
        a change under agam-dev/verify/ needs two readings (architect, Claude Code) — RV-<milestone>-VP
        gate/<milestone> created from BASE; both tips merged → GATE_HEAD
        merge result compared record for record with the two manifests — tool from TOOLS
        candidate catalogue compared with the trusted inventory       — tool from TOOLS
        │
 WSL2:  system tools first: bytecode caches and build output removed, cache writing off,
        empty cache prefix outside the tree — before any candidate code is imported
        run.py gate <milestone> — refuses to run unless the working tree equals GATE_HEAD
        and holds no cache or unlisted file
        │
 owner: summary compared with the trusted inventory                   — tool from TOOLS
        live check repeated
        │
        ├─ any failure: gate branch deleted, main untouched, new tips, start again
        └─ pass + reviews + owner's acceptance:
               record commit GATE_REC — checked against the judged files: exact file set,
               archived bytes equal judged bytes, ACCEPTED grown by exactly one line;
               the archived summary is judged again
               live check repeated; main fast-forwarded and tagged by the owner
```

The tools that judge a tip are never taken from that tip, and the list of tests that must pass is never taken from it either. The Git-free tree check inside the runner shows that the files tested are the files of `GATE_HEAD`; it does not show that a branch still points there, which is why the live check is made three times (`TEST_PLAN.md` §8.5).

Results are valid only for the exact baseline, tips and merge commit they name; any change makes them void.

---

## 7. Technology stack (exact versions in `DECISIONS.md` §8)

| Area | Choice | Status |
|---|---|---|
| Kernel language | C17 + assembly | C-02 |
| Kernel structure | Modular monolithic | O-08 |
| Bootloader | Limine **12.9.6** | C-03; pin P-02; identity evidence reproduced, disposition pending (B-02) |
| Boot protocol | Limine protocol, **base revision 6** | O-03 (conditional) |
| Protocol header | `limine-protocol` commit `6e85ddd…`, vendored | P-02 |
| Compiler / linker | Clang 18.1.3, LLD 18.1.3, target `x86_64-unknown-none-elf` | C-04; flags P-04; versions observed by Codex and owner-reported |
| LLVM binary utilities | `llvm-objdump`, `llvm-readelf`, `llvm-nm`, `llvm-objcopy`, `llvm-ar` 18.1.3 — under those names or with the `-18` suffix | Owner-reported under the `-18` names (2026-11-13); resolved by `check_env.sh` |
| Assembler | Clang integrated assembler, `.S`, AT&T syntax | P-05 |
| Emulator / debugger | QEMU 8.2.2, GDB 15.1 | C-05 |
| Build | GNU Make 4.3; never invokes git | P-06 |
| Build/test environment | WSL2, Ubuntu 24.04 LTS, working in `/mnt/d/Agam-Computing/…` | O-01 |
| Version control | Git for Windows only | O-01 |
| AgamLang host | CPython ≥ 3.11 (3.12.3 on Ubuntu 24.04), standard library only at run time | P-20 |
| AgamLang tests | `pytest` (development dependency) | P-27 |

Versions are those offered by Ubuntu 24.04 on 2026-11-12. The owner reports Ubuntu 24.04.5 with Clang 18.1.3, LLD 18.1.3, QEMU 8.2.2, GDB 15.1 and Make 4.3 installed, and the second review observed the same versions in WSL (`DECISIONS.md` §8.2). The recorded evidence comes at M0 (B-01).

---

## 8. Cross-cutting concerns

### 8.1 Reproducibility

- Limine binaries come from one exact release asset, verified by SHA-256; `limine.h` is vendored at one commit that the same Limine release itself pins.
- Tool versions are captured by `agam-dev/verify/check_env.sh` into every evidence report.
- QEMU runs with `-accel tcg`, machine `q35`, CPU `qemu64`, one CPU, explicit memory size.
- `limine.conf` sets `timeout: 0` and `randomise_hhdm_base: no`; the kernel still never hard-codes the HHDM base.
- LF line endings are enforced, because files are edited on Windows and built on Linux.

### 8.2 Security posture (Phase 1)

What is protected:

- Kernel segments get minimum permissions through ELF program headers; Limine maps them so and sets `CR0.WP`; the kernel requires NX and checks it at boot.
- `-Werror`; no floating point or vector instructions in kernel code; no red zone.
- The allocator validates every free before changing anything, and panics on an invalid one.
- AgamLang never calls `eval`/`exec` on user text, has no file or network access, and bounds values and call depth.
- Two external inputs, both pinned by hash.
- The debugger port is bound to the loopback interface.

What is **not** protected, deliberately, in Phase 1 (R-20, R-24; finding AR-23):

- Under Limine's page tables the HHDM maps memory read/write/execute, so kernel memory is also writable and executable through that alias until K6.
- Stacks have no guard pages; frames handed out by the allocator are not zeroed.
- AgamLang's limits are provisional policy. A single operation can still be costly, and the step limit is optional. The interpreter is not a sandbox.
- No untrusted kernel code and no untrusted AgamLang program is in scope for any security claim.

### 8.3 Diagnostics and observability

- AOS: serial log with fixed line formats; full register dump on any exception; QEMU `-d int,cpu_reset` log kept with every test run; a clear distinction between the guest's exit value and the host's exit status (`AOS_KERNEL_SPEC.md` §12.5).
- AgamLang: every error has a stable code, a source span, a plain-language message and a tip, in English or Hinglish; Python tracebacks never reach users.

### 8.4 Versioning

Root `VERSION` holds the ecosystem version. Tags (created by the owner): `aos-v0.1.0-sprint1`, `agamlang-v0.1.0`. Each document carries its own status line.

---

## 9. What was read or run, and by whom

Exact values are in `DECISIONS.md` §8.

| Item | By | Result |
|---|---|---|
| Independent architecture review of revision 2 (commit `349c48a`) | Codex | Not ready: 0 Critical, 14 Major, 8 Minor, 2 Informational. Report untracked in `codex-worktree`; cited by commit and SHA-256. |
| Independent re-review of revision 3 (commit `fd2670d`) | Codex | Not ready for owner approval. Of AR-01…24: 19 Resolved, 5 Partially Resolved, at document level. New: 7 Major, 2 Minor (AR-25…33). Report untracked; cited the same way. |
| Read-only exercise of the manifest and tree-listing flags; WSL tool versions | Codex (second review, evidence R3-E03, R3-E05) | Flags behave as specified on the review commits. Versions as in `DECISIONS.md` §8.2; unversioned LLVM utility names not found. |
| Limine asset and header hashes; release/header pairing; base revision | Codex (downloads, hashes, source search); architect (clone, and a second archive download) | Reproduced. Identity and pairing only. |
| GDT, TSS, IDT and interrupt-frame arithmetic | Codex (probe and manuals) | Consistent. Not a CPU execution result. |
| Exit-value formula | Codex (QEMU 8.2.2 source); architect (boot sectors on QEMU 8.2.2 in a scratch sandbox) | Guest `0x10` → host 33; `0x11` → 35; reset → 0 |
| C flag set and header identifiers | Architect, throwaway probe, Clang 18.1.3, scratch sandbox | Compiles, zero warnings. Not linked; no assembly. |
| Independent re-review of revision 4 (commit `cbf2cff`) | Codex | Not ready as a whole. Of the 14 findings re-reviewed: 11 Resolved, 3 Partially Resolved (AR-14, AR-27, AR-31), at specification level. New: 3 Major (AR-34…36), 3 Minor (AR-37…39). AOS track: ready for scoped owner consideration. Report untracked; cited by commit and SHA-256. |
| Stale-bytecode observation; read-only blob export and attribute-filter comparison | Codex (third review, evidence R4-E02, R4-E03), Windows CPython 3.13.3 and Git for Windows | A cache with matching size and timestamp is executed in place of the source, also with `-B`. Normalising one CRLF evidence file changes its blob ID. |
| Cache containment settings | Architect, scratch fixtures in a cloud sandbox, CPython 3.12.3 and 3.13.18, pytest 9.1.1 — not the owner's WSL, not project code | Timestamp caches and unchecked hash-based caches are executed in place of the source, also with `-B`; `-E` switches the environment variables off. With `-B -X pycache_prefix=<new empty directory>` the source is used and nothing is written. A sourceless `.pyc` is still importable, so absence must be checked. A stale `pytest` cache can make a failing test pass. Details: `DECISIONS.md` §8.3. |
| Branch and worktree state for revision 5 | Architect, by reading the repository's `HEAD`, ref files and objects; **no git command was run** | `ecosystem` on `review/architecture-v0.1` at `cbf2cff` (revision 4, committed by the owner), the nine documents identical to that commit before editing; `agent/codex` at `fd2670d`; `main` and `agent/claude` at `4b2edd6` |
| Owner's WSL2 toolchain | Owner (reported) | Ubuntu 24.04.5; Clang 18.1.3; LLD 18.1.3; QEMU 8.2.2; GDB 15.1; Make 4.3; `clang-18`, `ld.lld-18`, `llvm-objdump-18`, `llvm-readelf-18`, `llvm-nm-18`, `llvm-objcopy-18`, `llvm-ar-18` in PATH. No output filed yet. |
| Revision 4 and 5 document checks | Architect, scratch scripts | Cross-document identifier, table, inventory and catalogue consistency; arithmetic and caret positions of the worked examples. Listed in each revision's change summary. Not a substitute for re-review. |
| **Not performed by anyone** | — | Any project build, link, ISO, boot, exception, allocator or interpreter run; any gate step other than the read-only flags above; signature check of the Limine asset |

No kernel, interpreter, harness, test or checker code was written. Nothing was committed, merged, tagged or pushed by the architect for revision 4 or 5, and Git configuration was not changed; revision 4 was committed by the owner. The review reports and their evidence were not modified.

---

## 10. Blockers and top risks

Blockers (full table: `DECISIONS.md` §9). None is cleared by this revision.

| Blocker | Gate |
|---|---|
| B-07 process contract (AR-01…04, AR-15, AR-18, AR-25…27, AR-34, AR-35, AR-37…39) — pending re-review | G-BOOT |
| B-04 approved documents on `main` | G-BOOT |
| B-01 toolchain evidence; B-03 attribution evidence | M0 acceptance |
| B-02 owner's disposition of the Limine pin evidence; B-08 kernel contract — pending re-review | G-AOS |
| B-06 Hinglish wording; B-09 language semantics; B-10 diagnostics — pending re-review and owner review | G-LANG |
| B-05 independent review with no Critical or Major finding open, judged per track (O-23) | G-AOS for the AOS track; G-LANG for the language track |

| Risk | Summary | Mitigation |
|---|---|---|
| R-02 | Git run from WSL could prune the Windows worktrees | Git only on Windows; WSL tools are Git-free |
| R-03 | CRLF checkouts can break scripts, Makefiles and golden files | `.gitattributes` first; content hashes checked before every gate run |
| R-11 | Two agents editing the same file | Manifest checker run from the trusted copy; merge verified record by record |
| R-12 | Unverified claims | Acceptance only from a gate run bound to exact commits |
| R-19 | Tests copying expectations from the implementation | Independent hand-written fixtures |
| R-25 | Stale gate results | Live refs checked before capture, after the run and before promotion; results void when any SHA or tool version changes |
| R-26 | A branch weakening its own check, or dropping a required test | Checkers and the test inventory exported from the baseline, outside every worktree |

The full register is `ROADMAP.md` §8.
