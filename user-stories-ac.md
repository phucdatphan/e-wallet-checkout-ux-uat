# User Stories & Acceptance Criteria (AI-Assisted Agile Spec)

> **Dự án**: E-Wallet Checkout UX, Spec & UAT Suite (MoMo & ZaloPay)  
> **Nguồn chuyển đổi**: `docs/BUSINESS_ANALYSIS.md` (Project 1)  
> **Chuẩn viết**: INVEST + Gherkin (`Given - When - Then`)

---

## US-01: Chọn phương thức thanh toán Ví điện tử tại Checkout
**As a** Khách hàng (End-user)  
**I want** chọn thanh toán bằng Ví MoMo hoặc Ví ZaloPay trên trang Checkout  
**So that** tôi có thể thanh toán nhanh chóng không cần nhập số thẻ ngân hàng.

- **Acceptance Criteria:**
  - **AC1 (Hiển thị lựa chọn)**:
    - **Given** khách hàng đang ở trang Checkout (`GET /`) với giỏ hàng hợp lệ (`amount > 0`).
    - **When** trang tải hoàn tất.
    - **Then** hệ thống hiển thị 2 lựa chọn ví: "Ví MoMo" và "Ví ZaloPay" cùng nút "Thanh toán ngay".
  - **AC2 (Chặn khi chưa chọn ví)**:
    - **Given** khách hàng chưa chọn phương thức thanh toán nào.
    - **When** khách hàng bấm "Thanh toán ngay".
    - **Then** hệ thống hiển thị thông báo lỗi "Vui lòng chọn phương thức thanh toán" và không gọi API `POST /api/payment/create`.

---

## US-02: Khởi tạo đơn hàng & Khóa tồn kho 15 phút (15-Minute Reservation)
**As a** Hệ thống Merchant  
**I want** tạo mã đơn hàng duy nhất (`appTransId`), đặt trạng thái `PENDING` và giữ tồn kho 15 phút  
**So that** tránh tình trạng bán quá số lượng (overselling) trong lúc khách đang quét QR.

- **Acceptance Criteria:**
  - **AC1 (Tạo đơn thành công)**:
    - **Given** khách hàng đã chọn ví (MoMo/ZaloPay) và bấm "Thanh toán ngay".
    - **When** hệ thống gọi `POST /api/payment/create` và chuyển tiếp sang Gateway `POST /v2/gateway/pay/create` kèm chữ ký `HMAC-SHA256`.
    - **Then** đơn hàng được lưu với `status = PENDING`, `expiredAt = createdAt + 15 phút`, tồn kho bị khóa tạm thời (Hold Stock), và trả về `payUrl` / `qrCode`.
  - **AC2 (Chống bấm đúp - Double Submit)**:
    - **Given** khách hàng bấm nút "Thanh toán ngay" 2 lần liên tiếp trong 1 giây cùng `idempotencyKey`.
    - **When** hệ thống nhận request thứ 2.
    - **Then** nút thanh toán bị disable ở UI ngay sau lần bấm đầu; ở backend, request thứ 2 trả về đúng `payUrl` của đơn đã tạo, không sinh đơn hàng thứ 2.

---

## US-03: Hiển thị màn hình QR động & Đồng hồ đếm ngược 15 phút
**As a** Khách hàng  
**I want** nhìn thấy mã QR động và thời gian đếm ngược 15:00  
**So that** tôi biết chính xác mình còn bao nhiêu thời gian để hoàn tất thanh toán.

- **Acceptance Criteria:**
  - **AC1 (Hiển thị QR & Timer)**:
    - **Given** Gateway trả về `payUrl` và `qrCode` hợp lệ (`GET /checkout?transId=...`).
    - **When** màn hình thanh toán mở ra.
    - **Then** mã QR SVG hiển thị rõ nét, số tiền khớp với đơn hàng, và đồng hồ đếm ngược chạy từ `15:00` về `00:00`.
  - **AC2 (Deep-link trên Mobile)**:
    - **Given** khách hàng truy cập bằng thiết bị di động.
    - **When** khách hàng bấm nút "Mở App Ví".
    - **Then** hệ thống kích hoạt deep-link mở trực tiếp ứng dụng MoMo hoặc ZaloPay tương ứng.

---

## US-04: Xử lý IPN Webhook & Xác thực bảo mật 3 lớp
**As a** Hệ thống Merchant Backend  
**I want** nhận và xác thực IPN Webhook (`POST /api/payment/callback`) từ Gateway theo 3 lớp bảo mật  
**So that** chỉ những giao dịch hợp lệ, chưa xử lý và đúng số tiền mới được chốt đơn.

- **Acceptance Criteria:**
  - **AC1 (Lớp 1 - Sai chữ ký HMAC-SHA256)**:
    - **Given** Gateway hoặc kẻ tấn công gửi IPN với `signature` không khớp `HMAC-SHA256(rawData, secretKey)`.
    - **When** Merchant nhận request tại `POST /api/payment/callback`.
    - **Then** hệ thống từ chối xử lý, trả về HTTP `400/403` và ghi log cảnh báo bảo mật.
  - **AC2 (Lớp 3 - Lệch số tiền / Amount Tampering)**:
    - **Given** chữ ký hợp lệ nhưng `ipn.amount !== order.amount`.
    - **When** Merchant kiểm tra đối soát số tiền.
    - **Then** hệ thống không chốt đơn `PAID`, đánh dấu đơn cần kiểm tra (`DISCREPANCY_FLAG`) và trả lỗi.
  - **AC3 (Chốt đơn thành công)**:
    - **Given** chữ ký hợp lệ, `ipn.amount === order.amount`, và `resultCode === 0`.
    - **When** Merchant xử lý IPN.
    - **Then** đơn hàng chuyển sang `PAID`, kích hoạt xuất kho và trả về HTTP `200 { returnCode: 0 }`.

---

## US-05: Đảm bảo Idempotency khi Gateway Retry IPN nhiều lần
**As a** Hệ thống Merchant Backend  
**I want** bỏ qua các IPN trùng lặp bằng khóa `ipn_{transId}_{resultCode}`  
**So that** đơn hàng không bị xuất kho 2 lần hoặc cộng điểm 2 lần khi Gateway retry.

- **Acceptance Criteria:**
  - **AC1 (IPN trùng lặp)**:
    - **Given** đơn hàng `#ORD-001` đã được xử lý thành công ở lần nhận IPN đầu tiên.
    - **When** Gateway gửi lại cùng một payload IPN lần thứ 2 và thứ 3 (do timeout mạng trước đó).
    - **Then** hệ thống nhận diện `idempotencyKey` đã tồn tại, KHÔNG chạy lại logic trừ kho/gửi email, và trả về ngay `HTTP 200 { returnCode: 0, message: "Idempotent OK" }`.

---

## US-06: Hiển thị kết quả Thanh toán Thành công (PAID)
**As a** Khách hàng  
**I want** thấy màn hình xác nhận thanh toán thành công ngay sau khi quét QR  
**So that** tôi yên tâm đơn hàng đã được ghi nhận.

- **Acceptance Criteria:**
  - **AC1 (Chuyển hướng sau IPN)**:
    - **Given** khách hàng vừa xác nhận thanh toán trên ví giả lập (`POST /v2/gateway/pay/confirm`).
    - **When** Merchant nhận IPN `resultCode = 0`.
    - **Then** màn hình chuyển sang `GET /order-result?status=PAID`, hiển thị mã giao dịch, số tiền và nút "Xem chi tiết đơn hàng".

---

## US-07: Tự động hết hạn sau 15 phút & Nhả tồn kho (Timeout / EXPIRED)
**As a** Hệ thống Merchant  
**I want** tự động chuyển đơn hàng `PENDING` quá 15 phút sang `EXPIRED` và mở khóa tồn kho  
**So that** sản phẩm không bị giữ chỗ vô thời hạn bởi các đơn bỏ dở.

- **Acceptance Criteria:**
  - **AC1 (Hết hạn trên UI & Backend)**:
    - **Given** đơn hàng đang ở trạng thái `PENDING` và đồng hồ đếm ngược về `00:00`.
    - **When** quá 15 phút mà chưa có IPN thành công.
    - **Then** đơn hàng chuyển sang `EXPIRED`, tồn kho được giải phóng (Release Stock), và UI hiển thị màn hình Timeout kèm nút "Đặt lại đơn hàng".

---

## US-08: Xử lý Timeout Refund khi khách thanh toán sát giờ hết hạn (14:59)
**As a** Bộ phận CSKH & Kế toán  
**I want** hệ thống phát hiện giao dịch thanh toán thành công nhưng đơn hàng đã `EXPIRED`  
**So that** khách hàng được hoàn tiền kịp thời và không xảy ra khiếu nại.

- **Acceptance Criteria:**
  - **AC1 (Late IPN sau khi Expired)**:
    - **Given** đơn hàng đã chuyển sang `EXPIRED` lúc `15:00` và tồn kho đã nhả.
    - **When** Merchant nhận được IPN `resultCode = 0` đến trễ lúc `15:02`.
    - **Then** hệ thống KHÔNG chuyển đơn sang `PAID`, gắn cờ `DISCREPANCY_FLAG` (hoặc `REFUND_REQUIRED`), và kích hoạt quy trình hoàn tiền tự động / tạo ticket CSKH.

---

## US-09: Người dùng chủ động Hủy giao dịch (FAILED / CANCELLED)
**As a** Khách hàng  
**I want** bấm nút "Hủy giao dịch" trên màn hình QR  
**So that** tôi có thể đổi sang phương thức thanh toán khác nếu muốn.

- **Acceptance Criteria:**
  - **AC1 (Hủy đơn chủ động)**:
    - **Given** khách hàng đang ở màn hình QR (`PENDING`).
    - **When** khách hàng bấm "Hủy giao dịch" và xác nhận.
    - **Then** đơn hàng chuyển sang `FAILED`, tồn kho được mở khóa ngay lập tức, và hiển thị màn hình Hủy đơn với nút "Thử lại với ví khác".

---

## US-10: Chủ động Query Order Status khi mạng rớt / IPN chậm (Polling)
**As a** Khách hàng  
**I want** hệ thống tự động kiểm tra lại trạng thái giao dịch khi IPN về chậm  
**So that** tôi không bị báo lỗi oan dù tiền đã bị trừ trong ví.

- **Acceptance Criteria:**
  - **AC1 (Polling khi chưa có IPN)**:
    - **Given** khách hàng đã thanh toán trên app ví nhưng mạng chập chờn khiến IPN chưa về Merchant sau 5 giây.
    - **When** frontend hiển thị trạng thái "Đang xác nhận giao dịch...".
    - **Then** hệ thống tự động gọi `POST /v2/gateway/pay/query` mỗi 5 giây (tối đa 6 lần) để đồng bộ trạng thái `PAID` về cho khách hàng.
