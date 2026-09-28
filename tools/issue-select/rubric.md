# Rubric: is this a good first issue?

## Checks

| Check | Evidence | Pass condition | Weight |
|---|---|---|---|
| commits-alive | The last 10 default-branch commit dates in the repo-facts block. | At least 2 commits on the default branch within the last 180 days. If the repo is archived, this check fails automatically. | required |
| responds-to-issues | Reply times in the last 5 issues or PRs from users with write, maintain, or admin role, in the repo-facts block and sampled threads. | At least 1 maintainer reply, review, or triage action (labeling, closing, assigning) within the last 90 days. | required |
| shipped-recently | The releases list or tags in the repo-facts block. | At least 1 release or version tag published within the last 365 days. If the project does not use releases (no release history at all), fall back to the commits-alive check and mark this check as pass if commits-alive passes. | required |
| scope-fits | The issue title, body, labels, and any linked discussion or PR. Cross-check against references/evidence-guide.md. | The issue describes one bounded change: a single bug with a stated trigger, a single well-scoped feature, or a single isolated task. The issue body includes enough specification to know when the fix is done. It does not require cross-cutting architectural changes, new external service dependencies, or database migrations. If the issue body is too vague to identify a concrete change, it fails. | required |
| unclaimed | The issue's assignee field, the full comment thread, and linked pull requests in the repo-facts block. | No current assignee, no open or draft PR referencing this issue, and no comment in the last 45 days expressing intent to work on it (e.g. "I'll take this", "working on it", "claiming"). A comment that only asks a question or requests clarification is not a claim. A closed, unmerged PR with no recent activity does not count as a claim. | required |
| ai-policy-ok | CONTRIBUTING.md, README.md, CODE_OF_CONDUCT.md, and any governance or policy files in the repo-facts block. | The repository has no explicit rule banning AI-assisted or AI-generated contributions. Silence (no mention of AI) counts as pass. A policy requiring disclosure of AI use counts as pass. Only an outright prohibition counts as fail. | required |
| good-first-label | The issue's labels in the repo-facts block. | The issue carries a label like `good first issue`, `good-first-issue`, `beginner`, `starter`, or `help wanted`. | preferred |

## Verdict rule

Accept if every required check passes. Reject if any required check fails. Unclear on any required check counts as fail. Preferred checks never change the verdict; they only rank accepted issues — an accepted issue with more preferred checks passing ranks higher.