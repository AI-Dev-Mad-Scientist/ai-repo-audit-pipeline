## AI generated README
# AI Repo Audit Pipeline
 
A three-stage, prompt-based workflow for using an AI coding assistant to **hunt bugs and security vulnerabilities in a codebase**, **independently check the results**, and **fix what's real**, with a human approval gate before any code is touched.
 
Each stage is a copy-paste prompt template meant to run in its **own separate session**, one repository at a time. Keeping the sessions separate stops the auditor from grading its own work and stops the fixer from deciding on its own what "best patch" means.
 
```
  ┌────────────┐      ┌────────────────┐      ┌─────────────────────────────┐
  │ 1. AUDIT   │ ───▶ │ 2. VERIFY      │ ───▶ │ 3. REMEDIATE                │
  │ find issues│      │ independent    │      │ Phase 1: propose patches    │
  │ (read-only)│      │ review         │      │   ── human approval gate ── │
  │            │      │ (read-only)    │      │ Phase 2: apply + test       │
  └────────────┘      └────────────────┘      └─────────────────────────────┘
   *-AUDIT-FINDINGS.md  *-AUDIT-REVIEW.md      *-AUDIT-PATCH-PROPOSAL.md
                                               *-AUDIT-REMEDIATION.md
```
 
## The prompts
 
| Stage | File | Modifies repo code? | Output |
|---|---|---|---|
| 1. Audit | [`REPO_AUDIT_PROMPT.md`](REPO_AUDIT_PROMPT.md) | No | `<REPO>-AUDIT-FINDINGS.md` |
| 2. Verify | [`REPO_AUDIT_VERIFICATION_PROMPT.md`](REPO_AUDIT_VERIFICATION_PROMPT.md) | No | `<REPO>-AUDIT-REVIEW.md` |
| 3. Remediate | [`REPO_AUDIT_REMEDIATION_PROMPT.md`](REPO_AUDIT_REMEDIATION_PROMPT.md) | Only in Phase 2, only after explicit approval | `<REPO>-AUDIT-PATCH-PROPOSAL.md`, `<REPO>-AUDIT-REMEDIATION.md` |
 
### 1. Audit
A skeptical auditor reviews one repository (source, tests, config, build/release files, scripts, docs, dependencies) and writes a severity-ranked findings report. It must give file paths and line numbers, separate confirmed findings from suspected ones, report which tests it actually ran, and list what it did *not* review.
 
### 2. Verify
A different session acts as an independent reviewer of the audit report. It re-traces every finding against the code, checks severity and recommendations, looks for false positives and missed issues, and gives a verdict: **Supported**, **Supported with corrections**, or **Not sufficiently supported**.
 
### 3. Remediate
Two hard-separated phases:
- **Phase 1 (proposal):** reconcile the audit and the review, re-confirm each finding against current code, and write a patch plan with diffs, risk notes, and a test strategy. Then **stop and wait for approval**, which can be per group (e.g. "approve P0 only").
- **Phase 2 (execution):** apply only what was approved, run the test suite after each group, convert regression guards into permanent tests, and write a remediation report.
## Usage
 
1. Copy the prompt block from the stage's file into a fresh AI session that has file and shell access to your repo (for example Claude Code or a similar agentic coding tool).
2. Replace the placeholders: `<REPO_NAME>`, `<ABSOLUTE_PATH>`, and the report paths for stages 2 and 3.
3. Run **one repo per session, one stage per session.** Read each report before moving to the next stage.
4. Don't carry findings from one repo over to another.
## Design principles
 
- **Separation of roles.** Audit, review, and fix happen in different sessions to preserve independence.
- **Evidence over assertion.** Every finding needs file/line evidence. "Verified by running" is distinguished from "verified by reading."
- **No false assurance.** "I found nothing" is never treated as proof of security, and unreviewed areas must be disclosed.
- **Read-only until approved.** Audit and review never change code, and remediation proposes before it acts.
- **Safe by default.** No real secrets in reports, no destructive commands, no external systems, and local testing only.
- **No gaming the suite.** Remediation may not weaken tests, validations, or exit-code contracts to get green results.
- **Severity ≠ priority.** Findings are ranked on a Critical / High / Medium / Low / Informational scale, with fix priority tracked separately.
## Limitations
 
- These are prompts, not a scanner. Results depend on the model, its tooling, and the repo size and quality. Treat the output as a structured second opinion, not a certification.
- The AI can still miss issues or report false positives. The verification stage reduces this but does not eliminate it. Human review of every report is expected.
- Large repos may exceed what one session can cover. The audit prompt requires the model to say what it skipped.
- The remediation prompt's regression-guard language (`xfail(strict=True)`) assumes a pytest-style test setup. Adapt it for your language and framework.
## Contributing
 
Issues and PRs that improve the prompts are welcome, especially adaptations for specific languages or ecosystems and examples of sanitized reports.
 
## License
 
Add a license of your choice (MIT is a common default for prompt/template repos).
 
Claude finished the response
