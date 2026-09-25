# Unit 2 — Claim and Reproduce

Path: `beat-1-sandbox/unit-2/reproduction.md`

Record of your claim and reproduction on the issue you chose in Unit 1, and of the
evaluation runs that produced `eval-run.txt`. This file is graded at the path above; a copy
kept anywhere else in the repository is not read.

Complete every labelled field below. Each is graded on its own; content placed under the wrong
label is not graded.

---

## Your identity upstream

**GitHub username**

mikeng07

---

## Posted upstream

**Claim comment**

https://github.com/codepath/pathreview-ai301-fa26-s1/issues/72#issuecomment-5785826794
Hi, I'd like to take on this as a first contribution. `verify_password` currently lets `passlib.exc.UnknownHashError` escape when the stored hash isn't a recognizable bcrypt hash, instead of failing closed and returning `False` — I'll reproduce this locally and report back with the details, then follow up with a fix that returns `False` and removes the `xfail` marker on `test_verify_with_wrong_hash_format` (manifest H-05).


**Reproduction comment**

https://github.com/codepath/pathreview-ai301-fa26-s1/issues/72#issuecomment-5785839790
**Environment:** macOS 14.7.7 (arm64), Python 3.13.5, repo at commit `f89c06f`. This bug lives entirely in `core/security.py`, and `Settings()` (`core/config.py`) has a default for every field, so no `.env`/Docker stack is needed to trigger it — just a venv with the relevant deps:

```
$ python3 -m venv .venv && source .venv/bin/activate
$ pip install "passlib[bcrypt]>=1.7.4" "bcrypt>=4.0.1,<5.0.0" "python-jose[cryptography]>=3.3.0" "pydantic[email]>=2.5.0" "pydantic-settings>=2.1.0"
```
Installed: passlib 1.7.4, bcrypt 4.3.0.

**Steps and observed:**

Control run (a valid bcrypt hash, to confirm `verify_password` works normally first):
```
$ python3 -c "
from core.security import verify_password, hash_password
h = hash_password('password')
print('control (valid hash):', verify_password('password', h))
"
control (valid hash): True
```

The reported trigger — the exact string the repo's own covering test uses (`wrong_hash = "not_a_valid_bcrypt_hash"` in `tests/unit/test_security.py`):
```
$ python3 -c "
from core.security import verify_password
verify_password('password', 'not_a_valid_bcrypt_hash')
"
Traceback (most recent call last):
  ...
passlib.exc.UnknownHashError: hash could not be identified
```

Also ran the repo's own covering test directly:
```
$ python3 -m pytest tests/unit/test_security.py -k test_verify_with_wrong_hash_format -v
tests/unit/test_security.py::TestSecurity::test_verify_with_wrong_hash_format XFAIL
```
`XFAIL` matches the `@pytest.mark.xfail(strict=True, reason="issue #72 (manifest H-05): ...")` marker already on that test — the bug is present exactly as the marker describes it.

**Expected:** `verify_password` returns `False` for a hash it cannot identify (fail closed), the same way it returns `False` for a merely wrong password.
**Actual:** `passlib.exc.UnknownHashError` propagates out of `verify_password` uncaught, as shown above.

One unrelated note for honesty: passlib 1.7.4 against bcrypt 4.3.0 also prints a harmless `(trapped) error reading bcrypt version` warning on every call (a known passlib/bcrypt version-detection quirk). It shows up in both the control and bug runs and has nothing to do with this issue — flagging it so it isn't mistaken for evidence of anything else.



## Eval iterations

Answer all four sections. Quote source text directly; paraphrase does not satisfy these
fields.

**Run history**

agreement: 16/20 scored items  (bar: 18/20: below the bar)
agreement: 19/20 scored items  (bar: 18/20: PASS)

**Package analysis**

I analyzed `pkg-01`. My rubric decided `reject`, while the gold label
was `accept`.

The package is a faithful reproduction of HTTPie's missing
`Content-Type: application/json` behavior. It records HTTPie, Python,
multidict, macOS, and architecture details; gives an offline command
that a stranger can run; shows the missing header; and includes a
control run where the header is present. The rubric rejected it because
the `repo-conventions` check was interpreted too strictly even though
the package's repo facts state that httpie/cli has no AI policy and does
not require disclosure. This was the one remaining disagreement in the
confirming run.

**Check rationale**

```md
| repo-conventions | repo-facts block and claim/repro comments | pass when the comments follow the stated reporting policy. Apply an AI-disclosure failure only if the repo-facts text explicitly requires the issue or comment to name AI use and the candidate omits it; human-authorship, human-voice, or human-review guidance without that explicit disclosure requirement must pass | required |
```

I chose this check because the eval set includes a disclosure-wall
package. The condition makes disclosure mandatory only when the repo
explicitly requires it, while avoiding an invented disclosure rule for
repositories that have no AI policy or only ask for human review. This
distinction correctly handles the explicit disclosure case while
preserving the ordinary communication checks elsewhere.

**Trade-offs**

I loosened the environment and steps checks to accept documented
differences from the issue environment and honest cannot-reproduce
attempts when their commands and results are concrete. I reran
`pkg-09` as a canary for the cannot-reproduce change and `pkg-11` as a
canary for documented environment differences. I also reran `pkg-03`
and `pkg-20` after revising `repo-conventions`: `pkg-03` covers a
human-authorship policy without a disclosure requirement, while
`pkg-20` covers an explicit disclosure requirement. The trade-off is
that a careful failed attempt can be accepted even when it does not
reach the original trigger, but unsupported claims, wrong-target
artifacts, missing environments, and missing mandatory disclosures
remain rejects.

---

Related paths: `eval-run.txt` in this directory; your skill's files in
`tools/repro-check/`.
