<p align="center">
  <img src="https://img.shields.io/badge/Stage%202-UX%20%26%20UAT%20Suite-blueviolet?style=for-the-badge&logo=figma" alt="Stage 2" />
  <img src="https://img.shields.io/badge/Agile%2FScrum-User%20Stories%20%26%20AC-success?style=for-the-badge&logo=jira" alt="Agile" />
  <img src="https://img.shields.io/badge/Testing-26%20UAT%20Cases%20%7C%20Postman-informational?style=for-the-badge&logo=postman" alt="Testing" />
  <img src="https://img.shields.io/badge/License-MIT-orange?style=for-the-badge" alt="License" />
</p>

<h1 align="center">🎨 E-Wallet Checkout UX, AI-Assisted Spec & UAT Suite</h1>

<h3 align="center">
  Stage 2: Hiện thực hóa Đặc tả Nghiệp vụ thành Giao diện Người dùng (UI/UX), User Stories & Bộ Kiểm thử UAT/API
</h3>

<p align="center">
  Dự án nối tiếp trực tiếp từ <strong>Stage 1 (Core Backend & BRD)</strong>, chuyển đổi toàn bộ yêu cầu nghiệp vụ (giữ tồn kho 15 phút, Webhook IPN 3 lớp bảo mật, Idempotency chống double-charge, Timeout refund) thành bản thiết kế Wireframe trên Figma, 10 User Stories chuẩn Gherkin và 26 kịch bản kiểm thử UAT + Postman Collection chạy thật.
</p>

---

## 🔗 Chuỗi 3 Dự Án (E-Wallet Payment System)

| Giai đoạn | Tên Repo | Vai trò & Trọng tâm | Liên kết |
| :--- | :--- | :--- | :--- |
| **Stage 1** | **e-wallet-payment-simulator** | Core Backend Node.js, BRD/SRS, HMAC-SHA256, IPN | [Xem Repo Stage 1](https://github.com/phucdatphan/e-wallet-payment-simulator) |
| **Stage 2** *(Hiện tại)* | **e-wallet-checkout-ux-uat** | Wireframes Figma, 10 User Stories, 26 UAT Cases, Postman | [Xem Repo Stage 2](https://github.com/phucdatphan/e-wallet-checkout-ux-uat) |
| **Stage 3** | **e-wallet-post-launch-analytics** | 10,500+ Orders SQL Analytics, Funnel & Đối soát 3 bên | [Xem Repo Stage 3](https://github.com/phucdatphan/e-wallet-post-launch-analytics) |

---

## 📌 Mục Lục
- [1. Bối cảnh & Mục tiêu Stage 2](#-1-bối-cảnh--mục-tiêu-stage-2)
- [2. User Journey & 5 Màn hình Wireframes (Figma)](#-2-user-journey--5-màn-hình-wireframes-figma)
- [3. AI-Assisted Agile Artifacts (User Stories & AC)](#-3-ai-assisted-agile-artifacts-user-stories--ac)
- [4. Bộ Kiểm thử UAT & Postman Collection](#-4-bộ-kiểm-thử-uat--postman-collection)
- [5. Cấu trúc Thư mục Dự án](#-5-cấu-trúc-thư-mục-dự-án)
- [6. Hướng dẫn Chạy Kiểm thử với Stage 1](#-6-hướng-dẫn-chạy-kiểm-thử-với-stage-1)

---

## 🎯 1. Bối cảnh & Mục tiêu Stage 2

Sau khi hoàn thiện kiến trúc lõi ở **Stage 1**, đội ngũ sản phẩm cần một bộ tài liệu chuẩn mực để **Dev, QA và BA** cùng hiểu đúng trước khi Go-live:
1. **Chuẩn hóa trải nghiệm bất đồng bộ (Async UX)**: Khách quét QR trên app ví khác nên web phải có bộ đếm ngược **15:00** và tách bạch 3 trạng thái cuối (`PAID`, `EXPIRED`, `FAILED`).
2. **Không để lọt Edge Cases tài chính**: Quy định rõ hành vi khi user bấm thanh toán 2 lần (Double Submit), Gateway retry IPN 3 lần (Idempotency), hoặc thanh toán sát giờ hết hạn (`14:59` → Timeout Refund).
3. **Kiểm thử tự động hóa**: Kết nối Postman trực tiếp với Backend Node.js (`localhost:3000` & `localhost:4000`) của Stage 1 để xác minh 100% API pass trước khi lên Production.

---

## 🖼️ 2. User Journey & 5 Màn hình Wireframes (Figma)

```
[01. Chọn Ví (MoMo/ZaloPay)] ──(Khóa kho 15p)──> [02. Màn hình QR + Đếm ngược 15:00]
                                                          │
          ┌───────────────────────────────────────────────┼───────────────────────────────────────────────┐
          ▼ (IPN Code 0)                                  ▼ (Quá 15 phút)                                 ▼ (User Hủy / Lỗi)
 [03. Thành công (PAID)]                         [04. Hết hạn (EXPIRED)]                         [05. Thất bại (FAILED)]
```

- **Figma Design (Interactive)**: 👉 [Mở Wireframes trên Figma](https://www.figma.com/design/eedWAY1oMYerrjZztJNXIc?node-id=4-3&autoplay-code-preview=4-3)
- **Bản HTML Offline**: [wireframes/user-journey-wireframes.html](wireframes/user-journey-wireframes.html)

---

## 📝 3. AI-Assisted Agile Artifacts (User Stories & AC)

Toàn bộ yêu cầu từ `BUSINESS_ANALYSIS.md` (Stage 1) được chuyển đổi thành **10 User Stories** chuẩn **INVEST** kèm **Acceptance Criteria (`Given - When - Then`)** tại [user-stories-ac.md](user-stories-ac.md):

| Nhóm chức năng | User Stories tiêu biểu | Trọng tâm nghiệp vụ |
| :--- | :--- | :--- |
| **Checkout & Reservation** | `US-01`, `US-02`, `US-03` | Chọn ví, khóa tồn kho 15 phút, chống Double Submit, hiển thị QR + Timer |
| **IPN Security & Idempotency** | `US-04`, `US-05` | Xác thực 3 lớp (HMAC, Idempotency Key, Amount), chống trừ kho 2 lần |
| **Trạng thái & Ngoại lệ** | `US-06`, `US-07`, `US-08`, `US-09`, `US-10` | Màn hình Paid/Expired/Failed, Late IPN Refund (`14:59`), Active Polling |

---

## 🧪 4. Bộ Kiểm thử UAT & Postman Collection

- **26 Kịch bản UAT ([uat-test-cases.csv](uat-test-cases.csv))**: Bao gồm Happy Cases, Edge Cases (mạng rớt, bấm đúp, hết hạn 15 phút, đồng thời 2 user mua sản phẩm cuối) và Security Cases (giả mạo HMAC, sửa số tiền).
- **Postman Collection ([postman/e-wallet-payment-simulator.postman_collection.json](postman/e-wallet-payment-simulator.postman_collection.json))**: Gồm 4 request E2E kèm `pm.test(...)` scripts tự động kiểm tra tạo đơn, xác nhận IPN, Query Status và Idempotency.

---

## 📂 5. Cấu trúc Thư mục Dự án

```text
e-wallet-checkout-ux-uat/
├── wireframes/
│   ├── README.md
│   ├── index.html
│   └── user-journey-wireframes.html
├── postman/
│   └── e-wallet-payment-simulator.postman_collection.json
├── user-stories-ac.md
├── uat-test-cases.csv
└── README.md
```

---

## 🚀 6. Hướng dẫn Chạy Kiểm thử với Stage 1

1. Khởi động Backend Node.js từ **Stage 1** (`e-wallet-payment-simulator`):
   ```bash
   npm start
   # Merchant: http://localhost:3000 | Gateway: http://localhost:4000
   ```
2. Import file `postman/e-wallet-payment-simulator.postman_collection.json` vào **Postman**.
3. Chạy **Collection Runner** theo thứ tự từ Request 1 → 4 để xác nhận pass 100% các test case tạo đơn, IPN và Idempotency.
