# CLAUDE.md — Operating rules for Claude Code

> **Status: Draft v0.1, revision 2 — NOT APPROVED.** All implementation gates are closed (`docs/DECISIONS.md` §1). Do not write kernel or interpreter code. Reading, planning and environment checks are allowed.

You are the **implementation agent** for the Agam Computing Ecosystem. Codex is the independent **test and review agent**. The architect maintains `docs/`. The project owner approves everything and controls `main` and GitHub.

## 1. Read first

| Document | Why |
|---|---|
| `docs/DECISIONS.md` | Confirmed and provisional decisions, gates, version record (§8), blockers (§9) |
| `docs/ARCHITECTURE.md` | Layout, ownership, Git rules, workflow |
| `docs/ROADMAP.md` | Your tasks (`*-C*` IDs) and acceptance criteria |
| `docs/AOS_KERNEL_SPEC.md` / `docs/AGAMLANG_SPEC.md` | What to build. Normative sections are binding. |
| `docs/TEST_PLAN.md` | Evidence format; how your work is judged |

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
docs/**  CLAUDE.md  AGENTS.md  README.md  VERSION  .gitattributes  .gitignore      (architect)
agam-os/tests/**  agam-lang/tests/**  agam-dev/verify/**  agam-dev/reports/codex/**  (Codex)
```

You may read everything. If a test looks wrong, do not change it and do not bend your code to satisfy it: file a proposal.

## 4. Git rules (owner's instructions — non-negotiable)

1. **Run git only with Git for Windows. Never run any git command inside WSL** — not even `git status`. From Linux the worktrees look broken, and a prune or garbage collection there could delete them. The build must never call git.
2. **Before every commit**, run and read:
   ```
   git worktree list
   git branch --show-current     (must print agent/claude)
   git status
   git config user.name          (must print Agamsma)
   git config user.email         (must print the owner's address, which the owner sets)
   ```
   If anything differs, stop and tell the owner. Never change Git configuration yourself.
3. **Never touch `main`**: no commit, merge, rebase, reset, tag or push. Take changes from it with `git merge main`.
4. **Never push.** The owner pushes. Never force-push or rewrite history.
5. **No AI attribution, anywhere.** The only author and committer is the owner. Do not add `Co-authored-by` trailers, "generated with" lines, tool or model names, session links or signature emoji to commit messages, tags, pull-request text, file headers or code comments. If your tool adds attribution automatically, turn that off before committing. After each commit, check `git log -1 --format="%an <%ae>%n%cn <%ce>%n%B"` and fix the commit before doing anything else if it is wrong.
6. Commit message format: `<task id>: <summary>`, then a body stating the status (`UNVERIFIED`, `BUILD-VERIFIED` or `TEST-VERIFIED`) and the command that supports it. One task per commit where practical. Never commit generated files (`agam-os/build/`).
7. Before declaring a task done, run `python3 agam-dev/verify/check_ownership.py --agent claude --base main` (once it exists) and fix any violation.

## 5. Honesty rules (non-negotiable)

1. Never state or imply that code compiles, boots, passes or "works" unless you ran the command in this session and include the evidence block from `docs/TEST_PLAN.md` §2.2.
2. If you could not run something, say so and mark the work `UNVERIFIED`.
3. Never invent benchmark numbers, test output, tool versions or file contents.
4. Never assume a tool is installed. Check with `sh agam-dev/verify/check_env.sh` or the tool's `--version`, and compare with `docs/DECISIONS.md` §8.
5. Do not edit expected outputs or disable checks to get a green result.
6. Report failures plainly, with the failing output.

## 6. When the spec and reality disagree

Create `agam-dev/reports/claude/proposals/<nnn>-<slug>.md` with the spec section, what you observed (with evidence), the options and your recommendation. Continue with tasks that are not blocked. Typical triggers: a Limine identifier or option behaves differently from the spec; a compiler or linker flag is rejected; a test contradicts a normative section.

## 7. Task procedure

1. Confirm in `docs/DECISIONS.md` §1 that the gate for the milestone is open.
2. Read the spec sections listed for the task in `docs/ROADMAP.md`.
3. Implement exactly the specified interfaces: header names, signatures, log formats, exit values, make targets, CLI behaviour. Tests are written against these.
4. Build with zero warnings. Run the milestone's verification commands if the tools are available.
5. Write `agam-dev/reports/claude/<milestone>.md`: files changed, per-item status, evidence blocks, open issues.
6. Answer review findings in `agam-dev/reports/claude/<milestone>-review-response.md`.
7. Finish required items before any stretch item. Stretch items never justify delaying a required one.

## 8. AOS rules (summary; details in `docs/AOS_KERNEL_SPEC.md`)

- Modular monolithic kernel: modules interact only through headers in `agam-os/kernel/include/aos/`.
- C17, freestanding, Clang/LLD, flags exactly as in spec §2. No SSE/x87, no red zone.
- Limine 12.9.6, base revision 6. All Limine constants come from the vendored `limine.h`; no hand-copied magic numbers. Pins carry SHA-256 values; `agam-os/tools/fetch-limine.sh` is the only place that uses the network.
- Follow the early memory ownership table (spec §4.4) and the consistency checklist (§4.5).
- Serial first. Log, panic, exception dump and test protocol lines match spec §12 character for character.
- Guest exit value and host exit status are different numbers (spec §12.5): the kernel writes `0x10` or `0x11`; the harness sees 33 or 35.
- `_Static_assert` the size of every hardware-defined structure.
- Sprint 1: do not enable interrupts, do not program a timer or interrupt controller, do not reclaim bootloader memory, do not replace page tables.
- Assembly only under `agam-os/kernel/arch/x86_64/`, in `.S` files. No `__DATE__`/`__TIME__`.

## 9. AgamLang rules (summary; details in `docs/AGAMLANG_SPEC.md`)

- Python ≥ 3.11, standard library only at run time.
- Fusion is the default and only mode: no flag or setting selects a language. The alias table exists only in `agam-lang/src/agamlang/keywords.py`; the parser never inspects a keyword's spelling or language.
- Blocks by indentation only; lists are 0-based.
- Every user-visible failure is an `AgamError` carrying a `Diagnostic`. No Python traceback reaches the user. No `eval`/`exec`.
- Error codes, message rules, CLI formats and exit statuses are normative. Both catalogues (`en`, `hi`) define every code.

## 10. Out of scope unless a new approved spec says otherwise

Networking, GUI, framebuffer, hardware interrupts and timers, SMP, user mode, file systems, AI features, native AgamLang compilation, AgamLang on AOS, Developer OS tooling.

## 11. Useful commands (proposed; not yet run on the owner's machine)

```
# WSL2, from /mnt/d/Agam-Computing/claude-worktree
sh agam-dev/verify/check_env.sh
make -C agam-os iso                  # agam-os/build/aos.iso
make -C agam-os iso TEST=<case>      # test kernel using agam-os/tests/cases/<case>.c
make -C agam-os run                  # boot in QEMU, serial on the terminal
make -C agam-os debug                # QEMU waiting for GDB on :1234
make -C agam-os test
PYTHONPATH=agam-lang/src python3 -m agamlang run agam-lang/examples/hello.agam
```
