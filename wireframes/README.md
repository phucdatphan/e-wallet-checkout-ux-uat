# Wireframes & User Journey — E-Wallet Checkout

## User Journey Flow
```
[1. Giỏ hàng & Thanh toán] ──(Bấm Thanh toán)──> [2. Chọn Ví (MoMo / ZaloPay)]
                                                          │
                                                (Tạo đơn & Khóa kho 15p)
                                                          ▼
[4a. Thành công (PAID)] <──(IPN Code 0)── [3. Màn hình QR + Đếm ngược 15:00]
[4b. Hết hạn (EXPIRED)] <──(Timeout 15p)──┤
[4c. Hủy / Thất bại]    <──(User Hủy)─────┘
```

## Danh sách 5 màn hình cốt lõi
1. **Screen 01 — Checkout & Payment Method Selection**: Chọn phương thức thanh toán (Ví MoMo / Ví ZaloPay), tóm tắt đơn hàng, nút "Thanh toán ngay".
2. **Screen 02 — QR Payment & 15-Minute Countdown**: Hiển thị mã QR động SVG, đồng hồ đếm ngược `14:59`, hướng dẫn quét mã, nút mở Deep-link trên Mobile.
3. **Screen 03 — Payment Success (PAID)**: Xác nhận thanh toán thành công qua IPN, mã giao dịch, thời gian, nút "Xem đơn hàng".
4. **Screen 04 — Payment Timeout / Expired (EXPIRED)**: Thông báo hết hạn 15 phút, tồn kho đã được nhả (Release Stock), nút "Đặt lại đơn hàng".
5. **Screen 05 — Payment Cancelled / Failed (FAILED)**: Thông báo giao dịch bị hủy hoặc thất bại, hỗ trợ thử lại phương thức khác.

> Mở file `user-journey-wireframes.html` bằng trình duyệt để xem bản Wireframe tương tác đầy đủ của cả 5 màn hình.
