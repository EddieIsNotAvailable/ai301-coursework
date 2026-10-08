# Test evidence — issue #72

Captured on 2026-10-07 from the existing committed fix/72-malformed-password-hashes branch. Device username is replaced with user in captured paths. No new implementation edits or commits were made.

## Before (recorded Unit 3 baseline)

Baseline f89c06fc3ff292df2a04a39ac51319d32a76b779. Reusing the recorded output from unit3-evidence/before.txt; the same repro command is rerun below on the fix branch.

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

## Fresh branch checks and after reproduction

### make test-unit

```text
$ make test-unit
.venv/bin/pytest tests/unit -v -m unit
============================= test session starts ==============================
platform darwin -- Python 3.13.16, pytest-9.1.1, pluggy-1.6.0 -- /Users/user/Documents/codepath/pathreview-ai301-fa26-s1/.venv/bin/python
cachedir: .pytest_cache
rootdir: /Users/user/Documents/codepath/pathreview-ai301-fa26-s1
configfile: pyproject.toml
plugins: platformdirs-4.12.2
collecting ... collected 62 items / 17 errors

==================================== ERRORS ====================================
_____________ ERROR collecting tests/unit/test_batch_processor.py ______________
ImportError while importing test module '/Users/user/Documents/codepath/pathreview-ai301-fa26-s1/tests/unit/test_batch_processor.py'.
Hint: make sure your test modules/packages have valid Python names.
Traceback:
/opt/homebrew/Cellar/python@3.13/3.13.16/Frameworks/Python.framework/Versions/3.13/lib/python3.13/importlib/__init__.py:88: in import_module
    return _bootstrap._gcd_import(name[level:], package, level)
           ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
tests/unit/test_batch_processor.py:8: in <module>
    from ingestion.embeddings.batch_processor import BatchEmbeddingProcessor
ingestion/embeddings/batch_processor.py:1: in <module>
    import structlog
E   ModuleNotFoundError: No module named 'structlog'
______________ ERROR collecting tests/unit/test_bias_detector.py _______________
ImportError while importing test module '/Users/user/Documents/codepath/pathreview-ai301-fa26-s1/tests/unit/test_bias_detector.py'.
Hint: make sure your test modules/packages have valid Python names.
Traceback:
/opt/homebrew/Cellar/python@3.13/3.13.16/Frameworks/Python.framework/Versions/3.13/lib/python3.13/importlib/__init__.py:88: in import_module
    return _bootstrap._gcd_import(name[level:], package, level)
           ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
tests/unit/test_bias_detector.py:5: in <module>
    from safety.bias_detector import BiasDetector
safety/bias_detector.py:5: in <module>
    import structlog
E   ModuleNotFoundError: No module named 'structlog'
___________ ERROR collecting tests/unit/test_faithfulness_checker.py ___________
ImportError while importing test module '/Users/user/Documents/codepath/pathreview-ai301-fa26-s1/tests/unit/test_faithfulness_checker.py'.
Hint: make sure your test modules/packages have valid Python names.
Traceback:
/opt/homebrew/Cellar/python@3.13/3.13.16/Frameworks/Python.framework/Versions/3.13/lib/python3.13/importlib/__init__.py:88: in import_module
    return _bootstrap._gcd_import(name[level:], package, level)
           ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
tests/unit/test_faithfulness_checker.py:5: in <module>
    from rag.evaluator.faithfulness_checker import FaithfulnessChecker
rag/evaluator/faithfulness_checker.py:5: in <module>
    import structlog
E   ModuleNotFoundError: No module named 'structlog'
______________ ERROR collecting tests/unit/test_keyword_search.py ______________
ImportError while importing test module '/Users/user/Documents/codepath/pathreview-ai301-fa26-s1/tests/unit/test_keyword_search.py'.
Hint: make sure your test modules/packages have valid Python names.
Traceback:
/opt/homebrew/Cellar/python@3.13/3.13.16/Frameworks/Python.framework/Versions/3.13/lib/python3.13/importlib/__init__.py:88: in import_module
    return _bootstrap._gcd_import(name[level:], package, level)
           ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
tests/unit/test_keyword_search.py:5: in <module>
    from rag.retriever.keyword_search import KeywordSearcher
rag/retriever/keyword_search.py:3: in <module>
    import structlog
E   ModuleNotFoundError: No module named 'structlog'
__________ ERROR collecting tests/unit/test_llm_provider_contract.py ___________
ImportError while importing test module '/Users/user/Documents/codepath/pathreview-ai301-fa26-s1/tests/unit/test_llm_provider_contract.py'.
Hint: make sure your test modules/packages have valid Python names.
Traceback:
/opt/homebrew/Cellar/python@3.13/3.13.16/Frameworks/Python.framework/Versions/3.13/lib/python3.13/importlib/__init__.py:88: in import_module
    return _bootstrap._gcd_import(name[level:], package, level)
           ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
tests/unit/test_llm_provider_contract.py:3: in <module>
    import numpy as np
E   ModuleNotFoundError: No module named 'numpy'
______________ ERROR collecting tests/unit/test_output_parser.py _______________
ImportError while importing test module '/Users/user/Documents/codepath/pathreview-ai301-fa26-s1/tests/unit/test_output_parser.py'.
Hint: make sure your test modules/packages have valid Python names.
Traceback:
/opt/homebrew/Cellar/python@3.13/3.13.16/Frameworks/Python.framework/Versions/3.13/lib/python3.13/importlib/__init__.py:88: in import_module
    return _bootstrap._gcd_import(name[level:], package, level)
           ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
tests/unit/test_output_parser.py:7: in <module>
    from rag.generator.output_parser import (
rag/generator/output_parser.py:7: in <module>
    import structlog
E   ModuleNotFoundError: No module named 'structlog'
_______________ ERROR collecting tests/unit/test_pii_scrubber.py _______________
ImportError while importing test module '/Users/user/Documents/codepath/pathreview-ai301-fa26-s1/tests/unit/test_pii_scrubber.py'.
Hint: make sure your test modules/packages have valid Python names.
Traceback:
/opt/homebrew/Cellar/python@3.13/3.13.16/Frameworks/Python.framework/Versions/3.13/lib/python3.13/importlib/__init__.py:88: in import_module
    return _bootstrap._gcd_import(name[level:], package, level)
           ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
tests/unit/test_pii_scrubber.py:5: in <module>
    from safety.pii_scrubber import PIIScrubber
safety/pii_scrubber.py:5: in <module>
    import structlog
E   ModuleNotFoundError: No module named 'structlog'
______________ ERROR collecting tests/unit/test_prompt_defense.py ______________
ImportError while importing test module '/Users/user/Documents/codepath/pathreview-ai301-fa26-s1/tests/unit/test_prompt_defense.py'.
Hint: make sure your test modules/packages have valid Python names.
Traceback:
/opt/homebrew/Cellar/python@3.13/3.13.16/Frameworks/Python.framework/Versions/3.13/lib/python3.13/importlib/__init__.py:88: in import_module
    return _bootstrap._gcd_import(name[level:], package, level)
           ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
tests/unit/test_prompt_defense.py:5: in <module>
    from safety.prompt_defense import PromptDefense
safety/prompt_defense.py:5: in <module>
    import structlog
E   ModuleNotFoundError: No module named 'structlog'
_____________ ERROR collecting tests/unit/test_prompt_templates.py _____________
ImportError while importing test module '/Users/user/Documents/codepath/pathreview-ai301-fa26-s1/tests/unit/test_prompt_templates.py'.
Hint: make sure your test modules/packages have valid Python names.
Traceback:
/opt/homebrew/Cellar/python@3.13/3.13.16/Frameworks/Python.framework/Versions/3.13/lib/python3.13/importlib/__init__.py:88: in import_module
    return _bootstrap._gcd_import(name[level:], package, level)
           ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
tests/unit/test_prompt_templates.py:7: in <module>
    from rag.generator.prompt_templates import PROMPT_TEMPLATES, get_template
rag/generator/prompt_templates.py:3: in <module>
    import structlog
E   ModuleNotFoundError: No module named 'structlog'
_______________ ERROR collecting tests/unit/test_rate_limiter.py _______________
ImportError while importing test module '/Users/user/Documents/codepath/pathreview-ai301-fa26-s1/tests/unit/test_rate_limiter.py'.
Hint: make sure your test modules/packages have valid Python names.
Traceback:
/opt/homebrew/Cellar/python@3.13/3.13.16/Frameworks/Python.framework/Versions/3.13/lib/python3.13/importlib/__init__.py:88: in import_module
    return _bootstrap._gcd_import(name[level:], package, level)
           ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
tests/unit/test_rate_limiter.py:7: in <module>
    from safety.rate_limiter import RateLimiter
safety/rate_limiter.py:5: in <module>
    import redis
E   ModuleNotFoundError: No module named 'redis'
______________ ERROR collecting tests/unit/test_readme_scorer.py _______________
ImportError while importing test module '/Users/user/Documents/codepath/pathreview-ai301-fa26-s1/tests/unit/test_readme_scorer.py'.
Hint: make sure your test modules/packages have valid Python names.
Traceback:
/opt/homebrew/Cellar/python@3.13/3.13.16/Frameworks/Python.framework/Versions/3.13/lib/python3.13/importlib/__init__.py:88: in import_module
    return _bootstrap._gcd_import(name[level:], package, level)
           ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
tests/unit/test_readme_scorer.py:5: in <module>
    from agent.tools.readme_scorer import ReadmeScorer
agent/tools/readme_scorer.py:5: in <module>
    import structlog
E   ModuleNotFoundError: No module named 'structlog'
_____________ ERROR collecting tests/unit/test_relevance_scorer.py _____________
ImportError while importing test module '/Users/user/Documents/codepath/pathreview-ai301-fa26-s1/tests/unit/test_relevance_scorer.py'.
Hint: make sure your test modules/packages have valid Python names.
Traceback:
/opt/homebrew/Cellar/python@3.13/3.13.16/Frameworks/Python.framework/Versions/3.13/lib/python3.13/importlib/__init__.py:88: in import_module
    return _bootstrap._gcd_import(name[level:], package, level)
           ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
tests/unit/test_relevance_scorer.py:5: in <module>
    from rag.evaluator.relevance_scorer import RelevanceScorer
rag/evaluator/relevance_scorer.py:3: in <module>
    import structlog
E   ModuleNotFoundError: No module named 'structlog'
______________ ERROR collecting tests/unit/test_resume_parser.py _______________
ImportError while importing test module '/Users/user/Documents/codepath/pathreview-ai301-fa26-s1/tests/unit/test_resume_parser.py'.
Hint: make sure your test modules/packages have valid Python names.
Traceback:
/opt/homebrew/Cellar/python@3.13/3.13.16/Frameworks/Python.framework/Versions/3.13/lib/python3.13/importlib/__init__.py:88: in import_module
    return _bootstrap._gcd_import(name[level:], package, level)
           ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
tests/unit/test_resume_parser.py:8: in <module>
    from ingestion.parsers.resume_parser import ResumeParser
ingestion/parsers/resume_parser.py:4: in <module>
    from pypdf import PdfReader
E   ModuleNotFoundError: No module named 'pypdf'
______________ ERROR collecting tests/unit/test_review_service.py ______________
ImportError while importing test module '/Users/user/Documents/codepath/pathreview-ai301-fa26-s1/tests/unit/test_review_service.py'.
Hint: make sure your test modules/packages have valid Python names.
Traceback:
/opt/homebrew/Cellar/python@3.13/3.13.16/Frameworks/Python.framework/Versions/3.13/lib/python3.13/importlib/__init__.py:88: in import_module
    return _bootstrap._gcd_import(name[level:], package, level)
           ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
tests/unit/test_review_service.py:8: in <module>
    from core.services.review_service import (
core/services/review_service.py:5: in <module>
    import structlog
E   ModuleNotFoundError: No module named 'structlog'
_____________ ERROR collecting tests/unit/test_semantic_chunker.py _____________
ImportError while importing test module '/Users/user/Documents/codepath/pathreview-ai301-fa26-s1/tests/unit/test_semantic_chunker.py'.
Hint: make sure your test modules/packages have valid Python names.
Traceback:
/opt/homebrew/Cellar/python@3.13/3.13.16/Frameworks/Python.framework/Versions/3.13/lib/python3.13/importlib/__init__.py:88: in import_module
    return _bootstrap._gcd_import(name[level:], package, level)
           ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
tests/unit/test_semantic_chunker.py:6: in <module>
    from ingestion.chunking.semantic_chunker import SemanticChunker
ingestion/chunking/semantic_chunker.py:3: in <module>
    import tiktoken
E   ModuleNotFoundError: No module named 'tiktoken'
____________ ERROR collecting tests/unit/test_structural_chunker.py ____________
ImportError while importing test module '/Users/user/Documents/codepath/pathreview-ai301-fa26-s1/tests/unit/test_structural_chunker.py'.
Hint: make sure your test modules/packages have valid Python names.
Traceback:
/opt/homebrew/Cellar/python@3.13/3.13.16/Frameworks/Python.framework/Versions/3.13/lib/python3.13/importlib/__init__.py:88: in import_module
    return _bootstrap._gcd_import(name[level:], package, level)
           ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
tests/unit/test_structural_chunker.py:6: in <module>
    from ingestion.chunking.structural_chunker import StructuralChunker
ingestion/chunking/structural_chunker.py:3: in <module>
    import tiktoken
E   ModuleNotFoundError: No module named 'tiktoken'
______________ ERROR collecting tests/unit/test_tech_detector.py _______________
ImportError while importing test module '/Users/user/Documents/codepath/pathreview-ai301-fa26-s1/tests/unit/test_tech_detector.py'.
Hint: make sure your test modules/packages have valid Python names.
Traceback:
/opt/homebrew/Cellar/python@3.13/3.13.16/Frameworks/Python.framework/Versions/3.13/lib/python3.13/importlib/__init__.py:88: in import_module
    return _bootstrap._gcd_import(name[level:], package, level)
           ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
tests/unit/test_tech_detector.py:5: in <module>
    from agent.tools.tech_detector import TechDetector
agent/tools/tech_detector.py:3: in <module>
    import structlog
E   ModuleNotFoundError: No module named 'structlog'
=============================== warnings summary ===============================
core/config.py:7
  /Users/user/Documents/codepath/pathreview-ai301-fa26-s1/core/config.py:7: PydanticDeprecatedSince20: Support for class-based `config` is deprecated, use ConfigDict instead. Deprecated in Pydantic V2.0 to be removed in V3.0. See Pydantic V2 Migration Guide at https://errors.pydantic.dev/2.13/migration/
    class Settings(BaseSettings):

-- Docs: https://docs.pytest.org/en/stable/how-to/capture-warnings.html
=========================== short test summary info ============================
ERROR tests/unit/test_batch_processor.py
ERROR tests/unit/test_bias_detector.py
ERROR tests/unit/test_faithfulness_checker.py
ERROR tests/unit/test_keyword_search.py
ERROR tests/unit/test_llm_provider_contract.py
ERROR tests/unit/test_output_parser.py
ERROR tests/unit/test_pii_scrubber.py
ERROR tests/unit/test_prompt_defense.py
ERROR tests/unit/test_prompt_templates.py
ERROR tests/unit/test_rate_limiter.py
ERROR tests/unit/test_readme_scorer.py
ERROR tests/unit/test_relevance_scorer.py
ERROR tests/unit/test_resume_parser.py
ERROR tests/unit/test_review_service.py
ERROR tests/unit/test_semantic_chunker.py
ERROR tests/unit/test_structural_chunker.py
ERROR tests/unit/test_tech_detector.py
!!!!!!!!!!!!!!!!!!! Interrupted: 17 errors during collection !!!!!!!!!!!!!!!!!!!
======================== 1 warning, 17 errors in 1.37s =========================
make: *** [test-unit] Error 2

Exit status: 2
```

### make test-integration

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

### make lint

```text
$ make lint
.venv/bin/ruff check .
All checks passed!

Exit status: 0
```

### make typecheck

```text
$ make typecheck
.venv/bin/mypy api/ core/ ingestion/ rag/ agent/ safety/
error: INTERNAL ERROR -- Please try using mypy master on GitHub:
https://mypy.readthedocs.io/en/stable/common_issues.html#using-a-development-mypy-build
If this issue continues with mypy master, please report a bug at https://github.com/python/mypy/issues
version: 2.3.1
note: please use --show-traceback to print a traceback when reporting a bug
make: *** [typecheck] Error 2

Exit status: 2
```

### .venv/bin/python -m unit3-evidence.repro

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

### .venv/bin/python -m pytest tests/unit/test_security.py -v

```text
$ .venv/bin/python -m pytest tests/unit/test_security.py -v
============================= test session starts ==============================
platform darwin -- Python 3.13.16, pytest-9.1.1, pluggy-1.6.0 -- /Users/user/Documents/codepath/pathreview-ai301-fa26-s1/.venv/bin/python
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
======================== 29 passed, 1 warning in 9.17s =========================

Exit status: 0
```

### .venv/bin/python -m pytest tests/unit/test_security.py -k test_verify_with_wrong_hash_format -v --runxfail

```text
$ .venv/bin/python -m pytest tests/unit/test_security.py -k test_verify_with_wrong_hash_format -v --runxfail
============================= test session starts ==============================
platform darwin -- Python 3.13.16, pytest-9.1.1, pluggy-1.6.0 -- /Users/user/Documents/codepath/pathreview-ai301-fa26-s1/.venv/bin/python
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
================= 4 passed, 25 deselected, 1 warning in 0.22s ==================

Exit status: 0
```

### .venv/bin/black --check core/security.py tests/unit/test_security.py

```text
$ .venv/bin/black --check core/security.py tests/unit/test_security.py
All done! ✨ 🍰 ✨
2 files would be left unchanged.

Exit status: 0
```

### git diff main...HEAD --check

```text
$ git diff main...HEAD --check

Exit status: 0
```

### git diff main...HEAD --stat

```text
$ git diff main...HEAD --stat
 core/security.py            |  8 ++++++--
 tests/unit/test_security.py | 22 ++++++++++++++++------
 2 files changed, 22 insertions(+), 8 deletions(-)

Exit status: 0
```

## Local environment repair and retries

The initial full unit run could not collect 17 modules because the local venv lacked declared project dependencies. Installed the repository's declared project/dev dependencies with `.venv/bin/python -m pip install -e '.[dev]'` (exit 0). The first mypy failure came from a corrupt local SQLite cache; preserved that cache outside the checkout before retrying. The next run hit NumPy 2.4 stubs using Python 3.12-only type statements while the project targets Python 3.11. Installed `numpy>=1.26,<2.3` (NumPy 2.2.6, exit 0), within the project's declared dependency range, and reran typecheck. No repository dependency or configuration files changed.

Black's whole-directory scan flagged only the ignored `unit3-evidence/repro.py` helper; the subsequent command below checks all tracked Python files, matching the scope of a clean CI checkout. Installed the frontend's existing lockfile dependencies using `npm ci` (exit 0); no frontend source or lockfile changed. All original failures and final results are retained below.

### make test-unit

```text
$ make test-unit
.venv/bin/pytest tests/unit -v -m unit
============================= test session starts ==============================
platform darwin -- Python 3.13.16, pytest-9.1.1, pluggy-1.6.0 -- /Users/user/Documents/codepath/pathreview-ai301-fa26-s1/.venv/bin/python
cachedir: .pytest_cache
hypothesis profile 'default'
benchmark: 5.3.0 (defaults: timer=time.perf_counter disable_gc=False min_rounds=5 min_time=0.000005 max_time=1.0 calibration_precision=10 warmup=False warmup_iterations=100000)
rootdir: /Users/user/Documents/codepath/pathreview-ai301-fa26-s1
configfile: pyproject.toml
plugins: pytest_httpserver-1.2.0, hypothesis-6.168.5, platformdirs-4.12.2, cov-7.1.0, asyncio-1.4.0, benchmark-5.3.0, anyio-4.15.1
asyncio: mode=Mode.STRICT, debug=False, asyncio_default_fixture_loop_scope=None, asyncio_default_test_loop_scope=function
collecting ... collected 432 items

tests/unit/test_batch_processor.py::TestBatchEmbeddingProcessor::test_empty_chunks_list_returns_empty XFAIL [  0%]
tests/unit/test_batch_processor.py::TestBatchEmbeddingProcessor::test_normal_chunks_list_processes PASSED [  0%]
tests/unit/test_batch_processor.py::TestBatchEmbeddingProcessor::test_batches_chunks_correctly PASSED [  0%]
tests/unit/test_batch_processor.py::TestBatchEmbeddingProcessor::test_returns_chunk_and_embedding_id_tuples PASSED [  0%]
tests/unit/test_batch_processor.py::TestBatchEmbeddingProcessor::test_embedding_provider_called_with_texts PASSED [  1%]
tests/unit/test_batch_processor.py::TestBatchEmbeddingProcessor::test_chunk_metadata_preserved PASSED [  1%]
tests/unit/test_batch_processor.py::TestBatchEmbeddingProcessor::test_batch_size_limit PASSED [  1%]
tests/unit/test_batch_processor.py::TestBatchEmbeddingProcessor::test_many_chunks PASSED [  1%]
tests/unit/test_batch_processor.py::TestBatchEmbeddingProcessor::test_embedding_provider_exception_propagates PASSED [  2%]
tests/unit/test_batch_processor.py::TestBatchEmbeddingProcessor::test_store_embedding_called_for_each_chunk PASSED [  2%]
tests/unit/test_batch_processor.py::TestBatchEmbeddingProcessor::test_chunk_text_extraction PASSED [  2%]
tests/unit/test_bias_detector.py::TestBiasDetector::test_dismissive_bootcamp_language_detected XFAIL [  2%]
tests/unit/test_bias_detector.py::TestBiasDetector::test_bootcamp_lacks_rigor_detected XFAIL [  3%]
tests/unit/test_bias_detector.py::TestBiasDetector::test_bootcamp_inadequate_training_detected PASSED [  3%]
tests/unit/test_bias_detector.py::TestBiasDetector::test_self_taught_comparison_detected PASSED [  3%]
tests/unit/test_bias_detector.py::TestBiasDetector::test_positive_bootcamp_mention_not_flagged PASSED [  3%]
tests/unit/test_bias_detector.py::TestBiasDetector::test_neutral_bootcamp_mention_not_flagged PASSED [  3%]
tests/unit/test_bias_detector.py::TestBiasDetector::test_educational_background_positive_not_flagged PASSED [  4%]
tests/unit/test_bias_detector.py::TestBiasDetector::test_demographic_assumption_age_detected XFAIL [  4%]
tests/unit/test_bias_detector.py::TestBiasDetector::test_demographic_assumption_old_detected PASSED [  4%]
tests/unit/test_bias_detector.py::TestBiasDetector::test_demographic_assumption_background_detected PASSED [  4%]
tests/unit/test_bias_detector.py::TestBiasDetector::test_immigrant_developer_assumption_detected PASSED [  5%]
tests/unit/test_bias_detector.py::TestBiasDetector::test_international_developer_assumption_detected PASSED [  5%]
tests/unit/test_bias_detector.py::TestBiasDetector::test_clean_feedback_not_flagged PASSED [  5%]
tests/unit/test_bias_detector.py::TestBiasDetector::test_technical_feedback_not_flagged PASSED [  5%]
tests/unit/test_bias_detector.py::TestBiasDetector::test_development_suggestion_not_flagged PASSED [  6%]
tests/unit/test_bias_detector.py::TestBiasDetector::test_case_insensitive_detection PASSED [  6%]
tests/unit/test_bias_detector.py::TestBiasDetector::test_return_value_structure PASSED [  6%]
tests/unit/test_bias_detector.py::TestBiasDetector::test_unbiased_returns_empty_reason PASSED [  6%]
tests/unit/test_bias_detector.py::TestBiasDetector::test_biased_returns_reason PASSED [  6%]
tests/unit/test_bias_detector.py::TestBiasDetector::test_empty_text PASSED [  7%]
tests/unit/test_bias_detector.py::TestBiasDetector::test_whitespace_only PASSED [  7%]
tests/unit/test_bias_detector.py::TestBiasDetector::test_coding_bootcamp_variant XFAIL [  7%]
tests/unit/test_bias_detector.py::TestBiasDetector::test_online_course_bias_detected PASSED [  7%]
tests/unit/test_bias_detector.py::TestBiasDetector::test_developer_vs_programmer_distinction XFAIL [  8%]
tests/unit/test_bias_detector.py::TestBiasDetector::test_multiple_bias_indicators XFAIL [  8%]
tests/unit/test_bias_detector.py::TestBiasDetector::test_negative_educational_claim XFAIL [  8%]
tests/unit/test_bias_detector.py::TestBiasDetector::test_working_class_assumption PASSED [  8%]
tests/unit/test_bias_detector.py::TestBiasDetector::test_rich_poor_assumption XFAIL [  9%]
tests/unit/test_bias_detector.py::TestBiasDetector::test_foreign_developer_struggle_assumption PASSED [  9%]
tests/unit/test_bias_detector.py::TestBiasDetector::test_skill_assessment_not_biased PASSED [  9%]
tests/unit/test_bias_detector.py::TestBiasDetector::test_comparative_without_bias PASSED [  9%]
tests/unit/test_bias_detector.py::TestBiasDetector::test_assumption_vs_observation XFAIL [  9%]
tests/unit/test_faithfulness_checker.py::TestFaithfulnessChecker::test_feedback_fully_supported_by_context PASSED [ 10%]
tests/unit/test_faithfulness_checker.py::TestFaithfulnessChecker::test_feedback_with_no_support_in_context PASSED [ 10%]
tests/unit/test_faithfulness_checker.py::TestFaithfulnessChecker::test_partial_support_returns_middle_score XFAIL [ 10%]
tests/unit/test_faithfulness_checker.py::TestFaithfulnessChecker::test_empty_feedback_returns_zero PASSED [ 10%]
tests/unit/test_faithfulness_checker.py::TestFaithfulnessChecker::test_empty_context_chunks_returns_zero PASSED [ 11%]
tests/unit/test_faithfulness_checker.py::TestFaithfulnessChecker::test_both_empty_returns_zero PASSED [ 11%]
tests/unit/test_faithfulness_checker.py::TestFaithfulnessChecker::test_multiple_context_chunks XFAIL [ 11%]
tests/unit/test_faithfulness_checker.py::TestFaithfulnessChecker::test_extract_claims PASSED [ 11%]
tests/unit/test_faithfulness_checker.py::TestFaithfulnessChecker::test_extract_claims_with_punctuation PASSED [ 12%]
tests/unit/test_faithfulness_checker.py::TestFaithfulnessChecker::test_is_supported_with_keyword_overlap PASSED [ 12%]
tests/unit/test_faithfulness_checker.py::TestFaithfulnessChecker::test_is_supported_without_keywords PASSED [ 12%]
tests/unit/test_faithfulness_checker.py::TestFaithfulnessChecker::test_case_insensitive_support_check PASSED [ 12%]
tests/unit/test_faithfulness_checker.py::TestFaithfulnessChecker::test_score_never_returns_hardcoded_value PASSED [ 12%]
tests/unit/test_faithfulness_checker.py::TestFaithfulnessChecker::test_multiple_claims_varying_support XFAIL [ 13%]
tests/unit/test_faithfulness_checker.py::TestFaithfulnessChecker::test_very_long_feedback PASSED [ 13%]
tests/unit/test_faithfulness_checker.py::TestFaithfulnessChecker::test_very_long_context PASSED [ 13%]
tests/unit/test_faithfulness_checker.py::TestFaithfulnessChecker::test_common_words_filtered_in_overlap PASSED [ 13%]
tests/unit/test_faithfulness_checker.py::TestFaithfulnessChecker::test_minimum_overlap_required PASSED [ 14%]
tests/unit/test_faithfulness_checker.py::TestFaithfulnessChecker::test_none_context_chunk_text XFAIL [ 14%]
tests/unit/test_faithfulness_checker.py::TestFaithfulnessChecker::test_missing_text_key_in_chunk PASSED [ 14%]
tests/unit/test_faithfulness_checker.py::TestFaithfulnessChecker::test_score_consistency PASSED [ 14%]
tests/unit/test_faithfulness_checker.py::TestFaithfulnessChecker::test_specialized_technical_terms PASSED [ 15%]
tests/unit/test_keyword_search.py::TestKeywordSearcher::test_results_sorted_by_score_descending PASSED [ 15%]
tests/unit/test_keyword_search.py::TestKeywordSearcher::test_query_matching_no_documents PASSED [ 15%]
tests/unit/test_keyword_search.py::TestKeywordSearcher::test_query_matching_multiple_documents PASSED [ 15%]
tests/unit/test_keyword_search.py::TestKeywordSearcher::test_case_insensitive_matching PASSED [ 15%]
tests/unit/test_keyword_search.py::TestKeywordSearcher::test_top_k_limit PASSED [ 16%]
tests/unit/test_keyword_search.py::TestKeywordSearcher::test_top_k_larger_than_results PASSED [ 16%]
tests/unit/test_keyword_search.py::TestKeywordSearcher::test_results_have_bm25_score PASSED [ 16%]
tests/unit/test_keyword_search.py::TestKeywordSearcher::test_results_preserve_chunk_fields PASSED [ 16%]
tests/unit/test_keyword_search.py::TestKeywordSearcher::test_empty_index XFAIL [ 17%]
tests/unit/test_keyword_search.py::TestKeywordSearcher::test_index_not_called_returns_empty PASSED [ 17%]
tests/unit/test_keyword_search.py::TestKeywordSearcher::test_multi_word_query PASSED [ 17%]
tests/unit/test_keyword_search.py::TestKeywordSearcher::test_tokenization PASSED [ 17%]
tests/unit/test_keyword_search.py::TestKeywordSearcher::test_tokenization_case_handling PASSED [ 18%]
tests/unit/test_keyword_search.py::TestKeywordSearcher::test_large_corpus PASSED [ 18%]
tests/unit/test_keyword_search.py::TestKeywordSearcher::test_special_characters_in_query PASSED [ 18%]
tests/unit/test_keyword_search.py::TestKeywordSearcher::test_exact_phrase_matching PASSED [ 18%]
tests/unit/test_keyword_search.py::TestKeywordSearcher::test_single_word_chunks PASSED [ 18%]
tests/unit/test_llm_provider_contract.py::TestLLMProviderContract::test_mock_provider_returns_list_of_floats PASSED [ 19%]
tests/unit/test_llm_provider_contract.py::TestLLMProviderContract::test_openai_provider_would_return_floats PASSED [ 19%]
tests/unit/test_llm_provider_contract.py::TestLLMProviderContract::test_mock_provider_deterministic PASSED [ 19%]
tests/unit/test_llm_provider_contract.py::TestLLMProviderContract::test_mock_provider_embedding_dimension PASSED [ 19%]
tests/unit/test_llm_provider_contract.py::TestLLMProviderContract::test_providers_same_dimension PASSED [ 20%]
tests/unit/test_llm_provider_contract.py::TestLLMProviderContract::test_mock_provider_multiple_texts PASSED [ 20%]
tests/unit/test_llm_provider_contract.py::TestLLMProviderContract::test_mock_provider_empty_list PASSED [ 20%]
tests/unit/test_llm_provider_contract.py::TestLLMProviderContract::test_mock_provider_single_text PASSED [ 20%]
tests/unit/test_llm_provider_contract.py::TestLLMProviderContract::test_mock_provider_empty_string PASSED [ 21%]
tests/unit/test_llm_provider_contract.py::TestLLMProviderContract::test_mock_provider_different_inputs_different_outputs PASSED [ 21%]
tests/unit/test_llm_provider_contract.py::TestLLMProviderContract::test_mock_provider_embeddings_normalized PASSED [ 21%]
tests/unit/test_llm_provider_contract.py::TestLLMProviderContract::test_provider_interface_implements_embed PASSED [ 21%]
tests/unit/test_llm_provider_contract.py::TestLLMProviderContract::test_mock_provider_returns_floats_not_integers PASSED [ 21%]
tests/unit/test_llm_provider_contract.py::TestLLMProviderContract::test_multiple_providers_same_interface PASSED [ 22%]
tests/unit/test_llm_provider_contract.py::TestLLMProviderContract::test_mock_provider_consistency_across_calls PASSED [ 22%]
tests/unit/test_llm_provider_contract.py::TestLLMProviderContract::test_provider_embed_signature PASSED [ 22%]
tests/unit/test_llm_provider_contract.py::TestLLMProviderContract::test_mock_provider_with_very_long_text PASSED [ 22%]
tests/unit/test_llm_provider_contract.py::TestLLMProviderContract::test_mock_provider_with_special_characters PASSED [ 23%]
tests/unit/test_llm_provider_contract.py::TestLLMProviderContract::test_mock_provider_with_unicode PASSED [ 23%]
tests/unit/test_llm_provider_contract.py::TestLLMProviderContract::test_mock_provider_with_newlines PASSED [ 23%]
tests/unit/test_llm_provider_contract.py::TestLLMProviderContract::test_mock_provider_with_tabs PASSED [ 23%]
tests/unit/test_llm_provider_contract.py::TestLLMProviderContract::test_different_text_lengths_same_output_dimension PASSED [ 24%]
tests/unit/test_llm_provider_contract.py::TestLLMProviderContract::test_mock_provider_batch_processing PASSED [ 24%]
tests/unit/test_llm_provider_contract.py::TestLLMProviderContract::test_embedding_vector_values_in_valid_range PASSED [ 24%]
tests/unit/test_llm_provider_contract.py::TestLLMProviderContract::test_providers_inherit_from_abstract_base PASSED [ 24%]
tests/unit/test_llm_provider_contract.py::TestLLMProviderContract::test_abstract_base_class_has_embed_method PASSED [ 25%]
tests/unit/test_llm_provider_contract.py::TestLLMProviderContract::test_mock_provider_embedding_stability PASSED [ 25%]
tests/unit/test_output_parser.py::TestOutputParser::test_json_wrapped_in_code_fence PASSED [ 25%]
tests/unit/test_output_parser.py::TestOutputParser::test_raw_json_without_fence PASSED [ 25%]
tests/unit/test_output_parser.py::TestOutputParser::test_plain_text_fallback PASSED [ 25%]
tests/unit/test_output_parser.py::TestOutputParser::test_malformed_json_fallback PASSED [ 26%]
tests/unit/test_output_parser.py::TestOutputParser::test_feedback_section_has_required_fields PASSED [ 26%]
tests/unit/test_output_parser.py::TestOutputParser::test_json_with_multiple_sections PASSED [ 26%]
tests/unit/test_output_parser.py::TestOutputParser::test_confidence_scores_in_sections PASSED [ 26%]
tests/unit/test_output_parser.py::TestOutputParser::test_empty_json_object PASSED [ 27%]
tests/unit/test_output_parser.py::TestOutputParser::test_json_array_fallback XFAIL [ 27%]
tests/unit/test_output_parser.py::TestOutputParser::test_very_long_plain_text PASSED [ 27%]
tests/unit/test_output_parser.py::TestOutputParser::test_html_in_feedback PASSED [ 27%]
tests/unit/test_output_parser.py::TestOutputParser::test_code_in_feedback PASSED [ 28%]
tests/unit/test_output_parser.py::TestOutputParser::test_section_suggestions_extraction PASSED [ 28%]
tests/unit/test_output_parser.py::TestOutputParser::test_unicode_characters PASSED [ 28%]
tests/unit/test_output_parser.py::TestOutputParser::test_nested_json_structure PASSED [ 28%]
tests/unit/test_output_parser.py::TestOutputParser::test_mixed_content PASSED [ 28%]
tests/unit/test_output_parser.py::TestOutputParser::test_plaintext_output_helper PASSED [ 29%]
tests/unit/test_output_parser.py::TestOutputParser::test_multiple_code_fences PASSED [ 29%]
tests/unit/test_output_parser.py::TestOutputParser::test_no_suggestions_key PASSED [ 29%]
tests/unit/test_pii_scrubber.py::TestPIIScrubber::test_email_redaction PASSED [ 29%]
tests/unit/test_pii_scrubber.py::TestPIIScrubber::test_multiple_emails_redacted PASSED [ 30%]
tests/unit/test_pii_scrubber.py::TestPIIScrubber::test_us_phone_number_redaction XFAIL [ 30%]
tests/unit/test_pii_scrubber.py::TestPIIScrubber::test_us_phone_formats XFAIL [ 30%]
tests/unit/test_pii_scrubber.py::TestPIIScrubber::test_international_phone_redaction PASSED [ 30%]
tests/unit/test_pii_scrubber.py::TestPIIScrubber::test_ssn_redaction PASSED [ 31%]
tests/unit/test_pii_scrubber.py::TestPIIScrubber::test_ssn_variations PASSED [ 31%]
tests/unit/test_pii_scrubber.py::TestPIIScrubber::test_street_address_redaction PASSED [ 31%]
tests/unit/test_pii_scrubber.py::TestPIIScrubber::test_text_with_no_pii PASSED [ 31%]
tests/unit/test_pii_scrubber.py::TestPIIScrubber::test_detect_returns_list_of_pii PASSED [ 31%]
tests/unit/test_pii_scrubber.py::TestPIIScrubber::test_detect_email_pii PASSED [ 32%]
tests/unit/test_pii_scrubber.py::TestPIIScrubber::test_detect_phone_pii XFAIL [ 32%]
tests/unit/test_pii_scrubber.py::TestPIIScrubber::test_detect_ssn_pii PASSED [ 32%]
tests/unit/test_pii_scrubber.py::TestPIIScrubber::test_detect_positions_accurate PASSED [ 32%]
tests/unit/test_pii_scrubber.py::TestPIIScrubber::test_detect_multiple_pii_items PASSED [ 33%]
tests/unit/test_pii_scrubber.py::TestPIIScrubber::test_case_insensitive_email_matching PASSED [ 33%]
tests/unit/test_pii_scrubber.py::TestPIIScrubber::test_complex_email_addresses PASSED [ 33%]
tests/unit/test_pii_scrubber.py::TestPIIScrubber::test_phone_at_start_of_text XFAIL [ 33%]
tests/unit/test_pii_scrubber.py::TestPIIScrubber::test_phone_at_end_of_text PASSED [ 34%]
tests/unit/test_pii_scrubber.py::TestPIIScrubber::test_address_variations PASSED [ 34%]
tests/unit/test_pii_scrubber.py::TestPIIScrubber::test_empty_text PASSED [ 34%]
tests/unit/test_pii_scrubber.py::TestPIIScrubber::test_whitespace_only PASSED [ 34%]
tests/unit/test_pii_scrubber.py::TestPIIScrubber::test_mixed_pii_and_text XFAIL [ 34%]
tests/unit/test_pii_scrubber.py::TestPIIScrubber::test_scrub_idempotent PASSED [ 35%]
tests/unit/test_pii_scrubber.py::TestPIIScrubber::test_detect_no_false_positives PASSED [ 35%]
tests/unit/test_prompt_defense.py::TestPromptDefense::test_newline_injection_detected PASSED [ 35%]
tests/unit/test_prompt_defense.py::TestPromptDefense::test_system_role_switching_detected PASSED [ 35%]
tests/unit/test_prompt_defense.py::TestPromptDefense::test_human_role_switching_detected PASSED [ 36%]
tests/unit/test_prompt_defense.py::TestPromptDefense::test_assistant_role_switching_detected PASSED [ 36%]
tests/unit/test_prompt_defense.py::TestPromptDefense::test_template_injection_detected PASSED [ 36%]
tests/unit/test_prompt_defense.py::TestPromptDefense::test_jinja_injection_detected PASSED [ 36%]
tests/unit/test_prompt_defense.py::TestPromptDefense::test_ignore_instruction_detected PASSED [ 37%]
tests/unit/test_prompt_defense.py::TestPromptDefense::test_forget_instruction_detected PASSED [ 37%]
tests/unit/test_prompt_defense.py::TestPromptDefense::test_disregard_instruction_detected PASSED [ 37%]
tests/unit/test_prompt_defense.py::TestPromptDefense::test_override_instruction_detected PASSED [ 37%]
tests/unit/test_prompt_defense.py::TestPromptDefense::test_code_execution_attempt_detected PASSED [ 37%]
tests/unit/test_prompt_defense.py::TestPromptDefense::test_run_function_attempt_detected PASSED [ 38%]
tests/unit/test_prompt_defense.py::TestPromptDefense::test_eval_function_attempt_detected PASSED [ 38%]
tests/unit/test_prompt_defense.py::TestPromptDefense::test_clean_input_not_flagged PASSED [ 38%]
tests/unit/test_prompt_defense.py::TestPromptDefense::test_normal_text_not_flagged PASSED [ 38%]
tests/unit/test_prompt_defense.py::TestPromptDefense::test_sanitize_removes_template_delimiters PASSED [ 39%]
tests/unit/test_prompt_defense.py::TestPromptDefense::test_sanitize_removes_angle_brackets PASSED [ 39%]
tests/unit/test_prompt_defense.py::TestPromptDefense::test_sanitize_preserves_legitimate_content PASSED [ 39%]
tests/unit/test_prompt_defense.py::TestPromptDefense::test_sanitize_multiple_template_delimiters PASSED [ 39%]
tests/unit/test_prompt_defense.py::TestPromptDefense::test_separator_line_detected PASSED [ 40%]
tests/unit/test_prompt_defense.py::TestPromptDefense::test_long_separator_detected PASSED [ 40%]
tests/unit/test_prompt_defense.py::TestPromptDefense::test_case_insensitive_detection PASSED [ 40%]
tests/unit/test_prompt_defense.py::TestPromptDefense::test_whitespace_variations_detected XFAIL [ 40%]
tests/unit/test_prompt_defense.py::TestPromptDefense::test_multiple_injection_patterns PASSED [ 40%]
tests/unit/test_prompt_defense.py::TestPromptDefense::test_benign_mentions_not_flagged PASSED [ 41%]
tests/unit/test_prompt_defense.py::TestPromptDefense::test_empty_string PASSED [ 41%]
tests/unit/test_prompt_defense.py::TestPromptDefense::test_whitespace_only PASSED [ 41%]
tests/unit/test_prompt_defense.py::TestPromptDefense::test_sanitize_idempotent PASSED [ 41%]
tests/unit/test_prompt_defense.py::TestPromptDefense::test_complex_injection_attempt PASSED [ 42%]
tests/unit/test_prompt_defense.py::TestPromptDefense::test_legitimate_newlines_not_flagged PASSED [ 42%]
tests/unit/test_prompt_defense.py::TestPromptDefense::test_code_blocks_handled PASSED [ 42%]
tests/unit/test_prompt_defense.py::TestPromptDefense::test_sanitize_with_mixed_delimiters PASSED [ 42%]
tests/unit/test_prompt_templates.py::TestPromptTemplates::test_all_5_templates_exist PASSED [ 43%]
tests/unit/test_prompt_templates.py::TestPromptTemplates::test_skills_feedback_template_exists PASSED [ 43%]
tests/unit/test_prompt_templates.py::TestPromptTemplates::test_projects_feedback_template_exists PASSED [ 43%]
tests/unit/test_prompt_templates.py::TestPromptTemplates::test_presentation_feedback_template_exists PASSED [ 43%]
tests/unit/test_prompt_templates.py::TestPromptTemplates::test_gaps_feedback_template_exists PASSED [ 43%]
tests/unit/test_prompt_templates.py::TestPromptTemplates::test_first_impression_template_exists PASSED [ 44%]
tests/unit/test_prompt_templates.py::TestPromptTemplates::test_each_template_contains_context_placeholder PASSED [ 44%]
tests/unit/test_prompt_templates.py::TestPromptTemplates::test_each_template_contains_github_username_placeholder PASSED [ 44%]
tests/unit/test_prompt_templates.py::TestPromptTemplates::test_each_template_contains_project_count_placeholder PASSED [ 44%]
tests/unit/test_prompt_templates.py::TestPromptTemplates::test_get_template_returns_correct_template PASSED [ 45%]
tests/unit/test_prompt_templates.py::TestPromptTemplates::test_get_template_projects_feedback PASSED [ 45%]
tests/unit/test_prompt_templates.py::TestPromptTemplates::test_get_template_presentation_feedback PASSED [ 45%]
tests/unit/test_prompt_templates.py::TestPromptTemplates::test_get_template_gaps_feedback PASSED [ 45%]
tests/unit/test_prompt_templates.py::TestPromptTemplates::test_get_template_first_impression PASSED [ 46%]
tests/unit/test_prompt_templates.py::TestPromptTemplates::test_get_template_unknown_name_raises_error PASSED [ 46%]
tests/unit/test_prompt_templates.py::TestPromptTemplates::test_get_template_unknown_version_raises_error PASSED [ 46%]
tests/unit/test_prompt_templates.py::TestPromptTemplates::test_all_templates_have_v1 PASSED [ 46%]
tests/unit/test_prompt_templates.py::TestPromptTemplates::test_templates_are_strings PASSED [ 46%]
tests/unit/test_prompt_templates.py::TestPromptTemplates::test_templates_have_reasonable_length PASSED [ 47%]
tests/unit/test_prompt_templates.py::TestPromptTemplates::test_skills_feedback_mentions_technical_skills PASSED [ 47%]
tests/unit/test_prompt_templates.py::TestPromptTemplates::test_projects_feedback_mentions_code_quality PASSED [ 47%]
tests/unit/test_prompt_templates.py::TestPromptTemplates::test_gaps_feedback_mentions_missing_skills PASSED [ 47%]
tests/unit/test_prompt_templates.py::TestPromptTemplates::test_presentation_feedback_mentions_readme PASSED [ 48%]
tests/unit/test_prompt_templates.py::TestPromptTemplates::test_first_impression_is_concise PASSED [ 48%]
tests/unit/test_prompt_templates.py::TestPromptTemplates::test_get_template_default_version PASSED [ 48%]
tests/unit/test_prompt_templates.py::TestPromptTemplates::test_template_snapshot_content_hash PASSED [ 48%]
tests/unit/test_prompt_templates.py::TestPromptTemplates::test_skills_feedback_requests_json_format PASSED [ 49%]
tests/unit/test_prompt_templates.py::TestPromptTemplates::test_projects_feedback_requests_json_format PASSED [ 49%]
tests/unit/test_prompt_templates.py::TestPromptTemplates::test_presentation_feedback_requests_json_format PASSED [ 49%]
tests/unit/test_prompt_templates.py::TestPromptTemplates::test_gaps_feedback_requests_json_format PASSED [ 49%]
tests/unit/test_prompt_templates.py::TestPromptTemplates::test_first_impression_plain_text PASSED [ 50%]
tests/unit/test_prompt_templates.py::TestPromptTemplates::test_templates_have_portfolio_context PASSED [ 50%]
tests/unit/test_prompt_templates.py::TestPromptTemplates::test_template_get_logs_retrieval PASSED [ 50%]
tests/unit/test_prompt_templates.py::TestPromptTemplates::test_each_template_name_is_valid_identifier PASSED [ 50%]
tests/unit/test_prompt_templates.py::TestPromptTemplates::test_template_versions_are_strings PASSED [ 50%]
tests/unit/test_prompt_templates.py::TestPromptTemplates::test_no_hardcoded_usernames_in_templates PASSED [ 51%]
tests/unit/test_prompt_templates.py::TestPromptTemplates::test_templates_use_consistent_placeholders PASSED [ 51%]
tests/unit/test_rate_limiter.py::TestRateLimiter::test_first_request_allowed PASSED [ 51%]
tests/unit/test_rate_limiter.py::TestRateLimiter::test_requests_up_to_limit_allowed PASSED [ 51%]
tests/unit/test_rate_limiter.py::TestRateLimiter::test_request_at_limit_plus_one_denied PASSED [ 52%]
tests/unit/test_rate_limiter.py::TestRateLimiter::test_remaining_count_correct PASSED [ 52%]
tests/unit/test_rate_limiter.py::TestRateLimiter::test_rolling_window_removes_old_entries PASSED [ 52%]
tests/unit/test_rate_limiter.py::TestRateLimiter::test_different_identifiers_independent PASSED [ 52%]
tests/unit/test_rate_limiter.py::TestRateLimiter::test_key_format_correct PASSED [ 53%]
tests/unit/test_rate_limiter.py::TestRateLimiter::test_entry_added_to_redis PASSED [ 53%]
tests/unit/test_rate_limiter.py::TestRateLimiter::test_expiry_set_correctly PASSED [ 53%]
tests/unit/test_rate_limiter.py::TestRateLimiter::test_custom_window_size PASSED [ 53%]
tests/unit/test_rate_limiter.py::TestRateLimiter::test_redis_error_handling PASSED [ 53%]
tests/unit/test_rate_limiter.py::TestRateLimiter::test_zero_limit PASSED [ 54%]
tests/unit/test_rate_limiter.py::TestRateLimiter::test_negative_limit PASSED [ 54%]
tests/unit/test_rate_limiter.py::TestRateLimiter::test_large_limit PASSED [ 54%]
tests/unit/test_rate_limiter.py::TestRateLimiter::test_return_tuple_structure PASSED [ 54%]
tests/unit/test_rate_limiter.py::TestRateLimiter::test_multiple_requests_same_user PASSED [ 55%]
tests/unit/test_rate_limiter.py::TestRateLimiter::test_time_based_window_calculation PASSED [ 55%]
tests/unit/test_rate_limiter.py::TestRateLimiter::test_ip_address_as_identifier PASSED [ 55%]
tests/unit/test_rate_limiter.py::TestRateLimiter::test_api_key_as_identifier PASSED [ 55%]
tests/unit/test_readme_parser.py::TestReadmeParser::test_parse_standard_readme XFAIL [ 56%]
tests/unit/test_readme_parser.py::TestReadmeParser::test_parse_readme_longer_than_10000_chars PASSED [ 56%]
tests/unit/test_readme_parser.py::TestReadmeParser::test_parse_readme_with_code_blocks PASSED [ 56%]
tests/unit/test_readme_parser.py::TestReadmeParser::test_parse_readme_without_code_blocks PASSED [ 56%]
tests/unit/test_readme_parser.py::TestReadmeParser::test_parse_empty_readme PASSED [ 56%]
tests/unit/test_readme_parser.py::TestReadmeParser::test_parse_readme_with_only_whitespace PASSED [ 57%]
tests/unit/test_readme_parser.py::TestReadmeParser::test_parse_readme_bytes_input PASSED [ 57%]
tests/unit/test_readme_parser.py::TestReadmeParser::test_parse_readme_bytes_with_utf8 PASSED [ 57%]
tests/unit/test_readme_parser.py::TestReadmeParser::test_parse_invalid_content_type PASSED [ 57%]
tests/unit/test_readme_parser.py::TestReadmeParser::test_extract_heading_hierarchy XFAIL [ 58%]
tests/unit/test_readme_parser.py::TestReadmeParser::test_extract_heading_hierarchy_no_headings PASSED [ 58%]
tests/unit/test_readme_parser.py::TestReadmeParser::test_parse_readme_with_badges PASSED [ 58%]
tests/unit/test_readme_parser.py::TestReadmeParser::test_parse_readme_without_badges PASSED [ 58%]
tests/unit/test_readme_parser.py::TestReadmeParser::test_metadata_structure PASSED [ 59%]
tests/unit/test_readme_parser.py::TestReadmeParser::test_word_count_accuracy PASSED [ 59%]
tests/unit/test_readme_scorer.py::TestReadmeScorer::test_readme_with_all_quality_signals XFAIL [ 59%]
tests/unit/test_readme_scorer.py::TestReadmeScorer::test_readme_with_no_content PASSED [ 59%]
tests/unit/test_readme_scorer.py::TestReadmeScorer::test_readme_with_only_title PASSED [ 59%]
tests/unit/test_readme_scorer.py::TestReadmeScorer::test_word_count_category_minimal PASSED [ 60%]
tests/unit/test_readme_scorer.py::TestReadmeScorer::test_word_count_category_adequate PASSED [ 60%]
tests/unit/test_readme_scorer.py::TestReadmeScorer::test_word_count_category_comprehensive PASSED [ 60%]
tests/unit/test_readme_scorer.py::TestReadmeScorer::test_installation_section_detection PASSED [ 60%]
tests/unit/test_readme_scorer.py::TestReadmeScorer::test_usage_section_detection PASSED [ 61%]
tests/unit/test_readme_scorer.py::TestReadmeScorer::test_setup_keyword_counts_as_installation PASSED [ 61%]
tests/unit/test_readme_scorer.py::TestReadmeScorer::test_quickstart_counts_as_usage PASSED [ 61%]
tests/unit/test_readme_scorer.py::TestReadmeScorer::test_badge_detection PASSED [ 61%]
tests/unit/test_readme_scorer.py::TestReadmeScorer::test_demo_link_detection PASSED [ 62%]
tests/unit/test_readme_scorer.py::TestReadmeScorer::test_tech_stack_section_detection PASSED [ 62%]
tests/unit/test_readme_scorer.py::TestReadmeScorer::test_technologies_keyword_counts PASSED [ 62%]
tests/unit/test_readme_scorer.py::TestReadmeScorer::test_overall_score_calculation PASSED [ 62%]
tests/unit/test_readme_scorer.py::TestReadmeScorer::test_missing_readme_content_key PASSED [ 62%]
tests/unit/test_readme_scorer.py::TestReadmeScorer::test_result_has_all_required_fields PASSED [ 63%]
tests/unit/test_readme_scorer.py::TestReadmeScorer::test_case_insensitive_section_detection PASSED [ 63%]
tests/unit/test_readme_scorer.py::TestReadmeScorer::test_whitespace_only_readme PASSED [ 63%]
tests/unit/test_readme_scorer.py::TestReadmeScorer::test_example_keyword_counts_as_usage PASSED [ 63%]
tests/unit/test_readme_scorer.py::TestReadmeScorer::test_score_scales_with_word_count PASSED [ 64%]
tests/unit/test_readme_scorer.py::TestReadmeScorer::test_try_it_as_demo_indicator PASSED [ 64%]
tests/unit/test_readme_scorer.py::TestReadmeScorer::test_built_with_counts_as_tech_stack PASSED [ 64%]
tests/unit/test_relevance_scorer.py::TestRelevanceScorer::test_query_with_perfect_keyword_match PASSED [ 64%]
tests/unit/test_relevance_scorer.py::TestRelevanceScorer::test_query_with_zero_keyword_overlap PASSED [ 65%]
tests/unit/test_relevance_scorer.py::TestRelevanceScorer::test_query_with_partial_overlap XFAIL [ 65%]
tests/unit/test_relevance_scorer.py::TestRelevanceScorer::test_empty_chunks_list_returns_zero PASSED [ 65%]
tests/unit/test_relevance_scorer.py::TestRelevanceScorer::test_empty_query_returns_zero PASSED [ 65%]
tests/unit/test_relevance_scorer.py::TestRelevanceScorer::test_multiple_chunks_aggregated PASSED [ 65%]
tests/unit/test_relevance_scorer.py::TestRelevanceScorer::test_case_insensitive_matching PASSED [ 66%]
tests/unit/test_relevance_scorer.py::TestRelevanceScorer::test_tokenization PASSED [ 66%]
tests/unit/test_relevance_scorer.py::TestRelevanceScorer::test_empty_text_in_chunk PASSED [ 66%]
tests/unit/test_relevance_scorer.py::TestRelevanceScorer::test_very_long_query PASSED [ 66%]
tests/unit/test_relevance_scorer.py::TestRelevanceScorer::test_very_long_chunk PASSED [ 67%]
tests/unit/test_relevance_scorer.py::TestRelevanceScorer::test_special_characters_ignored PASSED [ 67%]
tests/unit/test_relevance_scorer.py::TestRelevanceScorer::test_multiple_keyword_matches PASSED [ 67%]
tests/unit/test_relevance_scorer.py::TestRelevanceScorer::test_single_word_chunks PASSED [ 67%]
tests/unit/test_relevance_scorer.py::TestRelevanceScorer::test_score_ranges_from_zero_to_one PASSED [ 68%]
tests/unit/test_relevance_scorer.py::TestRelevanceScorer::test_common_words_not_preventing_scoring PASSED [ 68%]
tests/unit/test_relevance_scorer.py::TestRelevanceScorer::test_chunk_without_text_key PASSED [ 68%]
tests/unit/test_relevance_scorer.py::TestRelevanceScorer::test_whitespace_only_in_chunks PASSED [ 68%]
tests/unit/test_relevance_scorer.py::TestRelevanceScorer::test_average_relevance_calculation PASSED [ 68%]
tests/unit/test_resume_parser.py::TestResumeParser::test_parse_single_column_resume_text XFAIL [ 69%]
tests/unit/test_resume_parser.py::TestResumeParser::test_parse_resume_no_work_experience XFAIL [ 69%]
tests/unit/test_resume_parser.py::TestResumeParser::test_parse_multipage_pdf PASSED [ 69%]
tests/unit/test_resume_parser.py::TestResumeParser::test_parse_markdown_resume XFAIL [ 69%]
tests/unit/test_resume_parser.py::TestResumeParser::test_parse_invalid_content_type PASSED [ 70%]
tests/unit/test_resume_parser.py::TestResumeParser::test_parse_invalid_list_content PASSED [ 70%]
tests/unit/test_resume_parser.py::TestResumeParser::test_detect_sections XFAIL [ 70%]
tests/unit/test_resume_parser.py::TestResumeParser::test_strip_markdown_syntax XFAIL [ 70%]
tests/unit/test_resume_parser.py::TestResumeParser::test_pdf_parsing_error_handling PASSED [ 71%]
tests/unit/test_resume_parser.py::TestResumeParser::test_parse_preserves_text_content PASSED [ 71%]
tests/unit/test_review_service.py::TestReviewService::test_create_review_returns_review_with_pending_status PASSED [ 71%]
tests/unit/test_review_service.py::TestReviewService::test_get_review_returns_review_for_correct_owner XFAIL [ 71%]
tests/unit/test_review_service.py::TestReviewService::test_get_review_returns_none_for_wrong_user XFAIL [ 71%]
tests/unit/test_review_service.py::TestReviewService::test_list_reviews_returns_paginated_results XFAIL [ 72%]
tests/unit/test_review_service.py::TestReviewService::test_list_reviews_page_2_returns_correct_offset XFAIL [ 72%]
tests/unit/test_review_service.py::TestReviewService::test_list_reviews_returns_tuple XFAIL [ 72%]
tests/unit/test_review_service.py::TestReviewService::test_create_review_calls_db_add PASSED [ 72%]
tests/unit/test_review_service.py::TestReviewService::test_create_review_calls_db_commit PASSED [ 73%]
tests/unit/test_review_service.py::TestReviewService::test_create_review_calls_db_refresh PASSED [ 73%]
tests/unit/test_review_service.py::TestReviewService::test_get_review_uses_select_and_join XFAIL [ 73%]
tests/unit/test_review_service.py::TestReviewService::test_list_reviews_default_pagination XFAIL [ 73%]
tests/unit/test_review_service.py::TestReviewService::test_list_reviews_custom_page_size XFAIL [ 74%]
tests/unit/test_review_service.py::TestReviewService::test_create_review_with_uuid_ids PASSED [ 74%]
tests/unit/test_review_service.py::TestReviewService::test_get_review_verifies_ownership XFAIL [ 74%]
tests/unit/test_review_service.py::TestReviewService::test_list_reviews_counts_total XFAIL [ 74%]
tests/unit/test_review_service.py::TestReviewService::test_list_reviews_returns_reviews_list XFAIL [ 75%]
tests/unit/test_review_service.py::TestReviewService::test_review_sections_and_score_initially_none PASSED [ 75%]
tests/unit/test_review_service.py::TestReviewService::test_get_review_with_valid_uuid XFAIL [ 75%]
tests/unit/test_review_service.py::TestReviewService::test_list_reviews_ordered_by_created_at XFAIL [ 75%]
tests/unit/test_security.py::TestSecurity::test_hash_password_returns_bcrypt_hash PASSED [ 75%]
tests/unit/test_security.py::TestSecurity::test_verify_password_correct PASSED [ 76%]
tests/unit/test_security.py::TestSecurity::test_verify_password_incorrect PASSED [ 76%]
tests/unit/test_security.py::TestSecurity::test_verify_password_case_sensitive PASSED [ 76%]
tests/unit/test_security.py::TestSecurity::test_hash_same_password_different_hash PASSED [ 76%]
tests/unit/test_security.py::TestSecurity::test_create_access_token_returns_string PASSED [ 77%]
tests/unit/test_security.py::TestSecurity::test_create_access_token_is_jwt PASSED [ 77%]
tests/unit/test_security.py::TestSecurity::test_decode_access_token_valid PASSED [ 77%]
tests/unit/test_security.py::TestSecurity::test_decode_access_token_invalid_token PASSED [ 77%]
tests/unit/test_security.py::TestSecurity::test_decode_access_token_malformed PASSED [ 78%]
tests/unit/test_security.py::TestSecurity::test_decode_access_token_empty_string PASSED [ 78%]
tests/unit/test_security.py::TestSecurity::test_roundtrip_token_with_data PASSED [ 78%]
tests/unit/test_security.py::TestSecurity::test_create_access_token_with_custom_expiry PASSED [ 78%]
tests/unit/test_security.py::TestSecurity::test_access_token_includes_expiration PASSED [ 78%]
tests/unit/test_security.py::TestSecurity::test_password_hash_different_for_different_passwords PASSED [ 79%]
tests/unit/test_security.py::TestSecurity::test_verify_password_with_empty_strings PASSED [ 79%]
tests/unit/test_security.py::TestSecurity::test_verify_password_with_special_characters PASSED [ 79%]
tests/unit/test_security.py::TestSecurity::test_token_with_empty_data PASSED [ 79%]
tests/unit/test_security.py::TestSecurity::test_token_with_special_characters_in_data PASSED [ 80%]
tests/unit/test_security.py::TestSecurity::test_token_with_unicode_data PASSED [ 80%]
tests/unit/test_security.py::TestSecurity::test_hash_password_long_input PASSED [ 80%]
tests/unit/test_security.py::TestSecurity::test_verify_with_wrong_hash_format[not_a_valid_bcrypt_hash] PASSED [ 80%]
tests/unit/test_security.py::TestSecurity::test_verify_with_wrong_hash_format[] PASSED [ 81%]
tests/unit/test_security.py::TestSecurity::test_verify_with_wrong_hash_format[plaintext] PASSED [ 81%]
tests/unit/test_security.py::TestSecurity::test_verify_with_wrong_hash_format[$2b$notarealhash] PASSED [ 81%]
tests/unit/test_security.py::TestSecurity::test_verify_password_propagates_runtime_errors PASSED [ 81%]
tests/unit/test_security.py::TestSecurity::test_token_tampering_detection PASSED [ 81%]
tests/unit/test_security.py::TestSecurity::test_create_token_consistency PASSED [ 82%]
tests/unit/test_security.py::TestSecurity::test_password_with_whitespace PASSED [ 82%]
tests/unit/test_semantic_chunker.py::TestSemanticChunker::test_empty_input_returns_empty_list PASSED [ 82%]
tests/unit/test_semantic_chunker.py::TestSemanticChunker::test_whitespace_only_input PASSED [ 82%]
tests/unit/test_semantic_chunker.py::TestSemanticChunker::test_short_text_returns_single_chunk PASSED [ 83%]
tests/unit/test_semantic_chunker.py::TestSemanticChunker::test_long_text_returns_multiple_chunks PASSED [ 83%]
tests/unit/test_semantic_chunker.py::TestSemanticChunker::test_chunk_metadata_includes_chunk_index PASSED [ 83%]
tests/unit/test_semantic_chunker.py::TestSemanticChunker::test_chunks_preserve_source_metadata PASSED [ 83%]
tests/unit/test_semantic_chunker.py::TestSemanticChunker::test_chunk_has_character_positions PASSED [ 84%]
tests/unit/test_semantic_chunker.py::TestSemanticChunker::test_bullet_points_not_split_mid_bullet PASSED [ 84%]
tests/unit/test_semantic_chunker.py::TestSemanticChunker::test_sentence_splitting PASSED [ 84%]
tests/unit/test_semantic_chunker.py::TestSemanticChunker::test_sentence_splitting_with_exclamation_marks PASSED [ 84%]
tests/unit/test_semantic_chunker.py::TestSemanticChunker::test_sentence_splitting_preserves_punctuation PASSED [ 84%]
tests/unit/test_semantic_chunker.py::TestSemanticChunker::test_overlap_between_chunks PASSED [ 85%]
tests/unit/test_semantic_chunker.py::TestSemanticChunker::test_chunks_have_non_zero_text PASSED [ 85%]
tests/unit/test_semantic_chunker.py::TestSemanticChunker::test_paragraph_boundaries_respected PASSED [ 85%]
tests/unit/test_semantic_chunker.py::TestSemanticChunker::test_metadata_copied_not_mutated PASSED [ 85%]
tests/unit/test_semantic_chunker.py::TestSemanticChunker::test_chunk_indexing_is_sequential PASSED [ 86%]
tests/unit/test_skill_extractor.py::TestSkillExtractor::test_text_with_python_imports PASSED [ 86%]
tests/unit/test_skill_extractor.py::TestSkillExtractor::test_text_with_python_type_annotations PASSED [ 86%]
tests/unit/test_skill_extractor.py::TestSkillExtractor::test_text_with_typescript_files XFAIL [ 86%]
tests/unit/test_skill_extractor.py::TestSkillExtractor::test_jupyter_ipynb_detection PASSED [ 87%]
tests/unit/test_skill_extractor.py::TestSkillExtractor::test_mixed_language_text PASSED [ 87%]
tests/unit/test_skill_extractor.py::TestSkillExtractor::test_frameworks_detected_with_high_confidence PASSED [ 87%]
tests/unit/test_skill_extractor.py::TestSkillExtractor::test_react_detection PASSED [ 87%]
tests/unit/test_skill_extractor.py::TestSkillExtractor::test_database_technology_detection XFAIL [ 87%]
tests/unit/test_skill_extractor.py::TestSkillExtractor::test_devops_tool_detection XFAIL [ 88%]
tests/unit/test_skill_extractor.py::TestSkillExtractor::test_skill_has_evidence_list PASSED [ 88%]
tests/unit/test_skill_extractor.py::TestSkillExtractor::test_filename_based_detection PASSED [ 88%]
tests/unit/test_skill_extractor.py::TestSkillExtractor::test_javascript_detection XFAIL [ 88%]
tests/unit/test_skill_extractor.py::TestSkillExtractor::test_docker_compose_detection XFAIL [ 89%]
tests/unit/test_skill_extractor.py::TestSkillExtractor::test_aws_gcp_azure_detection PASSED [ 89%]
tests/unit/test_skill_extractor.py::TestSkillExtractor::test_empty_text PASSED [ 89%]
tests/unit/test_skill_extractor.py::TestSkillExtractor::test_unrecognized_language PASSED [ 89%]
tests/unit/test_skill_extractor.py::TestSkillExtractor::test_confidence_scores_are_floats PASSED [ 90%]
tests/unit/test_skill_extractor.py::TestSkillExtractor::test_skill_detection_dataclass PASSED [ 90%]
tests/unit/test_structural_chunker.py::TestStructuralChunker::test_empty_input_returns_empty_list PASSED [ 90%]
tests/unit/test_structural_chunker.py::TestStructuralChunker::test_whitespace_only_input PASSED [ 90%]
tests/unit/test_structural_chunker.py::TestStructuralChunker::test_document_with_no_headings XFAIL [ 90%]
tests/unit/test_structural_chunker.py::TestStructuralChunker::test_document_with_nested_headings PASSED [ 91%]
tests/unit/test_structural_chunker.py::TestStructuralChunker::test_heading_path_format PASSED [ 91%]
tests/unit/test_structural_chunker.py::TestStructuralChunker::test_large_section_sub_chunked PASSED [ 91%]
tests/unit/test_structural_chunker.py::TestStructuralChunker::test_chunk_metadata_includes_heading_level PASSED [ 91%]
tests/unit/test_structural_chunker.py::TestStructuralChunker::test_chunk_metadata_structure PASSED [ 92%]
tests/unit/test_structural_chunker.py::TestStructuralChunker::test_preserve_source_metadata PASSED [ 92%]
tests/unit/test_structural_chunker.py::TestStructuralChunker::test_multiple_h1_headings PASSED [ 92%]
tests/unit/test_structural_chunker.py::TestStructuralChunker::test_heading_path_breadcrumb PASSED [ 92%]
tests/unit/test_structural_chunker.py::TestStructuralChunker::test_chunks_have_text_content PASSED [ 93%]
tests/unit/test_structural_chunker.py::TestStructuralChunker::test_section_extraction_with_multiple_levels PASSED [ 93%]
tests/unit/test_structural_chunker.py::TestStructuralChunker::test_heading_not_in_middle_of_content PASSED [ 93%]
tests/unit/test_structural_chunker.py::TestStructuralChunker::test_empty_sections_handled PASSED [ 93%]
tests/unit/test_tech_detector.py::TestTechDetector::test_single_language_repo PASSED [ 93%]
tests/unit/test_tech_detector.py::TestTechDetector::test_mixed_language_repo_python_primary PASSED [ 94%]
tests/unit/test_tech_detector.py::TestTechDetector::test_ipynb_counted_as_python_not_json PASSED [ 94%]
tests/unit/test_tech_detector.py::TestTechDetector::test_node_modules_excluded XFAIL [ 94%]
tests/unit/test_tech_detector.py::TestTechDetector::test_vendor_files_excluded PASSED [ 94%]
tests/unit/test_tech_detector.py::TestTechDetector::test_build_directory_excluded XFAIL [ 95%]
tests/unit/test_tech_detector.py::TestTechDetector::test_config_file_detection PASSED [ 95%]
tests/unit/test_tech_detector.py::TestTechDetector::test_dockerfile_detection PASSED [ 95%]
tests/unit/test_tech_detector.py::TestTechDetector::test_github_actions_detection PASSED [ 95%]
tests/unit/test_tech_detector.py::TestTechDetector::test_makefile_detection PASSED [ 96%]
tests/unit/test_tech_detector.py::TestTechDetector::test_typescript_detection PASSED [ 96%]
tests/unit/test_tech_detector.py::TestTechDetector::test_go_detection PASSED [ 96%]
tests/unit/test_tech_detector.py::TestTechDetector::test_rust_detection PASSED [ 96%]
tests/unit/test_tech_detector.py::TestTechDetector::test_java_detection PASSED [ 96%]
tests/unit/test_tech_detector.py::TestTechDetector::test_multiple_python_files PASSED [ 97%]
tests/unit/test_tech_detector.py::TestTechDetector::test_empty_file_list PASSED [ 97%]
tests/unit/test_tech_detector.py::TestTechDetector::test_no_files_key PASSED [ 97%]
tests/unit/test_tech_detector.py::TestTechDetector::test_result_structure PASSED [ 97%]
tests/unit/test_tech_detector.py::TestTechDetector::test_framework_detection PASSED [ 98%]
tests/unit/test_tech_detector.py::TestTechDetector::test_unknown_extensions PASSED [ 98%]
tests/unit/test_tech_detector.py::TestTechDetector::test_case_insensitive_extension_matching PASSED [ 98%]
tests/unit/test_tech_detector.py::TestTechDetector::test_multiple_extensions_same_file PASSED [ 98%]
tests/unit/test_tech_detector.py::TestTechDetector::test_all_languages_sorted PASSED [ 99%]
tests/unit/test_tech_detector.py::TestTechDetector::test_frameworks_sorted PASSED [ 99%]
tests/unit/test_tech_detector.py::TestTechDetector::test_ruby_detection PASSED [ 99%]
tests/unit/test_tech_detector.py::TestTechDetector::test_csharp_detection PASSED [ 99%]
tests/unit/test_tech_detector.py::TestTechDetector::test_cpp_detection PASSED [100%]/Users/user/Documents/codepath/pathreview-ai301-fa26-s1/.venv/lib/python3.13/site-packages/_pytest/unraisableexception.py:33: RuntimeWarning: coroutine 'AsyncMockMixin._execute_mock_call' was never awaited
  gc.collect()
RuntimeWarning: Enable tracemalloc to get the object allocation traceback


=============================== warnings summary ===============================
core/config.py:7
  /Users/user/Documents/codepath/pathreview-ai301-fa26-s1/core/config.py:7: PydanticDeprecatedSince20: Support for class-based `config` is deprecated, use ConfigDict instead. Deprecated in Pydantic V2.0 to be removed in V3.0. See Pydantic V2 Migration Guide at https://errors.pydantic.dev/2.13/migration/
    class Settings(BaseSettings):

-- Docs: https://docs.pytest.org/en/stable/how-to/capture-warnings.html
================== 380 passed, 52 xfailed, 1 warning in 8.12s ==================

Exit status: 0
```

### make typecheck

```text
$ make typecheck
.venv/bin/mypy api/ core/ ingestion/ rag/ agent/ safety/
.venv/lib/python3.13/site-packages/numpy/__init__.pyi:737: error: Type statement is only supported in Python 3.12 and greater  [syntax]
Found 1 error in 1 file (errors prevented further checking)
make: *** [typecheck] Error 2

Exit status: 2
```

### .venv/bin/black --check .

```text
$ .venv/bin/black --check .
would reformat /Users/user/Documents/codepath/pathreview-ai301-fa26-s1/unit3-evidence/repro.py

Oh no! 💥 💔 💥
1 file would be reformatted, 110 files would be left unchanged.

Exit status: 1
```

### make typecheck

```text
$ make typecheck
.venv/bin/mypy api/ core/ ingestion/ rag/ agent/ safety/
Success: no issues found in 76 source files

Exit status: 0
```

### git ls-files '*.py' | xargs .venv/bin/black --check

```text
$ git ls-files '*.py' | xargs .venv/bin/black --check
All done! ✨ 🍰 ✨
110 files would be left unchanged.

Exit status: 0
```

### npm test -- --run

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
