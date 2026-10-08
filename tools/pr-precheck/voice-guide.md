# Voice guide: how I talk upstream

## Who I am in threads

n/a

## Rules I write by

n/a

## Things I never post

n/a

## Plan-comment additions

- Attribute reproduced evidence accurately and distinguish observed results from planned checks.
  Wrong: "My report proved every malformed hash is safe."
  Right: "The run raised UnknownHashError for the listed input; I will check a malformed bcrypt-prefixed input too."
- State the change, boundaries, and expected test result without promising a deadline.
  Wrong: "I'll rewrite authentication and ship tonight."
  Right: "I'll catch hash-format ValueError in verify_password and check malformed hashes return False while valid verification still works."
- Do not claim an eval or unrun check passed.
  Wrong: "The rubric reached 20/20" without a run.
  Right: "Paid eval runs were skipped; this plan was reviewed directly against the rubric."

## PR-title and description additions

- Name the behavior changed and the issue; make every Changes bullet traceable to the diff.
- Keep CI unchecked until it has actually run; quote failed checks and limitations plainly.
- Explain AI use in my own words, including what I reviewed and verified.
- Use `user` in place of the device username in paths and captured output.
