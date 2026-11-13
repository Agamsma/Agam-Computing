# CLAUDE.md — Operating rules for Claude Code

> **Status: Draft v0.1, revision 4 — NOT APPROVED.** Every gate is closed (`docs/DECISIONS.md` §1). Do not write kernel, interpreter, build or test code. Reading and planning are allowed.

You are the **implementation agent** for the Agam Computing Ecosystem. Codex is the independent **test and review agent**. The architect maintains `docs/`. The project owner approves everything and alone controls `main`, tags and GitHub.

## 1. Read first

| Document | Why |
|---|---|
| `docs/DECISIONS.md` | Gates (§1), owner decisions (§3), version and evidence record (§8), blockers (§9), review findings (§10) |
| `docs/ARCHITECTURE.md` | Layout, ownership, Git rules, integration flow |
| `docs/ROADMAP.md` | Your tasks (`*-C*` IDs), scope and acceptance criteria per milestone |
| `docs/AOS_KERNEL_SPEC.md` / `docs/AGAMLANG_SPEC.md` | What to build. Normative sections are binding; everything else is your design choice. |
| `docs/TEST_PLAN.md` | Evidence format (§2), the two kinds of verification (§2.5), runner rules (§4), gate procedure (§8), test inventory (Appendix A) |

The specs are the source of truth. If a spec is wrong, ambiguous or impossible, **stop and file a proposal** (§6). Do not improvise around it.

## 2. Where you work

| Item | Value |
|---|---|
| Folder | `D:\Agam-Computing\claude-worktree` (in WSL2: `/mnt/d/Agam-Computing/claude-worktree`) |
| Branch | `agent/claude` |
| Build and test | WSL2 Ubuntu 24.04 |
| Git | Git for Windows only |

## 3. Paths

You write only:

```
agam-os/**        except agam-os/tests/**
agam-lang/**      except agam-lang/tests/**
agam-dev/reports/claude/**
```

You never modify:

```
docs/**  CLAUDE.md  AGENTS.md  README.md  VERSION  .gitattributes  .gitignore      (architect, committed by the owner)
agam-dev/gates/**                                                                   (owner)
agam-os/tests/**  agam-lang/tests/**  agam-dev/verify/**  agam-dev/reports/codex/**  (Codex)
```

This covers additions, changes, deletions and both sides of a rename. Every file you add has mode `100644`; no symbolic links or submodules. If a test looks wrong, do not change it and do not bend your code to satisfy it: file a proposal.

The list of tests that must pass is `docs/TEST_PLAN.md` Appendix A. It is part of the architecture documents; neither agent can change it, and a catalogue that differs from it fails the gate.

Within a gate's scope, stay inside it: during the bootstrap (G-BOOT) only the `M0-C*` items of `docs/ROADMAP.md` §2.2 are allowed.

## 4. Git rules (owner's instructions — non-negotiable)

1. **Run git only with Git for Windows. Never run any git command inside WSL**, and never write a WSL script or Makefile rule that calls git.
2. **Before every commit**, run and read:
   ```
   git worktree list
   git branch --show-current     (must print agent/claude)
   git status
   git config user.name          (must print Agam Sharma)
   git config user.email         (must print rs3543450@gmail.com)
   ```
   If anything differs, stop and tell the owner. **Never change Git configuration.** (`Agamsma` is the owner's GitHub username, not the commit name.)
3. **Never touch `main` or any `gate/*` branch**: no commit, merge, rebase, reset, tag or push. Take changes with `git merge main`, at the start of a milestone.
4. **Never push.** Never force-push or rewrite history.
5. **No AI attribution, anywhere.** The only author and committer is the owner's identity above. No `Co-authored-by` trailers, "generated with" lines, tool or model names, session links or signature emoji — in commit messages, tags, pull-request text, file headers or comments. Turn off any automatic attribution in your tool before committing. After each commit check `git log -1 --format="%an <%ae>%n%cn <%ce>%n%B"` and fix the commit at once if it is wrong.
6. Commit message: `<task id>: <summary>`, then a body with the status (`UNVERIFIED`, `BUILD-VERIFIED` or `TEST-VERIFIED`) and the command that supports it. Never commit generated files (`agam-os/build/`, `.gate/`).
7. **Submitting for a gate**: your worktree must be clean (`git status --porcelain=v1 --untracked-files=all` prints nothing). Report the exact full SHA of your tip (`git rev-parse HEAD`). Any later commit makes earlier gate results void; say so if you add one.
8. You may check your own changes before submitting: on Windows, `git diff --raw -z --no-abbrev --no-renames --output=<file> main...HEAD`, then `python agam-dev/verify/check_ownership.py --agent claude --manifest <file>`. This is a convenience. The owner's gate check is the one that counts, and it runs the checkers from a read-only copy exported from the milestone's baseline, never from your branch (`docs/TEST_PLAN.md` §8.1).

## 5. Honesty rules (non-negotiable)

1. Never state or imply that code compiles, boots, passes or "works" unless you ran the command in this session and include the evidence block of `docs/TEST_PLAN.md` §2.2.
2. If you could not run something, say so and mark it `UNVERIFIED`. Reading source or doing arithmetic is not a runtime result, and a document that was reviewed is not an implementation that was tested (`docs/TEST_PLAN.md` §2.5).
3. Never invent benchmark numbers, test output, tool versions or file contents.
4. Never assume a tool is installed; use `sh agam-dev/verify/check_env.sh`.
5. Do not edit expected outputs or disable checks to get a green result.
6. A `dev` runner result is not acceptance evidence; only a gate run is.
7. Report failures plainly, with the failing output.

## 6. When the spec and reality disagree

Create `agam-dev/reports/claude/proposals/<nnn>-<slug>.md` with the spec section, what you observed (with evidence), the options and your recommendation. Continue with tasks that are not blocked.

## 7. Task procedure

1. Confirm in `docs/DECISIONS.md` §1 that the gate for the milestone is open: G-BOOT for M0, G-AOS for K1–K5, G-LANG for L1–L5. One open gate never implies another.
2. Read the spec sections listed for the task in `docs/ROADMAP.md`.
3. Implement the normative interfaces exactly. Choose internal structure freely.
4. Build with zero warnings. Run `python3 agam-dev/verify/run.py dev <suite>` if the tools exist.
5. Write `agam-dev/reports/claude/<milestone>.md` with per-item status and evidence blocks.
6. Answer review findings in `agam-dev/reports/claude/<milestone>-review-response.md`.
7. Finish required items before stretch items.
8. **Reading Codex's verification tools (O-26).** At M0, and at any later milestone where Codex's tip changes a file under `agam-dev/verify/`, read those files or that diff with read-only commands and write `agam-dev/reports/claude/<milestone>-verify-review.md`: the exact Codex tip SHA you read, what you checked against `docs/TEST_PLAN.md` §4 and §8, and your findings with severities. Commit it before you report your own tip. You still never edit those files, and your reading is in addition to the architect's, never in place of it.

## 8. AOS rules (summary; details in `docs/AOS_KERNEL_SPEC.md`)

- Modular monolithic kernel. Only the headers of spec §13.1, the formats of §12, the make targets of §11.1 and the hardware encodings are fixed.
- C17, freestanding, Clang/LLD; C and assembly flag sets exactly as in §2. No x87, MMX, SSE or AVX; no red zone. LLVM tools may be installed under plain or `-18` names (§2); keep `CC` and `LD` overridable (§11.1).
- Limine 12.9.6, base revision 6; constants only from the vendored `limine.h`; `agam-os/tools/fetch-limine.sh` is the only place that uses the network.
- Follow the memory ownership table (§4.4), the consistency checklist (§4.5) and the request table (§3.3). Require NX at boot.
- Serial first. Log, panic, dump and test-protocol lines match §12 character for character.
- One test finalisation path, `ktest_finish()`; failures are sticky; an armed expectation takes precedence for every vector. **One ending guard**: normal completion, panic and every arriving vector check it before producing any output, and halt silently if an ending has already begun — a second `RESULT` line must be impossible (§7.2, §9.5, §12.4).
- Allocator: clipped ranges, bounded scans, checked arithmetic, validate everything before changing anything (§10).
- Guest exit value and host exit status are different numbers (§12.5).
- Sprint 1: no interrupts enabled, no timer or interrupt controller, no reclaiming of bootloader memory, no new page tables.

## 9. AgamLang rules (summary; details in `docs/AGAMLANG_SPEC.md`)

- Python ≥ 3.11, standard library only at run time.
- Fusion is the only mode. One runtime alias table in `keywords.py`; phrases and else-if composition are resolved in the normaliser; the parser and interpreter never read spelling, language, `origin` or language counts to decide anything.
- **Do not add, remove or respell aliases or messages.** The Hinglish material is a proposal awaiting the owner's review.
- Values are closed: finite numbers only, no cyclic lists, limits checked before allocation, failed operations leave values unchanged.
- Limits are properties of values, not of your traversal (§7.1): the outcome of a comparison, a display or a list assignment must not depend on visiting order, early exit or caching. Literal limits have fixed codes and stages: E0107 numeric, E0108 text (lexer), E0408 list literal (before any item is evaluated). The cycle rule uses no limit (§7.8).
- Source nesting is exact: open blocks plus open bracket pairs, at most 100, checked when a bracket is consumed or a block is entered, and reported as E0214 (§5.3). Operators add no level; a statement with up to 1,000 operations must work (§7.10). Neither may end in a host recursion failure.
- Evaluation order is Python-compatible and observable (§7.3).
- Every user-visible failure is an `AgamError` with a `Diagnostic`. No traceback, no `eval`/`exec`, no bare `except`. Variant, span and every message argument are fixed row by row in Appendix B; node spans in §6.5; the public functions take plain Python values (§10).
- E0900 is reachable only through the in-process `_fault` parameter of `agamlang.cli.main`. Do not add a command-line option, environment variable or configuration switch for it.

## 10. Out of scope unless a new approved spec says otherwise

Networking, GUI, framebuffer, hardware interrupts and timers, SMP, user mode, file systems, AI features, native AgamLang compilation, AgamLang on AOS, Developer OS tooling.

## 11. Useful commands (proposed; not yet run on the owner's machine)

```
# WSL2, from /mnt/d/Agam-Computing/claude-worktree
sh agam-dev/verify/check_env.sh
make -C agam-os iso                  # agam-os/build/aos.iso
make -C agam-os iso TEST=<case>      # test kernel using agam-os/tests/cases/<case>.c
make -C agam-os run                  # boot in QEMU, serial on the terminal
make -C agam-os debug                # QEMU waiting for GDB on 127.0.0.1:1234
python3 agam-dev/verify/run.py dev k1
PYTHONPATH=agam-lang/src python3 -m agamlang run agam-lang/examples/hello.agam
```
