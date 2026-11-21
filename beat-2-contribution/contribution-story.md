# Contribution: CLI discover fails with an opaque TypeError on a date column; add --index-col

Path: `beat-2-contribution/contribution-story.md`

This file is Assignment 5, and it is graded at the path above; a copy kept anywhere else in
the repository is not read. **Keep every section header below under its exact name** — the
format criteria grade exactly that. Your course repo link is the whole submission: no tool
files and no evidence uploads are part of Assignment 5.

**Contribution Number:** [1]
**Student:** Not listed (see GitHub Username)
**GitHub Username:** EddieIsNotAvailable
**Issue:** https://github.com/bloomberg/causal-ts/issues/130
**Pull Request:** https://github.com/bloomberg/causal-ts/pull/138

One repository, built across units 5 to 8, telling the whole story of your real
contribution. Fill it as you work, not at the end — the sections below ask for evidence that
is far easier to write down as it happens than to reconstruct in the last weekend.

Contribution Number stays `[1]` unless you make more than one contribution this term; it
just numbers them.

---

## Toolkit

Every story section below, from Why I Chose This Issue through Pull
Request, ends with a Toolkit block: which tool did that phase's work
and how you used it, what it produced, what difficulties or
shortcomings it presented, and what you changed or would change
because of them. Together the blocks are the thread the rubric's
Toolkit points read, and they are the raw material for your unit-9
portfolio, so keep them current as you work rather than
reconstructing them later.

A Toolkit block looks like this (copy it into each story section and
fill it there):

```text
**Toolkit**

- Tool: [which tool did this phase's work, and how you used it]
- What it produced: [described in a sentence or a short excerpt]
- Difficulties: [what fought you or fell short, or "none met"]
- What changed: [the patch, revision, or workaround, what you would
  update, or "nothing"]
```

This section also keeps one short dated entry per week of tool work,
starting below: what you built or changed that week, what broke, and
what you changed because of it. The blocks tell the story where it
happened; these entries are the toolkit's own timeline.

All four entries are graded. Keep every heading below, and fill each
one in the week it belongs to.

**Unit 5: Built issue-scout, then fixed a search gap and a rubric flaw that let untakeable issues through**

I turned my Unit 1 grader into issue-scout, which runs six `gh search issues` queries, one per language in my profile (Python, JS, TS, Go, Java, C), for unassigned `good first issue` + `bug` issues, drops issues the repo owner filed on their own solo repo, keeps one issue per repo, and grades the top five with my rubric. My first scan wasted two of its five grading slots on issues that already had open PRs, because search results don't show linked PRs, so I added `-linked:pr` to every query and confirmed that both issues disappear with it and return without it. Grading my Issue Selection site picks exposed a bigger flaw: the scout accepted session-desktop#255, a Linux-only icon bug I couldn't reproduce or test on my Mac, because no check asked whether an issue is testable on my platform. I added a required "Platform testable" check that fails issues confined to another OS (my platform is fixed as macOS arm64 so the grader doesn't have to detect it), and a re-grade now rejects #255 while the platform-independent pants#13825 still passes.

**Unit 6: Tightened issue-scout's search and availability check while choosing my issue**

I kept working in issue-scout rather than installing the starter voice guard and `CLAUDE.local.md`. My scout had accepted pants#13825, opened in 2021, because my queries only filtered on last update, so I added `--created ">=2025-11-20"` to all six queries and a matching ranking filter. That overcorrected toward issues opened the same day, and my `bug` requirement excluded #130, the issue I chose, so I removed that label from my queries and fit profile. I also changed the Available work check so my own open PR doesn't count as competition, and a direct grade then accepted #130 on all six required checks.

**Unit 7: Ran the project's checks by hand on my real branch before opening the PR**

I didn't install the starter ship-gate or write a manifest. I gated `cli-index-col-130` by hand through Claude Code with `pytest`, `black` 24, `isort` and `flake8`. The full suite went from 2443 passed on `26a4308` to 2455 passed, 29 skipped and 0 failed, and all 12 new tests failed on the original source. What fought me was my default Python 3.15, which is outside the project's supported versions, so I built the environment with 3.13.

**Unit 8: Read PR #138's state with `gh` and chose not to nudge a same-day PR**

I checked PR #138 with `gh` through Claude Code rather than installing the PR monitor or writing a playbook. On 2026-11-20 it was mergeable with no reviews, comments or check runs, because CI is waiting for a maintainer to approve workflows for a first-time contributor. With nothing to answer, I drafted no reply or nudge and left the thread alone, since pinging maintainers the day a PR opens would only add noise. The shortcoming is that without a monitor I have to recheck the PR by hand to catch review activity.

---

## Why I Chose This Issue

I searched with issue-scout, which runs one `gh search issues` query for each language in my profile (Python, JavaScript, TypeScript, Go, Java and C), looking for open, unassigned `good first issue` bugs. My first candidates didn't hold up. A maintainer had already said OpenSearch-Dashboards#6670 was no longer a bug, and pants#13825 dated from 2021. After I added a 12-month age limit, my third scan found 535 issues, ranked them down to 113, and accepted all five it graded. I picked bloomberg/causal-ts#131 because causal-ts was the most credible of the five projects, but another contributor opened PR #132 for it about an hour after the issue was filed, so I moved to #130 in the same repo. **I chose #130 before my scout had graded it.** It is labeled `enhancement`, and my queries required `bug` at the time, so no scan could surface it. I chose it because it fits the rest of my profile: it is small, it has a runnable reproduction and a stated way to test the result, it had no assignee or open PR, and it works the same on my Mac. It is also a real crash, even though the label says enhancement. When I graded it afterward, my scout accepted it on all six required checks, once I changed Available work so that my own open PR doesn't count as competition. I hope to get better at what my profile names: navigating an unfamiliar codebase, reading its tests, and making a focused change.

The scout was useful for narrowing hundreds of issues to a few, but it fell short on this real search in four ways. It filtered on last update, so an old issue looked fresh. It didn't read the threads of closed PRs, so it missed a maintainer saying a bug was gone. Its "alive" checks only look at recent commits, so it passed projects filing issues in bulk (mergepay-web opened 144 in one day). And my `bug` filter excluded the issue I ended up taking. I fixed the first by adding a `--created` filter and the last by removing the `bug` requirement from my queries and fit profile. I would fix the other two with a closed-PR thread check and a bulk-issue filter.

**Claim comment:** A claim comment was made.

**Toolkit**

- Tool: issue-scout (my search strategy and rubric), run on Sonnet through Claude Code: three scans, plus direct grades of my Issue Selection site picks.
- What it produced: Scan 3 (2026-11-20) found 535 issues, ranked them down to 113, and accepted all five it graded, including bloomberg/causal-ts#131. I moved to #130, in the same repo, after another contributor opened PR #132 for #131. A direct grade afterward accepted #130 on all six required checks.
- Difficulties: It accepted OpenSearch-Dashboards#6670 even though a maintainer had said in closed PR #12532 that the bug was gone, and pants#13825 even though it was opened in 2021, because my queries only filtered on last update. Once I added an age filter, all five picks were opened that same day, and some came from projects filing issues in bulk (mergepay-web opened 144 that day, flisk 51).
- What changed: I added `--created ">=2025-11-20"` to all six queries and a matching ranking filter, removed the `bug` label requirement from the queries and my fit profile, and changed Available work so my own open PR doesn't count as competition. Not fixed yet: reading the threads of closed linked PRs, and filtering out bulk-issue projects.

---

## Understanding the Issue

**Problem Description**

Most real time-series CSVs have a timestamp column, but `causal-ts discover` treats every column as a variable. It hands a date column to the engine, which crashes with an error that never says which column caused it. There is also no option to tell the CLI that a column is the index, so the only workaround is editing the file first.

**Expected Behavior**

`discover` should stop before running any engine, with a clear usage error that names the non-numeric columns and says how to fix it. A new `--index-col NAME_OR_POSITION` option on `discover`, `inspect` and `deconfound` should read that column as the index and keep it out of the variables.

**Current Behavior**

On `main` at `26a4308`, `discover dated.csv` exits with code 1 and a traceback ending in `TypeError: unsupported operand type(s) for -: 'str' and 'str'`. `deconfound` crashes on the same kind of file with a different `TypeError`. `inspect` handles it better, leaving the column out with a warning, but none of the three commands has an index option.

**Affected Components**

`causalts/cli.py` (data loading in the `inspect`, `discover` and `deconfound` commands), `causalts/utils/io.py` (`read_dataframe`), and `causalts/inspection.py` (`numeric_frame` and the non-numeric column rule). The crash itself surfaces in `_detect_discrete_cols` in `causalts/cdnots/phase3_utils.py`, but that is where bad input lands, not where it should be caught.

**Toolkit**

- Tool: Claude Code, which read issue #130 and traced the data loading through `causalts/cli.py`, `causalts/utils/io.py` and `causalts/inspection.py`.
- What it produced: A map of how `inspect`, `discover` and `deconfound` each load data. It also showed that `deconfound` crashes on a date column with a different `TypeError`, which the issue doesn't report.
- Difficulties: Issue #130 only reports `discover`, so reading the issue alone would have missed the separate `deconfound` crash.
- What changed: The clear error covers `deconfound` as well as `discover`.

---

## Reproduction Process

**Environment Setup**

macOS 27.2.1 on Apple Silicon (arm64), Python 3.13.18, uv 0.13.9, and causal-ts 0.28.1 at upstream `main` commit `26a4308`, with pandas 3.1.1, numpy 2.5.5, click 8.5.1 and causal-learn 0.1.4.8.

```bash
git clone https://github.com/bloomberg/causal-ts.git && cd causal-ts
uv venv -p 3.13 .venv
uv pip install -p .venv -e ".[dev]"
```

One challenge: my system Python is 3.15, and the project lists support for 3.10 through 3.14, so I created the environment with 3.13.

**Steps to Reproduce**

1. Create the CSV from the issue, with a `date` column next to two numeric columns:

   ```python
   import numpy as np, pandas as pd
   rng = np.random.default_rng(0)
   pd.DataFrame({"date": pd.date_range("2024-01-01", periods=300).strftime("%Y-%m-%d"),
                 "x": rng.normal(size=300), "y": rng.normal(size=300)}).to_csv("dated.csv", index=False)
   ```

2. Run `.venv/bin/causal-ts discover dated.csv --max-lag 1 --no-plot`.
3. Observed: exit code 1 and a traceback from inside the engine (`run_cdnots` → `_detect_discrete_cols`) that never names the `date` column.

**Reproduction Evidence**

- **Commit showing reproduction:** https://github.com/EddieIsNotAvailable/causal-ts/commit/d78dfb93e5d2ae4e2575c66ee9d615cc2f293509 (I didn't make a separate reproduction commit. This commit adds 12 tests, and all 12 fail on `26a4308` and pass with the change.)
- **Screenshots/logs:**

  ```text
  $ causal-ts discover dated.csv --max-lag 1 --no-plot     # exit 1
  TypeError: unsupported operand type(s) for -: 'str' and 'str'

  $ causal-ts deconfound <graph>.npy --data dated2.csv     # exit 1
  TypeError: ufunc 'isnan' not supported for the input types, ...

  $ causal-ts inspect dated.csv
  "Non-numeric column(s) were left out of the inspection and must be dropped or encoded before discovery: ['date']."
  ```
- **My findings:** The bug reproduced on the first try with the issue's steps. `inspect` already detects the column and warns, but `discover` doesn't use that check. `deconfound` fails the same way with a different error, and the failed `discover` run left an empty output folder behind.

**Toolkit**

- Tool: Claude Code, which built the environment and ran the issue's steps on `main`.
- What it produced: The traceback above, plus the same failure in `deconfound`.
- Difficulties: The default Python (3.15) is outside the project's supported versions.
- What changed: Used a Python 3.13 virtual environment.

---

## Solution Approach

**Analysis**

`discover` in `causalts/cli.py` loads data with `numeric_frame(read_dataframe(data))`. `numeric_frame` only converts Boolean columns, so a string date column reaches the engine and fails in `_detect_discrete_cols` (`causalts/cdnots/phase3_utils.py`). `deconfound` calls `read_dataframe` with no column check at all. `read_dataframe` (`causalts/utils/io.py`) also has no way to name an index column, so users had to edit the file first.

**Proposed Solution**

`discover` and `deconfound` now load data through one helper that raises a usage error (exit code 2) naming any non-numeric column, before an engine runs or an output folder is created. A shared `--index-col NAME_OR_POSITION` option on `inspect`, `discover` and `deconfound` reads that column as the index.

Deliberately not changed: `inspect` still warns and leaves non-numeric columns out, because it is the pre-flight check, and the `dowhy` commands that read CSVs directly are untouched.

**Implementation Plan**

1. `causalts/utils/io.py`: add `index_col` to `read_dataframe`, and `IndexColumnError` for a column the file lacks.
2. `causalts/inspection.py`: move the non-numeric rule into `non_numeric_columns`, so `inspect` and the new check share it.
3. `causalts/cli.py`: add the shared option and the `_read_data` and `_read_numeric_data` helpers, and use them in the three commands.
4. `tests/test_cli.py` and `tests/test_utils.py`: add 12 tests.
5. `docs/cli.md` and `CHANGELOG.md`: document the option and the fix.

Deviations from the issue's suggestions:

- The index column is resolved after the file is read, instead of passing `index_col` to `pd.read_csv`, so CSV, parquet and feather share one code path and one error.
- `deconfound` also gets the clear error, and now reads Booleans as 0/1 as `discover` does.
- The output folder is now created after the data check, so a rejected file leaves nothing behind.

**Toolkit**

- Tool: Claude Code, which wrote the plan and the change on branch `cli-index-col-130`.
- What it produced: Commit `d78dfb9`, touching three source files, two test files, the CLI docs and the changelog.
- Difficulties: `read_dataframe` raises `ValueError` for unsupported formats too, so catching it would have mislabeled those as `--index-col` errors.
- What changed: Added a dedicated `IndexColumnError`, following the existing `NoNumericColumnsError` pattern.

---

## Testing Strategy

**Reproduction steps, before and after:**

| Command on `dated.csv` | Before (`26a4308`) | After (`d78dfb9`) |
|---|---|---|
| `discover --max-lag 1 --no-plot` | exit 1, `TypeError` traceback | exit 2, `Error: column(s) ['date'] are not numeric; drop them or pass --index-col date` |
| `discover ... --index-col date` | no such option | exit 0, variables `["x","y"]`, shape `[300,2]` |
| `inspect --index-col date` | no such option | `warnings` is `[]` |
| `discover --index-col time` | no such option | exit 2, usage error listing `['date', 'x', 'y']` |

**Automated checks:**

- New tests: all 12 fail on the original source and pass with the change.
- Full suite: 2443 passed, 29 skipped on `26a4308`; 2455 passed, 29 skipped, 0 failed with the change. The difference is the 12 new tests.
- `black` 24, `isort` and `flake8` are clean on the five changed Python files.
- The project's own CI has not run yet. It is waiting for a maintainer to approve workflows on a first-time contributor's PR.

**Toolkit**

- Tool: Claude Code running `pytest`, `black`, `isort` and `flake8`.
- What it produced: The before and after results above.
- Difficulties: None of the existing tests covered non-numeric columns or `read_dataframe`, and CI can't run until a maintainer approves it.
- What changed: Added the 12 tests and confirmed they fail without the change.

---

## Implementation Notes

**2026-11-20**

Started on causal-ts#131 and reproduced it, but before I claimed it another contributor opened PR #132 for it. I deleted my #131 work and moved to #130 in the same repo. Claiming first would have avoided the lost work.

**2026-11-20**

Reproduced #130 on `26a4308`, then added the usage error and `--index-col`. Found that `deconfound` crashes the same way, so I included it. Chose to resolve the index column after reading so all three file formats behave the same.

**2026-11-20**

Ran the full suite with the change: 2455 passed, 29 skipped, 0 failed. Then opened PR #138 with before and after examples, the test results, and the AI-use disclosure the repo's PR template asks for.

**Toolkit**

- Tool: Claude Code, used for the implementation and for keeping this log.
- What it produced: The dated entries above.
- Difficulties: Losing #131 to another contributor after I had already reproduced it.
- What changed: Moved to #130, and I would now claim an issue before starting work on it.

---

## Pull Request

**PR Link:** https://github.com/bloomberg/causal-ts/pull/138

**PR Description:** Fixes #130. `discover` and `deconfound` now stop with a usage error naming any non-numeric column before an engine runs, and `inspect`, `discover` and `deconfound` gain `--index-col NAME_OR_POSITION`. The description includes before and after examples, the test results, the design decisions for review, and an AI-use disclosure.

**Maintainer Feedback:**

- 2026-11-20: none yet. CI is waiting for a maintainer to approve workflows, since this is my first PR to the repo.

**Status:** Awaiting review

**Toolkit**

- Tool: Claude Code, which drafted the description from the repo's PR template, then pushed the branch to my fork and opened the PR after I reviewed it.
- What it produced: PR #138, linked to #130 so that merging closes the issue.
- Difficulties: The project's CI can't run until a maintainer approves workflows for a first-time contributor, so the PR opened without automated checks.
- What changed: Put the before and after examples and my local test results in the description, so the evidence is there while CI waits.

---

## Learnings and Reflections

**Technical Skills Gained**

- Click: sharing one option across commands, and using `UsageError` and `BadParameter` so bad input exits with code 2 instead of a traceback.
- pandas: how `select_dtypes` separates numeric columns from dates and labels, and `set_index` for keeping a column out of the variables.
- Testing: proving a regression test is real by running it against the original source first. All 12 of mine failed there and passed with the change.
- GitHub: `gh search issues` qualifiers (`--updated` versus `--created`, `-linked:pr`), DCO sign-off, and why CI waits for approval on a first-time contributor's PR.

**Challenges Overcome**

- **Finding an issue that was actually available.** Three candidates fell through: one was already fixed, one was five years old, and one was taken by another PR while I was working on it. I tightened the search and moved to #130.
- **An error in more places than reported.** The issue describes `discover`, but `deconfound` crashes the same way, so I routed both through one loading helper.

**What I'd Do Differently Next Time**

- Have the scout grade an issue before I commit to it, instead of grading #130 only after opening my PR.
- Check for competing PRs immediately before starting work. Fresh, well-written issues can be taken within an hour.

---

## Resources Used

- [causal-ts issue #130](https://github.com/bloomberg/causal-ts/issues/130): the reproduction, suggested approach and test plan
- [causal-ts CONTRIBUTING.md](https://github.com/bloomberg/causal-ts/blob/main/CONTRIBUTING.md): the test expectations for new PRs
- [causal-ts CLI reference](https://github.com/bloomberg/causal-ts/blob/main/docs/cli.md): how the three commands and their options are documented
