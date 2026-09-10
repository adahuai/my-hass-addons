# Security

## Báo cáo vấn đề

Không mở issue công khai có chứa tài khoản, mật khẩu, token REST, MQTT
username/password/client ID/topic, serial đầy đủ, QR, thông tin ngân hàng, KYC,
mobile-user hoặc raw payload.

Nếu đã lỡ công khai credential, hãy xóa nội dung, đổi mật khẩu Tingbox và tải
lại integration để tạo phiên mới. Diagnostics của integration đã được rút gọn,
nhưng vẫn nên kiểm tra file trước khi chia sẻ.

## TLS MQTT legacy

Chế độ `allow_insecure_mqtt` chỉ tồn tại để tương thích broker quan sát được có
certificate hết hạn. Chế độ này tắt xác minh certificate cho MQTT; REST vẫn dùng
TLS chuẩn. Hãy tắt tùy chọn ngay khi broker có certificate hợp lệ.

