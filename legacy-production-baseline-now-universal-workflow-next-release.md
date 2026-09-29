# Legacy Production Baseline — Dispatcher Only

此檔不再作為可執行提示詞。

原因：Production Baseline 必須一個產品一個任務，避免 TARGET_PRODUCT、repo、release evidence 與 delivery model 混用。

請改用以下已填好目標產品的專用提示詞：

- DM: legacy-production-baseline-dm.md
- Post: legacy-production-baseline-post.md
- OCR: legacy-production-baseline-ocr.md
- Sign: legacy-production-baseline-sign.md
- 591: legacy-production-baseline-591.md

規則：

1. 一次只跑一個產品。
2. 不需要使用者手動修改 TARGET_PRODUCT 或 PRODUCT_REPO。
3. 本輪不讀取、不套用未完成的新 Universal Workflow。
4. clean preflight + artifact/target/rollback confirmed + no destructive mutation 時，直接 Production promotion，不再次詢問。
5. 遇到 DB migration、destructive schema、secret、DNS、port、runtime architecture、非預期 source/config mutation 才 STOP。
6. Production deploy 後仍需 runtime/health/auth/commercial/canary 驗證；DEPLOYED != ACCEPTED。
