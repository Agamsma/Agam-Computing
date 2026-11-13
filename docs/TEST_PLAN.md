# Agam Computing Ecosystem — Test Plan

| Field | Value |
|---|---|
| Status | **Draft v0.1, revision 5 — NOT APPROVED.** A focused correction after the third independent review: AR-34 (record verification inputs), AR-35 (bytecode caches and other unbound inputs), AR-37 (first `ACCEPTED` state), AR-38 and AR-39 (preserved material, line endings), and the test side of AR-36 (`DECISIONS.md` §10.3). All are *addressed in draft — pending independent re-review*. |
| Date | 2026-11-13 (revision 5) |
| Codex owns | `agam-os/tests/**`, `agam-lang/tests/**`, `agam-dev/verify/**`, `agam-dev/reports/codex/**` |
| Paths | Relative to the repository root |

No project test, harness, runner or checker exists yet, and none has been run. What has been executed so far is listed in `DECISIONS.md` §8.3 and §8.5. Every git and tool command in this document is a proposal that has not been run as part of a gate. Timeouts and counts here are budgets chosen by the architect, not measurements.

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
| `agam-dev/reports/codex/ARCHITECTURE_REVIEW*.md` and their evidence folders | Codex | Architecture reviews of the document revisions |
| `agam-dev/gates/<milestone>/` | Owner | Gate record: manifests, runner summary, acceptance (§8) |

### 2.4 Identifiers and severities

| Prefix | Meaning |
|---|---|
| `T-<milestone>-nn` | Automated test in the runner's catalogue |
| `E-<milestone>-nn` | Manual evidence: a command run by hand, or an owner action, recorded with an evidence block |
| `RV-<milestone>-nn` | Mandatory review item, answered in Codex's review |

Severities: **Critical** and **Major** block acceptance; **Minor** should be fixed but does not block; **Informational** records a result. (These replace the High/Medium/Low names of earlier revisions: Critical and Major correspond to High and Medium, Minor to Low.)

### 2.5 Two kinds of verification

| Kind | What it is | When it can happen | Who |
|---|---|---|---|
| **Specification-level** | Reading the documents; checking them against each other and against primary sources; arithmetic; dry-running a procedure on paper or on synthetic data | Now, before any gate opens | Independent re-review (Codex), then the owner |
| **Implementation-level** | Running a `T-` test, producing `E-` evidence, or answering an `RV-` review item about real code | Only after the relevant gate opens and the code exists | Gate procedure of §8 |

A finding from an architecture review can be **resolved at specification level** by a re-review. That never means its implementation-level items have passed: those do not exist yet and stay `UNVERIFIED`. `DECISIONS.md` §10 lists both kinds separately for every finding.

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

Two things must agree:

- The **inventory** — Appendix A of this document: the complete list of test IDs with their suite and class. It belongs to the architecture documents, so neither agent can change it, and it is the only definition of what "all required tests" means.
- The **catalogue** — `agam-dev/verify/catalogue.json`, written by Codex: the runner's registration of each test, with its mode and implementation.

For every suite of a gate, the catalogue's set of (ID, class) pairs MUST equal the inventory's. An ID that is missing, duplicated, moved to another suite, given another class, or present without being in the inventory is an error (§4.4, §8.4).

Every test has an ID, a suite, a **class** and a **mode**:

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

**`ACCEPTED` always exists (AR-37).** The owner creates `agam-dev/gates/ACCEPTED` as an **empty file (0 bytes)** in the pre-agent baseline (`ROADMAP.md` M0-A1), so it is part of every `BASE` and of every tree the runner checks, including the first M0 attempt. An empty file means that no milestone has been accepted. A missing file is exit status 2; it is never read as an empty history. Format: one milestone ID per line, every line ended by LF, no blank line, no duplicate. Only the record commit of a gate changes it (§8.3).

### 4.3 Commands

```
python3 agam-dev/verify/run.py gate <MILESTONE> [--stretch] [--mem SIZE] [--matrix] [--repeat N]
python3 agam-dev/verify/run.py dev <suite>... [--only TEST_ID] [--stretch]
```

- `gate` selects every required test of §4.2 and requires the binding files of §4.6. It is the only command whose result counts for acceptance.
- `dev` is for an agent's own worktree. Its summary starts with `DEV RUN — not acceptance evidence`.
- `make -C agam-os test MILESTONE=<id>` is a convenience that calls `gate`.
- A `gate` run counts only when it is started as §8.2 step 8 prescribes: caches and build output removed first, and the interpreter options given on the command line. Started any other way, the runner refuses to run (§4.6 checks 4–6).
- `--matrix` and `--repeat` exist only from K5 (suite `k5` uses them internally).

### 4.4 Result accounting and exit status (normative)

One line per test: `PASS|FAIL|SKIP|MISSING <class> <test id> <name> (<seconds>s)[: <reason>]`. Then:

```
REQUIRED selected=<n> passed=<n> failed=<n> skipped=<n> missing=<n>
STRETCH  selected=<n> passed=<n> failed=<n> skipped=<n>
OPTIONAL selected=<n> passed=<n> failed=<n> skipped=<n>
```

- `MISSING`: a required test of a selected suite has no registered implementation.
- **Inventory mismatch**: before selecting anything, the runner compares the catalogue with Appendix A as found in the working tree, for every selected suite. Any difference listed in §4.1 is exit status 2 and no test is run. This is the candidate runner checking itself; the check that counts for acceptance is made with the trusted tools in §8.2 steps 7 and 8.
- **Exit status 0 only if** `REQUIRED selected > 0` **and** `passed == selected` (so `failed == skipped == missing == 0`).
- Exit status 1 if any required test failed, was skipped or is missing, or if no required test was selected.
- Exit status 2 for a usage or environment error, including a failed binding check (§4.6).
- Stretch and optional results never change the exit status.
- Artefacts go to `agam-os/build/test-results/<UTC timestamp>/`, with a `summary.json` holding every line above, the binding, and `check_env.sh --brief`.

### 4.5 Stretch triage

Every stretch `FAIL` produces a finding in Codex's review. The architect classifies it before acceptance: test defect; behaviour outside the normative text; or **violation of a normative requirement**, which becomes a blocking Major finding. Acceptance needs every stretch failure classified. A stretch test that was not run is recorded as `SKIPPED`, and the behaviour it covers stays `UNVERIFIED`.

### 4.6 Binding results to commits; inputs and outputs (normative)

A `gate` run reads `.gate/<MILESTONE>/gate.json` and `.gate/<MILESTONE>/tree.lst`, produced on Windows in §8:

- `gate.json` holds `BASE`, `TIP_C`, `TIP_X`, `GATE_HEAD` (full SHAs) and the SHA-256 of `claude.raw`, `codex.raw`, `merged.raw` and `tree.lst`.
- `tree.lst` is the byte-exact output of `git ls-tree -r -z --full-tree <GATE_HEAD>`.

Before running any test, the runner does the following without Git. Any failure is exit status 2.

1. **Binding file**: the SHA-256 of `tree.lst` equals the value in `gate.json`.
2. **Tracked content**: for every entry of `tree.lst`, the file exists, its blob ID recomputed from the working tree (SHA-1 of `blob <size>\0<content>`) equals the listed one, and the listed mode is `100644`. This also catches CRLF conversion.
3. **No untracked inputs**: every file in the working tree (outside `.git`) is either listed in `tree.lst` or lies in a **permitted output location**: `agam-os/build/` or `.gate/`. Any other file fails the run — an untracked source, header, test, module or report, a bytecode cache, or a file that an ignore rule hides from `git status`. Ignore rules play no part in this check.
4. **No bytecode caches**: no directory named `__pycache__` or `.pytest_cache`, and no file matching `*.pyc` or `*.pyo`, exists anywhere in the working tree, the permitted output locations included.
5. **Interpreter settings**: `sys.dont_write_bytecode` is true, and `sys.pycache_prefix` names an existing, empty directory outside the working tree.
6. **Outputs start empty**: `agam-os/build/` contains nothing except `agam-os/build/cache/`, and that directory contains only the downloaded files named in a `PIN` file, each re-verified against its pinned SHA-256 before use (T-M0-02). Of `.gate/`, the runner reads only `gate.json` and `tree.lst`.

The runner gives every Python child process the options `-B` and `-X pycache_prefix=<that directory>` **on its command line**, and every `pytest` process `-p no:cacheprovider` as well. It does not rely on environment variables for this: `-E` and `-I` make Python ignore every `PYTHON*` variable.

**Why caches are inputs, not outputs (AR-35).** Python executes a `.pyc` file in place of the source in three cases: a timestamp cache whose recorded size and modification time match the source file; an *unchecked* hash-based cache, which Python uses without looking at the source at all; and a `.pyc` that lies beside the sources where no source file of that name exists. Only a *checked* hash-based cache is compared with the content of the source. `pytest` keeps its own rewritten copies of test modules in the same `__pycache__` directories and reuses them under the timestamp rule, so a stale one can make a failing test pass. A stale or planted cache can therefore run bytes that are in no tree while every tracked file still matches `tree.lst`.

**What each setting does, and does not do:**

| Setting | Effect | Limit |
|---|---|---|
| `-B`, `PYTHONDONTWRITEBYTECODE`, `sys.dont_write_bytecode` | Python and `pytest` write no `.pyc` file | **Existing caches are still read and executed.** On its own this setting protects nothing. |
| `-X pycache_prefix=DIR`, `PYTHONPYCACHEPREFIX`, `sys.pycache_prefix` | Python and `pytest` read and write caches only under `DIR`, and ignore every `__pycache__` directory in the source tree | A `.pyc` beside the sources with no source file is still importable. A `DIR` that is not empty is itself a cache that is read. |
| `-p no:cacheprovider` | `pytest` neither reads nor writes `.pytest_cache`, its record of earlier runs | Does not concern bytecode |
| `-E`, `-I` | — | Make Python ignore `PYTHONDONTWRITEBYTECODE` and `PYTHONPYCACHEPREFIX`; the command-line options still apply |

The rule that follows has three parts, and none replaces another: **absence** — no cache and no sourceless `.pyc` in the tree (step 8a, check 4); **an empty prefix outside the tree** — so that no cache exists to be read (check 5); **no writing** — so that the prefix is still empty afterwards, which step 8c confirms. Checks 3–6 are made by the candidate runner and are a second line only. The first line is the owner's launch procedure (§8.2 step 8a), carried out with system tools **before any candidate Python code is imported**.

**Which worktree (AR-38).** These checks apply to the worktree in which `run.py gate` runs: the integration worktree, `D:\Agam-Computing\ecosystem`. Material that exists only in an agent's worktree is outside them; what the agent worktrees must satisfy is a clean `git status` when a tip is reported (§8.1). An ignore rule can satisfy that second rule. It cannot satisfy check 3: an ignored file that appears in the integration worktree fails the run.

**What this proves and what it does not.** Steps 1–3 prove that the files on disk are exactly the listed tree and nothing else. They do **not** prove which commit or branch that tree belongs to, that `gate.json` tells the truth about the refs, or that a ref has not moved since. Commit and ref freshness are established only by the live checks the owner makes with Git for Windows (§8.2 steps 0, 8 and 10).

`summary.json` repeats the four SHAs and the SHA-256 of `gate.json`. **A gate result is valid only for exactly those SHAs and that tool environment.** It is void if `main` no longer equals `BASE`, if either agent tip has a different SHA, if the merge was redone, if the tree fails the checks above, or if `check_env.sh --brief` differs from the recorded one.

### 4.7 Running kernel tests (`agam-os/tests/harness/qemu.py`, from K1)

1. Build with `make -C agam-os iso TEST=<case>` (or `iso` for the normal kernel). A build failure is a test failure.
2. Launch QEMU with the reference command of `AOS_KERNEL_SPEC.md` §11.5, serial to `serial.log`, plus `-d int,cpu_reset -D qemu.log`. Memory from `--mem` (default `256M`).
3. Timeout budget: 20 s by default; 60 s for `pmm_exhaust`; 120 s for 4 GiB runs. On timeout, kill QEMU and fail.
4. Save `cmd.txt`, `serial.log`, `qemu.log`, `exit_status`. Normalise `\r\n` to `\n`.
5. `exit` mode: pass only if the **host** exit status equals the expected one (33; 35 for tests that expect a failure result), every required pattern matches in order, and the log contains **exactly one** `@@AOS-TEST@@ RESULT` line, with the expected verdict. No `RESULT` line, or more than one, is a failure (`AOS_KERNEL_SPEC.md` §12.4).
6. `observe` mode: pass only if the required lines appear in order ending with `[aos] halt` within the budget; the harness then terminates QEMU. The exit device is not involved, and host status 33 is not expected.

### 4.8 Build once

Within one runner invocation each image is built at most once and cached by case name; `--repeat` and `--matrix` reuse it. Tests that rebuild from clean (`T-K1-01`, `T-K2-05`) run first, in a separate build directory, and never inside a repeat or matrix loop.

### 4.9 Parsers

`agam-os/tests/harness/aosdump.py` parses the formats of `AOS_KERNEL_SPEC.md` §12. A line that deviates from a normative format is itself a failure.

### 4.9 Active text files and byte-exact files (normative; AR-39)

Every tracked file belongs to exactly one of two sets.

- **Byte-exact set**: files that must keep the bytes they were created with — gate records and archived review evidence. They are the paths matched by the four patterns below, which `.gitattributes` marks `-text` (`ROADMAP.md` §2.2). Git never converts them; they may contain NUL bytes or CRLF; the SHA-256 values cited for them stay valid.

  ```
  agam-dev/gates/**
  agam-dev/reports/codex/ARCHITECTURE_REVIEW*.md
  agam-dev/reports/codex/architecture-review*-evidence/**
  agam-dev/reports/codex/proposals/00[1-3]-*.md
  ```

- **Active text set**: every other tracked file. These are stored and checked out with LF, and a carriage-return byte in one of them is an error.

Both sets are bound in the same way: the blob ID recomputed from the working tree must equal the one in `tree.lst` (§4.6 check 2). Only the line-ending rules — T-M0-03 and E-M0-03 — are limited to the active text set. Extending the byte-exact set is a change to this section and to `.gitattributes`, made by the architect and committed by the owner; an agent cannot do it.

Committing a file of the byte-exact set preserves a record. It is not acceptance of anything: it approves no document, closes no finding, clears no blocker and opens no gate (`DECISIONS.md` §3.6).

---

## 5. AOS test catalogue

Class is `required` unless marked **stretch** or **optional**. Expected host status is 33 for `exit` tests unless stated.

### M0 (suite `m0`)

| ID | Name | Method | Pass condition |
|---|---|---|---|
| T-M0-01 | env_tools | `sh agam-dev/verify/check_env.sh` | Exit 0; every tool of `AOS_KERNEL_SPEC.md` §2 reported with path and version; the script runs no `git` |
| T-M0-02 | pin_fetch | `sh agam-os/tools/fetch-limine.sh` twice | Asset and header SHA-256 equal the PIN files; second run reports already verified; the files of spec §11.2 exist |
| T-M0-03 | tree_integrity | Binding check of §4.6, plus a scan of every listed file | Blob IDs and modes match for every tracked file; no carriage-return byte in any tracked file of the active text set (§4.9) |
| T-M0-04 | runner_selftest | The runner executed on synthetic catalogues, inventories and trees | (a) all pass → 0; (b) a required `SKIP` → 1; (c) a required `MISSING` → 1; (d) empty required selection → 1; (e) a stretch `FAIL` → 0 and reported; (f) tree differs from `tree.lst` → 2; (g) a suite outside §4.2's set is not selected; (h) `tree.lst` hash differs from `gate.json` → 2; (i) an unlisted file outside the permitted output locations → 2, also when an ignore rule covers it; (j) catalogue differs from the inventory → 2; (k) a `__pycache__` directory, a `.pytest_cache` directory or a `.pyc` file anywhere in the tree → 2; (l) started with bytecode writing enabled, or without an empty cache prefix outside the tree → 2; (m) `agam-dev/gates/ACCEPTED` missing → 2, and an empty one read as no accepted milestone; (n) a file under `agam-os/build/` outside `cache/` at the start → 2; (o) every Python child process the runner starts has both settings, also when the two environment variables are unset |
| T-M0-05 | ownership_selftest | `check_ownership.py` (§8.3) on synthetic manifests and record inputs | Accepts valid manifests, including a deletion (zero new mode and blob); rejects a foreign path, a rename into a foreign path (seen as delete plus add), a mode other than `100644`, a symlink or submodule entry, a type change, overlapping manifests, and a merge manifest that differs from the union in path, blob or mode. `--verify-record` accepts the exact record delta with its judged directory, for a first record (empty old `ACCEPTED`) and a later one; it rejects: any other path; a missing required file; an extra file in the record directory; `tools-reading-architect.md` missing when required or present when not; an archived file whose blob differs from the judged file (for example a substituted summary); a `gate.json` whose digests do not match the manifests or the tree listing; a summary that names another `gate.json`; an `accepted.old` whose blob differs from the raw record (wrong prefix); and an `ACCEPTED` that drops or changes an earlier line, appends another milestone, appends two lines, repeats a milestone, or lacks the final line ending. |
| T-M0-06 | make_skeleton | `make -C agam-os print-flags` | Exit 0; prints every flag of spec §2.1 and §2.2 |
| T-M0-07 | inventory_selftest | `check_inventory.py` on synthetic inventories, catalogues and summaries | `catalogue` mode rejects: a required ID removed; an ID duplicated; a required ID changed to stretch; an ID moved to another suite; an ID not in the inventory. `summary` mode rejects: a required ID absent; a required ID not `PASS`; a binding that differs from `gate.json`. Both accept the matching case. |

| ID | Manual evidence |
|---|---|
| E-M0-01 | Corrupting the cached Limine asset makes `fetch-limine.sh` exit non-zero |
| E-M0-02 | Git for Windows: `git config user.name`, `user.email` and `git log --format="%an <%ae> / %cn <%ce>%n%B"` over the bootstrap commits show only the identity of `DECISIONS.md` O-18 and no attribution text |
| E-M0-03 | Git for Windows, in all three worktrees: `git ls-files --eol` shows `i/lf w/lf` for every file of the active text set (§4.9). Every file of the byte-exact set shows the attribute `-text`, and `git hash-object --no-filters <file>` prints the blob ID that `git ls-files -s <file>` shows. |
| E-M0-04 | The versions printed by T-M0-01 compared with `DECISIONS.md` §8.2; differences accepted by the owner |
| E-M0-05 | Git for Windows: `git worktree list` and `git branch --show-current` in each worktree |
| E-M0-06 | The owner's recorded disposition of the Limine pin evidence (B-02) |
| E-M0-07 | **M0 bootstrap check, by hand** (§8.6): from readable `git diff --raw --no-abbrev --no-renames` output for each tip and for the merge, the owner checks and files a table of every record — path ownership, status, modes, blob IDs, no overlap, and merge records equal to the union |
| E-M0-08 | By hand: the `m0` entries of the candidate catalogue compared with Appendix A |
| E-M0-09 | **M0 record check, by hand** (§8.6): the listing of `git diff --raw --no-abbrev --no-renames <GATE_HEAD> <GATE_REC>` shows exactly the files of §8.3 rule 1; for each archived file, `git hash-object --no-filters .gate/M0/<name>` prints the new blob ID of its record; the old blob of `ACCEPTED` is the empty blob `e69de29bb2d1d6434b8b29ae775ad8c2e48c5391`, and its new blob equals `git hash-object --no-filters` of a file that contains exactly `M0` and one LF |
| E-M0-10 | **Cache control**, in a scratch copy of the tree, never in a worktree: a stale `.pyc` is planted for one runner module (source unchanged, same size and timestamp). With the launch procedure of §8.2 step 8a the cache is found and removed before the run; with step 8a skipped, the runner exits 2 by check 4 of §4.6 |
| RV-M0-01 | Independent reading of every candidate gate tool (`check_ownership.py`, `check_inventory.py`, `gate_manifest.ps1`) and of the runner's selection, accounting and binding code against §4 and §8, by parties other than their author: **two separate readings, the architect's and Claude Code's**, filed as §8.1 describes (O-26) |

### K1 (suite `k1`)

| ID | Name | Mode | Pass condition |
|---|---|---|---|
| T-K1-01 | build_clean | host | `make -C agam-os distclean iso` exits 0 with no `warning:` |
| T-K1-02 | elf_header | host | ELF64, X86-64, type EXEC, entry ≥ `0xffffffff80000000` |
| T-K1-03 | elf_segments | host | Exactly three LOAD segments, flags `R E`, `R`, `RW`, each page-aligned, none W+E; no `INTERP` or `DYNAMIC` header; no `.rela*`/`.rel*` section to apply |
| T-K1-04 | no_fp_simd | host | In `llvm-objdump -d` output of the kernel (tool name resolved as in spec §2): no operand naming `%xmm`, `%ymm`, `%zmm`, `%mm0`–`%mm7`, `%st` or `%k0`–`%k7`; and no x87, MMX, SSE/AVX, or state-save mnemonic (`f…` x87 forms, `emms`, `fxsave`/`fxrstor`, `xsave…`/`xrstor…`, `ldmxcsr`/`stmxcsr`, VEX/EVEX-encoded forms). Codex derives the exact pattern from the installed tool's real output and records it. |
| T-K1-05 | boot_banner | observe | Banner, `limine:`, `hhdm:`, `kernel:`, `cpu: nx enabled`, `init complete`, `halt` in order and in the formats of spec §12.1; HHDM offset ≥ `0xffff800000000000`; kernel virt ≥ `0xffffffff80000000` |
| T-K1-06 | smoke | exit | `BEGIN smoke`, `RESULT PASS smoke` |
| T-K1-07 | panic_exit | exit, host 35 | Panic block with `reason: test panic 42`, then `RESULT FAIL panic_exit` |
| T-K1-08 | smoke_stability | exit | T-K1-06 ten times from one built image |
| T-K1-09 | exit_convention | host | On the owner's QEMU: `smoke` gives host 33; `panic_exit` gives 35; the normal kernel run in `exit` mode is reported as a timeout failure; a recorded host status 0 is classified as failure. On synthetic logs: a log with no `RESULT` line, and a log with two, are each classified as failure even with host status 33. |
| T-K1-10 | kt_check_fail_return | exit, host 35 | A failed `KTEST_CHECK` followed by a normal return: `CHECK-FAILED`, then `RESULT FAIL` |
| T-K1-11 | requests_layout | host | From `llvm-nm`/`llvm-readelf` (names resolved as in spec §2): start marker, requests and end marker lie in that order inside the R+W segment, below `__bss_start`; symbols ordered and aligned as in spec §4.2; `__kernel_end ≤ 0xffffffffc0000000` |

| ID | Manual evidence / review |
|---|---|
| E-K1-01 | Resolution of V-01…V-05, V-12, V-14 with evidence blocks |
| E-K1-02 | **Weakened-checker control** (§8.6), in a scratch clone only: a tip that replaces `check_ownership.py` with one that accepts everything, and also changes a path it does not own, is taken through §8.2 steps 0–3; the checker exported from `BASE` must reject it |
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
| RV-K3-03 | The ending guard of spec §12.4: every event (normal completion, `panic`, each arriving vector including #BP) checks it before producing any output; nothing after the check can start a second ending sequence — mandatory because an interrupted ending sequence cannot be provoked from test code |

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
| T-K5-01 | matrix_128M | Every required `exit`/`observe` test of `k1`–`k4` with 128 MiB of guest memory |
| T-K5-02 | matrix_256M | Every required `exit`/`observe` test of `k1`–`k4` with 256 MiB of guest memory |
| T-K5-03 | matrix_1G | Every required `exit`/`observe` test of `k1`–`k4` with 1 GiB of guest memory |
| T-K5-04 | matrix_4G | Every required `exit`/`observe` test of `k1`–`k4` with 4 GiB of guest memory; in addition, at least one usable memory-map entry has base ≥ `0x100000000` |
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
| T-L1-04 | Literals | Numbers and text per §4.4–§4.5, including `1e3`, leading zeros, the 1,000-digit boundary (accepted) and 1,001 digits (E0107), decimal overflow (E0107); a text literal of exactly 1,000,000 characters (accepted) and of 1,000,001 (E0108, span on the opening quote); first-problem order inside a literal (E0106 before a later E0108 or E0102) |
| T-L1-05 | Lexical errors | E0100–E0108 with code, variant, line, column; E0100 offset with and without a byte-order mark |
| T-L1-06 | Fusion tokens | `tokens --canonical` identical across each keyword-substitution triple; `IDENT` lines carry the name |
| T-L1-07 | Lexer fuzz | 500 seeded inputs raise only `AgamError` |
| T-L2-01 | Golden ASTs | At least one per grammar production; default JSON contains no span, origin or language counts; `--spans` output for the cases of spec §6.5, including Appendix C.10 exactly |
| T-L2-02 | Precedence | ≥ 20 expressions |
| T-L2-03 | Syntax errors | E0201–E0214 with code, line, column; one case for every row of spec §5.4, including E0201 for each of the seven header forms and E0203 for an `INDENT` on the first line, at top level and inside a block; the outcomes of Appendix C.15; E0208 for each case (a)–(c) of §5.2 and raised by the parser; E0210 for `stop` in a function nested in a loop; error precedence of §5.2; E0214 at every boundary of spec §5.3 and Appendix C.11 — 100 levels accepted and 101 rejected for each of the five bracket uses, for blocks and for a mixture, the unclosed-bracket case, and E0214 taking precedence at the first token of a block; a flat chain of 1,000 operators and 200 successive index operations parse (§7.10) |
| T-L2-04 | Fusion ASTs | Default AST JSON identical across each keyword-substitution triple |
| T-L2-05 | Parser fuzz | 500 seeded mutations raise only `AgamError` |
| T-L3-01 | Operators | Every row of §7.4 for the types available (numbers, text, booleans, nothing); unsupported combinations give the stated E0302 variant; a flat sum with 1,000 `+` evaluates (§7.10) |
| T-L3-02 | Display forms | Every available value type |
| T-L3-03 | Control flow | if / else-if / else, while, repeat, stop, skip |
| T-L3-04 | Runtime errors | E0302c, E0401, E0404 |
| T-L3-05 | CLI | Exit statuses; stdout/stderr separation; `--version`; `--messages` on every command; `--max-steps` validation |
| T-L3-06 | Numeric closure | E0406 (negative base, fractional exponent); E0401 for `0 ** -1`; E0407 for decimal overflow, whole-number overflow at the 1,000-digit boundary, and conversion of an oversized whole number; `10 ** 999` accepted and `10 ** 1000` rejected; whole/decimal comparison exact |
| T-L4-01 | Functions | Arguments, return, recursion, the closure example of §7.9, E0303, E0304, E0213 |
| T-L4-02 | Lists and text | Literals, indexing, stores, E0305, E0306a–c, E0302d, for loops, snapshot iteration |
| T-L4-03 | Built-ins | Each built-in under both names; each error of §7.11; `number()` grammar accept/reject table; `ask` with `.in` and E0405 |
| T-L4-04 | Limits | E0402 without a traceback; E0403; `range` and concatenation at exactly 1,000,000 (accepted) and one more (E0408); a list literal with 1,000,001 items raises E0408 **before its first item is evaluated** (the first item is a call with a visible effect that must not occur); display longer than the limit (E0408); every row of Appendix C.11 |
| T-L4-05 | Examples | Every file in `agam-lang/examples/` exits 0 |
| T-L4-06 | Built-in alias behaviour | Each `builtin_alias` pair gives identical output and status; `length == lambai` is `true` |
| T-L4-07 | No cycles | E0308 for `append(a, a)`, `a[0] = a`, an indirect cycle through a nested list, and an augmented indexed store; after each error the list's display form is unchanged; sharing without a cycle is accepted; a value of depth 5,000 built in a loop can be appended (no limit takes part in the cycle rule) and an indirect cycle through it is still E0308 |
| T-L4-08 | Equality and display | §7.5 table including `[true] == [1]` false and function identity; text escaping inside lists; depth 100 displayed and compared, depth 101 (built with `append`, since a literal that deep is E0214) gives E0408; expanded count exactly 1,000,000 compared, 1,000,001 gives E0408 even when the lengths differ; a list above the count limit equals itself; the shared-list example of §7.5 (`a == a` true, `a == b` E0408) |
| T-L4-09 | Evaluation order | Programs whose functions record their call order: binary operands, arguments, `a[f()] = g()` (g first), `a[f()] += g()` (f first, once), `show`, short-circuit, error precedence of §7.3 rule 10 |
| T-L5-01 | Catalogue completeness | The implementation's codes equal `codes_expected.json`; both catalogues equal `messages_expected.json`; every code E01xx–E04xx is raised by at least one program test |
| T-L5-02 | Rendered diagnostics | Appendix C.1–C.9 and C.12–C.14 reproduced byte for byte; further golden texts written by hand from Appendix A and B, English and Hinglish, covering one row per catalogue variant, tab expansion, zero-width spans, end of file, diagnostics without a span, and keywords quoted as typed |
| T-L5-03 | Suggestions | 10 misspellings suggested; 5 unrelated words not; tie-break and case-only cases of §8.4 |
| T-L5-04 | Message language | `auto` by part counts, including a mixed phrase, a lexical error part-way through a file, and E0100 |
| T-L5-05 | Whole-program fuzz | 2,000 seeded cases; exit status only 0, 1 or 2; no traceback text; never 70 |
| T-L5-06 | Fusion corpus | ≥ 15 keyword-substitution triples covering every keyword, phrase and else-if form: token, AST and output identity |
| T-L5-07 | Internal error | Calling `agamlang.cli.main` in-process with a raising `_fault`: the output of Appendix C.7, exit status 70, no traceback. No command-line or environment route to this exists (checked by review RV-L5-01). |

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

- All `git` commands are run by the **owner**, with Git for Windows, in `D:\Agam-Computing\ecosystem`. Agents never run them. `sh`, `python3` and `make` run in WSL2. Git-free Python tools may run on either side.
- **`BASE`**: the full SHA of `main` when the milestone starts. It is recorded then and does not change during the milestone. If `main` has to change, the milestone restarts from the new `BASE`: both agents merge it and every earlier manifest and result is void.
- **`TIP_C`, `TIP_X`**: the exact full SHAs of `agent/claude` and `agent/codex` submitted for the gate, each reported from a clean worktree.
- **`gate/<milestone>`**: a temporary branch created from `BASE`. `main` is never used for trial merges.
- **`GATE_HEAD`**: the commit that was tested. **`GATE_REC`**: the commit that is promoted — `GATE_HEAD` plus the gate record.
- **Trusted tools (`TOOLS`)**: `check_ownership.py`, `check_inventory.py`, `gate_manifest.ps1` and every file they import — all under `agam-dev/verify/` — **as they exist in `BASE`**, together with `docs/TEST_PLAN.md` at `BASE` (for the inventory). The owner exports them to a directory **outside every worktree** (`D:\Agam-Computing\.gate-tools\<BASE>\`; O-27), byte-exactly from Git's object store, checks each exported file's blob ID against `BASE`, and makes the directory read-only. **Every checker call below uses `TOOLS`, never a path inside a worktree**, so a tip cannot replace the tool that judges it (AR-25).
- **The trusted directory stays exact.** It contains exactly the exported files and nothing else; a listing that differs from the `git ls-tree` list of step 1 fails. Trusted tools are run as `python -B …`, so that no cache file is ever added to it; and because it holds nothing but the exported files, no cache exists there to be read.
- **Candidate tools**: the runner and catalogue at `GATE_HEAD`. They execute the tests; their catalogue and their summary are judged by `TOOLS`.
- **Verification-path change**: any record under `agam-dev/verify/` in Codex's manifest. It needs review item `RV-<milestone>-VP` before the trial merge: **two separate readings of that diff, one by the architect and one by Claude Code** (O-26). Each reader is a party other than the author, names the exact Codex tip it read, and must leave no Critical or Major finding open; neither reading replaces the other. Claude Code reads with read-only commands, never edits `agam-dev/verify/`, and commits its reading under `agam-dev/reports/claude/` before it submits its own tip — so Codex reports its tip first whenever verification tools changed. The architect's reading is given to the owner, who stores it in the gate record. The changed tools become trusted only when they are part of a later `BASE`.

### 8.2 Steps

```
# 0. Live state — Git for Windows; repeated in steps 8 and 10
git rev-parse main agent/claude agent/codex              # must print BASE, TIP_C, TIP_X
git status --porcelain=v1 --untracked-files=all          # must print nothing (ignored files are not shown here;
                                                         #  in this worktree the runner rejects them, §4.6 check 3)
git merge-base --is-ancestor <BASE> <TIP_C>              # exit 0
git merge-base --is-ancestor <BASE> <TIP_X>              # exit 0

# 1. Trusted tools exported from BASE (from K1 on; for M0 see §8.6)
git ls-tree -r <BASE> -- agam-dev/verify docs/TEST_PLAN.md   # paths and blob IDs
git cat-file blob <blob id>                                  # each file, captured byte-exactly into TOOLS
git hash-object --no-filters <exported file>                 # must print the same blob ID

# 2. Per-agent manifests: every change from BASE to the tip, with modes and blob IDs
git diff --raw -z --no-abbrev --no-renames --output=.gate/<M>/claude.raw <BASE> <TIP_C>
git diff --raw -z --no-abbrev --no-renames --output=.gate/<M>/codex.raw  <BASE> <TIP_X>

# 3. Ownership and form of each manifest
python <TOOLS>/agam-dev/verify/check_ownership.py --agent claude --manifest .gate/<M>/claude.raw
python <TOOLS>/agam-dev/verify/check_ownership.py --agent codex  --manifest .gate/<M>/codex.raw
#    a verification-path change in codex.raw: both readings of RV-<M>-VP must be filed before step 4

# 4. Trial merge on the gate branch
git switch -c gate/<M> <BASE>
git merge --no-ff -m "gate <M>: merge agent/claude" <TIP_C>
git merge --no-ff -m "gate <M>: merge agent/codex"  <TIP_X>
git rev-parse HEAD                                       # GATE_HEAD

# 5. Merge result against the two approved manifests
git diff --raw -z --no-abbrev --no-renames --output=.gate/<M>/merged.raw <BASE> <GATE_HEAD>
python <TOOLS>/agam-dev/verify/check_ownership.py --verify-merge .gate/<M>/claude.raw .gate/<M>/codex.raw .gate/<M>/merged.raw

# 6. Binding files
git ls-tree -r -z --full-tree <GATE_HEAD>                # captured byte-exactly into .gate/<M>/tree.lst
#    gate.json: BASE, TIP_C, TIP_X, GATE_HEAD, and the SHA-256 of claude.raw, codex.raw, merged.raw, tree.lst

# 7. Candidate catalogue against the trusted inventory
python <TOOLS>/agam-dev/verify/check_inventory.py catalogue --test-plan <TOOLS>/docs/TEST_PLAN.md \
       --catalogue agam-dev/verify/catalogue.json --milestone <M> --accepted agam-dev/gates/ACCEPTED

# 8. Run — WSL2, in /mnt/d/Agam-Computing/ecosystem, all of it in one shell
# 8a. Before any candidate code runs: system tools only
find . -path ./.git -prune -o \( -name __pycache__ -o -name .pytest_cache -o -name '*.pyc' -o -name '*.pyo' \) -print
find agam-os/build -mindepth 1 -not -path 'agam-os/build/cache' -not -path 'agam-os/build/cache/*' -print
#     both must print nothing on standard output (a missing agam-os/build is fine).
#     Whatever they print is deleted by the owner, and both are repeated.
export PYTHONDONTWRITEBYTECODE=1
export PYTHONPYCACHEPREFIX="$(mktemp -d)"                # new, empty, outside every worktree
# 8b. The run. The options repeat the variables on purpose: -E or -I would switch the variables off, not the options.
python3 -B -X pycache_prefix="$PYTHONPYCACHEPREFIX" agam-dev/verify/run.py gate <M>
# 8c. After the run: system tools only
find . -path ./.git -prune -o \( -name __pycache__ -o -name .pytest_cache -o -name '*.pyc' -o -name '*.pyo' \) -print
find "$PYTHONPYCACHEPREFIX" -type f -print
#     both must print nothing; otherwise the result is void
cp <summary.json written by the run> .gate/<M>/summary.json
# 8d. Windows: judge the copy that will be archived, then re-check live state
python -B <TOOLS>/agam-dev/verify/check_inventory.py summary --test-plan <TOOLS>/docs/TEST_PLAN.md \
       --summary .gate/<M>/summary.json --gate .gate/<M>/gate.json --milestone <M> --accepted agam-dev/gates/ACCEPTED
#    repeat step 0; additionally: git rev-parse gate/<M>  must print GATE_HEAD

# 9. After reviews and the owner's acceptance: the record commit — one commit on gate/<M>, parent GATE_HEAD.
#    It adds, under agam-dev/gates/<M>/, a byte-for-byte copy of each of these files of .gate/<M>/ :
#      claude.raw  codex.raw  merged.raw  tree.lst  gate.json  summary.json  record.md
#      tools-reading-architect.md      — exactly when <M> is M0 or codex.raw has a record under agam-dev/verify/
#    and it appends the line <M> to agam-dev/gates/ACCEPTED. Nothing else.
git rev-parse HEAD                                       # GATE_REC
git rev-list --parents -n 1 <GATE_REC>                   # must print exactly: <GATE_REC> <GATE_HEAD>
git diff --raw -z --no-abbrev --no-renames --output=.gate/<M>/record.raw <GATE_HEAD> <GATE_REC>
git cat-file blob <old blob ID of agam-dev/gates/ACCEPTED in record.raw>     # captured byte-exactly into .gate/<M>/accepted.old
python <TOOLS>/agam-dev/verify/check_ownership.py --verify-record .gate/<M>/record.raw --milestone <M> \
       --judged .gate/<M> --accepted-old .gate/<M>/accepted.old
python <TOOLS>/agam-dev/verify/check_inventory.py summary --test-plan <TOOLS>/docs/TEST_PLAN.md \
       --summary .gate/<M>/summary.json --gate .gate/<M>/gate.json --milestone <M> --accepted .gate/<M>/accepted.old
#    the second command judges again the summary that step 9 has just proved to be the archived one

# 10. Immediately before promotion: repeat step 0, with gate/<M> now equal to GATE_REC; then
git switch main
git merge --ff-only gate/<M>
git tag -a accepted-<M> -m "<M>: BASE <sha>, TIP_C <sha>, TIP_X <sha>, tested <GATE_HEAD>, promoted <GATE_REC>"
git branch -d gate/<M>
```

`.gate/` is untracked and ignored. `gate_manifest.ps1` automates the byte-exact captures of steps 1, 2, 5, 6 and 9 with read-only git commands; like the checkers, it is run from `TOOLS`. `record.md` is written by the owner, and the architect's reading is placed, into `.gate/<M>/` before they are copied, so that every archived file has a judged original there. These commands are proposals. Read-only forms of `git ls-tree`, `git diff --raw`, `git cat-file blob` and `git hash-object --no-filters` were exercised by the second and third reviews (evidence R3-E03, R4-E03); the `find` and interpreter settings of step 8 were exercised by the architect on a scratch fixture (`DECISIONS.md` §8.3). No gate step has been run as a gate.

### 8.3 What `check_ownership.py` verifies (Git-free)

For each agent manifest:

- every record's path belongs to that agent's set in `ARCHITECTURE.md` §6.1 — for additions, modifications **and deletions**;
- because rename detection is off, a rename appears as a deletion plus an addition, and **both** paths must belong to the agent;
- status is `A`, `M` or `D` only (a type change fails);
- for `A` and `M`, the new mode is `100644`; symbolic links (`120000`) and submodules (`160000`) fail; for `D`, the new mode and blob are all zeros, which is valid;
- the two agent manifests have no path in common.

For the merge (`--verify-merge`): the set of records in `merged.raw` equals the union of the two agent manifests **record for record** — same path, status, old and new mode, old and new blob ID. Blob IDs identify contents, so this covers contents, modes, additions and deletions. Any extra, missing or different record fails; so does a merge conflict in step 4.

For the record commit (`--verify-record`; AR-34). Inputs: `record.raw` (the delta `GATE_HEAD` → `GATE_REC`), the milestone, the judged directory `.gate/<M>/`, and `accepted.old`. The check is Git-free: a blob ID is recomputed as the SHA-1 of `blob <size>\0<content>`, as in §4.6. A raw record carries paths, statuses, modes and blob IDs but no file content, which is why the content inputs are passed explicitly and each is tied to a blob ID of the record.

1. **Exact file set.** `record.raw` contains exactly one `A` record with mode `100644` for each required file under `agam-dev/gates/<M>/` — `claude.raw`, `codex.raw`, `merged.raw`, `tree.lst`, `gate.json`, `summary.json`, `record.md`, and `tools-reading-architect.md` exactly when `<M>` is `M0` or `codex.raw` holds a record under `agam-dev/verify/` — and exactly one `M` record for `agam-dev/gates/ACCEPTED` (`100644` before and after). A missing file, an extra file, or any other path, status or mode fails.
2. **Archived bytes are the judged bytes.** For each archived file, the new blob ID in `record.raw` equals the blob ID recomputed from the file of the same name in `.gate/<M>/`.
3. **The judged files belong together.** The SHA-256 values in `gate.json` equal those recomputed from `claude.raw`, `codex.raw`, `merged.raw` and `tree.lst`; `summary.json` names the same four SHAs and the SHA-256 of that `gate.json`; `record.md` and the reading, when required, are not empty.
4. **`ACCEPTED` grows by exactly one line.** The blob ID recomputed from `accepted.old` equals the old blob ID in `record.raw`. `accepted.old` is empty, or a sequence of LF-terminated milestone IDs without blank lines or duplicates, and does not contain `<M>`. The blob ID recomputed from `accepted.old` followed by `<M>` and one LF equals the new blob ID in `record.raw`. The new content is thereby proved without being read, and every earlier line is preserved. At the first gate `accepted.old` is the empty file created in the baseline (§4.2).

Step 9 then runs `check_inventory.py summary` once more on `.gate/<M>/summary.json`, whose bytes rule 2 has just shown to be the archived ones. What is archived is therefore exactly what passes judgement at record time, even if a file under `.gate/<M>/` was changed by mistake after step 8.

**Why the tested and the promoted commit may differ without a second test run.** The record delta holds two kinds of file. The files under `agam-dev/gates/<M>/` are read by no runner, test or checker. `ACCEPTED` **is** read — by the runner (§4.2) and by both inventory modes (§8.4) — and is the one deliberate exception: its only permitted change is the appended current milestone. That line cannot change what the finished run had to execute, because the run and its summary were judged against the old content; it takes effect from the next gate.

### 8.4 What `check_inventory.py` verifies (Git-free)

Its inventory is Appendix A of the **trusted** `TEST_PLAN.md`; its suite set is §4.2 for `<M>` plus the suites of the milestones in `ACCEPTED`.

- `catalogue` mode: for every suite in the set, the catalogue's (ID, class) pairs equal the inventory's. It fails on an ID that is missing, duplicated, in another suite, of another class, or not in the inventory.
- `summary` mode: the summary's SHAs equal `gate.json`'s; every **required** inventory ID of the set appears exactly once with `PASS`; no required ID has another status; every stretch or optional line names an inventory ID of that class; the totals agree with the lines.

Together with step 3 this means a tip can neither drop a required test from the denominator nor weaken the check that would notice (AR-26).

### 8.5 Failure and stale results

- On any failure: `git merge --abort` if needed, `git switch main`, `git branch -D gate/<M>`. `main` is untouched. The owning agent fixes its branch, which produces a new tip, and the gate restarts from step 0 with new manifests.
- Manifests, binding files and runner results are valid only for the exact `BASE`, `TIP_C`, `TIP_X` and `GATE_HEAD` they name. Nothing is carried over from an earlier attempt.
- Freshness is checked **three times** with live refs (steps 0, 8 and 10). The Git-free tree check of §4.6 complements this; it does not replace it.

### 8.6 The M0 exception, and the control that follows it

At the M0 gate no trusted tools exist, because M0 creates them. Their place is taken by:

1. **E-M0-07** — the owner's complete check by hand, for each tip and for the merge, using readable `git diff --raw --no-abbrev --no-renames` output: ownership of every path, status letters, modes, blob IDs, no overlap, and merge records equal to the union. M0 changes about a dozen bootstrap files, which keeps this practical. The archived review material committed under O-31 adds about fifty records to Codex's manifest; they are all additions under `agam-dev/reports/codex/` and are checked in the same way. The table of records is filed.
2. **E-M0-08** — by hand: the `m0` entries of the candidate catalogue against Appendix A.
3. **RV-M0-01** — two independent readings of the candidate gate tools and of the runner's selection, accounting and binding code, one by the architect and one by Claude Code, neither being their author (O-26).
4. The candidate tools' own self-tests (T-M0-04, T-M0-05, T-M0-07) run in the gate. They corroborate the reading; they are not the basis of trust.
5. **E-M0-09** — the record commit of M0 checked by hand against the four rules of §8.3, using Git's own blob IDs in place of the checker.
6. **E-M0-10** — the cache control, once, in a scratch copy.

Step 8a of §8.2 uses system tools only, so it applies at M0 unchanged.

From K1 on, `TOOLS` exists. **E-K1-02** then demonstrates the protection once, in a scratch clone: a tip that replaces the checker with one that accepts everything, and also changes a path it does not own, must be rejected by the checker exported from `BASE`.

### 8.7 Acceptance

The milestone is accepted only when: steps 0–8 succeeded for one set of SHAs; the runner exited 0 **and** `check_inventory.py summary` accepted its summary; all `E-` and `RV-` items of the milestone are filed; no Critical or Major finding is open; every stretch failure is classified (§4.5); and the owner says so. Steps 9 and 10 follow. **Only the owner advances `main`**, by fast-forward, and only the owner tags and pushes. Both the tested SHA (`GATE_HEAD`) and the promoted SHA (`GATE_REC`) are recorded, in the annotated tag and as the next milestone's `BASE`.

Every commit, including the merge and record commits, uses the identity of O-18 and carries no attribution text.

---

## 9. Known gaps (accepted for now)

- No real-hardware testing; no KVM; TCG only.
- No multi-processor, timing or interrupt-latency tests.
- No kernel code-coverage measurement.
- No continuous integration service (Q-11); the gate is run by hand.
- UEFI boot is optional (Q-03).
- Double-fault delivery, write-protection and no-execute enforcement, and invalid-argument panics of the allocator are covered only by stretch tests (O-09, O-19) plus mandatory review; without those runs the behaviour ships as `UNVERIFIED`.
- An interrupted ending sequence of a test kernel is covered by review only (RV-K3-03).
- The trusted tools judge the manifests, the catalogue and the summary. That the candidate runner really executed each test it reports rests on the review of verification-path changes (`RV-<milestone>-VP`) and on the architect's spot checks, not on a mechanical proof.
- The runner passes the interpreter options to its child processes on the command line. A candidate tool that deliberately started a child without them would not be caught mechanically; that is excluded by the two readings of every verification-path change.
- The release signature of the Limine asset is not verified (V-13).
- AgamLang performance is not tested, and its limits are provisional policy, not measured guarantees (O-14).

---

## Appendix A. Test inventory (normative; machine-readable; AR-26)

This is the only definition of which tests exist and which are required. It has one line per test: suite, ID, class. It MUST agree with the tables of §5 and §6; a difference between them is a defect of this document. Manual evidence (`E-`) and review (`RV-`) items are not in the inventory: they are listed per milestone in §5 and §6 and are checked by the owner at acceptance (§8.7).

Tools read the lines between the two marker comments and ignore blank lines, code-fence lines and lines starting with `#`.

<!-- inventory:begin -->
```
# suite id class
m0 T-M0-01 required
m0 T-M0-02 required
m0 T-M0-03 required
m0 T-M0-04 required
m0 T-M0-05 required
m0 T-M0-06 required
m0 T-M0-07 required
k1 T-K1-01 required
k1 T-K1-02 required
k1 T-K1-03 required
k1 T-K1-04 required
k1 T-K1-05 required
k1 T-K1-06 required
k1 T-K1-07 required
k1 T-K1-08 required
k1 T-K1-09 required
k1 T-K1-10 required
k1 T-K1-11 required
k2 T-K2-01 required
k2 T-K2-02 required
k2 T-K2-03 required
k2 T-K2-04 required
k2 T-K2-05 required
k3 T-K3-01 required
k3 T-K3-02 required
k3 T-K3-03 required
k3 T-K3-04 required
k3 T-K3-05 required
k3 T-K3-06 required
k3 T-K3-07 stretch
k3 T-K3-08 stretch
k3 T-K3-09 stretch
k3 T-K3-10 required
k3 T-K3-11 required
k3 T-K3-12 required
k3 T-K3-13 required
k3 T-K3-14 required
k3 T-K3-15 required
k3 T-K3-16 required
k4 T-K4-01 required
k4 T-K4-02 required
k4 T-K4-03 required
k4 T-K4-04 required
k4 T-K4-05 required
k4 T-K4-06 required
k4 T-K4-07 required
k4 T-K4-08 required
k4 T-K4-09 stretch
k4 T-K4-10 required
k4 T-K4-11 required
k4 T-K4-12 stretch
k5 T-K5-01 required
k5 T-K5-02 required
k5 T-K5-03 required
k5 T-K5-04 required
k5 T-K5-05 required
k5 T-K5-06 optional
l1 T-L1-01 required
l1 T-L1-02 required
l1 T-L1-03 required
l1 T-L1-04 required
l1 T-L1-05 required
l1 T-L1-06 required
l1 T-L1-07 required
l2 T-L2-01 required
l2 T-L2-02 required
l2 T-L2-03 required
l2 T-L2-04 required
l2 T-L2-05 required
l3 T-L3-01 required
l3 T-L3-02 required
l3 T-L3-03 required
l3 T-L3-04 required
l3 T-L3-05 required
l3 T-L3-06 required
l4 T-L4-01 required
l4 T-L4-02 required
l4 T-L4-03 required
l4 T-L4-04 required
l4 T-L4-05 required
l4 T-L4-06 required
l4 T-L4-07 required
l4 T-L4-08 required
l4 T-L4-09 required
l5 T-L5-01 required
l5 T-L5-02 required
l5 T-L5-03 required
l5 T-L5-04 required
l5 T-L5-05 required
l5 T-L5-06 required
l5 T-L5-07 required
```
<!-- inventory:end -->

Totals: 91 tests — 85 required, 5 stretch, 1 optional.
