# Báo cáo phân tích Tingbox và thiết kế Home Assistant

**Ngày báo cáo:** 30/08/2026 · **Phiên bản integration:** 0.25.5  
**Phạm vi:** APK Tingbox do người dùng cung cấp, artefact phân tích tĩnh và
protocol cloud mà tài khoản được phép truy cập.  
**Kết luận kiến trúc:** custom integration; chưa cần add-on hoặc daemon riêng.

## 1. Tóm tắt

Đầu vào là `Tingbox – Loa Quốc Dân_2.5.5.zip`, package Android
`vn.nextpay.mpos360`, version code `128`, version name `2.5.5`, Flutter/Shorebird.
SHA-256 của archive gốc:
`e5fa3a1a4e7a42120259542a54221f2c863c297a1f50d41c0b05e42a2b0f707b`.

Luồng cloud đủ rõ để chạy trực tiếp trong Home Assistant gồm REST cho đăng nhập,
cấu hình và danh sách loa, cùng MQTT v5/TLS cho push payment, QR và
`SpeakerNotificationPlayed`. Integration 0.25.5 tạo entity cấp tài khoản và
nhóm entity động cho từng loa: trạng thái/capability, giao dịch, chi tiết đã
mask, câu thông báo nhận tiền, xác nhận âm báo, QR metadata và ảnh QR.

Không sửa APK gốc. Chưa tuyên bố end-to-end trên tài khoản hoặc loa thật trong
lần này; các test integration hiện là offline/synthetic và smoke probe đã làm
trước đó không thu được payment live.

## 2. Đã xác minh từ APK và cloud

### REST và xác thực

- App-core: `https://tingbox-appcore.nextpay.vn/`.
- Đăng nhập: `POST api/auth/login`, body tối thiểu gồm `value`, `password`,
  `deviceToken` và `os`; token được gửi ở header `Authorization` không thêm
  tiền tố `Bearer`.
- Danh sách loa: `POST api/transfer-device/list-device`, body `{}`.
- Lấy cấu hình merchant/cloud: `POST Mpos360GetCauHinhByMerchant` với
  `merchantId`, `username`, `os=ANDROID`, `deviceToken`, `versionChange=2`.
- Cấu hình cloud cung cấp broker, MQTT username/password, client ID, topic,
  tổng tiền, số giao dịch và mode. Các credential chỉ tồn tại trong config entry
  và runtime, không thành entity.
- APK có màn hình giao dịch và các field transaction; integration đã có parser
  response offline bounded/redacted. Chưa bật REST polling lịch sử vì endpoint
  phân trang và semantics của dữ liệu giao dịch chưa được xác minh đủ.

### MQTT

- APK dùng package `mqtt5_client`, MQTT v5 và TLS; broker, credential, client ID
  và topic đều lấy động từ REST.
- Port được quan sát là `8883`; topic riêng được subscribe với QoS 1.
- Nhánh payment đọc `broadcast_type` và `money`; nhánh QR đọc các field như
  `homeqrcode`, `qr_type`, `qr_id`, `payment_amount`, `device_id`,
  `account_number`, `account_name` và `mobile_user`.
- APK có field `SpeakerNotificationPlayed` trong nhánh QR/notification. Static
  evidence cũng chỉ ra hai asset `NUM_RECEIVED.mp3` và `NUM_UNIT_VND.mp3` dùng
  trong luồng đọc số tiền.
- Parser integration giới hạn payload 256 KiB, tối đa 20 record, độ sâu 6 và
  không giữ raw MQTT payload. Payment, QR và notification có bộ dedupe riêng;
  fingerprint canonical thay đổi khi nội dung record thay đổi.
- Payment acknowledgement vẫn đi theo payment path; notification độc lập không
  bị nhận nhầm thành giao dịch.

### Độ sáng loa

- Loa có `isBrightness=true` được app hiển thị control.
- Đọc: `POST api/mc-device/get-info-config` với `{mcId, clientId}`, tìm
  `brightLevel`.
- Ghi: `POST api/mc-device/publish-message-config` với `{mcId, clientId,
  backlight_level}`.
- Giao diện Home Assistant dùng mức `1..7`, đổi sang wire bằng
  `backlight_level = 7 - mức_HA`.
- Smoke response trước đó có ACK nhưng thiếu `brightLevel`; integration giữ
  `unknown` thay vì đoán giá trị.

### Âm báo và giới hạn điều khiển audio

- APK gọi `Mpos360DeviceGetTypeReceiverTingTing` để đọc và
  `Mpos360DeviceUpdateTypeReceiverTingTing` để cập nhật boolean
  `type_receiver_tingting`; integration expose switch account-level cho âm báo
  trên ứng dụng điện thoại.
- APK chứa bộ asset số tiếng Việt trong
  `assets/flutter_assets/lib/app/res/sound/`, gồm `NUM_RECEIVED.mp3`,
  `NUM_UNIT_VND.mp3` và các clip số/đơn vị. Integration chỉ expose đường dẫn
  asset đã biết cùng câu `Đã nhận <amount> đồng` để automation dùng.
- Nút “Nghe thử” trong app ghép/phát audio cục bộ trên điện thoại. Chưa có
  protocol volume hoặc lệnh phát thử phần cứng đủ chắc chắn, nên không tạo
  `media_player`, volume control hay nút giả trong Home Assistant.

### QR và ảnh

- QR source có thể là text, data URL PNG/JPEG hoặc base64 ảnh; integration chỉ
  nhận dữ liệu embedded/local, không fetch URL bên ngoài.
- Raw source và bytes ảnh chỉ được giữ trong RAM để `image` entity phục vụ ảnh
  đã được Home Assistant xác thực. State, event và diagnostics chỉ chứa
  fingerprint, loại, amount, timestamp và field đã mask.
- Nếu payload là text QR, ảnh được render cục bộ bằng `segno`; lỗi overflow,
  input quá dài và output quá lớn đều bị từ chối an toàn.
- Entity ảnh có một bản cấp tài khoản và một bản cho từng loa; QR metadata cũng
  được tạo thành sensor cấp tài khoản và sensor theo loa.

## 3. Ánh xạ sang Home Assistant

### Cấp tài khoản

- Sensor tổng tiền, số giao dịch, mode, payment amount/time/type, câu thông báo,
  chi tiết payment, lịch sử MQTT tối đa 20 record, QR type/time và thời điểm
  notification.
- Sensor số loa, số loa hỗ trợ brightness và thời điểm REST refresh.
- Binary sensor kết nối MQTT, QR mặc định đã cấu hình và trạng thái âm báo gần
  nhất.
- Event entity cho `payment`, `qr` và `notification`; đồng thời phát bus event
  `tingbox_payment`, `tingbox_qr` và `tingbox_speaker_notification`.
- Image entity cho QR gần nhất, switch âm báo trên app, button refresh cloud.

### Theo từng loa

Với mỗi loa đang được API trả về, integration tự tạo và duy trì nhóm entity riêng:

- Sensor status/status text/category/channel, serial suffix và identifier hash.
- Sensor payment amount/time/type, câu loa thông báo, chi tiết transaction,
  QR type/time và thời điểm notification.
- Binary sensor brightness capability, mobile-user configured (boolean),
  other-data present (boolean) và `SpeakerNotificationPlayed`.
- Event entity cho payment, QR và notification của đúng loa.
- Image entity QR riêng của loa; number slider brightness chỉ có khi loa quảng
  cáo capability tương ứng.

Correlation ưu tiên `device_id`, `mutb_id`, serial hoặc identifier hash. Nếu tài
khoản chỉ có một loa mà MQTT không gửi reference, event được gán cho loa duy
nhất; với nhiều loa và thiếu reference, dữ liệu chỉ cập nhật cấp tài khoản để
tránh gán nhầm.

### Dữ liệu transaction an toàn

Transaction/event có thể gồm amount, currency, source, method, broadcast type,
channel, status, thời gian, fingerprint, câu thông báo, metadata asset âm thanh
và các giá trị `account_name_masked`, `account_number_masked`,
`mobile_user_masked`. Không có raw bank account, QR, token hoặc payload trong
state/event/diagnostics.

## 4. Giả thuyết và giới hạn

- Chưa bắt được payment live trong smoke probe; callback, entity và correlation
  đã được kiểm thử bằng payload tổng hợp.
- `get-info-config` có thể chỉ trả `brightLevel` khi loa online và đúng trạng
  thái màn hình; chưa gửi lệnh ghi trong reverse-engineering probe.
- Hai endpoint âm báo đã được xác định bằng static APK nhưng chưa gửi mutation
  thật trong probe; integration chỉ gọi khi người dùng chủ động thao tác switch.
- Chưa có protocol local HTTP/TCP/UDP/BLE đủ rõ để thay app cho provisioning;
  Wi-Fi/BLE/SoftAP vẫn cần ứng dụng chính thức và thao tác vật lý.
- Không có bằng chứng đáng tin cậy cho volume phần cứng hoặc publish audio từ
  Home Assistant.
- Firmware/cloud có thể thêm field mới; parser bỏ phần không hiểu thay vì lưu
  raw dữ liệu.

## 5. Artefact bằng chứng

Artefact dưới đây là evidence phân tích, không được đóng gói vào release ZIP:

- Manifest và version hiện tại:
  `/opt/apk-lab/input/tingbox/analysis/tingbox_run_20260829/apktool/base/AndroidManifest.xml`
  và `apktool/base/apktool.yml`.
- Chỉ mục chuỗi/function/reference đã làm sạch:
  `/opt/apk-lab/input/tingbox/analysis/tingbox_run_20260829/evidence/relevant_refs.txt`.
- Chuỗi binary/AOT của run hiện tại:
  `analysis/tingbox_run_20260829/evidence/libapp.strings.txt`,
  `libapp.readelf.txt` và `unflutter.log`.
- Asset âm thanh của APK:
  `/opt/apk-lab/input/tingbox/analysis/tingbox_run_20260829/apktool/base/assets/flutter_assets/lib/app/res/sound/`.
- Các dump AOT chi tiết cũ được dùng để đối chiếu protocol:
  `/opt/apk-lab/analysis/tingbox_run_20260814/unflutter252/asm/MqttManager/`,
  `HomeController/_handleMqttDataPayment@1794116085_266dd8.txt`,
  `HomeController/_handleMqttDataQR@1794116085_265a0c.txt`,
  `_onSubmitBrightness@2136035704_691654.txt` và các dump sound-setting.

Không đưa raw login response, token, MQTT credential, QR, bank, KYC, serial đầy
đủ hoặc payload giao dịch vào repository/release.

## 6. Kiểm thử và phát hành

- Unit tests standard library + fixture Home Assistant cho parser payment/QR,
  redaction, correlation, notification, translation và dynamic entity count.
- Test QR giới hạn 20 record, fingerprint content và `segno.DataOverflowError`.
- Compile/import toàn bộ module với Home Assistant `2026.2.3`.
- `scripts/validate_release.py` kiểm tra JSON, compilation, file bắt buộc và
  secret/temp artifact.
- `scripts/build_release.py` tạo ZIP deterministic cùng SHA-256; release không
  chứa APK gốc, dump phân tích, cache hoặc dữ liệu runtime.

## 7. Chính sách an toàn

- Không sửa hoặc xóa APK gốc và artefact phân tích.
- Không lưu credential, token, raw MQTT, raw QR, số tài khoản, KYC hoặc
  mobile-user nguyên bản.
- Raw QR chỉ tồn tại tạm thời trong RAM cho image entity authenticated.
- Diagnostics chỉ trả count/flags và trạng thái tổng quát, không trả identifier
  loa, QR raw hay transaction raw.
- Không publish audio, transfer/assign loa hoặc thực hiện mutation ngoài thao
  tác người dùng chủ động trong Home Assistant.
