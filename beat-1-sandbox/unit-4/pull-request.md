# Unit 4 — Test and Submit

## Your pull request

**Pull request**

https://github.com/codepath/pathreview-ai301-fa26-s1/pull/102

Opened against `codepath/pathreview-ai301-fa26-s1:main` for issue #72 from the existing two-file implementation commit `b271732`, after publication approval.

**Branch**

`fix/72-malformed-password-hashes`

The installed skill's live precheck returned `accept`, with all six checks passing. Local evidence: `pr_draft.md`, `test_evidence.md`, and `pr_precheck_result.md` in this folder. The committed implementation remains the existing two-file fix; no code deviations were needed.

## Eval iterations

**Run history**

One full Sonnet run on 2026-10-07, using the installed skill and the supplied harness; no partial reruns or rubric revisions followed. The generated transcript was copied unchanged. Its header uses a relative skill path to avoid exposing the device username.

```text
categories: clear-accept 6/7  not-tested 4/4  silent-drift 4/4  standards-wall 2/2  unreviewable 3/3
agreement: 19/20 scored items  (bar: 18/20: PASS)
```

**Package analysis**

`pkg-19`: my rubric decided `reject`; the gold label is `accept`. The only failing check was `behavioral-evidence`. The evaluator's deciding evidence was:

> Test evidence shows executed before/after for the issue's input but only asserts 'controls unchanged' for the double-quoted twin and no-trailing-blank-line cases with no commands or output shown.

The diff faithfully removes the planned whitespace reset, adds a regression, and reports the repository suite result. My rubric still asks for observable relevant control results rather than accepting a narrative assurance. That made it stricter than the gold label on this otherwise ready submission. I kept the check because the full run already meets 18/20 and every category has a match, and because silently relaxing the evidence standard could accept cases that never exercise the actual bug.

**Check rationale**

```md
| behavioral-evidence | Reproduction and plan test plan alongside candidate Test evidence or test_evidence.md and PR Testing | Concrete executed commands and observable before/after results distinguish the issue's failure from the expected behavior through the real affected path, with relevant controls. Cover the materially distinct planned triggers or explicitly disclose an unrun case and its limitation without claiming full coverage. Added tests, checkboxes, bare 'tests pass', and comment-only assertions are not executed evidence. | required |
```

I rewrote the Unit 3 proposed-test check for executed PR evidence. The rule compares actual commands/results with the repro and plan, distinguishes test code from a test run, and allows an explicitly disclosed unrun case. I rejected requiring every possible input or perfect repository-check success: the assignment permits honest limitations, while unsupported claims must still hold a PR. No change to this check was made after the full run.

**Trade-offs**

The stricter observable-controls requirement rejects `pkg-19` despite the gold `accept`; it trades one false hold for better detection of unsupported verification. The same run rejects `pkg-07`, matching its gold `reject`, because its reported one-file control never exercises the two-file bug. Since I did not loosen a check, no canary rerun was needed. Both standards-wall packages (`pkg-01` and `pkg-20`) matched, including `pkg-20`'s missing explicit AI disclosure. A genuinely untested edge case can still be accepted when disclosed with its limitation; the skill does not independently execute a submitted package or prove exhaustive correctness.
