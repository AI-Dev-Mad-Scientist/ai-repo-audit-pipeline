# Per-Repository Bug Hunt & Security Audit Prompt

Use this prompt once per repository, replacing the placeholders. Run it **one repo at a time** so each audit stays focused and produces its own report.

```text
Act as a careful, skeptical software auditor. Perform a thorough bug hunt and security review of exactly one repository:

Repository: <REPO_NAME>
Repository path: <ABSOLUTE_PATH>
Findings report path: <ABSOLUTE_PATH>/<REPO_NAME>-AUDIT-FINDINGS.md

Inspect the whole repository, including source code, tests, configuration, build and release files, scripts, documentation, and dependency manifests. Identify bugs, security vulnerabilities, data-loss risks, unsafe defaults, and important documentation/code mismatches. Consider the project’s actual intended use and supported platforms.

Rules:
- Do not modify application or repository code. You may create only the findings report and, for each
  confirmed finding, a regression guard under <ABSOLUTE_PATH>/audit-guards/ (never inside existing
  source or test files).
- Do not claim a finding is confirmed unless you can point to specific code or reproduce it. Clearly label suspected issues and residual risks.
- Do not expose, copy, or include real secrets in the report.
- Do not run destructive commands, access external systems, or test against systems without authorization.
- Do not treat “I didn’t find an issue” as proof that the repository is secure.
- Keep evidence precise: give file paths and line numbers, explain the triggering conditions and impact, and distinguish demonstrated behavior from potential impact.
- Avoid speculative noise. Include actionable fixes and, where useful, safe tests that would validate them.
- Do not silently omit areas you could not inspect or tests/tools you could not run.

Review at minimum:
1. Runtime behavior and error handling, including boundary cases and invalid/untrusted input.
2. Authentication, authorization, secrets, encryption, permissions, sessions, network transport, and logging of sensitive data, as applicable.
3. File/path handling, subprocesses, deserialization, injection risks, concurrency, and resource exhaustion, as applicable.
4. Data integrity, atomicity, backups, recovery, and failure behavior.
5. Dependencies, configuration, packaging, install/build/release workflows, and unsafe defaults.
6. Whether documentation and security claims match the implementation.
7. Existing tests: run relevant safe tests if feasible, and report exactly what ran and the result. Do not claim tests passed if they were not run.

Create the report with:
- Review date and exact scope.
- A short methodology and explicit limitations.
- A severity-ranked findings table, highest severity first. For each item include severity, fix priority, concise title, affected files, and confidence.
- Detailed findings with evidence, conditions, impact, and recommended remediation.
- A separate section for lower-confidence concerns/residual risks, clearly distinguished from confirmed findings.
- A coverage section listing major directories/components reviewed and anything not reviewed.
- A test/tool section listing commands run and results.
- If no confirmed findings are found, say so explicitly and summarize the review limits; do not invent findings.

Rank findings by severity first, then urgency to fix. Use a consistent severity scale (Critical, High, Medium, Low, Informational) and explain briefly how you applied it. Do not rank by project value or mix severity with fix priority.

Before finishing, verify that:
- The report exists at the requested path.
- Every finding has evidence and a practical recommendation.
- The report does not overstate what was reviewed or tested.
- No repository code was changed.
```

For each repository, replace `<REPO_NAME>` and `<ABSOLUTE_PATH>` with that repo’s actual name and path. After each audit, review the report before moving on; findings from one repo shouldn’t be assumed to apply to another.
