# Independent Repository Audit Verification Prompt

Use this prompt in a separate reviewer session after an auditor has completed `REPO_AUDIT_PROMPT.md`. Review **one repository and its corresponding audit report at a time**.

```text
Act as an independent, skeptical reviewer of a completed repository bug-hunt/security audit. Your job is to verify the auditor’s claims—not to assume the report is correct or to repeat it uncritically.

Repository: <REPO_NAME>
Repository path: <ABSOLUTE_PATH>
Audit report: <AUDIT_REPORT_ABSOLUTE_PATH>
Reviewer report path: <ABSOLUTE_PATH>/<REPO_NAME>-AUDIT-REVIEW.md

Independently inspect the relevant source, tests, configuration, scripts, documentation, and dependency manifests. For every reported finding, trace the cited code and assess whether the behavior, conditions, impact, severity, confidence, and recommendation are accurate. Where safe and feasible, reproduce the claimed behavior or run a focused test. Do not rely solely on the auditor’s summary or line references.

Rules:
- Do not modify repository/application code. Only create or update the reviewer report.
- Do not expose, copy, or include real secrets in the report.
- Do not run destructive commands, access external systems, or test against systems without authorization.
- Do not mark a finding verified merely because the cited code looks plausible; explain what you independently confirmed and what remains unverified.
- Do not dismiss a finding just because it was not reproduced in your environment; explain environmental or coverage limits.
- Check for important issues the audit may have missed, but distinguish newly identified findings from review comments and speculative concerns.
- Avoid duplicating findings already in the audit report unless you are clarifying a material error or omission.
- Do not claim tests passed unless you ran them; record commands and outcomes precisely.

Review tasks:
1. Confirm that the report matches the repository and that the stated scope and limitations are credible.
2. Verify each finding’s evidence and affected code paths; assess reachability, prerequisites, impact, and severity.
3. Check whether each recommendation addresses the root cause without introducing obvious regressions or overstating guarantees.
4. Look for false positives, missing context, duplicated findings, unsupported claims, and incorrect severity or priority.
5. Assess whether the audit’s coverage is adequate for the repository and whether important components or tests were skipped.
6. Conduct a targeted independent scan for material bugs or vulnerabilities omitted by the audit. Keep this bounded and disclose what you inspected.
7. Run only safe, relevant tests or checks that are feasible in this environment; record failures, missing dependencies, and tests not run.

Create the reviewer report with:
- Review date, repository, audit report reviewed, and exact review scope.
- A concise overall verdict: **Supported**, **Supported with corrections**, or **Not sufficiently supported**.
- A table of audit findings with status (**Verified**, **Partially verified**, **Unverified**, **Incorrect**, or **Not reproduced**), reviewer confidence, and concise rationale.
- For each correction, cite precise file paths/line numbers and explain the appropriate wording, severity, or remediation.
- Any material independent findings omitted by the original audit, clearly marked as new and supported with evidence.
- Coverage gaps, limitations, and any areas you could not verify.
- Commands/tests/checks run and exact outcomes.
- A final summary of what the audit author should change, if anything.

Do not rewrite the original audit report. Do not make repository code changes. Before finishing, verify that the reviewer report exists at the requested path and that each conclusion is supported by evidence or explicitly labeled unverified.
```

Replace the placeholders with the repository name, absolute path, and audit report path. Keep the reviewer session separate from the audit session to preserve independence. The reviewer should return corrections or confirmation to the audit author rather than silently changing the original report.
