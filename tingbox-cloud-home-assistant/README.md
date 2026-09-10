# Tingbox Cloud cho Home Assistant

<p align="center">
  <img src="custom_components/tingbox/brand/icon.png" alt="Tingbox" width="128">
</p>

<p align="center">
  <a href="https://github.com/trankhanhduy2929-beep/tingbox-cloud-home-assistant/releases"><img src="https://img.shields.io/github/v/release/trankhanhduy2929-beep/tingbox-cloud-home-assistant?display_name=tag&sort=semver" alt="GitHub release"></a>
  <a href="https://www.hacs.xyz/"><img src="https://img.shields.io/badge/HACS-Custom-41BDF5" alt="HACS Custom"></a>
  <a href="https://www.home-assistant.io/"><img src="https://img.shields.io/badge/Home%20Assistant-2026.2%2B-18BCF2" alt="Home Assistant 2026.2+"></a>
  <a href="LICENSE"><img src="https://img.shields.io/github/license/trankhanhduy2929-beep/tingbox-cloud-home-assistant" alt="License"></a>
</p>

Custom integration **không chính thức** giúp kết nối tài khoản và loa Tingbox
với Home Assistant qua REST API và MQTT cloud của Tingbox.

> **Phiên bản:** `0.25.5`  
> **Kiểm thử tĩnh/offline:** Home Assistant `2026.2.3` ngày `30/08/2026`  
> **Repository:** <https://github.com/trankhanhduy2929-beep/tingbox-cloud-home-assistant>

## Cài đặt nhanh bằng HACS

[![Open your Home Assistant instance and open a repository inside HACS.](https://my.home-assistant.io/badges/hacs_repository.svg)](https://my.home-assistant.io/redirect/hacs_repository/?owner=trankhanhduy2929-beep&repository=tingbox-cloud-home-assistant&category=integration)

1. Bấm nút **Open in HACS** phía trên.
2. Xác nhận mở Home Assistant và thêm repository vào HACS.
3. Trong HACS, tải integration **Tingbox**.
4. Khởi động lại Home Assistant.
5. Vào **Settings → Devices & services → Add integration → Tingbox**.
6. Nhập tài khoản và mật khẩu đang dùng trên ứng dụng Tingbox.

Nếu nút tự động không hoạt động, thêm repository thủ công:

1. Mở **HACS → Integrations**.
2. Chọn menu ba chấm → **Custom repositories**.
3. Nhập repository:

   ```text
   https://github.com/trankhanhduy2929-beep/tingbox-cloud-home-assistant
   ```

4. Chọn loại **Integration**, sau đó bấm **Add**.
5. Tìm **Tingbox**, tải integration và khởi động lại Home Assistant.

## Cài đặt thủ công

1. Tải bản mới nhất tại trang
   [Releases](https://github.com/trankhanhduy2929-beep/tingbox-cloud-home-assistant/releases).
2. Giải nén gói phát hành.
3. Chép thư mục `custom_components/tingbox` vào thư mục cấu hình Home Assistant:

   ```text
   /config/custom_components/tingbox
   ```

4. Kiểm tra đường dẫn sau tồn tại:

   ```text
   /config/custom_components/tingbox/manifest.json
   ```

5. Khởi động lại Home Assistant và thêm integration **Tingbox** từ giao diện.

## Yêu cầu

- Home Assistant `2026.2.0` trở lên.
- Tài khoản Tingbox đang đăng nhập được trên ứng dụng chính thức.
- Home Assistant có kết nối Internet tới dịch vụ cloud Tingbox.
- Không cần cài hoặc cấu hình MQTT broker riêng.
- HACS chỉ cần thiết khi cài hoặc cập nhật qua HACS.

## MQTT là cloud hay local?

MQTT của integration này là **MQTT cloud**:

- Broker, cổng, username, password, client ID và topic được API Tingbox cấp động
  sau khi đăng nhập.
- Kết nối dùng MQTT v5 qua TLS; cổng thường là `8883`.
- Home Assistant kết nối trực tiếp tới broker Tingbox để nhận giao dịch.
- Integration không kết nối tới Mosquitto/local broker của Home Assistant.
- APK chưa cung cấp giao thức local ổn định để điều khiển đầy đủ loa qua LAN.

## Tính năng

- Đăng nhập bằng config flow; không cần sửa `configuration.yaml`.
- Tự tạo `device_token` riêng cho Home Assistant và tự đăng nhập lại khi phiên
  cloud hết hạn.
- Tải danh sách loa đang được gán cho tài khoản.
- Hiển thị trạng thái, mô tả trạng thái, loại và kênh của từng loa.
- Hiển thị tổng tiền, số giao dịch và chế độ cloud do API trả về.
- Theo dõi trạng thái kết nối MQTT cloud.
- Nhận giao dịch gần như tức thời qua MQTT, giữ giao dịch gần nhất và tối đa 20
  giao dịch gần đây trong RAM.
- Hiển thị chi tiết giao dịch đã che dữ liệu: thời gian, phương thức, trạng thái,
  kênh, fingerprint và các trường tên/số tài khoản/người dùng đã mask.
- Nhận/correlate QR theo tài khoản và từng loa; tạo ảnh QR cục bộ từ payload,
  không gọi URL bên ngoài và không đưa raw QR vào state/event.
- Theo dõi `SpeakerNotificationPlayed`, thời điểm xác nhận và trạng thái âm báo
  của từng loa.
- Hiển thị câu thông báo nhận tiền cùng metadata asset âm thanh trong APK; không
  giả lập lệnh phát âm thanh hay volume phần cứng chưa được chứng minh.
- Điều khiển độ sáng màn hình mức `1..7` trên loa hỗ trợ `isBrightness=true`.
- Bật/tắt âm báo giao dịch trong ứng dụng Tingbox trên điện thoại.
- Hiển thị trạng thái có/không của QR mặc định mà không lưu nội dung cấu hình.
- Nút làm mới dữ liệu cloud ngay lập tức.
- Diagnostics được rút gọn và che credential/dữ liệu nhạy cảm.

## Entity được tạo

Tên `entity_id` thực tế phụ thuộc tên thiết bị và entity registry của Home
Assistant. Hãy mở entity trong giao diện để sao chép đúng `entity_id`.

### Entity cấp tài khoản

| Platform | Tên hiển thị | Nội dung |
| --- | --- | --- |
| `binary_sensor` | Kết nối MQTT | Trạng thái kết nối tới MQTT cloud Tingbox |
| `binary_sensor` | QR mặc định đã cấu hình | Chỉ hiển thị có/không, không chứa QR |
| `sensor` | Tổng tiền hiện tại | Tổng tiền do API Tingbox trả về |
| `sensor` | Số giao dịch | Số giao dịch do API trả về |
| `sensor` | Chế độ hiện tại | Chế độ cloud hiện tại |
| `sensor` | Giao dịch gần nhất | Số tiền cuối nhận qua MQTT |
| `sensor` | Thời điểm giao dịch gần nhất | Thời điểm nhận event cuối |
| `sensor` | Loại giao dịch gần nhất | Phương thức hoặc loại broadcast |
| `sensor` | Câu thông báo nhận tiền | Ví dụ `Đã nhận 1.234.000 đồng` |
| `sensor` | Chi tiết giao dịch gần nhất | Fingerprint ở state, metadata an toàn ở attributes |
| `sensor` | Lịch sử giao dịch trong phiên | Tối đa 20 giao dịch nhận qua MQTT |
| `sensor` | QR gần nhất | Loại QR, amount và metadata đã che |
| `sensor` | Thời điểm QR gần nhất | Thời điểm nhận/cập nhật QR |
| `sensor` | Thời điểm loa xác nhận âm báo | Timestamp `SpeakerNotificationPlayed` gần nhất |
| `sensor` | Số loa được gán | Số loa hiện thuộc tài khoản |
| `sensor` | Số loa hỗ trợ độ sáng | Số loa có `isBrightness=true` |
| `sensor` | Cập nhật cloud gần nhất | Lần REST refresh thành công gần nhất |
| `event` | Nhận giao dịch | Event entity với event type `payment` |
| `event` | Nhận/cập nhật QR | Event entity với event type `qr` |
| `event` | Xác nhận âm báo loa | Event entity với event type `notification` |
| `image` | Ảnh QR gần nhất | Ảnh embedded hoặc QR được render cục bộ |
| `switch` | Âm báo giao dịch trên ứng dụng | Bật/tắt âm báo trong app điện thoại |
| `button` | Làm mới dữ liệu cloud | Yêu cầu coordinator refresh ngay |

### Entity theo từng loa

| Platform | Tên hiển thị | Nội dung |
| --- | --- | --- |
| `sensor` | Trạng thái loa | `status_code` của loa |
| `sensor` | Mô tả trạng thái | Chuỗi trạng thái do API trả về |
| `sensor` | Loại loa | Category/model thiết bị |
| `sensor` | Kênh thiết bị | `channelDescription` nếu API có trả |
| `sensor` | Đuôi số sê-ri loa | Chỉ 4 ký tự cuối để nhận diện |
| `sensor` | Mã định danh loa | Hash ổn định, không phải serial đầy đủ |
| `sensor` | Số tiền gần nhất của loa | Payment gần nhất correlate được với loa |
| `sensor` | Thời điểm giao dịch gần nhất của loa | Timestamp payment của loa |
| `sensor` | Loại giao dịch gần nhất của loa | Method/broadcast/source |
| `sensor` | Câu loa thông báo nhận tiền | Câu đọc số tiền của payment gần nhất |
| `sensor` | Chi tiết giao dịch gần nhất của loa | Fingerprint và attributes đã che |
| `sensor` | QR gần nhất của loa | Metadata QR đã che |
| `sensor` | Thời điểm QR gần nhất của loa | Timestamp QR của loa |
| `sensor` | Thời điểm loa xác nhận âm báo | Timestamp acknowledgement của loa |
| `binary_sensor` | Hỗ trợ điều chỉnh độ sáng | Capability độ sáng của loa |
| `binary_sensor` | Đã cấu hình người dùng di động | Chỉ có/không, nếu cloud trả field |
| `binary_sensor` | Có dữ liệu bổ sung | Chỉ có/không, không serialize raw object |
| `binary_sensor` | Loa đã phát âm báo | Kết quả `SpeakerNotificationPlayed` |
| `event` | Nhận giao dịch | Event `payment` riêng cho loa |
| `event` | Nhận/cập nhật QR | Event `qr` riêng cho loa |
| `event` | Xác nhận âm báo loa | Event `notification` riêng cho loa |
| `image` | Ảnh QR gần nhất của loa | Ảnh QR riêng cho loa |
| `number` | Độ sáng màn hình | Slider `1..7`, chỉ tạo trên loa hỗ trợ |

Mỗi loa được tạo nhóm entity riêng. Nếu danh sách loa thay đổi sau lần refresh,
integration tự thêm nhóm entity mới; entity của loa đã bị gỡ sẽ trở nên
unavailable.

### Dữ liệu nhận tiền, âm thanh và QR

- Payment MQTT tạo sensor account-level và sensor/event tương ứng của loa khi
  payload có thể correlate bằng `device_id`, `mutb_id` hoặc serial. Nếu tài khoản
  chỉ có một loa và payload không có reference, integration dùng loa duy nhất.
- QR image giữ raw source **chỉ trong RAM** để phục vụ entity `image` đã được
  Home Assistant xác thực. State, event và diagnostics chỉ chứa fingerprint,
  loại, amount và dữ liệu đã mask.
- Câu thông báo được tạo từ amount, ví dụ `Đã nhận 1.234.000 đồng`. Metadata asset
  phản ánh APK: `lib/app/res/sound/NUM_RECEIVED.mp3` và
  `lib/app/res/sound/NUM_UNIT_VND.mp3`; integration không đóng gói hoặc phát lại
  MP3, vì APK chưa chứng minh protocol điều khiển audio phần cứng.
- Parser transaction REST hiện chỉ dùng để kiểm thử offline schema; chưa bật
  polling lịch sử cloud khi endpoint phân trang chưa được xác minh. Lịch sử HA
  vì vậy là lịch sử MQTT trong phiên, tối đa 20 record.

## Cấu hình integration

### Thiết lập lần đầu

Config flow yêu cầu:

- **Tài khoản:** tài khoản đăng nhập Tingbox.
- **Mật khẩu:** mật khẩu tài khoản Tingbox.

Integration tự tạo device token riêng; không cần lấy token từ APK hoặc điện
thoại.

### Options

Mở **Settings → Devices & services → Tingbox → Configure**:

| Tùy chọn | Mặc định | Phạm vi/ý nghĩa |
| --- | --- | --- |
| Chu kỳ cập nhật REST | 5 phút | Từ 1 đến 60 phút |
| Cho phép TLS MQTT không xác minh chứng chỉ | Tắt | Chỉ bật khi broker Tingbox dùng certificate cũ |

MQTT vẫn nhận event liên tục; chu kỳ REST chỉ dùng để làm mới cấu hình, danh
sách loa và các sensor cloud.

## Automation mẫu

### Thông báo khi nhận giao dịch

```yaml
automation:
  - alias: "Tingbox - Thông báo nhận tiền"
    mode: queued
    triggers:
      - trigger: event
        event_type: tingbox_payment
    actions:
      - action: notify.mobile_app_dien_thoai
        data:
          title: "Tingbox"
          message: >-
            Đã nhận {{ trigger.event.data.amount }}
            {{ trigger.event.data.currency }}
```

Event bus chứa các field an toàn sau (field không có trong payload sẽ là `null`):

```yaml
amount: 100000
currency: VND
source: mqtt_payment
method: bank_transfer
broadcast_type: "1"
status: success
transaction_fingerprint: <sha256>
account_name_masked: N********
account_number_masked: ********1234
mobile_user_masked: ********5678
device_identifier: <hash>
received_at: "2026-08-30T12:00:00+00:00"
speaker_notification_played: true
announcement_text: "Đã nhận 100.000 đồng"
sound_asset: lib/app/res/sound/NUM_RECEIVED.mp3
sound_unit_asset: lib/app/res/sound/NUM_UNIT_VND.mp3
```

Event `tingbox_qr` chỉ phát metadata QR đã che và fingerprint; event
`tingbox_speaker_notification` chỉ phát kết quả xác nhận cùng timestamp.

### Điều chỉnh độ sáng

```yaml
action: number.set_value
target:
  entity_id: number.tingbox_do_sang_man_hinh
data:
  value: 5
```

Cloud dùng mức ngược với giao diện: `backlight_level = 7 - mức_HA`.

### Bật âm báo trong ứng dụng điện thoại

```yaml
action: switch.turn_on
target:
  entity_id: switch.tingbox_am_bao_giao_dich_tren_ung_dung
```

Đây là âm báo của ứng dụng Tingbox trên điện thoại, **không phải âm lượng phần
cứng của loa**.

### Làm mới dữ liệu cloud

```yaml
action: button.press
target:
  entity_id: button.tingbox_lam_moi_du_lieu_cloud
```

## Tính năng chưa hỗ trợ

| Tính năng trong app | Trạng thái |
| --- | --- |
| Provision Wi-Fi/BLE/SoftAP | Chưa hỗ trợ; cần điện thoại ở gần và thao tác vật lý trên loa |
| Cài đặt/kích hoạt SIM 4G | Chưa hỗ trợ |
| Gán hoặc chuyển nhượng loa | Không đưa vào HA để tránh thay đổi quyền sở hữu ngoài ý muốn |
| Lịch sử giao dịch REST đầy đủ | Chưa bật vì endpoint phân trang chưa được xác minh |
| Đổi QR, ngân hàng, KYC | Không đưa vào HA vì chưa có luồng mutation an toàn đã xác minh |
| Phát âm thanh/âm lượng phần cứng | APK chỉ cho thấy asset và audio demo cục bộ; chưa có protocol loa đáng tin cậy |
| Nút nghe thử loa | Không tạo nút giả để tránh nhầm audio điện thoại với lệnh test loa |

Integration tập trung vào phần cloud an toàn đã xác minh, không cam kết sao chép
100% mọi màn hình của ứng dụng chính thức.

## TLS MQTT legacy

Trong lần kiểm tra ngày **14/08/2026**, certificate của broker động do API trả
về không vượt qua xác minh CA chuẩn. Ứng dụng Android chính thức chấp nhận
certificate này, vì vậy config flow có bước xác nhận riêng.

- Mặc định integration vẫn xác minh TLS nghiêm ngặt.
- Chỉ bật **Cho phép TLS MQTT không xác minh chứng chỉ** khi config flow yêu cầu
  và bạn hiểu rủi ro.
- Tắt tùy chọn này nếu broker đã được nhà cung cấp cập nhật certificate.
- Không dùng chế độ insecure trên mạng không tin cậy.

## Xử lý lỗi

### Không tìm thấy integration sau khi cài

- Kiểm tra file
  `/config/custom_components/tingbox/manifest.json` có tồn tại.
- Khởi động lại toàn bộ Home Assistant, không chỉ reload YAML.
- Xóa cache trình duyệt hoặc tải lại trang **Devices & services**.
- Kiểm tra log để chắc chắn integration không lỗi import dependency.

### Báo sai tài khoản hoặc mật khẩu

- Thử đăng nhập cùng credential trên ứng dụng Tingbox chính thức.
- Kiểm tra khoảng trắng thừa trong tài khoản.
- Nếu vừa đổi mật khẩu, mở integration để thực hiện reauthentication.

### MQTT unavailable

- Kiểm tra Internet và sensor **Kết nối MQTT**.
- Mở Options và xem config flow có yêu cầu TLS legacy hay không.
- Không cần cài Mosquitto add-on; integration sử dụng MQTT cloud riêng.
- Bấm **Làm mới dữ liệu cloud** để lấy lại MQTT config động.

### Không có slider độ sáng

- Slider chỉ được tạo khi API trả `isBrightness=true` cho loa.
- Loa cần online và màn hình cần ở trạng thái hỗ trợ cấu hình.
- Thử làm mới cloud hoặc khởi động lại integration sau khi gán loa mới.

### Switch âm báo unavailable

Cloud phải trả ít nhất một giá trị hợp lệ cho `type_receiver_tingting`. Nếu API
chưa từng trả field này, integration để switch ở unavailable thay vì đoán sai
trạng thái. Sau khi có giá trị hợp lệ, lần refresh tạm thiếu field sẽ giữ trạng
thái gần nhất.

### Bật log debug

```yaml
logger:
  default: info
  logs:
    custom_components.tingbox: debug
    paho.mqtt.client: debug
```

Khởi động lại Home Assistant sau khi sửa cấu hình. Hãy kiểm tra và che thông tin
nhạy cảm trước khi đăng log công khai.

## Quyền riêng tư và bảo mật

- Tài khoản/mật khẩu chỉ được dùng để đăng nhập cloud Tingbox từ config entry.
- MQTT password, topic, client ID và token REST không được tạo thành entity.
- Không lưu raw MQTT payload vào coordinator hoặc diagnostics.
- Raw QR chỉ tồn tại trong RAM phục vụ image entity authenticated; không đưa raw
  QR vào state, event hoặc diagnostics.
- Không đưa số tài khoản, tên tài khoản, KYC hoặc mobile-user nguyên bản vào state;
  thông tin transaction/event chỉ dùng giá trị đã mask.
- Serial đầy đủ không được dùng làm tên entity; integration chỉ hiển thị suffix
  ngắn và identifier đã hash.
- Event `tingbox_payment` phát metadata giao dịch an toàn; event QR và
  `SpeakerNotificationPlayed` cũng không chứa raw payload.
- Không đăng diagnostics/log chưa kiểm tra vào issue công khai.

Xem thêm tại [SECURITY.md](SECURITY.md).

## Cập nhật

### Qua HACS

1. Mở **HACS → Integrations → Tingbox**.
2. Chọn **Update/Redownload** khi có phiên bản mới.
3. Khởi động lại Home Assistant.

### Thủ công

Chép đè thư mục `custom_components/tingbox` bằng bản mới rồi khởi động lại Home
Assistant. Không xóa config entry nếu chỉ nâng cấp integration.

## Gỡ cài đặt

1. Vào **Settings → Devices & services → Tingbox** và xóa config entry.
2. Gỡ Tingbox trong HACS hoặc xóa `/config/custom_components/tingbox`.
3. Khởi động lại Home Assistant.

## Phát triển và kiểm thử

```bash
ruff check .
python3 -m unittest discover -s tests -v
python3 scripts/validate_release.py
python3 scripts/build_release.py ./dist
```

Script phát hành tạo ZIP deterministic và file SHA-256; gói không chứa APK gốc,
dump reverse engineering, credential, token, QR, payload giao dịch hoặc cache.

Chi tiết protocol và bằng chứng đã rút gọn:

- [REPORT.md](REPORT.md)
- [docs/PROTOCOL.md](docs/PROTOCOL.md)
- [CHANGELOG.md](CHANGELOG.md)

## Hỗ trợ

- Báo lỗi: <https://github.com/trankhanhduy2929-beep/tingbox-cloud-home-assistant/issues>
- Bản phát hành: <https://github.com/trankhanhduy2929-beep/tingbox-cloud-home-assistant/releases>
- Repository: <https://github.com/trankhanhduy2929-beep/tingbox-cloud-home-assistant>

Khi báo lỗi, hãy ghi phiên bản Home Assistant, phiên bản integration, loại loa,
trạng thái entity liên quan và log đã che dữ liệu nhạy cảm. Không gửi tài khoản,
mật khẩu, token, MQTT credential, QR hoặc thông tin ngân hàng.

## Miễn trừ trách nhiệm

Đây là dự án cộng đồng, không phải integration chính thức của Tingbox, NextPay
hoặc Home Assistant. API cloud có thể thay đổi mà không báo trước. Bạn tự chịu
trách nhiệm khi sử dụng tùy chọn TLS legacy hoặc các entity điều khiển.

## Tài liệu tham khảo

- [HACS custom repositories](https://www.hacs.xyz/docs/use/repositories/dashboard/)
- [HACS integration publishing](https://www.hacs.xyz/docs/publish/integration/)
- [Home Assistant config flow](https://developers.home-assistant.io/docs/config_entries_config_flow_handler/)
- [Home Assistant integration structure](https://developers.home-assistant.io/docs/creating_integration_file_structure/)
