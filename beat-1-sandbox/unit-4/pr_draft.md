fix(api): fail closed on malformed stored password hashes

## Summary

Make `verify_password` return `False` when passlib rejects a malformed stored hash instead of letting its parsing exception escape. Catch `ValueError` around verification, which covers both `UnknownHashError` and malformed bcrypt-prefixed input, while preserving normal password checks and unrelated runtime errors.

## Issue

Closes #72

## Changes

- Catch `ValueError` only around `pwd_context.verify` in `core/security.py` and document the malformed-hash return behavior.
- Replace issue #72's strict xfail marker with regression cases for `not_a_valid_bcrypt_hash`, the empty string, `plaintext`, and `$2b$notarealhash` in `tests/unit/test_security.py`.
- Add a regression proving unrelated `RuntimeError` failures still propagate.

## Testing

- [ ] **CI is green on this PR (all five jobs)** — required for review
- [x] Unit tests pass (`make test-unit`)
- [ ] Integration tests pass (`make test-integration`)
- [x] Linter passes (`make lint`)
- [x] Type checker passes (`make typecheck`)
- [x] New/updated tests cover the changes
- [x] If this fixes a seeded bug: removed its `@pytest.mark.xfail` marker (and any matching suppression in `pyproject.toml`)

Local validation on macOS arm64, Python 3.13.16, passlib 1.7.4 and bcrypt 4.3.0. The before output is the recorded Unit 3 baseline at `f89c06f` (Python 3.13.15); the exact same repro script was rerun on this branch. Device usernames in paths are replaced with `user`. These results establish function-level behavior, not an HTTP/full-stack login test.

Reproduction command: `.venv/bin/python -m unit3-evidence.repro`. The script calls the real `hash_password` and `verify_password`, prints valid/wrong-password controls, and tests the four malformed strings. It catches exceptions only to report baseline behavior.

Before:

```text
(trapped) error reading bcrypt version
Traceback (most recent call last):
  File "/Users/user/Documents/codepath/pathreview-ai301-fa26-s1/.venv/lib/python3.13/site-packages/passlib/handlers/bcrypt.py", line 620, in _load_backend_mixin
    version = _bcrypt.__about__.__version__
              ^^^^^^^^^^^^^^^^^
AttributeError: module 'bcrypt' has no attribute '__about__'
UnknownHashError inherits ValueError: True
control (valid hash): True
control (wrong password): False
'not_a_valid_bcrypt_hash' -> raised passlib.exc.UnknownHashError hash could not be identified
'' -> raised passlib.exc.UnknownHashError hash could not be identified
'plaintext' -> raised passlib.exc.UnknownHashError hash could not be identified
'$2b$notarealhash' -> raised builtins.ValueError not enough values to unpack (expected 2, got 1)
Exit status: 0

```

After (fresh run):

```text
$ .venv/bin/python -m unit3-evidence.repro
(trapped) error reading bcrypt version
Traceback (most recent call last):
  File "/Users/user/Documents/codepath/pathreview-ai301-fa26-s1/.venv/lib/python3.13/site-packages/passlib/handlers/bcrypt.py", line 620, in _load_backend_mixin
    version = _bcrypt.__about__.__version__
              ^^^^^^^^^^^^^^^^^
AttributeError: module 'bcrypt' has no attribute '__about__'
UnknownHashError inherits ValueError: True
control (valid hash): True
control (wrong password): False
'not_a_valid_bcrypt_hash' -> False
'' -> False
'plaintext' -> False
'$2b$notarealhash' -> False
Exit status: 0
```

Repository checks and additional validation (full-suite output summarized; command outcomes shown):

```text
$ make test-unit
.venv/bin/pytest tests/unit -v -m unit
================== 380 passed, 52 xfailed, 1 warning in 8.12s ==================
Exit status: 0
```

```text
$ make test-integration
.venv/bin/pytest tests/integration -v -m integration
============================= test session starts ==============================
platform darwin -- Python 3.13.16, pytest-9.1.1, pluggy-1.6.0 -- /Users/user/Documents/codepath/pathreview-ai301-fa26-s1/.venv/bin/python
cachedir: .pytest_cache
rootdir: /Users/user/Documents/codepath/pathreview-ai301-fa26-s1
configfile: pyproject.toml
plugins: platformdirs-4.12.2
collecting ... collected 0 items

============================ no tests ran in 0.01s =============================
make: *** [test-integration] Error 5
Exit status: 2
```

```text
$ make lint
.venv/bin/ruff check .
All checks passed!
Exit status: 0
```

```text
$ make typecheck
.venv/bin/mypy api/ core/ ingestion/ rag/ agent/ safety/
Success: no issues found in 76 source files
Exit status: 0
```

```text
$ .venv/bin/python -m pytest tests/unit/test_security.py -v
============================= test session starts ==============================
======================== 29 passed, 1 warning in 9.17s =========================
Exit status: 0
```

```text
$ .venv/bin/python -m pytest tests/unit/test_security.py -k test_verify_with_wrong_hash_format -v --runxfail
============================= test session starts ==============================
================= 4 passed, 25 deselected, 1 warning in 0.22s ==================
Exit status: 0
```

```text
$ git ls-files '*.py' | xargs .venv/bin/black --check
All done! ✨ 🍰 ✨
110 files would be left unchanged.
Exit status: 0
```

```text
$ npm test -- --run

> pathreview-frontend@0.1.0 test
> vitest --run


 RUN  v1.6.1 /Users/user/Documents/codepath/pathreview-ai301-fa26-s1/frontend

 ✓ src/components/__tests__/ReviewSection.test.tsx  (10 tests) 223ms
 ✓ src/components/__tests__/ProfileForm.test.tsx  (8 tests) 1182ms

 Test Files  2 passed (2)
      Tests  18 passed (18)
   Start at  18:59:06
   Duration  2.31s (transform 135ms, setup 452ms, collect 210ms, tests 1.41s, environment 842ms, prepare 128ms)

Exit status: 0
```

`make test-integration` reports "no tests ran": pytest exits 5 and make exits 2 because the repository has no integration tests yet. This is the assignment's expected result; the CI workflow explicitly treats pytest exit 5 as success. I have not called the integration suite passing. CI remains unchecked because this PR has not run CI yet.

Initial local checks exposed missing project dependencies and a corrupt mypy cache. After installing the repo's declared dependencies and using a fresh cache, mypy then hit NumPy 2.4 stubs incompatible with the repo's Python 3.11 target; selecting NumPy 2.2.6 within the declared dependency range made `make typecheck` pass. These were local environment repairs; no dependency/config files changed. The full unit run retains the repository's other 52 seeded expected failures. It also emits Pydantic/AsyncMock warnings. Passlib's trapped bcrypt-version warning appears before and after, including on valid controls; dependency compatibility cleanup is outside this fix.

Black passes on the two changed files and all tracked Python files. A scan including ignored local artifacts flagged only `unit3-evidence/repro.py`; that helper is absent from the committed diff. The diff has no whitespace errors and contains only the two planned files. No suppression in `pyproject.toml` names issue #72.

## Screenshots / Demo

N/A — this changes backend password-verification behavior with no UI change. The printed reproduction above demonstrates the result.

## Notes for Reviewers

I used Codex to help reproduce the issue, draft the implementation and regression tests, and prepare this PR. Codex also helped write the PR-precheck skill and capture the local validation shown here; Claude Sonnet evaluated the skill and checked this draft. Codex compared the two-file diff against the posted plan and captured the actual command results; I approved publication after reviewing the prepared draft and its reported validation.

The implementation follows the posted plan with no code deviations: authentication routes, JWT behavior, hashing configuration, dependencies, and unrelated seeded bugs remain outside the change. Catching `ValueError` can also reject password-validation errors; it still fails closed. Other exception classes remain visible. The malformed strings are representative, not exhaustive, and validation covers the local environment stated above.
