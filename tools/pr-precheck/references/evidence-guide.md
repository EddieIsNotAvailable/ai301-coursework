# Evidence guide: where evidence lives in a PR package

## Plan fidelity (harness category: silent-drift)

Eval: read Plan context's scope, excluded work, promised behavior, files, and deviations beside Candidate PR's entire Diff and Description, including explicit limitations/notes. Live: pair plan.md and its Deviations with `git diff main...HEAD`, commit metadata, and pr_draft.md. Good evidence maps every material hunk to the plan or an explained deviation, and maps every promise to implementation or an honest disclosed omission. The description must not claim behavior the diff omits, or conceal added behavior. A shorter implementation can be ready when the limits and reason are explicit.

## Test evidence (harness category: not-tested)

Eval: locate the repro evidence and Test plan inside Plan context, repository checks in Repo facts, and executed commands/results in Candidate PR's Test evidence and Description. Live: compare the issue's posted repro (or attributed house pack), plan.md Test plan, and test_evidence.md with pr_draft.md Testing; check the PR template/CONTRIBUTING for repository commands. Good behavioral evidence shows the actual failing and corrected observable result with relevant controls through the real affected code, not only an added test or a narrative claim. Suite summaries show repository-check outcomes. Record command failures, expected no-tests results, and unavailable or skipped checks honestly with reasons; never transform them into green claims.

## Diff quality (harness category: unreviewable)

Eval: inspect Candidate PR's complete unified Diff, changed filenames, and Commits alongside the issue and Plan context boundary. Live: inspect all hunks in `git diff main...HEAD` and branch commits; report uncommitted changes separately. Good is a visible bounded fix with necessary regression coverage, not debug leftovers, dead/commented-out code, unrelated cleanup, generated noise, or bulk formatting churn. An explained relevant deviation does not license unrelated edits.

## Standards and comms (harness category: standards-wall)

Eval: read Repo facts' PR-template asks, contribution instructions and explicit AI policy, plus Thread highlights; compare them to Candidate PR Title and Description. Live: read .github/PULL_REQUEST_TEMPLATE.md, docs/CONTRIBUTING.md, issue comments, scope.md, and the course's explicit AI disclosure ask beside pr_draft.md; compare outgoing text with voice-guide.md separately. Good content substantively answers the repository's actual asks, references the issue, engages explicit maintainer direction, discloses AI use when required, and labels CI/unrun checks honestly. N/A with a reason can answer an inapplicable section. Do not impose a template or AI disclosure on an eval repo that asks for neither. Contradicted behavior claims are graded as plan fidelity, not merely writing style.
