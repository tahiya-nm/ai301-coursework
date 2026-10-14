# Procedure: how this tool grades a PR package

## Read order

1. Read the plan-context block first: the plan's scope pair (in-scope and not-in-scope), the approach, the test plan, and the deviation notes. List every file the plan names and the boundary it draws. This comes first so you know what the diff should contain before you see it.
2. Read the reproduction evidence second: the repro's observed behavior, error messages, and artifacts. This grounds the test evidence comparison that comes later.
3. Read the candidate PR's diff third. List every file it touches and every behavior it adds, removes, or changes. Compare this list against the plan's file list from step 1. Note any file that appears in one list but not the other.
4. Read the PR description and title fourth. Note what the description claims about the change, the files, and the scope. Note any limitations or shortfalls it discloses.
5. Read the test-evidence section fifth. Note the before/after outputs and whether they map onto the repro evidence from step 2 and the plan's test plan from step 1.
6. Read the repo-facts block last: the PR template sections, contributing instructions, and AI-use disclosure policy. Note what the repo asks for and whether the PR provides it.

The order matters because plan-before-diff prevents anchoring on what the diff does before knowing what it should do, and repro-before-evidence lets you judge whether the test evidence actually shows the bug fixed.

## Evidence gathering

1. **For diff-matches-plan:** From step 1, the plan's in-scope files and boundary. From step 3, the diff's changed files. Pair them: plan file → diff file. Record any file in the diff not in the plan (silent addition) and any file in the plan not in the diff (silent omission). Check deviation notes for explanations of mismatches.
2. **For description-matches-diff:** From step 4, the description's claims about what changed. From step 3, the diff's actual changes. Pair them: each description claim → the diff evidence that supports it. Record any claim without diff support and any diff change without a description mention.
3. **For test-evidence-decisive:** From step 5, the before/after outputs. From step 1, the plan's test plan (what to observe). From step 2, the repro's observed behavior. Check whether the before matches the repro and the after shows the fix.
4. **For repo-checks-run:** From step 5, any mention of running the repo's test suite, linter, or CI. Record whether the outcome is visible.
5. **For diff-reviewable:** From step 3, scan the diff for debug prints, commented-out code, formatting-only hunks, or unrelated changes. Record any debris found.
6. **For standards-followed:** From step 6, the repo's template asks and disclosure policy. From step 4, the PR description. Check whether each ask is met. Check whether AI-use disclosure is present if required.
7. **For shortfall-disclosed:** From step 4, any limitations or known-issues statement. From step 1, the plan's risks. From step 3, any gap between plan and diff. Check whether known gaps are disclosed.

## Check execution

1. Execute checks in this order: diff-matches-plan, description-matches-diff, test-evidence-decisive, repo-checks-run, diff-reviewable, standards-followed, shortfall-disclosed. Required checks first, preferred last.
2. For each check, compare the gathered evidence pairs against the pass condition in rubric.md. Grade as pass, fail, or unclear.
3. If evidence for a check is genuinely absent (e.g., no test-evidence section exists), grade that check as fail — absence of evidence is not a pass.
4. A check may be graded without re-reading the whole package if the gathered evidence from the read-order phase already contains everything the check needs.
5. Record a one-sentence reason for each grade, quoting the specific evidence that decided it.

## Verdict assembly

1. Apply the verdict rule from rubric.md: accept if every required check passes; reject if any required check fails; unclear on any required check counts as fail.
2. Preferred checks (shortfall-disclosed) never change the verdict. They appear in the output for ranking only.
3. When more than one required check failed, quote the first failing required check in rubric order as the deciding check in the output.
4. Two executors given the same package, the same rubric, and this procedure must reach the same verdict. If a check's grade depends on interpretation not covered by the pass condition, grade it as unclear (which counts as fail).
