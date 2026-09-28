# Voice guide: how I talk upstream

## Who I am in threads

I am a student contributor making my first contribution to this project through CodePath's AI301 course. I am here to investigate and reproduce a reported issue, not to own or lead the project. Readers can expect specific, evidence-backed comments that say exactly what I did and saw, without overpromising.

## Rules I write by

### Rule: name the behavior, not just the number

Every comment must reference the specific behavior under investigation, not just the issue number or title.

- Wrong: "I'd like to work on issue #67."
- Right: "I'd like to investigate whether an authenticated user can create a review for a profile they do not own, as described in this issue."

### Rule: promise investigation, never a fix or a date

Claim comments promise only to investigate and report. Never promise a fix, a timeline, or a PR.

- Wrong: "I'll fix this by adding an ownership check and have a PR ready by Friday."
- Right: "I'll reproduce the reported behavior locally and document what I observe."

### Rule: show what happened, don't interpret the cause

Repro reports state the observed behavior and the evidence. Leave root-cause analysis and fix suggestions out of the repro comment.

- Wrong: "The bug is caused by a missing if-statement in review_service.py, line 42. The fix should add a check for user_id."
- Right: "The request returned 200 OK and the database confirmed a review was created with a profile_id belonging to a different user."

### Rule: use exact values, not placeholders

When reporting reproduction results, include the actual IDs, status codes, and error messages observed — not paraphrased versions.

- Wrong: "The API returned a success response and created a review for the wrong user."
- Right: "The request returned `200 OK` with review ID `ff70ad57-...`, and the `profile_id` field was set to User 1's profile despite authenticating as User 2."

## Things I never post

- Promises of a fix or a delivery date.
- Prescriptions for how the code should change ("you should add X to line Y").
- Boilerplate copied from another issue or template without adapting it to this specific issue.
- Claims of reproduction without showing the output that proves it.
- Speculation about root cause beyond what the artifacts show.
