# Plan: issue #72 — fail closed on malformed stored hashes

Issue: https://github.com/codepath/pathreview-ai301-fa26-s1/issues/72
Author: EddieIsNotAvailable
Baseline: f89c06fc3ff292df2a04a39ac51319d32a76b779
Environment: macOS 27.0.1 arm64, Python 3.13.15, passlib 1.7.4, bcrypt 4.3.0,
pytest 9.1.1, pydantic 2.13.5, pydantic-settings 2.15.0.

## Diagnosis and reproduction evidence

Fresh local reproduction through core.security.verify_password confirms the
issue. This evidence was collected for Unit 3, independently of anyone else's
report. With a valid bcrypt hash, the correct password returns True and a wrong
password returns False. The same function currently lets stored-hash parsing
exceptions escape because pwd_context.verify has no exception handling.

Command: `.venv/bin/python -m unit3-evidence.repro` (script reproduced below).
Observed output, quoted verbatim:

```text
UnknownHashError inherits ValueError: True
control (valid hash): True
control (wrong password): False
'not_a_valid_bcrypt_hash' -> raised passlib.exc.UnknownHashError hash could not be identified
'' -> raised passlib.exc.UnknownHashError hash could not be identified
'plaintext' -> raised passlib.exc.UnknownHashError hash could not be identified
'$2b$notarealhash' -> raised builtins.ValueError not enough values to unpack (expected 2, got 1)
```

The script calls the real code, catching exceptions only to print the baseline:

```python
from core.security import hash_password, verify_password
from passlib.exc import UnknownHashError

print("UnknownHashError inherits ValueError:", issubclass(UnknownHashError, ValueError))
hashed = hash_password("password")
print("control (valid hash):", verify_password("password", hashed))
print("control (wrong password):", verify_password("wrong", hashed))
for malformed in ("not_a_valid_bcrypt_hash", "", "plaintext", "$2b$notarealhash"):
    try:
        print(repr(malformed), "->", verify_password("password", malformed))
    except ValueError as exc:
        print(repr(malformed), "-> raised", type(exc).__module__ + "." + type(exc).__name__, str(exc))
```

The installed passlib/exc.py defines `class UnknownHashError(ValueError)`.
Catching ValueError therefore handles both the unrecognized format and the
recognized-prefix malformed shape without catching every exception.

The repository regression confirms the same failure:

```text
$ .venv/bin/python -m pytest tests/unit/test_security.py -k test_verify_with_wrong_hash_format -v
 tests/unit/test_security.py::TestSecurity::test_verify_with_wrong_hash_format XFAIL
$ .venv/bin/python -m pytest tests/unit/test_security.py -k test_verify_with_wrong_hash_format -v --runxfail
E   passlib.exc.UnknownHashError: hash could not be identified
FAILED tests/unit/test_security.py::TestSecurity::test_verify_with_wrong_hash_format
```

## Scope and files

- core/security.py: catch ValueError only around pwd_context.verify and return
  False; document the malformed-hash return behavior.
- tests/unit/test_security.py: remove issue #72's strict xfail marker and
  parameterize the existing test with the four reproduced malformed strings.
  Add a focused check that unrelated runtime/backend failures still propagate.

Out of scope: authentication routes, JWT behavior, password generation,
dependency upgrades, hash migrations, configuration changes, unrelated seeded
bugs, frontend changes, and API-backed evaluations. plan.md, comment.md, and
local evidence artifacts stay out of the implementation commit.

## Approach

1. Record baseline controls, malformed inputs, XFAIL, and the underlying
   --runxfail failure before editing (completed above).
2. Add try/except ValueError around the current verification expression.
   Keep normal boolean verification unchanged and avoid a broad Exception catch.
3. Replace the regression's xfail decorator with parameterized malformed inputs;
   retain the identity assertion `result is False` and test runtime-error propagation.
4. Review the two-file diff, run the local test plan, then commit using a
   Conventional Commit and push fix/72-malformed-password-hashes to my fork.

## Test plan

Re-run the exact reproduction script through core.security after editing.
Expected: the four malformed strings print False without escaping exceptions,
correct-password control stays True, wrong-password control stays False.
Re-run the named regression with and without --runxfail: all four cases should
PASS, with no XFAIL or XPASS. Run the complete tests/unit/test_security.py to
cover valid hashes, case sensitivity, empty passwords, special characters,
whitespace, long inputs, and JWT controls. Confirm the monkeypatched RuntimeError
still propagates. Run ruff, Black --check, and mypy on the affected files.
Save raw commands, output, and exit statuses before and after in the course
write-up. No eval harness, Claude CLI, LLM call, or paid test is part of validation.

## Risks and unknowns

ValueError can include password validation errors as well as hash parsing errors;
returning False rejects those attempts rather than authenticating them. Other
exception classes remain visible so backend failures are not silently hidden.
The malformed-input cases are representative, not an exhaustive proof for every
possible hash. This validation is local on the environment listed above; no claim
of cross-platform or full-stack coverage is made. Passlib emits a trapped bcrypt
version warning even on the successful control; fixing that compatibility warning
would require unrelated dependency work and is excluded. Full repo/CI validation
belongs to the Unit 4 PR; this Unit 3 change uses focused checks.
At the time of implementation, paid rubric evals were skipped by request.
The student subsequently ran one full sonnet eval: 19/20 agreement, with at
least one match in every category. The unmodified generated eval-run.txt and
run history are now included in the course submission; no additional run is
planned.

## Deviations

The build followed the posted plan: verification now catches ValueError, the
strict xfail was removed, all four malformed hashes are regression cases, and
an unrelated RuntimeError remains visible. No implementation deviation was
needed. The local security module passed all 29 tests, and focused Ruff, Black,
mypy, and diff checks passed. No paid eval was invoked during implementation.
The student later supplied a full passing eval (19/20), which is now recorded
in the submission. This updated evaluation record does not change the posted
implementation approach. Both pushes were reversed at the student's request;
local work remains ready for review and later approved publication.
