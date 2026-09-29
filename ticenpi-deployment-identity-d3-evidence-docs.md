# Task D3 — Release Evidence + Documentation Inventory

推薦模型：MiMo-V2.6-Flash / DeepSeek V4 Flash

Release evidence: F:\00-Ticenpi-SaaS\.release-evidence\
System docs: F:\00-Ticenpi-SaaS\deploy\docs\system\

本輪只做 JSON/MD mechanical inventory；唯讀，不修改、不做架構決策。

## Part 1 — Release evidence

優先 DM，再抽查 Post/OCR。
列 accepted-staging / history / current-production 實際欄位：

FILE =
product =
environment =
status =
source_commit field =
artifact_digest/id field =
deploy_config_commit field =
release_id field =
rollback_release_id field =
runtime_identity_verified field =
accepted_at field =
OTHER_IDENTITY_FIELDS =

Secrets 必須遮蔽。

## Part 2 — 文件位置

只查：
new-workflow.md
PRODUCT_TO_STAGING.md
STAGING_TO_PRODUCTION.md
SYSTEM_OWNERSHIP.md
PRODUCT_INTEGRATION_CONTRACT.md
TICENPI_SYSTEM_CURRENT_STATE.md
STAGING_TEST_IDENTITY_CONTRACT.md

找：
requiredEnv
shared/.env
release identity
runtime identity
accepted Staging
Production promotion
same artifact/digest
Supabase target
database target
deploy-time injection

輸出：
DOCUMENT | SECTION/LINE | TOPIC | CURRENT_TEXT_SUMMARY

不改文件、不提出新規範。

最後：
WORKER_TASK = D3_EVIDENCE_DOCS
FILES_INSPECTED =
FACTS =
CANDIDATE_EVIDENCE =
UNRESOLVED =
NO_ARCHITECTURE_DECISION_MADE = YES
