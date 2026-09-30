# Procedure: how this skill grades a plan package

## Read order

1. Determine live or eval mode. In live mode read scope.md first, verify the issue repo, then read voice-guide.md. In eval mode ignore both and use only the bundle.
2. Read rubric.md and references/evidence-guide.md; list all checks, required weights, and the verdict rule.
3. Read the issue, thread highlights, repo facts, and reproduction evidence before the candidate plan. Then read the entire plan, deviations, and draft comment. Do not consult gold labels while grading.

## Evidence gathering

1. In live mode fetch the issue and comments with gh or the GitHub API, and read relevant contribution instructions from the repo. Locate the student's posted repro or a clearly attributed quoted repro pack; do not attribute another person's report to the student.
2. In eval mode locate each family in the frozen bundle; do not fetch outside facts or infer an implementation from code outside it.
3. Following the evidence guide, record the failing trigger, observed and expected results, controls, stated cause, affected areas, exclusions, implementation decisions, observable checks, material unknowns, thread constraints, and contribution requirements.
4. Keep candidate statements separate from supporting evidence. For each rubric row collect one decisive quote or fact and any contradiction; missing evidence stays missing.

## Check execution

1. Apply every rubric row's pass condition to its named evidence. Compare the diagnosis with the reproduction before accepting confident causal claims.
2. Check that the scope and chosen operations address that cause, that a contributor can start, and that the test would distinguish the original failure from success through the real system.
3. Check uncertainty and deviations against the evidence, then compare the comment with the plan, thread, and explicit repo conventions. Do not invent disclosure rules or treat classmate work as a blocker.
4. Assign pass when the condition is satisfied, fail for contradiction or an inadequate proposed outcome, and unclear when deciding evidence is genuinely absent. Record a one-line deciding quote or fact for each result; do not compensate a failure with strengths elsewhere.

## Verdict assembly

1. Apply the rubric rule: all required passes means accept; any required fail or unclear means reject. Report each row with its exact rubric name.
2. In live mode separately report broken voice rules with their exact text. Voice notes do not change the verdict unless a rubric row explicitly makes them relevant.
3. Identify procedure gaps rather than improvising new grading rules. For a hold, name the specific missing evidence or correction needed.
4. Emit the SKILL.md JSON schema in a final fenced JSON block: item, checks (name, grade, evidence), and verdict. Emit nothing after the block.
