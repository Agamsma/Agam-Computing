# Re-review Brief for Codex — Draft v0.1, revision 3

| Field | Value |
|---|---|
| Status | Review request for **Draft v0.1, revision 3** (2026-11-12) |
| Reviewer | Codex, independent of the architect and of Claude Code |
| Previous review | `agam-dev/reports/codex/ARCHITECTURE_REVIEW.md` — revision 2 at commit `349c48a46d49cfac725b687868173d9423930309`; report SHA-256 `b3f923ecaec34b9a2a318a37cd0ed3235627479618b4bee81ce5ad2cefeb86a9`; 0 Critical, 14 Major, 8 Minor, 2 Informational |
| Purpose | Decide, finding by finding, whether AR-01…AR-24 are resolved, and whether revision 3 introduced new problems. This is what can clear blockers B-05 and B-07…B-10 (`DECISIONS.md` §9). The architect cannot clear them. |
| Output | `agam-dev/reports/codex/ARCHITECTURE_REVIEW_R3.md`, with evidence in `agam-dev/reports/codex/architecture-review-r3-evidence/`, on `agent/codex` |

## 1. What to review

| Document | Location |
|---|---|
| `ARCHITECTURE.md`, `DECISIONS.md`, `ROADMAP.md`, `AOS_KERNEL_SPEC.md`, `AGAMLANG_SPEC.md`, `TEST_PLAN.md`, this brief | `docs/` |
| `CLAUDE.md`, `AGENTS.md` | Repository root |

Revision 3 exists as **uncommitted changes** in `D:\Agam-Computing\ecosystem` on branch `review/architecture-v0.1`, until the owner commits it. The two worktrees do not share uncommitted files: read revision 3 where the owner tells you it is, and record exactly what you reviewed — the commit SHA if it has been committed, otherwise the SHA-256 of each of the nine files.

Your previous report, its evidence and proposal 001 must stay unchanged. Write new files.

## 2. Rules

- **Read-only on the documents.** Every problem becomes a finding.
- **No implementation**: no kernel, interpreter, test, harness or runner code. Throwaway probes outside the repository are fine and must be described.
- **Independent.** Re-derive from primary sources. The architect's notes, including `DECISIONS.md` §8.3, are claims to check, not evidence.
- **Evidence** in the block format of `TEST_PLAN.md` §2.2. Source inspection and arithmetic are labelled as such.
- **Git**: `AGENTS.md` §4. Commit the report only if the owner asks.
- Severities: Critical, Major, Minor, Informational (`TEST_PLAN.md` §2.4).

## 3. Scope A — disposition of every previous finding

For each of AR-01…AR-24, using the resolution table in `DECISIONS.md` §10:

1. Read the "revised sections" named there.
2. State one disposition: **Resolved**, **Partly resolved** (say exactly what remains), **Not resolved**, or **Resolved, but introduces a new problem** (give it a new finding ID).
3. Quote or cite the specific sentence that resolves it, or the place where it is still missing.

Specific points to test hard:

| Finding | Check |
|---|---|
| AR-01 | Trace every gate and blocker: does any gate still need evidence that only the work it permits can produce? Is any implementation path inside the bootstrap scope (`ROADMAP.md` §2.2)? |
| AR-02 | Walk the gate procedure (`TEST_PLAN.md` §8) with a worked example: can a foreign-path change, a rename, a mode change, a deletion, an extra file introduced by the merge, or a stale tip pass unnoticed? Are the exact git commands valid for Git for Windows, and is their output captured byte-exactly? Is taking the checker from the baseline sufficient? |
| AR-03 | For each gate M0…K5, L1…L5: is the required suite set well defined, and can the first gates actually run with what exists then? |
| AR-04 | Can a required skip, a missing required test or an empty selection ever produce exit status 0? Are stretch and optional results unable to change it? |
| AR-05 | Enumerate the state machine of `AOS_KERNEL_SPEC.md` §12.4 and §9.5: failed check then return; failed check then matching fault; missing fault; wrong fault; #BP armed and unarmed; panic; fault during finalisation. Any undefined transition? |
| AR-06, AR-07 | Try to construct a memory map or call sequence for which the allocator text permits returning in-use memory, scanning padding bits, overflowing, or changing state before rejecting a request. |
| AR-08 | Count the keyword tables (52 words, 5 phrases, 6 components) from §3.2 yourself. Could a test still pass if the runtime table lost an entry? |
| AR-09, AR-10 | Is E0208 raised in exactly one stage? Is `IDENT` exported? Is any token- or AST-identity requirement left that a built-in alias substitution would violate? |
| AR-11 | Look for any expression whose result is outside §7.1, any host exception with no mapped code, and any limit check that can only happen after the expensive step. Verify the arithmetic behind the 6,644-bit bound and the 2^3322 > 10^1000 claim it rests on. |
| AR-12 | Look for a mutation path that could still create a cycle, and for any failed store that could leave a list changed. |
| AR-13 | Compare each rule of §7.3 with Python's documented order. Is every rule observable and unambiguous? |
| AR-14 | Could you write every rendered-diagnostic golden file from Appendix A and §8.2–§8.5 alone? List each message, span, suggestion or language case where you would have to guess. Confirm that `origin` and spans cannot affect AST equality or default JSON, and that E0900 has no public trigger. |

## 4. Scope B — new material in revision 3

Review these as new text, not only as fixes:

| Area | Sections |
|---|---|
| Gates and blockers | `DECISIONS.md` §1, §9 |
| Owner decisions O-13…O-21 faithfully applied | `DECISIONS.md` §3.2 and the sections each names |
| Runner contract and binding of results to commits | `TEST_PLAN.md` §4 |
| Integration gate | `TEST_PLAN.md` §8; `ARCHITECTURE.md` §6 |
| Bootstrap scope | `ROADMAP.md` §2 |
| Else-if composition and phrase metadata | `AGAMLANG_SPEC.md` §3.3, §3.4 |
| Limits, cycles, evaluation order | `AGAMLANG_SPEC.md` §7.1, §7.3, §7.4, §7.8 |
| Message catalogue | `AGAMLANG_SPEC.md` Appendix A — review the English for completeness and consistency with §8.6. **Do not approve or rewrite the Hinglish wording**; it is reserved for the owner (O-20). You may flag a Hinglish row whose placeholders differ from its English row. |
| Kernel changes | `AOS_KERNEL_SPEC.md` §2.1, §3.3, §4.2, §5, §9.5, §10, §12.1, §12.4, §13.1, §15 |
| Normative versus internal | Is anything now normative that no independent party depends on? Is anything left internal that a test in `TEST_PLAN.md` needs? |
| Cross-document consistency | Test IDs, `E-`/`RV-` IDs, section references, gate names, branch names, identity values, version numbers, exit numbers |

## 5. Evidence carried over, and what is still unperformed

Carried over from the first review, to be re-checked only if you have reason: Limine archive and header hashes, release/header pairing, descriptor and frame arithmetic, exit-value formula from source.

State explicitly in the report that the following remain **not performed**, unless you actually perform them: release signature verification; building the Limine host utility; assembling and linking with the specified flags; any boot; any exception delivery; any allocator or interpreter execution. The owner now reports WSL2 Ubuntu 24.04.5 with Clang 18.1.3, LLD 18.1.3, QEMU 8.2.2, GDB 15.1 and Make 4.3; if you use that environment, record the tool output yourself.

## 6. Report format

```
# Architecture re-review — Draft v0.1, revision 3
Reviewed material: <commit SHA, or SHA-256 of each of the nine files>
Reviewer environment: <tools and versions actually used>

## Summary
<verdict: ready for the bootstrap gate / ready after fixes / not ready; counts by severity>
<separate verdicts for the process sections (G-BOOT), the AOS documents (G-AOS) and the language documents (G-LANG)>

## Disposition of AR-01…AR-24
| Finding | Disposition | Where resolved or what remains | Evidence |

## New findings (numbered from AR-25)
| ID | Severity | Document § | Problem | Suggested change |

## Independent verifications performed
<evidence blocks>

## Not performed
<explicit list>

## Recommendation to the owner
<which blockers this review supports clearing, and which it does not>
```

## 7. Definition of done

- Every one of AR-01…AR-24 has a disposition with a citation.
- Every area of §4 has been read, and the report says so.
- Each new Critical or Major finding names a section and a concrete change.
- The report gives three separate verdicts (process, AOS, language), so that gates can be considered independently.
- Nothing under `docs/`, no operating-rule file, no implementation path and none of the first review's files was modified.
