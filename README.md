# Project 2 (Stage 2): E-Wallet Checkout UX, AI-Assisted Spec & UAT Suite

> **Chuỗi 3 Dự án (E-Wallet Payment System)**:
> - **Stage 1 (Core Backend & BRD)**: [e-wallet-payment-simulator](https://github.com/phucdatphan/e-wallet-payment-simulator)
> - **Stage 2 (UX, Agile Spec & UAT)** *(Repo hiện tại)*: [e-wallet-checkout-ux-uat](https://github.com/phucdatphan/e-wallet-checkout-ux-uat)
> - **Stage 3 (Post-Launch Analytics & Reconciliation)**: [e-wallet-post-launch-analytics](https://github.com/phucdatphan/e-wallet-post-launch-analytics)

---

## 1. Tổng quan
Hiện thực hóa các luồng nghiệp vụ từ **Stage 1** (giữ tồn kho 15 phút, Webhook IPN 3 lớp bảo mật, Idempotency chống double-charge, Timeout refund) thành giao diện người dùng (UI/UX), đặc tả Agile (User Stories + Acceptance Criteria) và bộ kiểm thử UAT/API trước khi Go-live.

## 2. Deliverables
- **`/wireframes`**:
  - [user-journey-wireframes.html](wireframes/user-journey-wireframes.html): 5 màn hình cốt lõi (Checkout → QR + Đếm ngược 15:00 → Paid / Expired / Failed).
  - **Figma Design**: https://www.figma.com/design/eedWAY1oMYerrjZztJNXIc?node-id=4-3&autoplay-code-preview=4-3
- **[user-stories-ac.md](user-stories-ac.md)**: 10 User Stories chuẩn INVEST + Gherkin (`Given - When - Then`).
- **[uat-test-cases.csv](uat-test-cases.csv)**: 26 Test Cases (Happy cases + Edge cases: double-submit, IPN retry 3 lần, hết hạn 15 phút, late IPN 14:59, sai HMAC, lệch số tiền).
- **`/postman`**: [e-wallet-payment-simulator.postman_collection.json](postman/e-wallet-payment-simulator.postman_collection.json) chạy test trực tiếp với Backend Node.js ở Stage 1.
