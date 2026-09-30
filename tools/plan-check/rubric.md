# Rubric: is this plan ready to post and build from?

## Checks

| Check | Evidence | Pass condition | Weight |
|---|---|---|---|
| diagnosis-grounding | Candidate diagnosis and approach compared with the repro steps, observed output, controls, and issue behavior | The stated cause explains the reproduced failure and controls without contradicting them; the proposed change reaches that cause rather than masking a downstream symptom. An unproved cause is labeled a hypothesis with a concrete verification step before implementation. | required |
| bounded-scope | Candidate in-scope and out-of-scope statements, files or areas, compared with the issue and thread | The change is bounded to resolving the reproduced issue, with identifiable affected areas and explicit exclusions where confusion is plausible; unrelated rewrites, migrations, features, and secondary symptoms are deferred. | required |
| executable-approach | Candidate approach and affected files or areas, read against repo facts and repro evidence | Another contributor can begin work at a named layer or location and follow concrete operations toward the fix; choices that determine the fix are resolved or have a bounded investigation with a decision criterion. | required |
| decisive-test-plan | Candidate test plan compared with the reproduction's trigger, controls, and observed behavior | The plan reruns the failing input through the real affected code or UI and names an observable post-fix result that distinguishes success from the recorded failure, plus relevant controls or adjacent cases. Manual checks are sufficient when reproducible; a full suite alone is not. | required |
| honest-uncertainty | Candidate claims, risks, unknowns, exclusions, and deviations compared with repro evidence and repo facts | Claims stay within the evidence; material unknowns or unsupported variants are named with a validation or deferral decision, and recorded deviations explain what changed and why. Do not require boilerplate risks when the package exposes no material uncertainty. | required |
| thread-and-conventions | Candidate comment compared with issue thread highlights, repo contribution policy, and candidate plan | The comment independently describes the same fix and validation as the plan, addresses relevant maintainer constraints, and follows explicit repository requirements, including AI disclosure when required. Classmates' plans do not block a Path Review plan; silence does not create a policy. | required |

## Verdict rule

Grade each row pass, fail, or unclear (?); quote the deciding evidence.
Accept (ready) only when every required check passes. Reject (hold) when any
required check fails or is unclear. Missing evidence is unclear; contradictory
or explicitly inadequate evidence is fail. Preferred checks, if added, never
change the verdict. Length, headings, and an automated test are not independent
requirements. An honest scoped deferral can pass; an unresolved choice that
prevents starting the fix cannot.
