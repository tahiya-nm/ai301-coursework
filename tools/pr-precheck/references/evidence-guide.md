# Evidence guide: where evidence lives in a PR package

## Plan fidelity (harness category: silent-drift)

- **Where it lives:** In an eval bundle, the plan-context block's scope pair (in-scope statement and not-in-scope line), the plan's approach and file list, and the `## Deviations` section. The candidate PR's diff shows what actually changed. The PR description states what the author claims changed. In live mode, `plan.md` in the working directory (including its Deviations section), the branch's diff (`git diff main...HEAD`), and the draft PR description.
- **What good looks like:** Every file the diff touches falls inside the plan's stated scope or is accounted for by a deviation note. The description's claims about the change match what the diff actually does. A deviation note that says "I also had to update X because Y" re-ties a mismatch honestly. Silent drift looks like a diff that adds a file the plan never mentioned (silent addition), or a plan that promised a test the diff doesn't contain with no deviation note explaining why (silent omission). A description claiming "only changes review_service.py" when the diff also touches routes.py is description drift.

## Test evidence (harness category: not-tested)

- **Where it lives:** In an eval bundle, the candidate PR's test-evidence section (before/after outputs), read against the plan-context block's test plan and the reproduction evidence's observed behavior. Also any CI or suite output. In live mode, the before/after output the user captured (from re-running the repro steps), and the output of running the repo's test suite (`pytest`, `make test`, etc.) against the branch.
- **What good looks like:** The before output matches the reproduction's observed behavior (the bug present — e.g., `200 OK` with wrong profile). The after output shows the fix (the bug absent — e.g., `403 Forbidden`). The expected-after matches what the plan's test plan predicted. The repo's own checks were run and their outcome is visible (pass, or fail with explanation). Decisive evidence names the observable behavior; "tests pass" without showing what changed is not decisive. An honest "cannot reproduce after fix" with the passing output shown is decisive.

## Diff quality (harness category: unreviewable)

- **Where it lives:** In an eval bundle, the candidate PR's diff (unified diff hunks) and commit list. In live mode, the output of `git diff main...HEAD` and `git log main..HEAD --oneline`.
- **What good looks like:** The diff contains only the fix and its directly related changes (a new test, an updated import). A reviewer can see the actual fix without reading through unrelated noise. Debris tells: `print()` or `console.log()` debug statements left in, blocks of code commented out instead of deleted, formatting-only changes to files the fix doesn't touch, drive-by variable renames, unrelated refactors that rode along. A clean diff has none of these. A commit message that describes the change (not "WIP" or "fix") is preferred but not required for the diff itself to be reviewable.

## Standards and comms (harness category: standards-wall)

- **Where it lives:** In an eval bundle, the repo-facts block's PR template sections (what the repo asks PRs to include), contributing instructions, and stated AI-use disclosure policy. The candidate PR's description and title are read against these asks. In live mode, the repo's `.github/PULL_REQUEST_TEMPLATE.md`, `CONTRIBUTING.md`, and any AI-use policy, read against the draft PR description.
- **What good looks like:** The PR fills in the repo's template sections with real content specific to this change (not placeholder text like "describe your changes here" left untouched). If the repo requires AI-use disclosure, the PR includes a disclosure statement (e.g., "AI tools were used to assist with this change"). If the contributing guide asks for a linked issue, the description links or references the issue. If a maintainer gave explicit direction in the thread, the PR acknowledges it. Boilerplate that could apply to any PR in any repo fails. Silence in the repo's policy (no template, no disclosure rule) counts as pass.
