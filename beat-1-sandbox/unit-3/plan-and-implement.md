# Unit 3 — Plan and Build

## Posted upstream

**GitHub username**

EddieIsNotAvailable

**Plan comment**

https://github.com/codepath/pathreview-ai301-fa26-s1/issues/72#issuecomment-5921522306

I reproduced #72 locally for Unit 3 at `f89c06f` on macOS 27.0.1 arm64, Python 3.13.15, passlib 1.7.4, and bcrypt 4.3.0. The real `verify_password` returns `True` for a correct password and `False` for a wrong password with a valid hash, but the malformed stored hashes fail as follows:

```text
'not_a_valid_bcrypt_hash' -> raised passlib.exc.UnknownHashError hash could not be identified
'' -> raised passlib.exc.UnknownHashError hash could not be identified
'plaintext' -> raised passlib.exc.UnknownHashError hash could not be identified
'$2b$notarealhash' -> raised builtins.ValueError not enough values to unpack (expected 2, got 1)
```

The existing covering test reports `XFAIL`; rerunning it with `--runxfail` exposes `UnknownHashError`. The cause is the unguarded `pwd_context.verify` call in `core/security.py`. I confirmed `UnknownHashError` inherits from `ValueError`, so catching only `UnknownHashError` would leave the malformed bcrypt-prefixed case uncovered.

My plan is to catch `ValueError` around verification and return `False`, document that behavior, then remove the strict `xfail` from `test_verify_with_wrong_hash_format` in `tests/unit/test_security.py` and parameterize it with these four inputs. I will also check that unrelated runtime failures still propagate. The change stays in those two files; JWTs, hashing, auth routes, dependency upgrades, and other seeded bugs are outside scope.

I will rerun the same inputs through the real function: all four malformed hashes must return `False`, while the valid-password control stays `True` and the wrong-password control stays `False`. I will run the named regression with and without `--runxfail`, the full local security test module, and focused lint/format/type checks, saving before-and-after output. Branch: `fix/72-malformed-password-hashes` on my fork.

The trapped bcrypt-version warning also occurs on the passing control and is outside this fix. These checks cover this local environment and representative malformed inputs. Paid eval runs are skipped to conserve API credits. Codex assisted with drafting and implementation; no passing checks are claimed before they run.

---

## Your branch

**Branch**

fix/72-malformed-password-hashes

Fork: https://github.com/EddieIsNotAvailable/pathreview-ai301-fa26-s1

**Evidence**

Fresh Unit 3 reproduction through the real core.security functions, before and
after the fix. Unit 2 files were not changed or used as my own report. The same
script is included in plan.md; local command logs follow below. Device usernames
in filesystem paths have been replaced with user for privacy; results are unchanged.
Only the local security module and focused static checks ran; no API-backed
checks ran. The trapped bcrypt-version warning occurs on both successful controls
and is outside the fix. The Pydantic warning is also pre-existing.

Before (baseline f89c06fc3ff292df2a04a39ac51319d32a76b779):

```text
$ git rev-parse HEAD
f89c06fc3ff292df2a04a39ac51319d32a76b779
Exit status: 0

$ .venv/bin/python -c 'import platform; from importlib.metadata import version; print(platform.platform()); print(platform.python_version()); print({n: version(n) for n in ("passlib", "bcrypt", "pytest", "pydantic", "pydantic-settings")})'
macOS-27.0.1-arm64-arm-64bit-Mach-O
3.13.15
{'passlib': '1.7.4', 'bcrypt': '4.3.0', 'pytest': '9.1.1', 'pydantic': '2.13.5', 'pydantic-settings': '2.15.0'}
Exit status: 0

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
'not_a_valid_bcrypt_hash' -> raised passlib.exc.UnknownHashError hash could not be identified
'' -> raised passlib.exc.UnknownHashError hash could not be identified
'plaintext' -> raised passlib.exc.UnknownHashError hash could not be identified
'$2b$notarealhash' -> raised builtins.ValueError not enough values to unpack (expected 2, got 1)
Exit status: 0

$ .venv/bin/python -m pytest tests/unit/test_security.py -k test_verify_with_wrong_hash_format -v
============================= test session starts ==============================
platform darwin -- Python 3.13.15, pytest-9.1.1, pluggy-1.6.0 -- /Users/user/Documents/codepath/pathreview-ai301-fa26-s1/.venv/bin/python
cachedir: .pytest_cache
rootdir: /Users/user/Documents/codepath/pathreview-ai301-fa26-s1
configfile: pyproject.toml
plugins: platformdirs-4.12.2
collecting ... collected 25 items / 24 deselected / 1 selected

tests/unit/test_security.py::TestSecurity::test_verify_with_wrong_hash_format XFAIL [100%]

=============================== warnings summary ===============================
core/config.py:7
  /Users/user/Documents/codepath/pathreview-ai301-fa26-s1/core/config.py:7: PydanticDeprecatedSince20: Support for class-based `config` is deprecated, use ConfigDict instead. Deprecated in Pydantic V2.0 to be removed in V3.0. See Pydantic V2 Migration Guide at https://errors.pydantic.dev/2.13/migration/
    class Settings(BaseSettings):

-- Docs: https://docs.pytest.org/en/stable/how-to/capture-warnings.html
================= 24 deselected, 1 xfailed, 1 warning in 0.16s =================
Exit status: 0

$ .venv/bin/python -m pytest tests/unit/test_security.py -k test_verify_with_wrong_hash_format -v --runxfail
============================= test session starts ==============================
platform darwin -- Python 3.13.15, pytest-9.1.1, pluggy-1.6.0 -- /Users/user/Documents/codepath/pathreview-ai301-fa26-s1/.venv/bin/python
cachedir: .pytest_cache
rootdir: /Users/user/Documents/codepath/pathreview-ai301-fa26-s1
configfile: pyproject.toml
plugins: platformdirs-4.12.2
collecting ... collected 25 items / 24 deselected / 1 selected

tests/unit/test_security.py::TestSecurity::test_verify_with_wrong_hash_format FAILED [100%]

=================================== FAILURES ===================================
_______________ TestSecurity.test_verify_with_wrong_hash_format ________________

self = <tests.unit.test_security.TestSecurity object at 0x10c931b50>

    @pytest.mark.xfail(
        strict=True,
        reason="issue #72 (manifest H-05): password verify raises UnknownHashError instead of returning False",
    )
    def test_verify_with_wrong_hash_format(self):
        """Test verify_password with non-bcrypt hash."""
        wrong_hash = "not_a_valid_bcrypt_hash"
    
        # Should handle gracefully, return False
>       result = verify_password("password", wrong_hash)
                 ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

tests/unit/test_security.py:227: 
_ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ 
core/security.py:37: in verify_password
    return bool(pwd_context.verify(plain_password, hashed_password))
                ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
.venv/lib/python3.13/site-packages/passlib/context.py:2343: in verify
    record = self._get_or_identify_record(hash, scheme, category)
             ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
.venv/lib/python3.13/site-packages/passlib/context.py:2031: in _get_or_identify_record
    return self._identify_record(hash, category)
           ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
_ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ 

self = <passlib.context._CryptConfig object at 0x10c8d86e0>
hash = 'not_a_valid_bcrypt_hash', category = None, required = True

    def identify_record(self, hash, category, required=True):
        """internal helper to identify appropriate custom handler for hash"""
        # NOTE: this is part of the critical path shared by
        #       all of CryptContext's PasswordHash methods,
        #       hence all the caching and error checking.
        # FIXME: if multiple hashes could match (e.g. lmhash vs nthash)
        #        this will only return first match. might want to do something
        #        about this in future, but for now only hashes with
        #        unique identifiers will work properly in a CryptContext.
        # XXX: if all handlers have a unique prefix (e.g. all are MCF / LDAP),
        #      could use dict-lookup to speed up this search.
        if not isinstance(hash, unicode_or_bytes_types):
            raise ExpectedStringError(hash, "hash")
        # type check of category - handled by _get_record_list()
        for record in self._get_record_list(category):
            if record.identify(hash):
                return record
        if not required:
            return None
        elif not self.schemes:
            raise KeyError("no crypt algorithms supported")
        else:
>           raise exc.UnknownHashError("hash could not be identified")
E           passlib.exc.UnknownHashError: hash could not be identified

.venv/lib/python3.13/site-packages/passlib/context.py:1132: UnknownHashError
=============================== warnings summary ===============================
core/config.py:7
  /Users/user/Documents/codepath/pathreview-ai301-fa26-s1/core/config.py:7: PydanticDeprecatedSince20: Support for class-based `config` is deprecated, use ConfigDict instead. Deprecated in Pydantic V2.0 to be removed in V3.0. See Pydantic V2 Migration Guide at https://errors.pydantic.dev/2.13/migration/
    class Settings(BaseSettings):

-- Docs: https://docs.pytest.org/en/stable/how-to/capture-warnings.html
=========================== short test summary info ============================
FAILED tests/unit/test_security.py::TestSecurity::test_verify_with_wrong_hash_format
================= 1 failed, 24 deselected, 1 warning in 0.15s ==================
Exit status: 1
```

After (built change):

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

$ .venv/bin/python -m pytest tests/unit/test_security.py -k test_verify_with_wrong_hash_format -v
============================= test session starts ==============================
platform darwin -- Python 3.13.15, pytest-9.1.1, pluggy-1.6.0 -- /Users/user/Documents/codepath/pathreview-ai301-fa26-s1/.venv/bin/python
cachedir: .pytest_cache
rootdir: /Users/user/Documents/codepath/pathreview-ai301-fa26-s1
configfile: pyproject.toml
plugins: platformdirs-4.12.2
collecting ... collected 29 items / 25 deselected / 4 selected

tests/unit/test_security.py::TestSecurity::test_verify_with_wrong_hash_format[not_a_valid_bcrypt_hash] PASSED [ 25%]
tests/unit/test_security.py::TestSecurity::test_verify_with_wrong_hash_format[] PASSED [ 50%]
tests/unit/test_security.py::TestSecurity::test_verify_with_wrong_hash_format[plaintext] PASSED [ 75%]
tests/unit/test_security.py::TestSecurity::test_verify_with_wrong_hash_format[$2b$notarealhash] PASSED [100%]

=============================== warnings summary ===============================
core/config.py:7
  /Users/user/Documents/codepath/pathreview-ai301-fa26-s1/core/config.py:7: PydanticDeprecatedSince20: Support for class-based `config` is deprecated, use ConfigDict instead. Deprecated in Pydantic V2.0 to be removed in V3.0. See Pydantic V2 Migration Guide at https://errors.pydantic.dev/2.13/migration/
    class Settings(BaseSettings):

-- Docs: https://docs.pytest.org/en/stable/how-to/capture-warnings.html
================= 4 passed, 25 deselected, 1 warning in 0.80s ==================
Exit status: 0

$ .venv/bin/python -m pytest tests/unit/test_security.py -k test_verify_with_wrong_hash_format -v --runxfail
============================= test session starts ==============================
platform darwin -- Python 3.13.15, pytest-9.1.1, pluggy-1.6.0 -- /Users/user/Documents/codepath/pathreview-ai301-fa26-s1/.venv/bin/python
cachedir: .pytest_cache
rootdir: /Users/user/Documents/codepath/pathreview-ai301-fa26-s1
configfile: pyproject.toml
plugins: platformdirs-4.12.2
collecting ... collected 29 items / 25 deselected / 4 selected

tests/unit/test_security.py::TestSecurity::test_verify_with_wrong_hash_format[not_a_valid_bcrypt_hash] PASSED [ 25%]
tests/unit/test_security.py::TestSecurity::test_verify_with_wrong_hash_format[] PASSED [ 50%]
tests/unit/test_security.py::TestSecurity::test_verify_with_wrong_hash_format[plaintext] PASSED [ 75%]
tests/unit/test_security.py::TestSecurity::test_verify_with_wrong_hash_format[$2b$notarealhash] PASSED [100%]

=============================== warnings summary ===============================
core/config.py:7
  /Users/user/Documents/codepath/pathreview-ai301-fa26-s1/core/config.py:7: PydanticDeprecatedSince20: Support for class-based `config` is deprecated, use ConfigDict instead. Deprecated in Pydantic V2.0 to be removed in V3.0. See Pydantic V2 Migration Guide at https://errors.pydantic.dev/2.13/migration/
    class Settings(BaseSettings):

-- Docs: https://docs.pytest.org/en/stable/how-to/capture-warnings.html
================= 4 passed, 25 deselected, 1 warning in 0.27s ==================
Exit status: 0

$ .venv/bin/python -m pytest tests/unit/test_security.py -v
============================= test session starts ==============================
platform darwin -- Python 3.13.15, pytest-9.1.1, pluggy-1.6.0 -- /Users/user/Documents/codepath/pathreview-ai301-fa26-s1/.venv/bin/python
cachedir: .pytest_cache
rootdir: /Users/user/Documents/codepath/pathreview-ai301-fa26-s1
configfile: pyproject.toml
plugins: platformdirs-4.12.2
collecting ... collected 29 items

tests/unit/test_security.py::TestSecurity::test_hash_password_returns_bcrypt_hash PASSED [  3%]
tests/unit/test_security.py::TestSecurity::test_verify_password_correct PASSED [  6%]
tests/unit/test_security.py::TestSecurity::test_verify_password_incorrect PASSED [ 10%]
tests/unit/test_security.py::TestSecurity::test_verify_password_case_sensitive PASSED [ 13%]
tests/unit/test_security.py::TestSecurity::test_hash_same_password_different_hash PASSED [ 17%]
tests/unit/test_security.py::TestSecurity::test_create_access_token_returns_string PASSED [ 20%]
tests/unit/test_security.py::TestSecurity::test_create_access_token_is_jwt PASSED [ 24%]
tests/unit/test_security.py::TestSecurity::test_decode_access_token_valid PASSED [ 27%]
tests/unit/test_security.py::TestSecurity::test_decode_access_token_invalid_token PASSED [ 31%]
tests/unit/test_security.py::TestSecurity::test_decode_access_token_malformed PASSED [ 34%]
tests/unit/test_security.py::TestSecurity::test_decode_access_token_empty_string PASSED [ 37%]
tests/unit/test_security.py::TestSecurity::test_roundtrip_token_with_data PASSED [ 41%]
tests/unit/test_security.py::TestSecurity::test_create_access_token_with_custom_expiry PASSED [ 44%]
tests/unit/test_security.py::TestSecurity::test_access_token_includes_expiration PASSED [ 48%]
tests/unit/test_security.py::TestSecurity::test_password_hash_different_for_different_passwords PASSED [ 51%]
tests/unit/test_security.py::TestSecurity::test_verify_password_with_empty_strings PASSED [ 55%]
tests/unit/test_security.py::TestSecurity::test_verify_password_with_special_characters PASSED [ 58%]
tests/unit/test_security.py::TestSecurity::test_token_with_empty_data PASSED [ 62%]
tests/unit/test_security.py::TestSecurity::test_token_with_special_characters_in_data PASSED [ 65%]
tests/unit/test_security.py::TestSecurity::test_token_with_unicode_data PASSED [ 68%]
tests/unit/test_security.py::TestSecurity::test_hash_password_long_input PASSED [ 72%]
tests/unit/test_security.py::TestSecurity::test_verify_with_wrong_hash_format[not_a_valid_bcrypt_hash] PASSED [ 75%]
tests/unit/test_security.py::TestSecurity::test_verify_with_wrong_hash_format[] PASSED [ 79%]
tests/unit/test_security.py::TestSecurity::test_verify_with_wrong_hash_format[plaintext] PASSED [ 82%]
tests/unit/test_security.py::TestSecurity::test_verify_with_wrong_hash_format[$2b$notarealhash] PASSED [ 86%]
tests/unit/test_security.py::TestSecurity::test_verify_password_propagates_runtime_errors PASSED [ 89%]
tests/unit/test_security.py::TestSecurity::test_token_tampering_detection PASSED [ 93%]
tests/unit/test_security.py::TestSecurity::test_create_token_consistency PASSED [ 96%]
tests/unit/test_security.py::TestSecurity::test_password_with_whitespace PASSED [100%]

=============================== warnings summary ===============================
core/config.py:7
  /Users/user/Documents/codepath/pathreview-ai301-fa26-s1/core/config.py:7: PydanticDeprecatedSince20: Support for class-based `config` is deprecated, use ConfigDict instead. Deprecated in Pydantic V2.0 to be removed in V3.0. See Pydantic V2 Migration Guide at https://errors.pydantic.dev/2.13/migration/
    class Settings(BaseSettings):

-- Docs: https://docs.pytest.org/en/stable/how-to/capture-warnings.html
======================== 29 passed, 1 warning in 6.31s =========================
Exit status: 0

$ .venv/bin/ruff check core/security.py tests/unit/test_security.py
All checks passed!
Exit status: 0

$ .venv/bin/black --check core/security.py tests/unit/test_security.py
All done! ✨ 🍰 ✨
2 files would be left unchanged.
Exit status: 0

$ .venv/bin/mypy --follow-imports=silent core/security.py tests/unit/test_security.py
pyproject.toml: note: unused section(s): module = ['agent.tools.market_analyzer', 'api.main', 'api.routes.health', 'api.routes.profiles', 'api.routes.reviews', 'core.logging', 'core.services.profile_service', 'core.services.review_service', 'ingestion.chunking.semantic_chunker', 'ingestion.chunking.structural_chunker', 'ingestion.parsers.skill_extractor', 'ingestion.pipeline', 'rag.generator.output_parser', 'rag.generator.review_generator']
Success: no issues found in 2 source files
Exit status: 0

$ git diff --check
Exit status: 0
```

## Eval iterations

**Run history**

One full run was completed by the student on September 30, 2026, using the
installed skill and the harness's pinned sonnet model:

```bash
python3 run_eval.py --rubric ~/.claude/skills/plan-check/rubric.md --evidence ~/.claude/skills/plan-check/references/evidence-guide.md --save-run eval-run.txt
```

```text
categories: clear-accept 6/7  scope-creep 4/4  thread-convention 2/2  unbuildable 3/3  wrong-cause 4/4
agreement: 19/20 scored items  (bar: 18/20: PASS)
```

This is the only recorded full run. No partial reruns or canaries were performed.
The last score above matches eval-run.txt. Its device username in the graded
path was replaced with user for privacy; scores and fingerprints are unchanged. All four recorded
skill fingerprints match both the installed files and tools/plan-check/.
The only disagreement was pkg-14: gold accept, model reject, with
`failed: executable-approach`. No rubric changes were made after this run.
The earlier direct plan reviews are separate from this scored eval and are
not counted as eval iterations. A live Claude CLI plan check is being completed
separately for assignment step 8.

**Package analysis**

In the saved full run, the rubric/model verdict for pkg-01 was reject and the
staff gold label was also reject. The recorded row agrees with the gold label.
The package
states: "the error is raised by argparse's `parse_args` while consuming
positionals; the request items are never handed to HTTPie's item parser."
Its candidate instead says: "The `REQUEST_ITEM` tokenizer in
`httpie/cli/requestitems.py` is the problem." The control accepts the same items
without -v, and the debug evidence locates failure before that tokenizer. The
proposed tokenizer rewrite therefore fails diagnosis-grounding and cannot
resolve the evidenced cause. This evidence explains why the saved model verdict
and the gold label agree.

**Check rationale**

Exact row from the submitted tools/plan-check/rubric.md:

```md
| diagnosis-grounding | Candidate diagnosis and approach compared with the repro steps, observed output, controls, and issue behavior | The stated cause explains the reproduced failure and controls without contradicting them; the proposed change reaches that cause rather than masking a downstream symptom. An unproved cause is labeled a hypothesis with a concrete verification step before implementation. | required |
```

The original installed draft only asked whether the plan named a cause. That
would accept a confidently wrong explanation like pkg-01's tokenizer claim.
This replacement compares the cause and fix with actual failing output and
controls, and rejects a change at a layer that the evidence rules out. A
hypothesis can still pass if it is labeled and has a concrete verification step
before implementation. The procedure gathers evidence before assigning grades.

**Trade-offs**

The diagnosis check permits a labeled, bounded hypothesis rather than requiring
proof of every internal detail before a plan exists. That may admit a plausible
hypothesis that a later verification disproves; the verification must happen
before implementation. This preserves useful investigation plans without
accepting certainty that contradicts a reproduction.

The full run agreed on 19 of 20 packages, but pkg-14 shows a cost of the
executable-approach check: the model rejected a plan naming the Unix reattach
handshake and affected crates, reporting only `failed: executable-approach`.
The saved output does not include the model's detailed reasoning; deferring
exact functions until tracing is a possible source of the rejection. The gold label accepts that bounded investigation. The package says
"exact functions to be pinned in the PR after tracing the query issuance with
debug logs" and gives the criterion "before pane input is wired". My reading
is that these can satisfy the rubric's allowance for a bounded investigation,
but the recorded model verdict still rejected this check. Requiring an executable starting
point helps hold vague plans, yet can also reject a workable plan that defers
exact function names. I kept the scored rubric unchanged rather than loosen it
after a passing run and invalidate the saved fingerprints. No partial reruns
or canaries were performed; no improvement beyond this run is claimed. Manual
regressions can satisfy decisive-test-plan when their trigger and outcome are
concrete; automated tests are not a universal requirement. In this build,
automated security regressions were appropriate and all 29 passed.

---

The saved full eval meets the assignment's 18/20 target and every-category
condition. The posted comment above is preserved exactly as posted; its note
about skipping paid runs describes the intention at posting time, before the
student subsequently ran the full eval. The implementation did not change.
The implementation branch and coursework updates are local, awaiting approval
to push after the earlier pushes were reversed. Submit the entire course repo:
https://github.com/EddieIsNotAvailable/ai301-coursework
