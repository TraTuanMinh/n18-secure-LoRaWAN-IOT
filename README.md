# n18-secure-LoRaWAN-IOT
# Bảo mật LoRaWAN trong Hệ thống IoT

## 1. Mô tả đề tài

Đề tài nghiên cứu và thực nghiệm về bảo mật trong mạng **LoRaWAN (Long Range Wide Area Network)** — giao thức truyền thông không dây tầm xa, năng lượng thấp, được sử dụng phổ biến trong các hệ thống IoT như đo đạc từ xa, nông nghiệp thông minh, giám sát môi trường và thành phố thông minh.

Nội dung thực hiện:

- Phân tích kiến trúc mạng LoRaWAN gồm End Device, Gateway, Network Server và Application Server.
- Phân tích cơ chế bảo mật của LoRaWAN: mã hóa AES-128, khóa `AppKey`/`NwkKey`, quy trình kích hoạt thiết bị OTAA (Over-The-Air Activation) so với ABP (Activation By Personalization), và cơ chế toàn vẹn dữ liệu bằng MIC (Message Integrity Code).
- Triển khai hệ thống thử nghiệm sử dụng **The Things Stack** làm Network Server/Application Server, kết hợp thiết bị đầu cuối chạy firmware dựa trên thư viện **MCCI Arduino LoRaWAN**.
- Đối chiếu và đánh giá hệ thống thử nghiệm theo tiêu chuẩn **OWASP IoT Security Verification Standard (ISVS)**, tập trung vào các mục liên quan đến quản lý khóa, xác thực thiết bị và mã hóa truyền tải.
- Ghi nhận kết quả thực nghiệm, ảnh minh chứng và giới hạn an toàn của quá trình thử nghiệm.

## 2. Nguồn đã sử dụng

- **The Things Stack**: đóng vai trò Network Server, xử lý join-request theo cơ chế OTAA, giải mã và định tuyến dữ liệu từ thiết bị.
- **MCCI Arduino LoRaWAN**: thư viện firmware chạy trên thiết bị đầu cuối, thực hiện mã hóa AES-128 và quy trình OTAA khi gửi dữ liệu lên Network Server.
- **OWASP ISVS**: bộ tiêu chuẩn dùng làm khung tham chiếu để rà soát và đánh giá mức độ bảo mật của hệ thống thử nghiệm.

## 3. Cách chạy

### 3.1. Phần cứng yêu cầu

- Board vi điều khiển hỗ trợ LoRa (ví dụ Arduino MKR WAN 1300/1310 hoặc module LoRa kết hợp MCU tương thích).
- Gateway LoRaWAN, hoặc sử dụng gateway ảo/kết nối tới The Things Stack Sandbox.
- Máy tính có kết nối Internet để truy cập console quản trị của The Things Stack.

### 3.2. Cài đặt Network Server (The Things Stack)

```bash
git clone --branch v3.34 https://github.com/TheThingsNetwork/lorawan-stack.git
cd lorawan-stack
docker compose up
```

### 3.3. Cấu hình thiết bị đầu cuối (Firmware)

```bash
git clone --branch master https://github.com/mcci-catena/arduino-lorawan.git
```

1. Cài thư viện vào Arduino IDE thông qua `Sketch > Include Library > Add .ZIP Library`.
2. Cấu hình `AppEUI`, `DevEUI`, `AppKey` lấy từ trang quản trị The Things Stack.
3. Nạp sketch mẫu OTAA trong thư mục `examples/` vào board.
4. Mở Serial Monitor để theo dõi quá trình Join (OTAA) và gửi dữ liệu.

### 3.4. Đánh giá theo OWASP ISVS

Rà soát hệ thống thử nghiệm theo checklist mục V3 (Communication Security) và V4 (Software Platform Security) của OWASP ISVS, tập trung vào quản lý khóa, xác thực thiết bị và mã hóa truyền tải.

## 4. Kết quả

- Thiết bị Join thành công vào Network Server bằng cơ chế OTAA; khóa phiên `AppSKey` và `NwkSKey` được sinh tự động sau quá trình bắt tay (handshake).
- Dữ liệu cảm biến được mã hóa AES-128 trước khi truyền qua sóng LoRa; xác minh bằng cách bắt gói tin và kiểm tra payload không thể đọc được ở dạng plaintext.
- Frame counter hoạt động đúng cơ chế, ngăn chặn được thử nghiệm replay gói tin cũ.
- Hệ thống đáp ứng các yêu cầu cơ bản về quản lý khóa và xác thực thiết bị theo checklist OWASP ISVS mục V3.


## 5. Giới hạn an toàn

- Thử nghiệm được thực hiện trong môi trường lab cô lập, không tấn công lên hạ tầng LoRaWAN của bên thứ ba hoặc hệ thống sản xuất thực tế.
- Không thực hiện jamming, brute-force khóa hoặc các hình thức tấn công gây gián đoạn dịch vụ trên thiết bị không thuộc quyền sở hữu của nhóm.
- Không công bố `AppKey`, `DevEUI` hoặc bất kỳ thông tin định danh thiết bị thật nào của bên thứ ba.
- Kết quả đánh giá theo OWASP ISVS chỉ mang tính chất tham khảo học thuật, không thay thế cho một audit bảo mật chính thức.
- Mọi khóa mẫu sử dụng trong tài liệu này đều là khóa giả lập cho mục đích thử nghiệm.

## 6. Tài liệu tham khảo kỹ thuật

| Tài liệu | Nguồn | Nhánh/Thẻ | Ngày truy cập | Phần sử dụng |
|---|---|---|---|---|
| [The Things Stack](https://github.com/TheThingsNetwork/lorawan-stack) | TheThingsNetwork | `v3.34` | 09/07/2026 | Mã nguồn Network Server/Application Server dùng để triển khai môi trường thử nghiệm: cấu hình Docker Compose, xử lý join-request OTAA, quản lý khóa AppKey/NwkKey |
| [MCCI Arduino LoRaWAN](https://github.com/mcci-catena/arduino-lorawan) | MCCI Corporation | `master` | 09/07/2026 | Thư viện firmware cho thiết bị đầu cuối; sketch mẫu trong `examples/` triển khai quy trình OTAA và mã hóa AES-128 khi gửi dữ liệu cảm biến |
| [OWASP ISVS](https://github.com/OWASP/IoT-Security-Verification-Standard-ISVS) | OWASP | `master` | 09/07/2026 | Checklist mục V3 (Communication Security) và V4 (Software Platform Security) dùng làm khung tham chiếu đánh giá hệ thống thử nghiệm |

