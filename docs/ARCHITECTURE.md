# Agam Computing Ecosystem — Architecture

| Field | Value |
|---|---|
| Status | **Draft v0.1, revision 2 — NOT APPROVED.** The owner's provisional decisions of 2026-11-12 are incorporated. Nothing in this document authorises implementation. |
| Date | 2026-11-12 (revision 2) |
| Owner | Project owner (approves); Architect (maintains) |
| Related | `DECISIONS.md`, `ROADMAP.md`, `AOS_KERNEL_SPEC.md`, `AGAMLANG_SPEC.md`, `TEST_PLAN.md`, `CODEX_REVIEW_BRIEF.md`, `../CLAUDE.md`, `../AGENTS.md` |

Every statement here is tracked by ID in `DECISIONS.md`:

- **C-nn — Confirmed** in the original brief.
- **O-nn — Owner decision, provisional** (2026-11-12).
- **P-nn — Proposal** by the architect, with its status.
- **Q-nn — Open question.** **B-nn — Blocker.**

No AOS or AgamLang code exists. §9 lists exactly what was read or run while preparing this revision.

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
2. **The spec is the arbiter.** Implementation (Claude Code) and tests (Codex) are both written from the spec, independently. Disagreements are resolved by changing the spec through `DECISIONS.md`, not by working around it.
3. **Evidence or it did not happen.** No "tested", "passes" or "works" without the command, exit status, output, tool versions and commit (`TEST_PLAN.md` §2).
4. **One writer per path** (§6).
5. **Pinned inputs.** Bootloader, protocol header and toolchain versions are recorded (`DECISIONS.md` §8); the build never downloads "latest".
6. **The owner controls `main` and the public history** (O-10, O-11): nothing is merged, pushed or attributed without the owner.
7. **Required before stretch** (O-09): stretch items never delay or block a milestone.

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
    ├── verify/                  Codex  — check_env.sh, check_ownership.py, run.py
    └── reports/
        ├── claude/              Claude — task reports, evidence, proposals
        └── codex/               Codex  — evidence, reviews, proposals
```

Rules that make the layout work:

- **Build → test contract.** `agam-os/Makefile` (Claude) exposes `iso [TEST=<case>]`, `run`, `debug`, `clean`, `test`. The harness (Codex) calls `make -C agam-os iso …` and launches QEMU itself. Neither side edits the other's files; the contract is in `AOS_KERNEL_SPEC.md` §11 and `TEST_PLAN.md` §4.
- **In-kernel test cases belong to Codex.** `agam-os/tests/cases/<case>.c` is written against the public headers in `agam-os/kernel/include/aos/` and compiled into a test kernel by the Makefile.
- **`docs/` has one writer.** Agents request spec changes through proposal files in their own `agam-dev/reports/<agent>/proposals/` folder.
- **Empty folders are not stored by Git** (R-23): today only `README.md` is tracked. The project folders appear in the agent worktrees once their first files are committed and merged.

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

Fusion is the default and only mode (O-07): one lexer, one grammar, one AST, one interpreter. Blocks use Python-style indentation with no terminator word (O-05); lists are 0-based (O-06).

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
| `docs/**`, `CLAUDE.md`, `AGENTS.md`, `README.md`, `VERSION`, `.gitattributes`, `.gitignore` | Architect, with the owner's approval |
| `agam-os/**` except `agam-os/tests/**` | Claude Code |
| `agam-lang/**` except `agam-lang/tests/**` | Claude Code |
| `agam-dev/reports/claude/**` | Claude Code |
| `agam-os/tests/**`, `agam-lang/tests/**` | Codex |
| `agam-dev/verify/**`, `agam-dev/reports/codex/**` | Codex |
| `agam-os/build/**` and other generated output | Untracked |

The two agents' path sets are disjoint, so merging both into `main` cannot produce textual conflicts. `agam-dev/verify/check_ownership.py` checks each branch's diff against this table at every gate.

### 6.2 Branches (O-02; read from the repository on 2026-11-12)

| Folder | Branch | Rule |
|---|---|---|
| `D:/Agam-Computing/ecosystem` | `main` | Integration. Changed only by the owner or with the owner's explicit approval (O-11). |
| `D:/Agam-Computing/claude-worktree` | `agent/claude` | Claude Code's work. Starts each task with `git merge main`. |
| `D:/Agam-Computing/codex-worktree` | `agent/codex` | Codex's work. Starts each task with `git merge main`. |

### 6.3 Git rules (O-01, O-10, O-11)

- **Git runs only with Git for Windows.** It is never run inside WSL: from Linux the agent worktrees look "prunable", and a prune or garbage collection there could delete them (R-02).
- **Before every commit**: `git worktree list`, `git branch --show-current` and `git status` are run, and the folder and branch are confirmed.
- **Identity**: every commit's author and committer is the owner's GitHub identity (`Agamsma`, with the email the owner sets in the repository's Git configuration). Agents do not change Git configuration.
- **No AI attribution anywhere**: no `Co-authored-by` trailers, no "generated with" lines, no tool or model names, no session links — in commit messages, tags, pull requests or as author/committer.
- **No pushes by agents.** The owner pushes. Nothing is force-pushed.

### 6.4 Milestone flow

```
 spec approved ─► Codex writes tests from the spec ─┐
              └─► Claude implements from the spec ──┼─► owner merges both into main ─► gate on main
                                                    │        │ fail: fixed by the agent that owns the path
                                                    │        ▼ pass
                                                    └─► Codex reviews Claude's diff ─► findings fixed ─► owner accepts ─► tag
```

---

## 7. Technology stack (exact versions in `DECISIONS.md` §8)

| Area | Choice | Status |
|---|---|---|
| Kernel language | C17 + assembly | C-02 |
| Kernel structure | Modular monolithic | O-08 |
| Bootloader | Limine **12.9.6** | C-03; pin P-02, conditional on B-02 |
| Boot protocol | Limine protocol, **base revision 6** | O-03 (conditional) |
| Protocol header | `limine-protocol` commit `6e85ddd…`, vendored | P-02 |
| Compiler / linker | Clang 18.1.3, LLD 18.1.3, target `x86_64-unknown-none-elf` | C-04; flags P-04 |
| Assembler | Clang integrated assembler, `.S`, AT&T syntax | P-05 |
| Emulator / debugger | QEMU 8.2.2, GDB 15.1 | C-05 |
| Build | GNU Make 4.3; never invokes git | P-06 |
| Build/test environment | WSL2, Ubuntu 24.04 LTS, working in `/mnt/d/Agam-Computing/…` | O-01 |
| Version control | Git for Windows only | O-01 |
| AgamLang host | CPython ≥ 3.11 (3.12.3 on Ubuntu 24.04), standard library only at run time | P-20 |
| AgamLang tests | `pytest` (development dependency) | P-27 |

Versions are those offered by Ubuntu 24.04 on 2026-11-12; the owner's installation is verified at M0 (B-01).

---

## 8. Cross-cutting concerns

### 8.1 Reproducibility

- Limine binaries come from one exact release asset, verified by SHA-256; `limine.h` is vendored at one commit that the same Limine release itself pins.
- Tool versions are captured by `agam-dev/verify/check_env.sh` into every evidence report.
- QEMU runs with `-accel tcg`, machine `q35`, CPU `qemu64`, one CPU, explicit memory size.
- `limine.conf` sets `timeout: 0` and `randomise_hhdm_base: no`; the kernel still never hard-codes the HHDM base.
- LF line endings are enforced, because files are edited on Windows and built on Linux.

### 8.2 Security posture (Phase 1)

- Kernel segments get minimum permissions through ELF program headers (text R+X, rodata R, data R+W); Limine maps them so and sets `CR0.WP` and `EFER.NX`.
- **Known limitation (R-20):** under Limine's page tables the HHDM maps memory read/write/execute, so kernel memory is also reachable, writable and executable, through that alias until the kernel builds its own page tables (K6). Sprint 1 runs no untrusted code.
- `-Werror`; no floating point or SIMD in kernel code; no red zone.
- The PMM refuses double frees and invalid frees by panicking.
- AgamLang never calls `eval`/`exec` on user text, has no file or network access in v0.1, and enforces call-depth and optional step limits.
- Supply chain: two external inputs (Limine release, protocol header), both pinned by hash.

### 8.3 Diagnostics and observability

- AOS: serial log with fixed line formats; full register dump on any exception; QEMU `-d int,cpu_reset` log kept with every test run; a clear distinction between the guest's exit value and the host's exit status (`AOS_KERNEL_SPEC.md` §12.5).
- AgamLang: every error has a stable code, a source span, a plain-language message and a tip, in English or Hinglish; Python tracebacks never reach users.

### 8.4 Versioning

Root `VERSION` holds the ecosystem version. Tags (created by the owner): `aos-v0.1.0-sprint1`, `agamlang-v0.1.0`. Each document carries its own status line.

---

## 9. What was read or run while preparing this revision

Details and exact values are in `DECISIONS.md` §8.

| Item | How | Result |
|---|---|---|
| Folder layout under `D:/Agam-Computing` | Listed | `ecosystem` (`README.md`, and empty `agam-dev/`, `agam-lang/`, `agam-os/`, `docs/`), `claude-worktree`, `codex-worktree` |
| Git state | Read-only git commands, no locks taken | One commit `4b2edd6`; `main`, `agent/claude`, `agent/codex` all at it; `main` tracks `origin/main`; only `README.md` tracked |
| Limine release and header | Official repositories cloned at tag `v12.9.6` and at `limine-protocol` commit `6e85ddd…`; release page read | Versions, hashes and compatibility as in `DECISIONS.md` §8.1. Asset digest read from the page, not recomputed. |
| Compiler flags and header identifiers | Throwaway compile probe with Clang 18.1.3 in a scratch sandbox | Compiles with zero warnings; no SIMD/x87 instructions. Not linked. |
| QEMU exit convention | Boot sectors on QEMU 8.2.2 in a scratch sandbox | Guest `0x10` → host 33; `0x11` → 35; reset → 0 |
| Ubuntu 24.04 package versions | Package index read in a scratch sandbox | `DECISIONS.md` §8.2 |
| **The owner's WSL2 and installed tools** | — | **Not inspected. Nothing is assumed installed** (B-01). |

No kernel or interpreter code was written. Nothing was committed, merged or pushed.

---

## 10. Critical blockers and top risks

Blockers (full table: `DECISIONS.md` §9): B-01 toolchain not verified on the owner's machine; B-02 Limine pin not independently verified; B-03 Git identity not yet set for O-10; B-04 documents not committed; B-05 Codex review outstanding; B-06 Hinglish keyword table undecided (interpreter only).

| Risk | Summary | Mitigation |
|---|---|---|
| R-02 | Git run from WSL could prune the Windows worktrees | Git only on Windows; build never calls git |
| R-04 | Limine changes quickly | Exact release and header pinned by hash, independently re-verified |
| R-11 | Two agents editing the same file | Disjoint path ownership; automated check at every gate |
| R-12 | Unverified "it works" claims | Mandatory evidence format; gate re-runs everything on `main` |
| R-20 | HHDM is writable and executable | Recorded limitation; removed at K6 |
| R-21 | AI attribution or wrong email in a commit | O-10 rules; identity set by the owner; `git log` check at M0 and every gate |
| R-22 | Building on `/mnt/d` from WSL | Verified at M0; fallback documented |

The full register is `ROADMAP.md` §8.
