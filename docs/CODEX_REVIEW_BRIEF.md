# Focused Re-review Brief for Codex — Draft v0.1, revision 4

| Field | Value |
|---|---|
| Status | Review request for **Draft v0.1, revision 4** (2026-11-13), a document-only correction pass |
| Reviewer | Codex, independent of the architect and of Claude Code |
| Previous reviews | **First**: `agam-dev/reports/codex/ARCHITECTURE_REVIEW.md` — revision 2 at commit `349c48a46d49cfac725b687868173d9423930309`; SHA-256 `b3f923ecaec34b9a2a318a37cd0ed3235627479618b4bee81ce5ad2cefeb86a9`. **Second**: `agam-dev/reports/codex/ARCHITECTURE_REVIEW_R3.md` — revision 3 at commit `fd2670d7e9c664c8b1b3cd14dfee3c46d4a45299`; SHA-256 `a1b51c9a8fb2f08a05b8a52dd6c695ced74e937c33952e06304e7599629eaab7`; 19 Resolved and 5 Partially Resolved of AR-01…24; new AR-25…33 (7 Major, 2 Minor) |
| Purpose | Decide whether the five partly resolved findings and AR-25…AR-33 are resolved by revision 4, and whether revision 4 introduced new problems. Only this can support clearing B-05 (judged per track, O-23) and B-07…B-10 (`DECISIONS.md` §9). The architect cannot clear them and has closed nothing. |
| Output | `agam-dev/reports/codex/ARCHITECTURE_REVIEW_R4.md`, with evidence in `agam-dev/reports/codex/architecture-review-r4-evidence/` |

## 1. What to review

| Document | Location |
|---|---|
| `ARCHITECTURE.md`, `DECISIONS.md`, `ROADMAP.md`, `AOS_KERNEL_SPEC.md`, `AGAMLANG_SPEC.md`, `TEST_PLAN.md`, this brief | `docs/` |
| `CLAUDE.md`, `AGENTS.md` | Repository root |

Revision 4 exists as **uncommitted changes** in `D:\Agam-Computing\ecosystem` on branch `review/architecture-v0.1` (whose tip `fd2670d` is revision 3), until the owner commits it. The worktrees do not share uncommitted files: read revision 4 where the owner tells you it is, and record exactly what you reviewed — the commit SHA if it has been committed, otherwise the SHA-256 of each of the nine files.

Both previous reports, both evidence folders and proposals 001 and 002 must stay unchanged. Write new files.

## 2. Rules

- **Read-only on the documents.** Every problem becomes a finding.
- **No implementation**: no kernel, interpreter, test, harness, runner or checker code. Throwaway probes outside the repository are fine and must be described.
- **Independent.** Re-derive from primary sources. The architect's notes, including `DECISIONS.md` §8.3 and §10.2, are claims to check, not evidence.
- **Evidence** in the block format of `TEST_PLAN.md` §2.2. Source inspection and arithmetic are labelled as such.
- **Two kinds of verification** (`TEST_PLAN.md` §2.5). Your dispositions are specification-level. Say explicitly that no implementation-level item has been run.
- **Git**: `AGENTS.md` §4. Commit the report only if the owner asks.
- Severities: Critical, Major, Minor, Informational (`TEST_PLAN.md` §2.4).

## 3. Scope A — the findings left open by the second review

The resolution table is `DECISIONS.md` §10.2. For each row give one disposition — **Resolved**, **Partially Resolved** (say exactly what remains), **Not Resolved**, or **Resolved, but introduces a new problem** (new finding ID) — and cite the sentence that resolves it or the place where it is still missing.

| Finding | Sections | Test it by |
|---|---|---|
| AR-02 remainder, AR-25 | `TEST_PLAN.md` §8.1–§8.3, §8.6; `ROADMAP.md` §2.2–§2.4; `ARCHITECTURE.md` §4, §6.4 | Walking §8.2 as an adversarial tip: one that replaces `check_ownership.py`, `check_inventory.py` or `gate_manifest.ps1`, or a file they import, and also changes a foreign path. Is any checker call resolved inside a worktree? Are the export commands valid for Git for Windows and byte-exact? Is the trusted directory really outside every worktree, and does anything stop a later step from using a stale export for a different `BASE`? Is the M0 by-hand check (§8.6) complete — modes, blobs, statuses, overlap, union, and how the manifests themselves are captured? |
| AR-26 | `TEST_PLAN.md` §4.1, §4.4, §8.2 steps 7–8, §8.4, Appendix A | Comparing Appendix A line by line with the §5 and §6 tables (IDs, suites, classes; counts 91 / 85 / 5 / 1). Then trying to make a gate pass with a required test omitted, duplicated, demoted to stretch, moved to another suite, or replaced by an ID that is not in the inventory. Can the candidate runner's summary misreport in a way `summary` mode accepts? What remains is recorded as R-28 — is that residual stated accurately, and is it acceptable? |
| AR-27 | `TEST_PLAN.md` §4.6, §8.2 steps 0, 6, 8, 9, 10, §8.3, §8.5, §8.7 | Moving a ref, adding an untracked or ignored input, or editing a file between each pair of steps. Is the permitted-output list exact and free of anything a test reads? Does `--verify-record` admit only the record files and the one-line `ACCEPTED` change? Does any sentence still claim that blob equality identifies a commit or branch? |
| AR-05 remainder, AR-28 | `AOS_KERNEL_SPEC.md` §7.2, §9.5, §12.4; `TEST_PLAN.md` §4.7, T-K1-09, RV-K3-03 | Enumerating every event (normal completion, panic, vector arriving armed-equal, armed-different, unarmed #BP, unarmed other) in both states (ending not begun, ending begun), including a fault at each instruction boundary of the ending sequence. Is a second `RESULT` line impossible? Is "exactly one" promised only where it can hold? Does the harness rule reject zero and two? |
| AR-11 remainder, AR-29 | `AGAMLANG_SPEC.md` §4.5, §7.1, §7.3, §8.6, Appendix A (E0107, E0108, E0408), B.3, C.11 | Taking each kind of literal and each computed value at limit − 1, limit and limit + 1: exactly one code and one stage each? Is the first-problem order inside a text literal unambiguous (bad escape, over-length, unterminated)? Is "before any item is evaluated" observable and consistent with §7.3? The stages were approved by the owner (O-24); report any technical defect you find in them. |
| AR-30 | `AGAMLANG_SPEC.md` §7.1, §7.5, §7.6, §7.8, §7.10, C.11 | Checking that `depth` and `count` are total, well defined for shared sub-lists, and independent of traversal order, early exit and memoisation. Recompute the §7.5 example (20 doublings → count 3,145,726). Find any pair of values whose comparison or display outcome two conforming implementations could decide differently. Confirm the cycle rule uses no limit and that "cycle" and "limit" cannot both apply. The limits were approved by the owner (O-25): state the consequences you see, and confirm that a comparison over a limit can never yield `false`. |
| AR-14 remainder, AR-31 | `AGAMLANG_SPEC.md` §2, §6.2, §6.3, §6.5, §8, §10, Appendices A, B, C | Attempting to write, from the text alone, the rendered diagnostic for every row of Appendix A in both languages, and the `ast --spans` output for a small program. List every value you would still have to guess. Check B.3 against A row by row (53 rows each), every placeholder against B.1, and each worked example of Appendix C against the rules, character by character. Check that the API of §10 needs no private type. |
| AR-32 | `AGAMLANG_SPEC.md` Appendix A (E0308, both languages) | Is the tip true for a direct and for an indirect cycle? (Structure only for the Hinglish row.) |
| AR-17 remainder, AR-33 | `AOS_KERNEL_SPEC.md` §3.1, §3.3, §4.4 | Comparing the three passages with the protocol text at `limine-protocol` commit `6e85ddd6f928b5b512015c4d47d1cfce2fd28079`. |

For the AR-01…AR-24 findings you assessed as Resolved, no re-review is requested, except where a section they rely on changed in revision 4. State which, if any, you re-opened.

## 4. Scope B — new material in revision 4

Review as new text, not only as fixes:

| Area | Sections |
|---|---|
| Specification-level and implementation-level verification | `TEST_PLAN.md` §2.5; its use in `DECISIONS.md` §10.2 |
| Trusted tools, inventory checker, record commit | `TEST_PLAN.md` §8 as a whole; T-M0-04, T-M0-05, T-M0-07; E-M0-07, E-M0-08, RV-M0-01; E-K1-02 |
| Test inventory | `TEST_PLAN.md` Appendix A |
| Ending guard | `AOS_KERNEL_SPEC.md` §12.4 |
| LLVM tool-name rule | `AOS_KERNEL_SPEC.md` §2, §11.1; `ROADMAP.md` M0-O1; `DECISIONS.md` §8.2. Your R3-E05 observation and the owner's report of the `-18` names are both recorded: is the rule sufficient for the K1 static tests? |
| Value limits and literal limits | `AGAMLANG_SPEC.md` §7.1 |
| Diagnostic matrix and worked examples | `AGAMLANG_SPEC.md` Appendices B and C. **Do not approve or rewrite Hinglish wording** (O-20); this now includes the words of B.2. You may flag a Hinglish row whose placeholders or structure differ from its English row. |
| Owner decisions and open options | `DECISIONS.md` §3.4 (O-23…O-29, the final answers to Q-17…Q-23) and §11 (the remainder of Q-17, and Q-24): is each decision applied consistently in every document it names? For the open options give technical consequences, not a decision. |
| Source nesting limit | `AGAMLANG_SPEC.md` §5.2, §5.3, §7.1, §7.10, §8.6, Appendix A (E0214), B.3, C.11; `TEST_PLAN.md` T-L2-03, T-L3-01. Is the level of every token well defined by the grammar? Is the reporting token unique in both cases? Could two conforming parsers disagree at 100 or 101 levels for any bracket use, for blocks or for a mixture? Is the floor for long operator sequences (provisional, Q-24) stated without promising more than it guarantees? |
| Two readings of verification tools | `TEST_PLAN.md` RV-M0-01, §8.1, §8.2, §8.6; `ROADMAP.md` §2.2 (M0-C3); `CLAUDE.md` §7; `AGENTS.md` §3. Is the order of submission workable, and could either reading be skipped unnoticed? |
| B-05 scope | `DECISIONS.md` §1, §9 (O-23): B-05 is judged per track. Your three verdicts must say which findings bear on which track, and each new finding must be labelled process, AOS, language or shared. |
| Cross-document consistency | Test, `E-` and `RV-` IDs against the tables and Appendix A; section references moved by revision 4; gate, branch and identity values; version numbers; exit numbers; the 40 error codes and 53 catalogue rows; the 52 / 5 / 6 keyword counts |

## 5. Evidence carried over, and what is still unperformed

Carried over from the earlier reviews, to be re-checked only if you have reason: Limine archive and header hashes, release/header pairing, descriptor and frame arithmetic, exit-value formula from source, the read-only exercise of the manifest and tree-listing flags (R3-E03), the WSL version inspection (R3-E05).

State explicitly that the following remain **not performed**, unless you actually perform them: release signature verification; building the Limine host utility; assembling and linking with the specified flags; any boot; any exception delivery; any allocator or interpreter execution; any gate step beyond read-only commands; any checker or runner, because none exists. If you use the owner's WSL environment, record the tool output yourself, including which LLVM tool names resolve.

## 6. Report format

```
# Architecture re-review — Draft v0.1, revision 4
Reviewed material: <commit SHA, or SHA-256 of each of the nine files>
Reviewer environment: <tools and versions actually used>

## Summary
<overall verdict; counts by severity>
<three separate verdicts: process sections (G-BOOT), AOS documents (G-AOS), language documents (G-LANG)>

## Disposition of the findings open after the second review
| Finding | Disposition (specification level) | Where resolved or what remains | Evidence |

## Findings re-opened from AR-01…AR-24, if any

## New findings (numbered from AR-34)
| ID | Severity | Track | Document § | Problem | Suggested change |

## Remarks on the owner decisions O-23…O-29 and the open options Q-17, Q-24

## Independent verifications performed
<evidence blocks>

## Not performed
<explicit list>

## Recommendation to the owner
<which blockers this review supports clearing, and which it does not>
```

## 7. Definition of done

- Every row of `DECISIONS.md` §10.2 has a disposition with a citation.
- Every area of §4 has been read, and the report says so.
- Each new Critical or Major finding names a section and a concrete change.
- The report gives three separate verdicts (process, AOS, language).
- The report separates specification-level dispositions from implementation-level verification, and claims none of the latter.
- Nothing under `docs/`, no operating-rule file, no implementation path and no file of either earlier review was modified.
