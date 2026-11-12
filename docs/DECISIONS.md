# Agam Computing Ecosystem — Decision Log

| Field | Value |
|---|---|
| Status | **Draft v0.1, revision 2 — NOT APPROVED.** The owner's provisional decisions of 2026-11-12 are recorded. Implementation remains blocked (§1). |
| Date | 2026-11-12 (revision 2) |
| Rule | This file is the only place where a decision's status changes. Only the project owner can approve. |

## How to read this file

| Prefix | Meaning | Who can change it |
|---|---|---|
| **C-nn** | **Confirmed** — stated by the owner in the original brief | Owner only |
| **O-nn** | **Owner decision, provisional** — given on 2026-11-12 after reviewing the architecture summary. Binding for the documents; the owner may revise it after Codex's review. | Owner only |
| **P-nn** | **Proposal** — the architect's recommendation, with its current status | Owner approves, rejects or amends |
| **Q-nn** | **Open question** | Owner answers, or a tool run provides evidence |
| **B-nn** | **Blocker** — something that must be true before implementation starts (§9) | Cleared with evidence |

---

## 1. Implementation gates — all closed

"Proceed with provisional decisions" authorises revising these documents and preparing the review. It does **not** authorise implementation (C-11, owner's instruction of 2026-11-12).

| Gate | Opens when | Covers |
|---|---|---|
| **G-ENV** | B-01, B-03, B-04 cleared; owner says so | Milestone M0 work in the repository |
| **G-AOS** | G-ENV open; B-02 and B-05 cleared; owner explicitly approves the AOS documents | Any kernel code |
| **G-LANG** | G-ENV open; B-05 and B-06 cleared; owner explicitly approves the AgamLang specification | Any interpreter code |

---

## 2. Confirmed decisions (original brief)

| ID | Decision |
|---|---|
| C-01 | Three projects in one repository: AOS, AgamLang, Agam Developer OS |
| C-02 | AOS kernel is original, x86-64, written initially in C17 and assembly |
| C-03 | Bootloader is Limine |
| C-04 | Toolchain is Clang/LLD |
| C-05 | Emulator is QEMU; debugger is GDB |
| C-06 | Three working directories: `ecosystem` (integration), `claude-worktree` (Claude Code implementation), `codex-worktree` (Codex testing and review) |
| C-07 | First AgamLang implementation is a Python-based interpreter; native compilation and AOS integration are future work |
| C-08 | Fusion Mode supports English, Hinglish and mixed syntax in a single `.agam` file, through canonical tokens, not separate interpreters |
| C-09 | All architecture documents are Draft v0.1 until the owner explicitly approves them |
| C-10 | Priority is a minimal, reproducible AOS kernel; networking, GUI, AI integration and native AgamLang execution are out of scope for now |
| C-11 | No implementation starts before the architecture is reviewed and approved |
| C-12 | No invented benchmarks; no claim that code compiled or passed tests without evidence; no assumption that tools are installed |
| C-13 | Claude Code and Codex must be able to work independently without modifying the same files |
| C-14 | Architectural decisions are preserved in Markdown tracked by Git |
| C-15 | First sprint is five days: bootable kernel, CPU exception handling, physical memory management |

---

## 3. Owner decisions, provisional (2026-11-12)

| ID | Decision | Effect on earlier items |
|---|---|---|
| O-01 | Build and test in **WSL2 Ubuntu 24.04**. **Git operations stay on Windows** initially. | Answers Q-01; accepts P-03 |
| O-02 | Use the **existing branch names**; do not rename. Read from the repository on 2026-11-12: `main` (ecosystem), `agent/claude` (claude-worktree), `agent/codex` (codex-worktree). | Answers Q-02; replaces the names proposed in P-12 |
| O-03 | **Limine base revision 6**, on condition that the exact pinned release and header compatibility are independently verified. | Accepts P-01 and P-02 conditionally; condition tracked as B-02 |
| O-04 | **Hardware interrupts stay disabled during Sprint 1.** CPU exception handling only. | Accepts P-08 |
| O-05 | AgamLang uses **Python-style indentation**, with no mandatory `end` or `khatam`. | Answers Q-06; accepts P-25 |
| O-06 | **0-based list indexing.** | Answers Q-05 |
| O-07 | **Fusion is the default.** English, Hinglish and mixed keywords work in the same source file. | Accepts P-22 and P-23 |
| O-08 | AOS is a **modular monolithic kernel** with a **serial-first** debugging approach. | Accepts P-13; recorded in `ARCHITECTURE.md` §5.1 |
| O-09 | **Double-fault testing and advanced fault injection are stretch goals.** | K3 and K4 split into required and stretch items (`ROADMAP.md`, `TEST_PLAN.md`) |
| O-10 | **Commits and pushes use only the owner's GitHub identity (`Agamsma`).** No AI tool may appear as author, committer, co-author or contributor, and no AI attribution text may appear in commit messages, tags or pull requests. | New rule in `CLAUDE.md` §4, `AGENTS.md` §4; risk R-21 |
| O-11 | **`main` is not modified without the owner's approval.** The actual branch and worktree are verified before any commit. | New rule in the merge gate (`TEST_PLAN.md` §8) |
| O-12 | The project root is `D:/Agam-Computing`, with worktrees `ecosystem`, `claude-worktree`, `codex-worktree`, and top-level project folders `agam-os/`, `agam-lang/`, `agam-dev/`, `docs/`. Architecture documents belong in `docs/`, not only in `claude-worktree`. | Replaces the layout first proposed in P-11 |

Requested revisions of 2026-11-12 and where they were made:

| Request | Where |
|---|---|
| Double-fault testing and fault injection as stretch goals | O-09; `ROADMAP.md` K3, K4; `TEST_PLAN.md` §5; `AOS_KERNEL_SPEC.md` §9.6 |
| Verify the QEMU exit convention; separate guest exit value from host exit status | `AOS_KERNEL_SPEC.md` §12.5 (with observations); `TEST_PLAN.md` T-K1-09 |
| Make the higher-half address, linker configuration, Limine requirements and early memory ownership consistent | `AOS_KERNEL_SPEC.md` §4.4 (ownership table), §4.5 (consistency checklist), §3.1 |
| Record exact toolchain and Limine versions | §8 of this file |
| Identify remaining critical blockers | §9 of this file |

---

## 4. Proposals — environment and process

| ID | Proposal | Status |
|---|---|---|
| P-03 | Build/test environment is WSL2 with Ubuntu 24.04 LTS; version control stays on Git for Windows | **Provisionally accepted (O-01)** |
| P-06 | GNU Make; the build never runs `git`; the version string comes from the root `VERSION` file | Proposed. Now also required by O-01, since git is not used from WSL. |
| P-10 | Path ownership: one writer per path (table in `ARCHITECTURE.md` §6, revised for the real layout) | Proposed |
| P-11 | Repository layout as in `ARCHITECTURE.md` §4, built on the owner's folders (O-12): AOS code in `agam-os/`, interpreter in `agam-lang/`, tests in `agam-os/tests/` and `agam-lang/tests/`, shared verification tools and agent reports in `agam-dev/` | Proposed (revised) |
| P-12 | Agents start each task with `git merge main`; only the owner (or an explicitly approved action) merges into `main` | Proposed; branch names fixed by O-02 |
| P-15 | Evidence protocol and status vocabulary (`UNVERIFIED`, `BUILD-VERIFIED`, `TEST-VERIFIED`, `FAILED`, `SKIPPED`) as in `TEST_PLAN.md` §2 | Proposed |
| P-16 | `.gitattributes` forces LF line endings (`* text=auto eol=lf`) | Proposed |
| P-17 | These draft documents reach the agents by being committed to `main`, clearly labelled Draft, **by the owner or with the owner's approval**; agents then `git merge main`. Until then Codex reviews them read-only in `ecosystem/docs/`. | Proposed; see Q-13 and B-04 |

## 5. Proposals — AOS

| ID | Proposal | Status |
|---|---|---|
| P-01 | Limine boot protocol, base revision 6; the kernel halts if the bootloader does not report it as supported | **Provisionally accepted, conditional (O-03, B-02)** |
| P-02 | Pin **Limine 12.9.6** (release asset verified by SHA-256) and vendor `limine.h` from `limine-protocol` at commit `6e85ddd6f928b5b512015c4d47d1cfce2fd28079`. Exact values in §8.1. | **Provisionally accepted, conditional (O-03, B-02)** |
| P-04 | Compiler and linker flags as in `AOS_KERNEL_SPEC.md` §2 | Proposed. The compile flags were accepted by Clang 18.1.3 in a scratch probe (§8.3); linking is untested. |
| P-05 | Assembly in `.S` files, Clang integrated assembler, AT&T syntax | Proposed |
| P-07 | Virtual memory layout: kernel image at `0xFFFFFFFF80000000`, non-relocatable; HHDM base read at run time; 4-level paging; Sprint 1 keeps Limine's page tables | Proposed. Cross-checked against the protocol text; see `AOS_KERNEL_SPEC.md` §4.5. |
| P-08 | Sprint 1 interrupt handling: own GDT and TSS; IDT with 256 gates; handlers for vectors 0–31; interrupts disabled; no timer | **Provisionally accepted (O-04)** |
| P-09 | Physical memory manager: bitmap allocator, 4 KiB frames, Usable memory at or above 1 MiB only; bootloader-reclaimable memory not reclaimed in Sprint 1 | Proposed. Ownership rules in `AOS_KERNEL_SPEC.md` §4.4. |
| P-13 | Serial (COM1, 115200 8N1) is the only console in Sprint 1 | **Provisionally accepted (O-08)** |
| P-14 | Test kernels are selected at compile time (`make iso TEST=<case>`) and end through the `isa-debug-exit` device: guest exit value `0x10` gives host exit status 33 (pass), `0x11` gives 35 (fail) | Proposed. The value-to-status mapping was observed on QEMU 8.2.2 (§8.3). |
| P-18 | Modular monolithic kernel: one statically linked image, one address space, subsystems as modules behind headers in `agam-os/kernel/include/aos/` | **Provisionally accepted (O-08)** |

## 6. Proposals — AgamLang

| ID | Proposal | Status |
|---|---|---|
| P-20 | CPython ≥ 3.11, standard library only at run time | Proposed |
| P-21 | Hand-written lexer and recursive-descent parser | Proposed |
| P-22 | Fusion through one alias table mapping every surface word or two-word phrase to a canonical token kind | **Provisionally accepted (O-07)** |
| P-23 | Every alias word is reserved in every file | **Provisionally accepted (O-07)** — this is what makes mixing inside one file unambiguous |
| P-24 | Keywords are case-insensitive; identifiers are case-sensitive | Proposed |
| P-25 | Python-style blocks: colon plus indentation; spaces or tabs, used consistently | **Provisionally accepted (O-05)** |
| P-26 | Keyword-first grammar in v0.1: Hinglish uses the same statement order as English | Proposed |
| P-27 | `pytest` for tests (development dependency only) | Proposed |
| P-28 | Strict semantics: boolean conditions only; `text + number` is an error with a hint; `/` always gives a decimal; `range` excludes its stop value; lists are 0-based | 0-based indexing **provisionally accepted (O-06)**; the rest proposed |
| P-29 | Diagnostics: stable codes, one error at a time, source excerpt with caret, English and Hinglish catalogues, "did you mean", no Python tracebacks | Proposed |
| P-30 | v0.1 language surface exactly as in `AGAMLANG_SPEC.md` §4–§8 | Proposed |

## 7. Proposals — Developer OS

| ID | Proposal | Status |
|---|---|---|
| P-40 | Developer OS is delivered in stages D0–D3 (`ARCHITECTURE.md` §5.3). For now `agam-dev/` holds only the shared verification tools (`agam-dev/verify/`) and agent reports (`agam-dev/reports/`). | Proposed |

---

## 8. Version record

"Expected" means what the architect found on 2026-11-12. "Host" means the owner's machine, which has **not** been inspected; every Host cell is filled at milestone M0.

### 8.1 Pinned external inputs

| Input | Exact value | How it was obtained | Independent check |
|---|---|---|---|
| Limine release | **12.9.6** — tag `v12.9.6`, tag commit `89a9512c623a4779cc73e3869383876d54264b28`, release dated 2026-10-27/28 | Tag listed and cloned from the official repository | Codex, M0-X4 |
| Limine binary asset | `limine-binary.tar.xz`, SHA-256 `939f670e0831cc940453fd3a0ddfb2b324c0b4e3c5d0e741c190aa0d13df0103` | Digest **read from the GitHub release page**; the file was not downloaded or hashed by the architect | Codex, M0-X4 (download and recompute) |
| Alternative asset | `limine-binary.tar.gz`, SHA-256 `b38aabb042b0c0f9f562b7741be3c1204838d7bbacfa78bb411c674bd5b984ee` | Same page | — |
| Base revision supported by that release | `#define SUPPORTED_BASE_REVISION 6` in `common/protos/limine.c` | Read in the cloned tag | Codex, M0-X4 |
| Protocol header | `limine-protocol` commit `6e85ddd6f928b5b512015c4d47d1cfce2fd28079` (2026-10-27); `include/limine.h` SHA-256 `0aadf2633c85d8cb145344c968add158aeb1deb68af567d55ba8431c492b6d4c` | Cloned; hash computed from the clone | Codex, M0-X4 |
| Header ↔ release compatibility | Limine 12.9.6's own `bootstrap` script pins `limine-protocol` at the same commit `6e85ddd…` | Read in the cloned tag | Codex, M0-X4 |
| Protocol base revision requested by AOS | 6 | O-03 | — |

### 8.2 Toolchain (WSL2 Ubuntu 24.04 LTS)

Expected versions are the candidate package versions offered by the Ubuntu 24.04 archive on 2026-11-12. Security updates can change the Ubuntu suffix; the upstream version is what matters. M0 records the exact strings printed on the host.

| Tool (command) | Ubuntu package | Expected version | Used by the architect on 2026-11-12? | Host |
|---|---|---|---|---|
| Clang (`clang`) | `clang` → `clang-18` | 18.1.3 (`1:18.1.3-1ubuntu1`) | Yes — compile probe (§8.3) | Pending M0 |
| LLD (`ld.lld`) | `lld` → `lld-18` | 18.1.3 (`1:18.1.3-1ubuntu1`) | Installed; version printed; no link performed | Pending M0 |
| LLVM tools (`llvm-objdump`, `llvm-readelf`, `llvm-nm`) | `llvm` → `llvm-18` | 18.1.3 (`1:18.1.3-1ubuntu1`) | Yes — probe inspection | Pending M0 |
| GNU Make (`make`) | `make` | 4.3 (`4.3-4.1build2`) | No — version read only | Pending M0 |
| QEMU (`qemu-system-x86_64`) | `qemu-system-x86` | 8.2.2 (`1:8.2.2+ds-0ubuntu1.18`), with SeaBIOS 1.16.3 | Yes — exit-status experiment (§8.3) | Pending M0 |
| xorriso | `xorriso` | 1.5.6 (`1:1.5.6-1.1ubuntu3`) | No — version read only | Pending M0 |
| GDB (`gdb`) | `gdb` | 15.1 (`15.1-1ubuntu1~24.04.1`) | No — version read only | Pending M0 |
| OVMF (optional, UEFI test) | `ovmf` | 2024.02 (`2024.02-2ubuntu0.10`) | No — version read only | Pending M0 |
| Python (`python3`) | `python3` | 3.12.3 (`3.12.3-0ubuntu2.1`) | No — version read only | Pending M0 |
| Git | Git for Windows (not WSL) | Unknown | No | Pending M0 |

### 8.3 What the architect actually ran (scratch sandbox, Ubuntu 24.04, 2026-11-12)

None of this ran on the owner's machine, and none of it is AOS or AgamLang code.

| Check | Result |
|---|---|
| Compile a throwaway C file that includes the pinned `limine.h`, declares the Sprint 1 requests, and reads their responses, using the full flag set of `AOS_KERNEL_SPEC.md` §2.1 with Clang 18.1.3 | Exit 0, zero warnings; sections `.limine_requests_start`, `.limine_requests`, `.limine_requests_end` emitted as writable data; no `xmm`/`ymm`/`st` operands in the disassembly. Not linked. |
| Boot hand-assembled boot sectors in QEMU 8.2.2 with the `isa-debug-exit` device and `-no-reboot` | Guest value `0x10` → host status 33; `0x11` → 35; `0x00` → 1; no device or no exit call → no exit (timeout); guest reset → host status 0. Full table in `AOS_KERNEL_SPEC.md` §12.5. |
| Read `CONFIG.md` and `USAGE.md` at Limine tag `v12.9.6` | Option names in `agam-os/boot/limine.conf` and the `xorriso` command in the kernel specification match. |
| Read the Limine protocol specification at commit `6e85ddd…` | Entry state, mapping and memory-map statements in `AOS_KERNEL_SPEC.md` §3.1 match. |

### 8.4 Repository state (read without modifying anything, 2026-11-12)

| Item | Value |
|---|---|
| Remote | `origin` → `https://github.com/Agamsma/Agam-Computing` |
| Commits | One: `4b2edd6` "Initial ecosystem structure" |
| `D:/Agam-Computing/ecosystem` | Branch `main`, at `4b2edd6`, tracking `origin/main`, working tree clean before these documents were placed |
| `D:/Agam-Computing/claude-worktree` | Branch `agent/claude`, at `4b2edd6` |
| `D:/Agam-Computing/codex-worktree` | Branch `agent/codex`, at `4b2edd6` |
| Tracked files | `README.md` only. The folders `agam-os/`, `agam-lang/`, `agam-dev/`, `docs/` in `ecosystem` are empty, so Git does not store them yet (R-23). |
| Git identity | No `user.name`/`user.email` is set in the repository's own configuration. The existing commit was made with a different email address from the one the owner has now specified for this project (B-03). |
| Seen from Linux | Both agent worktrees are reported as "prunable" because their links are Windows paths (R-02). |

---

## 9. Critical blockers before implementation

| ID | Blocker | Why it is critical | Cleared by |
|---|---|---|---|
| **B-01** | The toolchain is not known to be installed in WSL2 Ubuntu 24.04, and WSL2 itself has not been confirmed on the owner's machine. | Nothing can be built or tested, and no version in §8.2 is confirmed. | Owner installs the packages (M0-O1); `check_env.sh` output filed (M0) |
| **B-02** | The Limine pin has not been independently verified: the asset digest was only read from a web page, and the release/header pairing was checked only by the architect. | Condition attached to O-03. | Codex report `agam-dev/reports/codex/M0-limine-pin.md` (M0-X4) agreeing with §8.1 |
| **B-03** | Git identity. The repository has no local `user.name`/`user.email`; the existing commit used a different email address from the one now required; and AI tools add attribution lines by default. | A single commit with the wrong identity or an AI trailer, once pushed, puts it in the public history (O-10). | Owner sets the identity in the repository with Git for Windows; attribution is switched off in each tool; `git log` check passes (M0 criterion 8). The owner also decides whether the existing commit's address matters (Q-14). |
| **B-04** | The documents are untracked files in the `ecosystem` working folder. Nothing is committed, so neither agent worktree contains them. | Agents cannot follow `CLAUDE.md`/`AGENTS.md` or the specifications from their own worktrees. | Owner decides how they are committed (Q-13) and performs or approves the commit |
| **B-05** | The architecture has not been reviewed by Codex. | Required by the owner before implementation. | `agam-dev/reports/codex/reviews/architecture-v0.1.md` with no High or Medium finding open, then the owner's explicit approval |
| **B-06** | The Hinglish keyword table and related questions (Q-07, Q-08, Q-09) are undecided. | Blocks the interpreter only; changing keywords later invalidates tests and golden files. | Owner approves or edits `AGAMLANG_SPEC.md` §3.2 |

Not blockers, but to watch: building on `/mnt/d` from WSL may be slow or behave differently (R-22); git must never be run from WSL (R-02); the HHDM is writable and executable under Limine's page tables (R-20).

---

## 10. Open questions

| ID | Question | Architect's suggestion | Blocks |
|---|---|---|---|
| Q-03 | Is BIOS boot the single required boot path for Sprint 1, with UEFI as an optional extra check? | Yes | G-AOS (minor) |
| Q-04 | Largest guest RAM the test matrix may use (128 MiB, 256 MiB, 1 GiB, 4 GiB proposed)? | 4 GiB required, 8 GiB optional | K5 |
| Q-07 | Hinglish keyword table: please review `AGAMLANG_SPEC.md` §3.2 word by word (spellings, and choices such as `kaam` for "function"). Should a later version add natural-order forms such as `5 baar dohrao:`? | Approve with your edits; defer natural-order forms | G-LANG (B-06) |
| Q-08 | Should `show` print `true`/`false`/`nothing` always, or follow the file's dominant language (`sach`/`jhooth`/`khaali`)? | Fixed words in v0.1 | G-LANG |
| Q-09 | Should Devanagari-script keywords be planned for? | Not in v0.1; the alias table allows adding them later | G-LANG |
| Q-10 | Licence for the repository | — | Later |
| Q-11 | Is continuous integration wanted? | — | Later |
| Q-12 | Real-hardware boot target after QEMU milestones | — | Later |
| Q-13 | How should these draft documents be committed so that Codex and Claude Code can read them in their worktrees: (a) commit to `main` as clearly labelled drafts, then each agent runs `git merge main`; or (b) leave them uncommitted until after Codex's review, with Codex reading `ecosystem/docs/` directly? | (b) for the review itself, then (a) once you approve | B-04 |
| Q-14 | The existing commit `4b2edd6` on `main` (already on GitHub) was made with a different email address from the one you specified. Leave it as is, or rewrite it? Rewriting needs a force-push to `main`. | Leave it unless the address must not be public; it is your decision | B-03 |
| Q-15 | Should the `agent/claude` and `agent/codex` branches ever be pushed to GitHub? Their names would be visible there. | Keep them local; push only `main` | Later |

Resolved: Q-01 (O-01), Q-02 (O-02), Q-05 (O-06), Q-06 (O-05).

---

## 11. Change history

| Date | Change | By |
|---|---|---|
| 2026-11-12 | Draft v0.1 created | Architect |
| 2026-11-12 | Revision 2: owner's provisional decisions O-01…O-12; real repository layout and branch names; version record; blockers; stretch goals; exit-code clarification | Architect |
