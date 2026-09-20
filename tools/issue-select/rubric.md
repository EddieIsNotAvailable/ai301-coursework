## Checks

| Check | Evidence | Pass condition | Weight |
|---|---|---|---|
| Maintainer Alive | the last 5 default-branch commits | At least 3 of said commits are < 1yr old, or at least 1 issue has a maintainer response within 180 days. Fails with neither condition met. | required |
| Repo Alive | archived status, last push to any branch, and latest release | Fails when the latest push/release is > 1yr old or the repo is archived. | required |
| Bounded Scope | Issue body, labels, and comments | Has a practical end goal. Do not reject for open implementation decisions. Reject for incomplete issue submission steps, undocumented third-party behavior | required |
| Available work | Repo assignees and linked PR states; Comments section for current work or claim statements | Pass if there is no open linked pull request and no indication that a contributor is currently working on the issue within 30 days | required |
| Contribution policy | contribution policy and any AI policy files or templates | Pass if the repository has no stated AI restriction or permits AI-assisted contributions with conditions; fail only for an outright ban on AI-generated code or documentation | required |

## Verdict rule

Accept if every required check passes. Preferred checks never change the
verdict; they rank issues that are accepted. Treat `unclear` as fail for
required checks and as not passed for preferred checks.
