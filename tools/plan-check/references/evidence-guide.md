# Evidence guide: where evidence lives in a plan package

## Diagnosis and grounding

- **Where it lives:** In an eval bundle, the plan's root-cause or diagnosis section states the cause. The repro-evidence block contains the observed behavior (error messages, status codes, output excerpts, database states) that the diagnosis must explain. In live mode, the plan's diagnosis is in the draft `plan.md`, and the repro evidence is in the student's posted repro comment on the issue thread.
- **What good looks like:** The stated cause cites behavior the repro evidence actually shows. A diagnosis that says "the function doesn't check ownership" is grounded when the repro evidence shows a `200 OK` for a cross-user request. A diagnosis that names a different function or a different error than what the repro evidence demonstrated is not grounded, even if it sounds plausible.

## Scope

- **Where it lives:** In an eval bundle, the plan's scope or in-scope/not-in-scope section, and the files or areas listed in the approach. In live mode, the scope section of the draft `plan.md`.
- **What good looks like:** The plan names the specific files it will change (e.g., `core/services/review_service.py` and `tests/unit/test_review_service.py`) and states what it will not touch. One bounded change looks like "add an ownership check to `create_review()` and update its unit tests." A drive-by rewrite looks like "refactor the entire service layer and add logging while we're at it." If the plan names files or changes unrelated to the diagnosis, scope is blown.

## Executability

- **Where it lives:** In an eval bundle, the plan's approach or implementation section, including file names, function names, and order of work. In live mode, the approach section of the draft `plan.md`.
- **What good looks like:** A stranger with the repo cloned can read the plan and start coding without asking "but where?" or "but how?" The plan names the file, the function, the kind of edit (add a guard clause, modify a query, add a test case), and if multiple files are involved, the order to work in. "Fix the bug in the service" is not executable. "In `create_review()` in `core/services/review_service.py`, add a query that fetches the profile by `profile_id` and checks `profile.user_id == user_id` before creating the review; raise `403` if they don't match" is executable.

## Test plan

- **Where it lives:** In an eval bundle, the plan's test plan or verification section, read against the repro-evidence block's steps and observed output. In live mode, the test plan section of the draft `plan.md`, read against the posted repro comment.
- **What good looks like:** The test plan re-runs the repro steps (or a meaningful subset) and names the expected output after the fix. If the repro showed `200 OK` with a wrong profile, the test plan says "re-run the same `POST /reviews` as User 2 with User 1's profile_id; expect `403 Forbidden` instead of `200 OK`." A test plan that says "run pytest and make sure it passes" without naming what the test checks is not decisive.

## Honesty

- **Where it lives:** In an eval bundle, the plan's risks, unknowns, or deviations section. Also scan the approach for any claims of certainty. In live mode, the risks section of the draft `plan.md` and the `## Deviations` heading.
- **What good looks like:** Uncertain things are marked as uncertain. "I believe the root cause is X because the repro shows Y, but I haven't confirmed whether Z also contributes" is honest. "This fix will definitely solve the problem with no side effects" is false confidence unless the change is trivially scoped. A plan with no risks section is acceptable only for trivial changes (one-line typo, a single guard clause where the test already exists). Deviations filled in after the build — even "nothing changed" in the author's own words — show honest tracking.

## Comms

- **Where it lives:** In an eval bundle, the plan comment text, read against the thread highlights and the repo-facts block (stated templates, contribution policy, AI-use disclosure requirements). In live mode, the draft `comment.md`, read against the issue thread on GitHub and the repo's `CONTRIBUTING.md`.
- **What good looks like:** The comment addresses this specific issue's behavior (not a generic "I plan to fix this bug"). It references or builds on relevant thread context — if a maintainer said "the fix should go in the service layer, not the route," the comment acknowledges that. It follows repo conventions: if the repo requires AI-use disclosure, the comment discloses. If the repo has a plan comment template, the comment uses it. A comment that could be copy-pasted onto any issue without changing a word is boilerplate and fails.
