# W5-C — Read-only Release Verification

Model: KIMI 3 or MiMo V2.6 Pro

Workspace: F:\00-Ticenpi-SaaS

Read-only task. Do not modify repositories or environments.

Check Post:
- source commit
- CI run
- image digests
- deploy-config commit
- staging release
- runtime identity
- health/auth
- assigned/unassigned result
- rollback reference

Check DM:
- staging release 20260924-213157
- DM_SEAT_POLICY=require
- source/digest/config identity
- assigned 200
- unassigned 403
- health/ready
- rollback reference

Check OCR after W5-B:
- p_product_code='ocr'
- CI source/digest
- staging runtime digest
- assigned 200
- unassigned 403
- Letter remains internal
- accepted evidence matches runtime

Check release tooling:
- deploy to staging does not automatically mean ACCEPTED
- ACCEPTED requires mandatory E2E
- accepted evidence points to immutable release data
- status reads the correct release identity
- promote uses only accepted staging evidence
- promote does not rebuild
- promoted digest matches accepted staging digest
- rollback is recorded

Check production preflight without changing anything:
- environment identity
- schema/migration differences
- service identity
- required configuration presence
- rollback reference
- accepted artifact availability
- manifest drift
- Facebook/Studio prerequisites
- Launcher prerequisites

Audit 591 and Sign compatibility with the same release evidence pattern.

Return:
POST_STAGING_RELEASE_ACCEPTED =
DM_STAGING_RELEASE_ACCEPTED =
OCR_STAGING_RELEASE_ACCEPTED =
RELEASE_TOOLING_CONTRACT =
PRODUCTION_PREFLIGHT =
591_COMPATIBILITY =
SIGN_COMPATIBILITY =
EVIDENCE_GAPS =
PRODUCTION_BLOCKERS =
SMALLEST_NEXT_ACTIONS =