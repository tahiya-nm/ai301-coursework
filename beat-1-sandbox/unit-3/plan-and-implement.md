# Unit 3 — Plan and Build

Path: `beat-1-sandbox/unit-3/plan-and-implement.md`

Record of your plan, the branch you built it on, and the evaluation runs that produced
`eval-run.txt`. This file is graded at the path above; a copy kept anywhere else in the
repository is not read.

Complete every labelled field below. Each is graded on its own; content placed under the wrong
label is not graded.

---

## Posted upstream

**GitHub username**

tahiya-nm

**Plan comment**

https://github.com/codepath/pathreview-ai301-fa26-s1/issues/67#issuecomment-6031027164

I've diagnosed the profile ownership gap in `create_review()` and have a plan to fix it.

**Root cause:** `create_review()` in `core/services/review_service.py` receives `user_id` as a parameter but never uses it. It creates a `Review` from the supplied `profile_id` without checking that the profile belongs to the authenticated user. My reproduction on commit `f89c06fc3ff292df2a04a39ac51319d32a76b779` confirmed this: authenticating as User 2 and sending `POST /reviews` with User 1's profile ID returned `200 OK` and persisted a review under the wrong profile.

**Plan:**
1. In `create_review()`, before constructing the `Review`, query the `Profile` table by `profile_id` and verify `profile.user_id == user_id`. If they don't match, raise an error that the route layer maps to `403 Forbidden`.
2. Add unit tests: one confirming the rejection (wrong user → error), one confirming the happy path (own user → review created).

**Scope:** Only `core/services/review_service.py` and `tests/unit/test_review_service.py`. The pattern already exists in `get_review()` and `list_reviews()`, which scope through `Profile.user_id`.

**Test plan:** Re-run my Unit 2 repro steps after the fix — the same `POST /reviews` as User 2 with User 1's profile ID should return `403` instead of `200 OK`.

**Risks:** I haven't confirmed how the route layer maps service exceptions to HTTP status codes; I'll check before coding and adapt if needed.

---

## Your branch

**Branch**

fix/67-profile-ownership-check

**Evidence**

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
$ echo "=== User 2 tries User 1's profile (should get 403) ==="
$ curl -s -X POST http://localhost:8000/reviews \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer $USER2_TOKEN" \
  -d '{"profile_id":"3643aa64-fba5-42bb-bd34-6b2ecda13be1"}' | python3 -m json.tool
{
    "detail": "Not authorized to create a review for this profile"
}

$ echo "=== User 1 uses own profile (should get 200) ==="
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

## Eval iterations

**Run history**

1. Smoke test (`--limit 3`): 3/3 agreement.
2. Full run (20 scored packages): 19/20 agreement. Disagreement on pkg-14 (gold: accept, rubric: reject, failed plan-executable). Categories: clear-accept 6/7, scope-creep 4/4, thread-convention 2/2, unbuildable 3/3, wrong-cause 4/4. Bar: 19/20 PASS.
3. Saved final run with `--save-run eval-run.txt`: 19/20 agreement. Same result as run 2.

**Package analysis**

pkg-14: My rubric decided reject; the gold label is accept. The rubric's `plan-executable` check failed this package because the plan's approach section did not name the specific edits with enough detail for a stranger to start coding without asking the author anything. The gold label accepts it because the plan is executable enough — it names the files, the function, and the kind of change, even if it doesn't spell out every line. My `plan-executable` pass condition requires that the plan "names the files to change, describes the concrete edits or additions (not just 'fix the bug'), and states an order of work if more than one file is involved." pkg-14's plan sits at the boundary — it names the file and the function but describes the edit at a slightly higher level than my check demands. This is one of the 4 arguable packages in the eval set, and the disagreement reflects my check's preference for more specificity than the gold label requires.

**Check rationale**

The `diagnosis-grounded` check, as currently written in my uploaded rubric.md:

> "The plan names a specific cause, and that cause explains the behavior the repro evidence actually shows. If the plan's stated cause contradicts what the repro evidence demonstrates (e.g., claims a missing import when the repro shows a logic error), or if the plan ignores the repro evidence entirely and diagnoses from the issue title alone, it fails. A plan that honestly says 'cause is uncertain' and explains what the repro evidence narrows it to passes."

This check exists because the lecture's first failure family is "the diagnosis ignores or contradicts the reproduced evidence." The eval set's `wrong-cause` category (4 packages) tests exactly this: plans that state a cause that doesn't match what the repro evidence shows. My check requires that the stated cause "explains the behavior the repro evidence actually shows" — not just that a cause is stated, but that it's grounded in what was observed. The "honest uncertainty" clause ("cause is uncertain" with evidence-based narrowing passes) prevents false rejections on plans that acknowledge ambiguity rather than guessing. This check correctly handled all 4 wrong-cause packages in the eval.

**Trade-offs**

The `diagnosis-grounded` check cannot catch a plan that correctly quotes the repro evidence but then draws a wrong conclusion from it — the check verifies that the diagnosis addresses the observed behavior, but not that the causal reasoning is logically valid. A plan could say "the repro shows a 200 OK on a cross-user request, so the cause must be a missing rate limiter" — the evidence is cited, but the conclusion is wrong. In practice, the eval set's wrong-cause packages use cruder mismatches (ignoring the evidence entirely or citing a different error), so this gap didn't cost any agreements. I accept this limitation because evaluating causal reasoning quality is too subjective for a binary check that two graders must agree on.

---

Related paths: `plan.md` and `eval-run.txt` in this directory; your skill's files in
`tools/plan-check/`.
