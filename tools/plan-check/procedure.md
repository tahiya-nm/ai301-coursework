# Procedure: how this skill grades a plan package

## Read order

1. Read the issue context first (title, body, thread highlights). Note the reported behavior, any maintainer signals or explicit direction, and any stated templates or conventions from the repo-facts block.
2. Read the repro-evidence block second. Note the exact observed behavior: error messages, status codes, output excerpts, database states. This is the ground truth the plan must build on.
3. Read the candidate plan third. By reading repro evidence before the plan, you can tell whether the plan's diagnosis follows from what was observed or invents a cause. Note the stated cause, the files named, the scope boundary, the approach, the test plan, and any risks or unknowns.
4. Read the candidate plan comment last. Note whether it addresses the specific issue, references thread context, and follows repo conventions (including AI-use disclosure if the repo-facts block requires it).

The order matters because diagnosis-grounded and test-plan-decisive both compare the plan against the repro evidence. Reading the evidence first prevents anchoring on the plan's claims before seeing what was actually observed.

## Evidence gathering

1. **For diagnosis-grounded:** From the plan, extract the stated cause or root-cause claim. From the repro-evidence block, extract the observed behavior (error messages, output, artifacts). Record both side by side.
2. **For scope-bounded:** From the plan, extract the in-scope statement, the not-in-scope line (if any), and every file or area named in the approach. Check whether any named changes fall outside what the diagnosis requires.
3. **For plan-executable:** From the plan, extract the approach steps, file names, and description of edits. From the repo-facts block, note any relevant project structure. Check whether a stranger could start executing without asking the author anything.
4. **For test-plan-decisive:** From the plan, extract the test or verification section. From the repro-evidence block, extract the steps and observed output. Check whether the test plan re-runs those steps and names what the output should look like after the fix.
5. **For honesty-about-unknowns:** From the plan, extract the risks, unknowns, or deviations section. Scan the approach for claims stated as certainty. Check whether uncertain assumptions are flagged.
6. **For comment-thread-aware:** From the plan comment, extract the text. From the thread highlights, extract any maintainer signals. From the repo-facts block, extract stated templates, contribution policy, and AI-use disclosure requirements. Check whether the comment addresses the specific issue and follows conventions.
7. **For names-repro-evidence:** From the plan's diagnosis and test plan, check whether at least one concrete artifact from the repro evidence (a quoted error, a status code, a command) is referenced by value, not just by summary.

## Check execution

1. Execute checks in this order: diagnosis-grounded, scope-bounded, plan-executable, test-plan-decisive, honesty-about-unknowns, comment-thread-aware, names-repro-evidence. This order front-loads the checks most likely to reject, saving time.
2. For each check, compare the gathered evidence against the pass condition in rubric.md. Grade as pass, fail, or unclear.
3. If evidence for a check is genuinely absent from the package (e.g., no test plan section exists at all), grade that check as fail — absence of evidence is not a pass.
4. A check may be graded without re-reading the whole package if the gathered evidence from the read-order phase already contains everything the check needs. Re-read the relevant section only if the gathered notes are ambiguous.
5. Record a one-sentence reason for each grade, quoting the specific evidence that decided it.

## Verdict assembly

1. Apply the verdict rule from rubric.md: accept if every required check passes; reject if any required check fails; unclear on any required check counts as fail.
2. Preferred checks (names-repro-evidence) never change the verdict. They appear in the output for ranking but do not affect accept/reject.
3. In the output, quote the evidence from the deciding check — the first required check that failed (if rejecting) or a summary of all-pass (if accepting).
4. Two executors given the same package, the same rubric, and this procedure must reach the same verdict. If a check's grade depends on interpretation not covered by the pass condition, grade it as unclear (which counts as fail).
