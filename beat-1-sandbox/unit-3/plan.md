# Plan: Fix profile ownership check in review creation (#67)

## Diagnosis

The `POST /reviews` endpoint passes the authenticated user's `user_id` to `create_review()` in `core/services/review_service.py`, but `create_review()` ignores that argument. It creates a `Review` from the supplied `profile_id` without verifying that the profile belongs to the authenticated user.

This is confirmed by the reproduction: authenticating as User 2 (`f7ce7540-ff97-449d-811e-6056f2d6d9da`) and sending `POST /reviews` with User 1's profile ID (`3643aa64-fba5-42bb-bd34-6b2ecda13be1`) returned `200 OK` and created review `c80a34f4-34c1-46d8-ad1f-ba7f6f1455bf` with `profile_id` set to User 1's profile. The repro was run on commit `f89c06fc3ff292df2a04a39ac51319d32a76b779`.

The root cause is that `create_review()` receives `user_id` as a parameter but never queries the `Profile` table to confirm `profile.user_id == user_id`. By contrast, `get_review()` and `list_reviews()` in the same file already scope queries through `Profile.user_id`, so the pattern to follow exists in the codebase.

## Scope

**In scope:**
- `core/services/review_service.py`: modify `create_review()` to verify that the profile identified by `profile_id` belongs to the user identified by `user_id` before creating the review.
- `tests/unit/test_review_service.py`: add a test that confirms a `PermissionError` (or appropriate HTTP 403 equivalent) is raised when `user_id` does not match the profile's owner.

**Not in scope:**
- Changes to `get_review()`, `list_reviews()`, or `process_review()` — these already scope correctly.
- Changes to the API route layer (`api/routes/`) — the route already passes `user_id`; the bug is in the service.
- Refactoring, logging additions, or any other cleanup.

## Approach

1. In `core/services/review_service.py`, in `create_review()`, before the `Review(...)` constructor:
   - Query the `Profile` table: `SELECT * FROM profiles WHERE id = profile_id`.
   - If no profile is found, raise an appropriate error (the profile doesn't exist).
   - If `profile.user_id != user_id`, raise `PermissionError` or a domain-specific exception that the route layer maps to HTTP 403.
2. In `tests/unit/test_review_service.py`:
   - Add a test `test_create_review_rejects_wrong_user` that calls `create_review()` with a `profile_id` belonging to a different `user_id` and asserts the error is raised.
   - Add a test `test_create_review_accepts_own_profile` that confirms the happy path still works (own profile, own user_id).

## Test plan

Re-run the reproduction steps from commit `f89c06fc3ff292df2a04a39ac51319d32a76b779`:

**Before (from Unit 2 repro):**
1. Log in as User 2.
2. Send `POST /reviews` with `{"profile_id": "3643aa64-fba5-42bb-bd34-6b2ecda13be1"}` (User 1's profile).
3. Expected (current broken behavior): `200 OK`, review created with wrong profile.

**After (expected with fix):**
1. Log in as User 2.
2. Send `POST /reviews` with `{"profile_id": "3643aa64-fba5-42bb-bd34-6b2ecda13be1"}` (User 1's profile).
3. Expected: `403 Forbidden` (or equivalent), no review created.
4. Log in as User 1.
5. Send `POST /reviews` with `{"profile_id": "3643aa64-fba5-42bb-bd34-6b2ecda13be1"}` (User 1's own profile).
6. Expected: `200 OK`, review created normally.

Also run: `pytest tests/unit/test_review_service.py -q` to confirm the new tests pass and existing tests remain green.

## Risks and unknowns

- I have not confirmed how the route layer maps service-level exceptions to HTTP status codes. If the route catches `PermissionError` and returns 403, the plan works as stated. If not, I may need to raise an `HTTPException(status_code=403)` directly, or add a catch in the route. I'll check the route file before coding.
- The existing tests in `test_review_service.py` use mocked database sessions. My new tests will follow the same mock pattern, but I haven't verified the exact mock setup yet. If the async mock pattern from issue #65 applies here, I'll adapt.

## Deviations

The fix matched the plan. The route layer already catches `HTTPException` and re-raises it (line 54 of `api/routes/reviews.py`), so raising `HTTPException(status_code=403)` directly in `create_review()` worked without any route-layer changes. I used `HTTPException` instead of `PermissionError` because the route's existing exception handling pattern re-raises `HTTPException` directly and wraps everything else in a generic 500. No other deviations from the plan.
