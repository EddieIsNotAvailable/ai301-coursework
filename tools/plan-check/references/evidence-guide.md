# Evidence guide: where evidence lives in a plan package

## Diagnosis and grounding

Where: eval bundles' Issue and Repro evidence sections, especially trigger commands, output, controls, and timing; Candidate plan diagnosis and approach. In live mode use the issue body, attributed posted reproduction or quoted house pack, and draft diagnosis.
Good: the proposed cause accounts for the failing and control behavior and targets the implicated layer. Calib-01's successful push followed by stale color, then correct color after re-entry, supports a missed view refresh. A confident claim cannot override an artifact pointing to another layer.

## Scope

Where: Candidate plan's change/in-scope/out-of-scope statements, files and approach, compared with Issue and Thread highlights. Live: equivalent draft statements and actual thread constraints.
Good: one cohesive fix reaches the reproduced cause, identifies its affected areas, and excludes unrelated work. Calib-01 changes the push completion refresh scope and excludes push-status computation and other views. Related regression tests belong in scope; drive-by migrations do not.

## Executability

Where: Candidate plan's files or areas, operations, sequencing, and decision points, read against Repo facts and Repro evidence. Live: draft approach plus documented repo paths and contribution instructions.
Good: a stranger can start at a named callback, function, module, or UI path and perform the chosen change. Calib-01 names the sync controller's completion callback and the missing commits context; exact line numbers and a patch are unnecessary. 'Investigate somewhere' leaves the decisive choice unresolved.

## Test plan

Where: Candidate test commands or manual steps and expected outputs compared with Repro evidence's failing trigger and controls. Live: draft test plan compared with the attributed reproduction, with before/after artifacts recorded later.
Good: rerun the trigger through the actual code or UI and state what changes at the failure point, retaining useful controls. Calib-01 requires the color to flip at step 3 without re-entering, then checks shared-callback paths. 'Run tests' alone lacks a decisive observable; a reproducible manual check can be decisive.

## Honesty

Where: Candidate cause claims, risk/unknown notes, deferred variants, exclusions, and Deviations; compare with Repro evidence's environment and limitations and Repo facts. Live: the same draft fields and attributed report.
Good: a hypothesis is marked and given a bounded verification, an unsupported variant is deferred explicitly, and a deviation states the changed approach and reason. Do not demand a risks heading or pretend untested platforms are proven.

## Comms

Where: Candidate plan comment compared with Candidate plan, Thread highlights, Issue, and Repo facts' contribution policy/templates. Live: comment.md, live issue comments, and relevant README/CONTRIBUTING/AI policy instructions.
Good: an independent comment describes the intended change and decisive validation, responds to relevant maintainer direction, and observes stated disclosure requirements. In calib-01 the review-bandwidth note supports keeping the fix small. Absence of an AI policy imposes no disclosure requirement; explicit requirements must be honored. Classmates' plans in Path Review are allowed.
