# Rubric: is this reproduction package ready to post?

## Checks

| Check | Evidence | Pass condition | Weight |
|---|---|---|---|
| issue-context | issue description | all required issue steps and information provided | required |
| env-reproduced | repro report's environment record read against the issue's target | the report records the relevant OS, runtime, package, install, and dependency details; the environment need not match the issue exactly, but any material difference is identified and does not invalidate the stated outcome | required |
| steps-reproduced | repro report's commands, setup, and observed artifacts read against the issue description | the report provides executable or otherwise concrete setup and trigger steps plus an observed result that demonstrates the issue; for an honest cannot-reproduce report, the attempted steps and result are concrete enough to repeat and the missing trigger condition is identified; a control run may be used when it makes the comparison unambiguous | required |
| repo-conventions | repo-facts block and claim/repro comments | pass when the comments follow the stated reporting policy. Apply an AI-disclosure failure only if the repo-facts text explicitly requires the issue or comment to name AI use and the candidate omits it; human-authorship, human-voice, or human-review guidance without that explicit disclosure requirement must pass | required |


## Verdict rule

accept only if all required checks pass