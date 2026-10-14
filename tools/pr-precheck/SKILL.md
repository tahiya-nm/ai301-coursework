---
name: pr-precheck
description: Grade a PR package (a candidate pull request read against the plan it claims to implement and the issue that plan belongs to) and decide whether it is ready to submit. Use when checking your own branch, draft PR title, and description before opening the pull request, or when grading an eval package bundle.
---

# pr-precheck: rubric-driven PR grading

## The question

This tool answers one question: **is this pull request ready to submit?** A PR package consists of a candidate pull request (its diff, commits, title, description, and test evidence) read against the plan it claims to implement (the plan's scope boundary, approach, test plan, and deviation notes) and the issue that plan belongs to (the issue context and reproduction evidence). The tool grades whether the diff matches the plan, the evidence proves the fix, the change is reviewable, and the PR respects the repo's stated standards.

## Inputs and modes

**Live mode:** The tool reads these inputs from the working copy and GitHub:
- `plan.md` in the current directory, including its `## Deviations` section
- The branch's diff: the output of `git diff main...HEAD` (three dots), run from the working copy — this is everything the branch changes relative to the repo's default branch
- The draft PR title and description (either in a draft file or as provided by the user)
- Test evidence: the before/after output the user provides (repro steps re-run against the fix)
- The issue on GitHub: its title, body, and thread
- The repo's CONTRIBUTING.md, PR template, and any stated AI-use disclosure policy

**Eval mode:** The bundle is the whole world. Nothing is fetched from GitHub. The plan-context block, the candidate PR (diff, description, test evidence, commits), and the repo-facts block are all inside the package file. Every check runs with the full verdict rule. No scope.md, no voice-guide.md.

## The scope seam (live mode only)

In live mode, read `scope.md` before grading. If the `Repo:` line contains an unfilled placeholder or is empty, refuse to grade and report that the scope must be set first. If the `Repo:` line names a repo, use it to locate the issue and the repo's conventions. Eval mode ignores `scope.md` entirely.

## The voice seam (live mode only)

In live mode, read `voice-guide.md` after grading all rubric checks but before emitting the final output. Apply its rules to the PR title and description: if any rule is broken, report which rule and which line breaks it, but do not change the verdict — the voice guide gates outgoing text, it never holds a PR on its own. Eval mode ignores `voice-guide.md` entirely.

## Component reads

The tool's judgment comes from three files working together:
- `rubric.md` defines the checks (what to decide) and the verdict rule (how check grades combine into accept or reject).
- `references/evidence-guide.md` maps where each evidence family lives in a package or on GitHub (where to look).
- `procedure.md` is executed as written — it defines the read order, evidence gathering moves, check execution order, and verdict assembly steps (how to grade).

When the procedure is silent on a step, report the gap and stop — never improvise around a missing instruction. When `rubric.md` or `procedure.md` has no content (empty checks table or empty procedure sections), refuse to grade and report that the components must be filled first.

## Verdict and output

The verdict space is binary: **accept** means the PR is ready to submit; **reject** means hold and revise. End the reply with the following fenced JSON block, valid and last, with nothing after it:

```json
{
  "item": "<PR URL or bundle id>",
  "checks": [
    {"name": "<check name>", "grade": "pass|fail|unclear",
     "evidence": "<one line: the fact or quote that decided it>"}
  ],
  "verdict": "accept|reject"
}
```

## Grading discipline

1. Evidence first: gather all evidence before grading any check. Never grade a check before its evidence is in hand.
2. Grade the thing, not the polish: judge whether the diff matches the plan and the evidence proves the fix, not how long the description is or how many commits there are.
3. The rubric decides: every check grade comes from applying the rubric's pass condition to the gathered evidence. No check is graded on impression.
4. The procedure decides how: follow procedure.md's steps in order. Do not skip steps or reorder them.
5. Unclear grades: when a check's evidence is genuinely ambiguous or absent, grade it as `unclear`. If the rubric's verdict rule does not state how `unclear` is treated, treat it as fail.
