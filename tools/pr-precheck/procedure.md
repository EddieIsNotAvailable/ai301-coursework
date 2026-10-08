# Procedure: how this tool grades a PR package

## Read order

1. Determine the mode. In live mode read scope.md first and verify the intended base repo, then voice-guide.md. In eval mode use only the supplied bundle and ignore both files.
2. Read rubric.md and references/evidence-guide.md; list the exact checks and weights. Stop if components are empty as instructed by SKILL.md.
3. Read issue, thread, and repository requirements; record the bug trigger, expected outcome, explicit directions, template asks, and required checks.
4. Read the accepted plan and reproduction before the diff. Record promised files, behavior, exclusions, test cases, and deviations so the implementation cannot redefine the intended scope.
5. Read the entire diff and commit list, then compare title/description and test evidence with the plan and actual changes.

## Evidence gathering

1. Plan fidelity: pair every changed file and material behavior with a plan step or explicit deviation; also pair every promised outcome with the implementation or a disclosed omission. Record any description claim contradicted by the diff.
2. Test evidence: pair the plan's distinct repro cases and controls with commands, actual output, and expected-after behavior. Separate executed results from code added to tests, promises, comments, and checkboxes. Record unrun cases and whether limitations are explicit.
3. Repository checks: list checks the repo actually asks for, their commands, printed outcomes, failures, and explanations. A suite summary is enough for a suite result; behavioral proof must show the changed behavior.
4. Diff quality: inspect all hunks, not only the fix. Record unrelated changes, debug/dead code, unnecessary churn, and whether relevant deviations remain bounded to the issue.
5. Standards and communication: pair each explicit template/policy/thread ask with substantive PR content. Record issue reference, applicable AI disclosure, CI claims, and reviewer-facing limitations. In live mode also quote any broken voice rules separately.

## Check execution

1. Execute every rubric row in table order using the gathered evidence. Re-read the relevant source pair when a claim and implementation conflict. Do not consult gold labels.
2. Assign pass only when the stated condition holds, fail for a concrete contradiction or inadequate evidence, and unclear for genuinely unavailable deciding evidence. Name the source and deciding fact or absence in one evidence line.
3. Do not demand exhaustive tests or an undisclosed extra standard. Honor the rubric's honest-shortfall allowance while rejecting invented success and unrelated changes.
4. A check can use already gathered evidence without rereading the entire package when its deciding fact and all relevant contradictions are recorded. Report procedure gaps; do not silently add steps or checks.

## Verdict assembly

1. Apply the exact verdict rule: every required pass means accept; required fail or unclear means reject; preferred grades do not gate.
2. Print every check, grade, and deciding evidence. For reject identify the first failing/unclear required check in rubric order as the primary reason and list corrections for other failures too.
3. Report live voice notes separately. Emit the fixed SKILL.md JSON with every check and the binary verdict, preserving key order. Nothing follows the fenced JSON.
