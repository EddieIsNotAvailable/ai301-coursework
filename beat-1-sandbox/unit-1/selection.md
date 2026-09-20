# Unit 1 — Issue Selection

Path: `beat-1-sandbox/unit-1/selection.md`

Record of the issue carried into Unit 2, and of the evaluation runs that produced
`eval-run.txt`. This file is graded at the path above; a copy kept anywhere else in
the repository is not read.

Complete every labelled field below. Each is graded on its own; content placed under the
wrong label is not graded.

---

## Selected issue

**Issue link**

https://github.com/codepath/pathreview-ai301-fa26-s1/issues/72

**Verdict output**

**The verdict must record `accept` for this issue.** Choose an issue your own skill
accepts. If your skill rejects every candidate you try, that is a signal about your
rubric rather than about the issues: revise it and re-run — retries are unlimited and a
partial re-run costs about $0.20 — or run the skill on different candidates. Output
recording `reject` for the issue you chose earns no credit for this field.

```
All three pass repo-level checks (Maintainer Alive, Repo Alive, Contribution policy). No assignees, no linked PRs. Applying the house rule, jacho15's comment on #69 doesn't block claiming. Now the ranking by fit profile (small, clear problem, testable).

Summary

┌─────────────────────────┬────────────────┬─────────────────┬────────────────────────┬──────────────────────────────────────┬──────────────────┬─────────┐
│          Issue          │  Maintainer    │   Repo Alive    │     Bounded Scope      │            Available Work            │  Contribution    │ Verdict │
│                         │     Alive      │                 │                        │                                      │      Policy      │         │
├─────────────────────────┼────────────────┼─────────────────┼────────────────────────┼──────────────────────────────────────┼──────────────────┼─────────┤
│ #73                     │ pass — commits │ pass — pushed   │ pass — concrete file   │                                      │ pass — no policy │         │
│ (README/.env.example    │  <1yr, latest  │ 2026-09-16, not │ mismatch, clear fix,   │ pass — no assignee/PR; no comments   │  found (silence  │ accept  │
│ mismatch)               │ 2026-09-16     │  archived       │ doc-only               │                                      │ passes)          │         │
├─────────────────────────┼────────────────┼─────────────────┼────────────────────────┼──────────────────────────────────────┼──────────────────┼─────────┤
│                         │                │                 │ pass — one function,   │                                      │                  │         │
│ #72 (verify_password    │ pass           │ pass            │ exact fix behavior     │ pass — no assignee/PR/comments       │ pass             │ accept  │
│ UnknownHashError)       │                │                 │ specified, existing    │                                      │                  │         │
│                         │                │                 │ xfail test to un-skip  │                                      │                  │         │
├─────────────────────────┼────────────────┼─────────────────┼────────────────────────┼──────────────────────────────────────┼──────────────────┼─────────┤
│                         │                │                 │ pass — one function,   │ pass — no assignee/PR; a classmate's │                  │         │
│ #69 (output parser      │ pass           │ pass            │ clear expected fix,    │  investigation comment exists but    │ pass             │ accept  │
│ crashes on JSON array)  │                │                 │ xfail test exists      │ house rule says claim comments don't │                  │         │
│                         │                │                 │                        │  block                               │                  │         │
└─────────────────────────┴────────────────┴─────────────────┴────────────────────────┴──────────────────────────────────────┴──────────────────┴─────────┘

Ranked by fit (small scope, clear validation, focused change in unfamiliar code, avoid undocumented env deps):

1. #72 — top pick. Single function (verify_password), the exact required behavior is spelled out (fail closed instead of raising), and there's already an xfail unit test to flip — a clean, self-contained loop of read code → fix → un-skip test → verify green.
2. #69 — same shape (one function, existing xfail test to remove), slightly larger effort (2–4h vs 1–2h) and a classmate is already actively investigating it, so less room to do your own discovery even though claiming is fine.
3. #73 — smallest and truly trivial (sync two docs/config files), but it's a docs-only fix, so it gives less practice reading code/tests than the other two, despite being valid and testable.

[
  {
    "item": "https://github.com/codepath/pathreview-ai301-fa26-s1/issues/72",
    "checks": [
      {"name": "Maintainer Alive", "grade": "pass", "evidence": "3 of last 5 default-branch commits are from 2026-09-16, well under 1yr"},
      {"name": "Repo Alive", "grade": "pass", "evidence": "not archived, pushedAt 2026-09-16T21:48:27Z"},
      {"name": "Bounded Scope", "grade": "pass", "evidence": "issue names exact function, files, and required fail-closed behavior; existing xfail test to un-mark"},
      {"name": "Available work", "grade": "pass", "evidence": "assignees: [], no linked PRs, no comments"},
      {"name": "Contribution policy", "grade": "pass", "evidence": "no CONTRIBUTING.md or AI policy file found (404 on both paths) — silence passes"}
    ],
    "verdict": "accept"
  },
  {
    "item": "https://github.com/codepath/pathreview-ai301-fa26-s1/issues/69",
    "checks": [
      {"name": "Maintainer Alive", "grade": "pass", "evidence": "3 of last 5 default-branch commits are from 2026-09-16, well under 1yr"},
      {"name": "Repo Alive", "grade": "pass", "evidence": "not archived, pushedAt 2026-09-16T21:48:27Z"},
      {"name": "Bounded Scope", "grade": "pass", "evidence": "issue names exact function/file, precise crash trace, and existing xfail test to un-mark"},
      {"name": "Available work", "grade": "pass", "evidence": "assignees: [], no linked PRs; comment from jacho15 ('I'd like to attempt this') doesn't block per Path Review house rule allowing shared claims"},
      {"name": "Contribution policy", "grade": "pass", "evidence": "no CONTRIBUTING.md or AI policy file found — silence passes"}
    ],
    "verdict": "accept"
  },
  {
    "item": "https://github.com/codepath/pathreview-ai301-fa26-s1/issues/73",
    "checks": [
      {"name": "Maintainer Alive", "grade": "pass", "evidence": "3 of last 5 default-branch commits are from 2026-09-16, well under 1yr"},
      {"name": "Repo Alive", "grade": "pass", "evidence": "not archived, pushedAt 2026-09-16T21:48:27Z"},
      {"name": "Bounded Scope", "grade": "pass", "evidence": "issue names exact two files and the required change (make .env.example and README agree)"},
      {"name": "Available work", "grade": "pass", "evidence": "assignees: [], no linked PRs, no comments"},
      {"name": "Contribution policy", "grade": "pass", "evidence": "no CONTRIBUTING.md or AI policy file found — silence passes"}
    ],
    "verdict": "accept"
  }
]
```

---

## Eval iterations

Quote source text directly in each field below. Paraphrase does not satisfy them.

**Run history**

```
agreement: 18/20 scored items  (bar: 18/20: PASS)
agreement: 19/20 scored items  (bar: 18/20: PASS)
agreement: 17/20 scored items  (bar: 18/20: below the bar)
agreement: 19/20 scored items  (bar: 18/20: PASS)
```

**Issue analysis**

On `issue-15` the rubric decided `accept`, while the gold label was `reject`.

The eval output says:

`issue-15  reject  accept  NO     graded accept`

The rubric accepted it because all required checks passed according to the evidence available to the skill. However, the gold label treated the issue as unsuitable, so the rubric did not identify the evidence that made it a rejection. I suspect that rejection was expected since this issue has been abandoned for several years, and it relies on undocumented slack behavior that might no longer apply.

**Check rationale**

```
| Bounded Scope | Issue body, labels, and comments | Has a practical end goal. Do not reject for open implementation decisions. Reject for incomplete issue submission steps, undocumented third-party behavior | required |
```

The rational was to keep the scope bounding requirements broad to prevent unsound rejections, with more explicit rejection cases to cover known strong indicators of poor issues to work on

**Trade-offs**

The specific statement under Bounded Scopes pass condition "Do not reject for open implementation decisions" maintains flexibility, but the openness it causes could lead to wasting work/tokens on issues that are inherently unbounded in scope due to ambiguous open implementation decisions, because the model does not generate a quality solution, or because the maintainer disagrees. Overall maintainers are likely to ignore PRs that are too large, and when there are open implementation decisions, the fix a model will generate is at risk of becoming too large and unplanned for a maintainer to appreciate, since a large PR size creates a lot of work for them.

---

## Selection rationale

Graded on whether all three are answered, in your own words. Not on how good the
reasoning is, and not on length — a short honest answer to each earns the full marks.
This is also the basis for the claim comment you write in Unit 2.

**Selection rationale**

1. The issue's fit to your interests and to the time available.
It seems very simple to fix, with clear acceptance criteria, so it would take little time to fix.
2. What the verdict identified correctly, and what you weighed that the rubric could
   not.
The verdict correctly identified issue 72 (verify_password) as the simplest to fix. What it did not explicitly weigh but that I did was that the verify password issue only requires the context of a single file to evaluate correctness, whereas the other issues would involve scanning many files, and would take more testing effort to evaluate correctness.
3. The anticipated difficulty in claiming it.
Very little, since it presents an unambiguous problem and has clear acceptance criteria that is easy to write a test for to prove correctness.
---

Related paths: `eval-run.txt` in this directory; your skill's files in
`tools/issue-select/`.
