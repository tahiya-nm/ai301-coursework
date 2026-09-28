# Rubric: is this reproduction package ready to post?

## Checks

| Check | Evidence | Pass condition | Weight |
|---|---|---|---|
| environment-recorded | The report records enough environment detail that a stranger could set up the same context. At minimum it names the runtime or language version and either a commit SHA, branch, tag, or version of the software under test. Naming the OS is expected but its absence alone does not fail the check if the reproduction is containerized or the issue is OS-independent. Missing both the runtime version and the commit/branch fails this check. | required |
| steps-followable | The repro report's steps section, read from starting state to trigger. | A stranger with the environment above could execute the steps in order, starting from a clean clone, and reach the trigger without inventing any unstated step. Each step names the concrete action (command, API call, UI interaction) rather than summarizing it. If a step says "set up the project" without naming the commands, it fails. | required |
| behavior-matches-issue | The output excerpt, error message, log snippet, or screenshot in the repro report, read against the specific behavior the issue describes. | The artifact shows the same error, crash, or incorrect behavior the issue names — not an adjacent or different failure. If the issue says "raises UnknownHashError" and the report shows a different exception, it fails. If the report honestly states cannot-reproduce and shows the passing output as evidence, it passes. | required |
| outcome-honest | The repro report's conclusion, read against the artifacts it presents. | The report's conclusion matches what the artifacts actually show. A report claiming reproduction must include output that shows the bug. A report claiming cannot-reproduce must include output that shows the expected (correct) behavior. A conclusion that claims more than the artifacts demonstrate fails. | required |
| claim-names-issue | The claim comment, read against the issue title and description. | The claim comment identifies the specific issue (by title, number, or described behavior) and states what the author will investigate. It promises investigation only, not a fix or a timeline. A generic "I'd like to work on this" without naming the specific behavior fails. | required |
| conventions-respected | The claim and repro comments, read against the repo's CONTRIBUTING.md and any stated templates or AI-use disclosure requirements in the repo-facts block. | The comments follow any stated contribution policy. If the repo requires AI-use disclosure and the comment does not disclose, it fails. If the repo has a comment template and the comment ignores it entirely, it fails. Silence in the repo's policy (no template, no AI rule) counts as pass. | required |
| pinned-commit | The repro report's environment section. | The report names a specific commit SHA, tag, or branch so the reproduction is pinned to a verifiable snapshot of the code. "Latest main" without a SHA fails. | preferred |

## Verdict rule

Accept if every required check passes. Reject if any required check fails. Unclear on any required check counts as fail. Preferred checks never change the verdict; they rank accepted packages.
