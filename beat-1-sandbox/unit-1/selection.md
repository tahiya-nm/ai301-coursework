# Unit 1 — Issue Selection

Path: `beat-1-sandbox/unit-1/selection.md`

Record of the issue carried into Unit 2, and of the evaluation runs that produced `eval-run.txt`. This file is graded at the path above; a copy kept anywhere else in
the repository is not read.

Complete every labelled field below. Each is graded on its own; content placed under the
wrong label is not graded.

---

## Selected issue

**Issue link**

https://github.com/codepath/pathreview-ai301-fa26-s1/issues/67

**Verdict output**

```
Repo-level checks (same for all three issues; today = 2026-09-28):

- commits-alive: 10 commits all within 180 days; most recent 2026-09-16 by Aburke225. ✓ PASS
- responds-to-issues: Aburke225 (sole committer = maintainer) applied labels to issues on 2026-09-10 — a triage action 18 days ago. ✓ PASS
- shipped-recently: No releases or tags in the repo → fallback to commits-alive → PASS
- ai-policy-ok: docs/CONTRIBUTING.md exists; no mention of AI policy. Silence = PASS

---
Issue #67 — "Review creation does not verify profile ownership"
- scope-fits: One bounded bug in review_service.py — add ownership check to create_review(); trigger and fix location are specified. PASS
- unclaimed: No assignee; no open or draft PR found in timeline; ChinoUkaegbu's claim comments are overridden by the Path Review house rule. PASS
- good-first-label: Labels are bug and tier-2 — no good first issue or equivalent. FAIL (preferred only)
- VERDICT: ACCEPT

Issue #72 — "verify_password lets UnknownHashError escape"
- scope-fits: One bounded bug in core/security.py; relevant files and test marker named explicitly. PASS
- unclaimed: No assignee, but PR #75 ("fix(security): fail closed in verify_password on malformed stored hashes") is open and cross-references this issue. The house rule exempts claim comments only; open PRs still apply as written. FAIL
- good-first-label: good first issue label present. PASS (preferred — does not save verdict)
- VERDICT: REJECT (unclaimed fails)

Issue #68 — "KeywordSearcher.index() raises ZeroDivisionError on empty corpus"
- scope-fits: One bounded bug in rag/retriever/keyword_search.py; files, xfail marker, and expected behavior specified. PASS
- unclaimed: No assignee, but PR #78 ("fix: handle empty keyword indexes") is open and cross-references this issue. FAIL
- good-first-label: good first issue + tier-1 labels present. PASS (preferred — does not save verdict)
- VERDICT: REJECT (unclaimed fails)

---
Ranking of accepted issues: Only #67 is accepted.

[
  {
    "item": "https://github.com/codepath/pathreview-ai301-fa26-s1/issues/67",
    "checks": [
      {"name": "commits-alive", "grade": "pass", "evidence": "Most recent commit 2026-09-16 by Aburke225, 12 days ago; 10 commits all within 180 days."},
      {"name": "responds-to-issues", "grade": "pass", "evidence": "Aburke225 applied labels to issues on 2026-09-10 (18 days ago) — a triage action within 90 days."},
      {"name": "shipped-recently", "grade": "pass", "evidence": "No releases or tags; fallback to commits-alive, which passes."},
      {"name": "scope-fits", "grade": "pass", "evidence": "One bounded bug: add profile-ownership check to create_review() in review_service.py; trigger (POST /reviews with another user's profile_id) and fix location stated."},
      {"name": "unclaimed", "grade": "pass", "evidence": "No assignee; no open or draft PR found in issue timeline; ChinoUkaegbu claim comments exempted by Path Review house rule."},
      {"name": "ai-policy-ok", "grade": "pass", "evidence": "docs/CONTRIBUTING.md has no mention of AI — silence counts as pass."},
      {"name": "good-first-label", "grade": "fail", "evidence": "Labels are 'bug' and 'tier-2'; no 'good first issue' or equivalent label present."}
    ],
    "verdict": "accept"
  },
  {
    "item": "https://github.com/codepath/pathreview-ai301-fa26-s1/issues/72",
    "checks": [
      {"name": "commits-alive", "grade": "pass", "evidence": "Most recent commit 2026-09-16 by Aburke225, 12 days ago; 10 commits all within 180 days."},
      {"name": "responds-to-issues", "grade": "pass", "evidence": "Aburke225 applied labels to issues on 2026-09-10 (18 days ago) — a triage action within 90 days."},
      {"name": "shipped-recently", "grade": "pass", "evidence": "No releases or tags; fallback to commits-alive, which passes."},
      {"name": "scope-fits", "grade": "pass", "evidence": "One bounded bug: catch UnknownHashError in core/security.py and return False; relevant file and xfail test marker explicitly named."},
      {"name": "unclaimed", "grade": "fail", "evidence": "Open PR #75 ('fix(security): fail closed in verify_password on malformed stored hashes') cross-references this issue; house rule exempts claim comments only, not open PRs."},
      {"name": "ai-policy-ok", "grade": "pass", "evidence": "docs/CONTRIBUTING.md has no mention of AI — silence counts as pass."},
      {"name": "good-first-label", "grade": "pass", "evidence": "'good first issue' label is present on the issue."}
    ],
    "verdict": "reject"
  },
  {
    "item": "https://github.com/codepath/pathreview-ai301-fa26-s1/issues/68",
    "checks": [
      {"name": "commits-alive", "grade": "pass", "evidence": "Most recent commit 2026-09-16 by Aburke225, 12 days ago; 10 commits all within 180 days."},
      {"name": "responds-to-issues", "grade": "pass", "evidence": "Aburke225 applied labels to issues on 2026-09-10 (18 days ago) — a triage action within 90 days."},
      {"name": "shipped-recently", "grade": "pass", "evidence": "No releases or tags; fallback to commits-alive, which passes."},
      {"name": "scope-fits", "grade": "pass", "evidence": "One bounded bug: add empty-corpus guard in KeywordSearcher.index() in keyword_search.py; files and xfail marker explicitly named."},
      {"name": "unclaimed", "grade": "fail", "evidence": "Open PR #78 ('fix: handle empty keyword indexes') cross-references this issue; house rule exempts claim comments only, not open PRs."},
      {"name": "ai-policy-ok", "grade": "pass", "evidence": "docs/CONTRIBUTING.md has no mention of AI — silence counts as pass."},
      {"name": "good-first-label", "grade": "pass", "evidence": "'good first issue' and 'tier-1' labels are present on the issue."}
    ],
    "verdict": "reject"
  }
]
```

---

## Eval iterations

**Run history**

1. Smoke test (`--limit 3`): 3/3 agreement.
2. Full run (20 scored issues): 18/20 agreement. Disagreements on issue-04 (gold: accept, rubric: reject, failed scope-fits) and issue-19 (gold: accept, rubric: reject, failed scope-fits). Categories: claimed 4/4, clear-accept 6/8, dead-repo 3/3, policy 1/1, scope 4/4. Bar: 18/20 PASS.
3. Saved final run with `--save-run eval-run.txt`: 18/20 agreement. Same result as run 2.

**Issue analysis**

issue-04: My rubric decided reject; the gold label is accept. The rubric's scope-fits check failed this issue because the described change appeared to require touching multiple subsystems or lacked a single concrete trigger statement in the issue body. The gold label accepts it because the issue does describe a bounded fix — the scope is arguable but falls within the range a reasonable first contribution could handle. My scope-fits check requires the issue body to identify "a single bug with a stated trigger, a single well-scoped feature, or a single isolated task," and issue-04's description was specific enough to meet the gold label's bar but ambiguous enough that my check's requirement for a clearly stated trigger caused a false rejection. This is one of the 4 genuinely arguable scope calls the assignment describes.

**Check rationale**

The `unclaimed` check, as currently written in my uploaded rubric.md:

> "The issue has no current assignee, no open or draft PR referencing this issue, and no comment in the last 45 days expressing intent to work on it (e.g. 'I'll take this', 'working on it', 'claiming'). A comment that only asks a question or requests clarification is not a claim. A closed, unmerged PR with no recent activity does not count as a claim."

This check exists because the lecture's fourth family — "Issue is Unclaimed" — identifies three signals: no assignee, no open PR, and no fresh claim comment. I chose a 45-day window rather than 30 because stale claims older than a month are unlikely to result in a completed PR, but I wanted a small buffer beyond 30 days to avoid edge cases. The clarification that questions are not claims prevents false rejections on issues where someone asked a follow-up question without intending to work on it. The closed-unmerged-PR exception avoids rejecting issues where a previous attempt was abandoned.

**Trade-offs**

The unclaimed check does not account for implicit claims — someone might be actively working on an issue without commenting or opening a PR. It also cannot detect coordination that happens outside GitHub (e.g., on Slack or Discord). I accept this limitation because the check relies on publicly observable evidence, which is all the skill has access to. The 45-day window means a claim at 46 days old is ignored, which could lead to a conflict if that person is still working but slow. In practice, for the Path Review sandbox repo, the house rules override claim comments anyway, so the open-PR detection is the more important signal — and that part of the check correctly rejected issues #72 and #68 in my live-mode run.

---

## Selection rationale

**Selection rationale**

1. Issue #67 is a security authorization bug — verifying profile ownership before allowing review creation. This fits my interest in backend security and API design, and the fix is scoped to one service file (`review_service.py`) plus its tests, which is realistic for the time available between now and Unit 2.

2. The verdict correctly identified that the repo is active (commits within 12 days), that the issue is well-scoped (one bounded change with a clear trigger — sending another user's profile_id to POST /reviews), and that it is unclaimed (no assignee, no open PR). What the rubric could not weigh is that this is a tier-2 issue, which means it is slightly more ambitious than the tier-1 issues most classmates will choose — but still bounded enough to be a realistic first contribution.

3. The issue has one existing claim comment from another student (ChinoUkaegbu), but the Path Review house rules state that a classmate's claim does not block you. Since there is no open PR linked to the issue, the practical risk of a merge conflict is low. The main anticipated difficulty is understanding the authentication flow well enough to add the ownership check in the right place, but the issue body names the exact function (`create_review()`) and contrasts it with `get_review()` and `list_reviews()` which already do the scoping correctly, so the pattern to follow is in the codebase.

---

Related paths: `eval-run.txt` in this directory; your skill's files in `tools/issue-select/`.
