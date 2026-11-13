# Focused Re-review Brief for Codex — Draft v0.1, revision 5

| Field | Value |
|---|---|
| Status | Review request for **Draft v0.1, revision 5** (2026-11-13), a focused document-only correction pass |
| Reviewer | Codex, independent of the architect and of Claude Code |
| Previous reviews | **First**: `ARCHITECTURE_REVIEW.md` — revision 2, commit `349c48a46d49cfac725b687868173d9423930309`; SHA-256 `b3f923ecaec34b9a2a318a37cd0ed3235627479618b4bee81ce5ad2cefeb86a9`. **Second**: `ARCHITECTURE_REVIEW_R3.md` — revision 3, commit `fd2670d7e9c664c8b1b3cd14dfee3c46d4a45299`; SHA-256 `a1b51c9a8fb2f08a05b8a52dd6c695ced74e937c33952e06304e7599629eaab7`. **Third**: `ARCHITECTURE_REVIEW_R4.md` — revision 4, commit `cbf2cffdc6f34307c70adddcc887907e1ec092e0`; SHA-256 `14f0fe9884b8a5fc17a078fac818886c76bba0129df664e7c34897b038bde220`; 11 Resolved and 3 Partially Resolved of the 14 findings re-reviewed; new AR-34…39 (3 Major, 3 Minor). All under `agam-dev/reports/codex/`. |
| Purpose | Decide whether the three partly resolved findings (AR-14, AR-27, AR-31) and AR-34…AR-39 are resolved by revision 5, and whether revision 5 introduced new problems. Only this can support clearing B-07, B-10 and B-05 for the language track (`DECISIONS.md` §9). The architect cannot clear them and has closed nothing. |
| Output | `agam-dev/reports/codex/ARCHITECTURE_REVIEW_R5.md`, with evidence in `agam-dev/reports/codex/architecture-review-r5-evidence/` |

## 1. What to review

| Document | Location | Changed in revision 5 |
|---|---|---|
| `TEST_PLAN.md`, `AGAMLANG_SPEC.md`, `DECISIONS.md`, `ROADMAP.md`, `ARCHITECTURE.md`, this brief | `docs/` | Yes |
| `CLAUDE.md`, `AGENTS.md` | Repository root | Yes |
| `AOS_KERNEL_SPEC.md` | `docs/` | **No** — byte-identical to its blob in `cbf2cff`. Its header therefore still reads "revision 4". |

Revision 5 exists as **uncommitted changes** in `D:\Agam-Computing\ecosystem` on branch `review/architecture-v0.1`, whose tip `cbf2cff` is revision 4, until the owner commits it. The worktrees do not share uncommitted files: read revision 5 where the owner tells you it is, and record exactly what you reviewed — the commit SHA if it has been committed, otherwise the SHA-256 of each of the nine files.

The three previous reports, their three evidence folders and proposals 001–003 must stay unchanged. Write new files.

## 2. Rules

- **Read-only on the documents.** Every problem becomes a finding.
- **No implementation**: no kernel, interpreter, test, harness, runner or checker code. Throwaway probes outside the repository are fine and must be described.
- **Independent.** Re-derive from primary sources. The architect's notes, including `DECISIONS.md` §8.3 and §10.3, are claims to check, not evidence.
- **Evidence** in the block format of `TEST_PLAN.md` §2.2. Source inspection and arithmetic are labelled as such.
- **Two kinds of verification** (`TEST_PLAN.md` §2.5). Your dispositions are specification-level. Say explicitly that no implementation-level item has been run.
- **Git**: `AGENTS.md` §4. Commit the report only if the owner asks, under the housekeeping plan of `DECISIONS.md` §3.6 (O-31). A committed report is a record, not an approval.
- Severities: Critical, Major, Minor, Informational (`TEST_PLAN.md` §2.4). Label every new finding process, AOS, language or shared (O-23).

## 3. Scope A — the findings left open by the third review

The resolution table is `DECISIONS.md` §10.3. For each row give one disposition — **Resolved**, **Partially Resolved** (say exactly what remains), **Not Resolved**, or **Resolved, but introduces a new problem** (new finding ID) — and cite the sentence that resolves it or the place where it is still missing. Then give AR-14, AR-27 and AR-31 the disposition that follows.

| Finding | Sections | Test it by |
|---|---|---|
| AR-34 | `TEST_PLAN.md` §8.2 steps 8c, 8d, 9; §8.3; §8.6; T-M0-05; E-M0-09 | Walking step 9 with each of: a wrong `ACCEPTED` prefix; an earlier line dropped or altered; two lines appended; another milestone appended; a missing artefact; a substituted summary; an extra file in the record directory; the architect's reading missing when `codex.raw` touches `agam-dev/verify/`, or present when it does not; a `GATE_REC` with another or a second parent. Is every content input named and tied to a blob ID of `record.raw`? Is recomputing a blob ID as specified valid for this repository's object format? Does the first gate work with an empty `accepted.old`? Is the stated `ACCEPTED` exception now accurate, and is anything else in the record delta read by a runner, test or checker? |
| AR-35 | `TEST_PLAN.md` §4.6 checks 3–6 and the paragraphs after them; §8.1; §8.2 steps 8a–8c; T-M0-04 (i), (k), (l), (n); E-M0-10; §9. `ROADMAP.md` §2.2, §4, R-29 | Walking step 8 for a planted or stale cache of each kind of module a gate run imports — the runner's own imports, a harness module, a test, a `conftest`, a language module — and for a sourceless `.pyc`, a `.pytest_cache`, a child process, the trusted tools' own directory, and leftover build output. Can any candidate Python code be imported before step 8a? Can any cache be read, written or left behind? Check the table of settings in §4.6 against Python's documentation and, if you can, on the WSL interpreter: the architect's fixtures ran on CPython 3.12.3 and 3.13.18 in a cloud sandbox, with `pytest` 9.1.1 on 3.13.18 only (`DECISIONS.md` §8.3). Cover the three ways a cache is executed, the effect of `-E` and `-I`, and whether any child process could start without the command-line options. Is the residual stated in `TEST_PLAN.md` §9 accurate and acceptable? |
| AR-36 | `AGAMLANG_SPEC.md` §5.2, §5.4, §8.6, Appendix A (E0203), B.1 (Rules F, X), B.3 (E0201, E0203, E0204, E0205a), C.12–C.15; `TEST_PLAN.md` T-L2-03, T-L5-02 | Taking every row of §5.4 and every kind of token: exactly one outcome each? Do the three lexer guarantees quoted in §5.4 follow from §4.2? Are your two counterexamples decided, and do C.12 and C.13 follow from the rendering rules character by character? Check C.14, and every row of C.15 against §5.2 and §5.4. Is the precedence list of §5.2 consistent with the table? Is every Rule X phrase in B.2? Is the reworded English E0203 message true on a first line, at top level and inside a block? **Do not approve or rewrite the Hinglish row**; structure and placeholders only. Look for a parser position or a token that is still undecided. |
| AR-37 | `TEST_PLAN.md` §4.2, §8.3 rule 4, T-M0-04 (m); `ROADMAP.md` §2.2 (M0-A1) | Is the file in every `BASE`, including the first? Does any rule still read a missing file as empty? Is it inside any agent's paths? |
| AR-38 | `TEST_PLAN.md` §4.6 ("Which worktree"), §8.2 step 0; `DECISIONS.md` §3.6 (O-31) | Are the two rules assigned to the right worktrees? Can an ignored file pass check 3 anywhere? Is the approved housekeeping plan consistent with both rules and with O-29? |
| AR-39 | `TEST_PLAN.md` §4.9, T-M0-03, E-M0-03; `ROADMAP.md` §2.2 (`.gitattributes` block), R-03 | Do the four patterns cover every archived file and no active one? With the last-match rule, does each path get the intended attributes — also if the owner keeps the per-extension lines before them? A read-only `git check-attr` and `git hash-object` comparison on real evidence files would settle it. Does any rule still demand LF of a binary manifest? |

For the findings you assessed as Resolved, no re-review is requested, except where a section they rely on changed in revision 5. State which, if any, you re-opened.

## 4. Scope B — new material and consistency

| Area | Sections |
|---|---|
| Record inputs and the judged directory | `TEST_PLAN.md` §8.2 step 9, §8.3; does `gate_manifest.ps1` need anything not yet specified to produce `accepted.old` and the copies? |
| Launch procedure | `TEST_PLAN.md` §8.2 step 8 as a whole: are the `find` commands correct for the layout, and is the order of 8a–8d workable for a gate that spans WSL and Windows? |
| Active text and byte-exact files | `TEST_PLAN.md` §4.9 |
| New evidence items | E-M0-09, E-M0-10; `ROADMAP.md` §2.3, §2.4 |
| Parse errors by position | `AGAMLANG_SPEC.md` §5.4 as new normative text, not only as a fix |
| Decision log | `DECISIONS.md` §1 (committing is not approving), §3.5 (O-30, O-31), §3.6 (housekeeping plan), §4 and §6 (P-49…P-53), §8.4, §8.6, §9, §10.3, §11 (Q-24) |
| Per-track statements | `DECISIONS.md` §9: B-05, B-08 and B-09 report your third-review conclusions without clearing anything. Are they reported accurately? |
| Cross-document consistency | Test, `E-` and `RV-` IDs against the tables and Appendix A — the inventory is unchanged at 91 tests (85 required, 5 stretch, 1 optional); 40 error codes and 53 catalogue rows, unchanged; section references moved by revision 5 (`TEST_PLAN.md` §4.6 check numbers, §4.9, §8.2 step 8 sub-steps); gate, branch and identity values; that `AOS_KERNEL_SPEC.md` needed no change |

## 5. Evidence carried over, and what is still unperformed

Carried over from the earlier reviews, to be re-checked only if you have reason: Limine archive and header hashes, release/header pairing, descriptor and frame arithmetic, exit-value formula from source, the read-only exercise of the Git capture commands (R3-E03, R4-E03), the WSL version inspection (R3-E05), your bytecode-cache observation (R4-E02).

State explicitly that the following remain **not performed**, unless you actually perform them: release signature verification; building the Limine host utility; assembling and linking with the specified flags; any boot; any exception delivery; any allocator or interpreter execution; any gate step beyond read-only commands; any checker or runner, because none exists.

## 6. Report format

```
# Architecture re-review — Draft v0.1, revision 5
Reviewed material: <commit SHA, or SHA-256 of each of the nine files>
Reviewer environment: <tools and versions actually used>

## Summary
<overall verdict; counts by severity>
<three separate verdicts: process sections (G-BOOT), AOS documents (G-AOS), language documents (G-LANG)>

## Disposition of the findings open after the third review
| Finding | Disposition (specification level) | Where resolved or what remains | Evidence |

## Findings re-opened from AR-01…AR-33, if any

## New findings (numbered from AR-40)
| ID | Severity | Track | Document § | Problem | Suggested change |

## Remarks on the housekeeping plan (O-31) and the open option Q-24

## Independent verifications performed
<evidence blocks>

## Not performed
<explicit list>

## Recommendation to the owner
<which blockers this review supports clearing, per track, and which it does not>
```

## 7. Definition of done

- Every row of `DECISIONS.md` §10.3 has a disposition with a citation, and AR-14, AR-27 and AR-31 each have the disposition that follows.
- Every area of §4 has been read, and the report says so.
- Each new Critical or Major finding names a track, a section and a concrete change.
- The report gives three separate verdicts (process, AOS, language).
- The report separates specification-level dispositions from implementation-level verification, and claims none of the latter.
- Nothing under `docs/`, no operating-rule file, no implementation path and no file of the three earlier reviews was modified.
