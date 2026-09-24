# W5-B — SONNET 5 OCR + Release Tooling Executor

MODEL: SONNET 5
ROLE: Implementation Executor

Workspace: F:\00-Ticenpi-SaaS
OCR: F:\00-Ticenpi-SaaS\TicenpiLetter
Deploy: F:\00-Ticenpi-SaaS\deploy

Goal:
- Fix the OCR Staging blocker.
- Finish OCR CI, Staging deployment and assigned/unassigned E2E.
- Add or complete release evidence, status and promote tooling.
- Do not change Production.

Known OCR defect:
product_seat_status is called without p_product_code='ocr', causing PGRST202/404 and fail-closed 503.

Rules:
- Use isolated worktree.
- Preserve unrelated WIP.
- Do not change Platform schema, OCR RLS, Letter authorization boundary, or product_code.
- Letter stays internal to OCR.
- Do not deploy Production.
- Do not repeat already-passed broad audits.

Release evidence:
First inspect existing deploy/release conventions and extend them instead of duplicating them.

Evidence should record:
product, environment, status, source_commit, backend_digest, frontend_digest, deploy_config_commit, release_id, rollback_release_id, ci_run_id, accepted_at, runtime_identity_verified, health_verified, auth_verified, commercial_verified, e2e_verified.

Keep both immutable history and one accepted Staging pointer/file per product.

Rules:
- Successful Staging deploy = DEPLOYED only.
- Mandatory E2E PASS is required before ACCEPTED.
- Production promotion may only read ACCEPTED Staging evidence.
- Production must use the same immutable digest; no rebuild.
- Rollback events must also be recorded.

Target operator commands:
.\deploy.ps1 post staging
.\deploy.ps1 dm staging
.\deploy.ps1 ocr staging

.\release.ps1 post status
.\release.ps1 dm status
.\release.ps1 ocr status

.\promote.ps1 post production
.\promote.ps1 dm production
.\promote.ps1 ocr production

If current deploy.ps1 syntax differs, add a compatible wrapper instead of breaking the working chain.

OCR work:
1. Confirm call site and missing request parameter.
2. Minimal fix: pass p_product_code='ocr'.
3. Focused tests: request body, assigned allow, unassigned 403, invalid JWT 401, RPC error/malformed fail closed, Letter internal, RLS unchanged.
4. Run relevant regression and git diff --check.
5. Commit.
6. Run CI.
7. Capture exact immutable digest.
8. Prepare OCR Staging deployment only. Run the supported dry-run/preflight path and produce the exact deploy command required by the repository/HANDOFF.
9. HARD GUARDRAIL: the executor MUST NOT invoke deploy.ps1 with -Yes (or any equivalent confirmed/mutating deployment flag). Stop and request the human operator to execute the exact Staging deploy command. Preserve state so execution can resume immediately afterward.
10. After the human confirms the deploy command completed, verify runtime identity, health and auth.
11. Browser E2E with isolated sessions:
   assigned -> OCR protected API 200 and Letter works
   unassigned -> auth succeeds and commercial access is 403
12. After PASS, write OCR ACCEPTED Staging evidence.

Post/DM:
Do not redeploy them.
If existing evidence is complete, import/build their ACCEPTED Staging evidence.
Never guess missing digests; report missing evidence for W5-C/OPUS to verify.

Final report:
OCR_STAGING_FIX =
OCR_STAGING_E2E =
OCR_STAGING_RELEASE_ACCEPTED =
POST_EVIDENCE_IMPORTED =
DM_EVIDENCE_IMPORTED =
DEPLOY_COMMAND_STANDARDIZED =
STATUS_COMMAND_STANDARDIZED =
PROMOTE_COMMAND_STANDARDIZED =
CHANGED_FILES =
COMMITS =
CI_RUNS =
EVIDENCE_PATHS =
COMMAND_EXAMPLES =

Do not perform any Production deployment.

Deployment authority rule:
- AI/Executor: prepare, dry-run, preflight, CI, digest pinning, evidence, and post-deploy verification.
- Human operator: executes the actual mutating Staging deploy command that requires deploy.ps1 -Yes (or equivalent confirmation).
- The AI must never bypass this HANDOFF guardrail.