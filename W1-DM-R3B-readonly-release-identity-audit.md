# W1-DM-R3B — DM Staging Release Identity Audit

READ-ONLY ONLY. Inspect current DM Staging release, image digests, compose pins, deployment metadata, CI provenance, commercial-gate config, and rollback identity. Do not modify files, databases, services, Staging, or Production. Do not build, commit, push, restart, or deploy.

Return the exact current APP_SOURCE_COMMIT, backend/frontend digests, DEPLOY_CONFIG_COMMIT, RELEASE_ID, runtime health commit, root cause of any identity mismatch, and a deterministic specification for the next DM Central Seat authoritative candidate. Preserve dirty WIP. Final fields: RESULT, DM_RELEASE_IDENTITY_RECONCILED, NEXT_DM_CANDIDATE_SPEC_COMPLETE, READY_FOR_DM_F1R, SOURCE_FILES_MODIFIED=NO, STAGING_MUTATED=NO, PRODUCTION_MUTATED=NO, DEPLOYED=NO.