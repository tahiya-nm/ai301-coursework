# Unit 4 — Test and Submit

Path: `beat-1-sandbox/unit-4/pull-request.md`

Record of your pull request, the branch it was opened from, and the evaluation runs that
produced `eval-run.txt`. This file is graded at the path above; a copy kept anywhere else
in the repository is not read.

Complete every labelled field below. Each is graded on its own; content placed under the wrong
label is not graded.

---

## Your identity upstream

**GitHub username**

tahiya-nm

---

## Pull request

**Pull request link**

https://github.com/codepath/pathreview-ai301-fa26-s1/pull/201

**Branch**

fix/67-profile-ownership-check

**Pull request description**

## Summary

Fixes #67. `create_review()` in `core/services/review_service.py` received `user_id` as a parameter but never used it, allowing an authenticated user to create a review for any profile by sending `POST /reviews` with another user's `profile_id`.

## Changes

- Added a profile ownership check in `create_review()`: queries the `Profile` table by `profile_id`, verifies `profile.user_id == user_id`, and raises `HTTPException(403)` if they don't match.
- Only `core/services/review_service.py` was changed. No route-layer, schema, or migration changes.

## Test evidence

**Before (commit f89c06fc):** `POST /reviews` as User 2 with User 1's profile_id → `200 OK`, review created with wrong profile.

**After (branch fix/67-profile-ownership-check):** Same request → `{"detail": "Not authorized to create a review for this profile"}` (403). User 1 creating a review for their own profile → `200 OK`, review created normally.

## AI-use disclosure

AI tools (Claude) were used to assist with drafting this change.

## Limitations

None known. The fix follows the same ownership-check pattern already used by `get_review()` and `list_reviews()` in the same file.

---

## Evidence

Before (from Unit 2 reproduction on commit `f89c06fc3ff292df2a04a39ac51319d32a76b779`):

```
$ curl -s -X POST http://localhost:8000/reviews \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer $USER2_TOKEN" \
  -d '{"profile_id":"3643aa64-fba5-42bb-bd34-6b2ecda13be1"}' | python3 -m json.tool
{
    "id": "c80a34f4-34c1-46d8-ad1f-ba7f6f1455bf",
    "profile_id": "3643aa64-fba5-42bb-bd34-6b2ecda13be1",
    "status": "pending",
    "sections": null,
    "overall_score": null,
    "error_message": null,
    "created_at": "2026-09-28T12:33:03.954770Z",
    "updated_at": "2026-09-28T12:33:03.954775Z"
}
```

User 2 created a review for User 1's profile. Status: 200 OK.

After (on branch `fix/67-profile-ownership-check`):

```
$ curl -s -X POST http://localhost:8000/reviews \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer $USER2_TOKEN" \
  -d '{"profile_id":"3643aa64-fba5-42bb-bd34-6b2ecda13be1"}' | python3 -m json.tool
{
    "detail": "Not authorized to create a review for this profile"
}

$ curl -s -X POST http://localhost:8000/reviews \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer $USER1_TOKEN" \
  -d '{"profile_id":"3643aa64-fba5-42bb-bd34-6b2ecda13be1"}' | python3 -m json.tool
{
    "id": "5d49d5f9-970b-4d6e-8bcc-5cd2ff0db622",
    "profile_id": "3643aa64-fba5-42bb-bd34-6b2ecda13be1",
    "status": "pending",
    "sections": null,
    "overall_score": null,
    "error_message": null,
    "created_at": "2026-10-07T09:34:40.299715Z",
    "updated_at": "2026-10-07T09:34:40.299719Z"
}
```

User 2 is now rejected with 403. User 1 can still create a review for their own profile with 200 OK. Fix confirmed.

---

## Eval iterations

**Run history**

1. Smoke test (`--limit 3`): 3/3 agreement.
2. Full run (20 scored packages): 20/20 agreement. Categories: clear-accept 7/7, not-tested 4/4, silent-drift 4/4, standards-wall 2/2, unreviewable 3/3. Bar: 20/20 PASS.
3. Saved final run with `--save-run eval-run.txt`: 20/20 agreement. Same result as run 2.

**Package analysis**

pkg-01: My rubric decided reject; the gold label is reject. The rubric's `standards-followed` check failed this package because the PR description did not include an AI-use disclosure despite the repo-facts block indicating a stated AI-use disclosure policy. The `standards-followed` pass condition requires: "If the repo requires AI-use disclosure and the PR does not include one, it fails." The repo-facts block for pkg-01 explicitly names a disclosure requirement, and the candidate PR's description contains no disclosure statement, so the check correctly graded as fail. This is a straightforward standards-wall case — the repo states an ask, the PR ignores it — and it demonstrates why the `standards-followed` check needs to read the repo-facts block's policy section before grading the PR description. The gold label agrees: reject.

**Check rationale**

The `diff-matches-plan` check, as currently written in my uploaded rubric.md:

> "Every file the diff touches falls inside the plan's stated scope or is covered by a deviation note. If the diff changes a file or adds a behavior the plan does not name and no deviation note explains the addition, it fails (silent addition). If the plan names a change the diff does not contain and no deviation note explains the omission, it fails (silent omission). An honest deviation note that explains why the diff differs from the plan re-ties the mismatch and passes."

This check exists because the lecture's first failure family for PRs is "silent drift" — the diff silently does more or less than the posted plan. The check works bidirectionally: it catches both silent additions (extra changes not in the plan) and silent omissions (planned changes missing from the diff). The deviation-note clause is critical because it implements the honest-outcome rule: a PR that honestly records why it deviated from the plan passes, while one that silently drifts fails. This mirrors Unit 2's honest-cannot-reproduce and Unit 3's recorded-deviation — the pattern across the sandbox is that honest disclosure earns full credit while silent inconsistency does not.

**Trade-offs**

The `diff-matches-plan` check cannot catch a deviation note that is technically present but dishonest — a note that says "added logging for debugging" when the diff actually adds a backdoor would pass the check as written, because the check verifies that a note exists, not that the note is truthful. In practice, the eval set's silent-drift packages test cruder mismatches (changes with no note at all, or descriptions that claim fidelity the diff contradicts), so this gap didn't cost any agreements. I accept this limitation because verifying the truthfulness of a deviation note requires understanding the code's semantics, which is too subjective for a binary check. The check correctly handled all 4 silent-drift packages and all 7 clear-accept packages in the eval.

---

Related paths: `eval-run.txt` in this directory; your skill's files in `tools/pr-precheck/`.
