# Changelog

## 0.25.5 - 2026-08-30

- Phân tích lại APK `2.5.5` (version code `128`) và đồng bộ báo cáo với run
  evidence ngày 29/08/2026.
- Đưa nhóm entity động cho từng loa lên Home Assistant: trạng thái, capability,
  brightness, payment, chi tiết transaction, QR, ảnh QR và notification.
- Thêm parser MQTT cho nhiều QR record, collection/columnar payload,
  `SpeakerNotificationPlayed`, dedupe và correlation theo loa.
- Thêm sensor/event cho câu thông báo nhận tiền, metadata asset âm thanh,
  transaction detail đã mask, QR metadata và ảnh QR render cục bộ.
- Giới hạn payload/record/depth/output, giữ raw QR chỉ trong RAM và siết
  diagnostics không chứa identifier hoặc dữ liệu nhạy cảm.
- Bổ sung regression tests cho QR bound/fingerprint, `DataOverflowError`,
  translation completeness, dynamic entity count và diagnostics redaction.
- CI cài `segno==1.6.6` trước khi lint/test/build validation.

## 0.2.2 - 2026-08-21

- Loại bỏ `ssl.create_default_context()` khỏi event loop khi xác minh và khởi
  động MQTT.
- Dùng SSL context đã cache/pre-warm từ `homeassistant.util.ssl`, gồm context
  strict và context legacy không xác minh certificate.
- Thêm kiểm thử hồi quy để ngăn `load_default_certs` và
  `set_default_verify_paths` bị gọi blocking trong coroutine của integration.

## 0.2.1 - 2026-08-16

- Sửa binary sensor **QR mặc định đã cấu hình** từ entity category `config`
  sang `diagnostic`. Home Assistant không cho phép binary sensor dùng category
  `config` và sẽ từ chối thêm entity.
- Thêm kiểm thử hồi quy cho entity category và dựng lại gói phát hành sạch.

## 0.2.0 - 2026-08-15

- Thêm công tắc cloud cho âm báo giao dịch trên ứng dụng Tingbox; entity chỉ
  khả dụng khi API trả trạng thái `type_receiver_tingting`.
- Thêm trạng thái QR mặc định dạng boolean, số loa, số loa hỗ trợ độ sáng và
  thời điểm cập nhật cloud gần nhất.
- Thêm entity chẩn đoán theo loa cho mô tả trạng thái, loại loa, kênh thiết bị
  và khả năng điều chỉnh độ sáng.
- Thêm nút làm mới dữ liệu cloud và giữ nguyên redaction cho QR, serial, MQTT,
  ngân hàng và dữ liệu định danh.
- Chuẩn hóa chuỗi boolean như `"false"` khi rút gọn trạng thái QR và giữ giá trị
  âm báo hợp lệ gần nhất nếu một lần refresh tạm thời thiếu field cloud.
- Viết lại README đầy đủ với nút cài HACS và đồng bộ metadata sang repository
  `trankhanhduy2929-beep/tingbox-cloud-home-assistant`.

## 0.1.0 - 2026-08-14

- Thêm config flow tài khoản/mật khẩu và reauthentication.
- Thêm REST polling cho danh sách loa, trạng thái, tổng tiền, số giao dịch và
  chế độ cloud.
- Thêm MQTT v5/TLS cho sự kiện giao dịch đã rút gọn dữ liệu.
- Thêm điều khiển độ sáng màn hình mức `1..7` cho thiết bị hỗ trợ.
- Thêm diagnostics với redaction nghiêm ngặt và tùy chọn TLS MQTT cũ có xác
  nhận rõ ràng.
