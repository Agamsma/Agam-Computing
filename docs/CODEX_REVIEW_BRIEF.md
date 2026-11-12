# Architecture Review Brief for Codex

| Field | Value |
|---|---|
| Status | Review request for **Draft v0.1, revision 2** (2026-11-12) |
| Reviewer | Codex, working independently of the architect and of Claude Code |
| Purpose | Clear blocker B-05 (and, for the Limine pin, B-02) in `DECISIONS.md` §9 |
| Output | `agam-dev/reports/codex/reviews/architecture-v0.1.md` on branch `agent/codex` |

## 1. What to review

| Document | Location |
|---|---|
| `ARCHITECTURE.md`, `DECISIONS.md`, `ROADMAP.md`, `AOS_KERNEL_SPEC.md`, `AGAMLANG_SPEC.md`, `TEST_PLAN.md` | `docs/` |
| `CLAUDE.md`, `AGENTS.md` | Repository root |

Until the owner commits them (question Q-13), the documents exist only as untracked files in `D:\Agam-Computing\ecosystem`. Read them there. Do not copy them into your worktree.

## 2. Rules for this review

- **Read-only on the documents.** Do not edit anything under `docs/`, `CLAUDE.md` or `AGENTS.md`. Every problem becomes a finding.
- **No implementation.** Do not write kernel code, interpreter code, tests or harness code. Small throwaway probes outside the repository are fine and must be described.
- **Independent.** Re-derive facts from primary sources (the Limine repositories, the QEMU source or your own run, the Intel/AMD manuals). Do not treat the architect's "observed" notes as proof; confirm or contradict them.
- **Evidence.** Use the evidence block of `TEST_PLAN.md` §2.2 for anything you ran. Anything you could not check is listed as "not verified", with the reason.
- **Git.** Follow `AGENTS.md` §4: Git for Windows only, branch `agent/codex`, the owner's identity only, no AI attribution, no push. Commit your report only if the owner asks you to.

## 3. Independent verifications requested

| # | Verify | Compare with |
|---|---|---|
| 1 | Download the Limine 12.9.6 asset `limine-binary.tar.xz` from the official release and compute its SHA-256. List the archive's files and confirm it contains `limine-bios.sys`, `limine-bios-cd.bin`, `limine-uefi-cd.bin`, `BOOTX64.EFI` and the host utility source. | `DECISIONS.md` §8.1; `AOS_KERNEL_SPEC.md` §11.2 |
| 2 | Obtain `include/limine.h` at `limine-protocol` commit `6e85ddd6f928b5b512015c4d47d1cfce2fd28079` and compute its SHA-256. Confirm that Limine 12.9.6 pins the same commit and supports base revision 6. | `DECISIONS.md` §8.1 |
| 3 | Confirm every identifier listed in `AOS_KERNEL_SPEC.md` §3.2 exists in that header with the field names the specification uses. | `AOS_KERNEL_SPEC.md` §3.2, §3.3 |
| 4 | Check each row of `AOS_KERNEL_SPEC.md` §3.1 against the protocol specification text at the same commit. | `AOS_KERNEL_SPEC.md` §3.1 |
| 5 | Check the consistency checklist row by row: link address, code model, segment permissions, request placement, memory ownership. | `AOS_KERNEL_SPEC.md` §4.4, §4.5 |
| 6 | Re-derive the GDT descriptor values, the TSS layout and size, the IDT gate encoding, the list of vectors that push an error code, and the interrupt-frame size and stack-alignment arithmetic. | `AOS_KERNEL_SPEC.md` §8, §9 |
| 7 | Confirm the exit convention: host status = (guest value × 2 + 1), and what QEMU returns on guest reset with `-no-reboot`. Use the QEMU source, and a run if QEMU is available to you. | `AOS_KERNEL_SPEC.md` §12.5 |
| 8 | Check the PMM design for a state in which it could hand out memory that is in use, or free memory it should not. | `AOS_KERNEL_SPEC.md` §10 |
| 9 | Check that the two agents' path sets are truly disjoint and that every file named in `ROADMAP.md` falls under exactly one owner. | `ARCHITECTURE.md` §6.1; `ROADMAP.md` |
| 10 | AgamLang: look for grammar ambiguities, phrase-alias conflicts (for example `nahi` versus `nahi to`), reserved words that will surprise beginners, error codes with no defined trigger, and semantics left undefined. | `AGAMLANG_SPEC.md` §3–§8 |
| 11 | Check that every acceptance criterion in `ROADMAP.md` is measurable and has a matching test ID in `TEST_PLAN.md`, and that every stretch item is clearly separated from required items. | `ROADMAP.md` §2–§4; `TEST_PLAN.md` §5–§6 |
| 12 | Check the eight documents against each other for contradictions: paths, branch names, version numbers, IDs, exit numbers. | All |

## 4. Questions to answer in the report

1. Could another engineer implement Sprint 1 (K1–K4) from `AOS_KERNEL_SPEC.md` alone? List every place where they would have to guess.
2. Could you write every required test in `TEST_PLAN.md` §5 before seeing any implementation? List every test that depends on an unspecified detail.
3. Is the five-day sprint realistic with the stretch items removed? Which day is most at risk?
4. Which decisions marked "Proposed" in `DECISIONS.md` do you recommend against, and why?
5. What is missing from the blocker list in `DECISIONS.md` §9?

## 5. Report format

```
# Architecture review — Draft v0.1, revision 2
Reviewer environment: <tools and versions actually used>
Scope covered / not covered: <list>

## Summary
<verdict: ready / ready after fixes / not ready; counts of High, Medium, Low>

## Independent verifications
<one evidence block or "not verified: reason" per row of section 3>

## Findings
| ID | Severity | Document § | Problem | Suggested change |

## Answers to the five questions

## Recommendation to the owner
```

Severity scale: `TEST_PLAN.md` §7 (High and Medium block approval).

## 6. Definition of done for the review

- Every row of §3 has evidence or an explicit "not verified".
- Every High and Medium finding names a document section and a concrete suggested change.
- The report states plainly whether the SHA-256 values in `DECISIONS.md` §8.1 were reproduced.
- Nothing under `docs/`, and no implementation path, was modified.
