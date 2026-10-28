# Per-Repository Audit Remediation Prompt

Use this prompt in a **separate, final session** after `REPO_AUDIT_PROMPT.md` (the audit) and
`REPO_AUDIT_VERIFICATION_PROMPT.md` (the independent reviewer) have both completed. Review **one
repository at a time**, and point it at both the audit report and its reviewer report.

This session has two hard-separated phases:

- **Phase 1 — Proposal (no code changes).** Read the audit + review reports, design the best patch
  set, and write it to a proposal file. **Stop. Wait for explicit approval.**
- **Phase 2 — Execution (only after approval).** Apply the approved patches, run the suite, update
  the regression guards, and produce a remediation report.

```text
Act as a careful remediation engineer. Your input is a completed audit plus an independent review
of that audit. Your first job is to propose patches — not to apply them. Do not modify any
repository file until the user has explicitly approved your proposal.

Repository: <REPO_NAME>
Repository path: <ABSOLUTE_PATH>
Audit report: <ABSOLUTE_PATH_TO_AUDIT-FINDINGS.md>
Audit review report: <ABSOLUTE_PATH_TO_AUDIT-REVIEW.md>
Patch proposal path: <ABSOLUTE_PATH>/<REPO_NAME>-AUDIT-PATCH-PROPOSAL.md
Remediation report path (written only in Phase 2): <ABSOLUTE_PATH>/<REPO_NAME>-AUDIT-REMEDIATION.md

Phase 1 — Build the patch proposal (book-phase; no repository changes):

1. Read the audit report and the reviewer report in full. Reconcile them: the reviewer's verdict,
   corrections, and any new findings (e.g. findings the reviewer "Verified only partially") become
   part of the patch backlog alongside the audit's findings. Where the reviewer disagreed with a
   severity, wording, or recommendation, adopt the corrected version.
2. Confirm each finding against the current source before planning a patch — code may have changed
   since the audit. If a defect is already fixed, record that and drop it from the plan.
3. Locate the audit's regression guards: xfail(strict=True) tests asserting desired post-fix
   behavior, standalone repro scripts, or both. Each accepted fix must end with its corresponding
   guard passing as a real test — for an xfail(strict) guard that means removing the marker only
   once the fix lands, and the suite staying green. If no guard exists for a finding you are fixing,
   write one as part of the patch.
4. For every finding in the backlog, propose the minimal fix that addresses the root cause:
   - Exact files and line regions, with the proposed diff (or a precise edit description) per change.
   - Which audit recommendation you follow, and — where the reviewer suggests an alternative or the
     audit offers several options — which option you chose and why (state trade-offs: behavior
     changes, backwards compatibility, `--help`/README updates).
   - Whether documentation or option contracts must change with the code (README, --help text,
     modification guide), inventory every user-visible behavior change.
   - Risk assessment: what the fix could regress, and which existing tests cover or would fail to
     cover that area.
5. Order the work P0 → P3 (the audit's fix priorities), grouping changes so the suite can be run and
   committed after each coherent group. Never bundle unrelated fixes. Omit Informational /
   hardening-only items unless the user asks for them — list them under "deferred" instead.
6. Write the proposal file containing: inputs used and any reconciliation notes, the full per-finding
   patch plan with diffs and risk notes, deferred items, the test/guard strategy per finding, an
   estimated scope per group, and an explicit approval request stating exactly what the user is
   approving.
7. Then STOP. Do not touch the repository. Ask the user to review the proposal and reply with an
   explicit approval, which may be per-group (e.g. "approve P0 only") — honor whatever they approve
   and Do not act until the user replies with explicit approval. Honor exactly what they approve (e.g.
"approve P0 only") and nothing more..

Only if and as approved, Phase 2 — Execute:

1. Apply the approved patches group by group, exactly as proposed. If reality forces a deviation,
   pause and re-confirm with the user rather than improvising.
2. After each group: run the project's test suite (and the audit's repro scripts for the touched
   findings where safe), fix only what your patch broke, and record commands + exact results. Do not
   claim tests passed that were not run.
3. When a fix lands, update its regression guard: remove the xfail(strict) marker so the test runs
   as a permanent regression test, or adopt the standalone repro as a real test if appropriate.
4. Do not weaken tests, security checks, validations, or exit-code contracts to make the suite pass.
   Do not mark a finding "fixed" by editing its test instead of its code.
5. Keep diffs minimal, match the codebase's style, and update README/guide/--help for every
   user-visible behavior change made.
6. Write the remediation report: finding-by-finding status (fixed / deferred / rejected with
   reason / already fixed), the diffs applied, the guard transitions (xfail → regression test), all
   commands run and outcomes, known remaining open items, and confirmation that no unapproved code
   paths were touched.

Rules (both phases):
- Do not modify repository/application code in Phase 1, and in Phase 2 only modify files within the
  approved scope.
- Do not expose, copy, or include real secrets in the proposal or reports.
- Do not run destructive commands or access external systems; keep all testing local (mocks,
  localhost).
- Do not re-audit from scratch; the audit and review are your inputs — verify against current code
  only where necessary for a correct patch.
- Distinguish "verified by running" from "verified by reading" in everything you report, as the
  audit and reviewer did.
- Before finishing, verify: the proposal existed with an explicit approval checkpoint recorded;
  only approved changes were applied; the suite is green with all adopted guards; the remediation
  report exists and every claim is backed by evidence or labeled unverified.
```

Replace the placeholders with the repository name, absolute path, and the two report paths. Keep this
session separate from the audit and reviewer sessions. The proposal-and-wait design is deliberate: it
prevents the fixer from interpreting "best patch" on its own, and gives the user a chance to reject
behavior changes ((e.g. stricter validation, changed defaults, or changed CLI flags)) before any code is touched. If the
reviewer report does not exist yet, run the verification session first — patches designed from an
unverified audit inherit its errors.
