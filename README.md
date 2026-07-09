# n18-secure-LoRaWAN-IOT
# Bảo mật LoRaWAN trong Hệ thống IoT

## 1. Mô tả đề tài

Đề tài nghiên cứu và thực nghiệm về **bảo mật trong mạng LoRaWAN (Long Range Wide Area Network)** — một giao thức truyền thông không dây tầm xa, năng lượng thấp, được sử dụng phổ biến trong các hệ thống IoT (Internet of Things) như đo đạc từ xa, nông nghiệp thông minh, giám sát môi trường, thành phố thông minh...

Nội dung chính bao gồm:

- Tìm hiểu kiến trúc mạng LoRaWAN (End Device – Gateway – Network Server – Application Server).
- Phân tích cơ chế bảo mật của LoRaWAN: mã hóa AES-128, khóa `AppKey`/`NwkKey`, quy trình OTAA (Over-The-Air Activation) và ABP (Activation By Personalization), toàn vẹn dữ liệu (MIC).
- Triển khai thử nghiệm một hệ thống LoRaWAN sử dụng **The Things Stack** làm Network Server và thiết bị đầu cuối chạy thư viện **MCCI Arduino LoRaWAN**.
- Đối chiếu, đánh giá mức độ tuân thủ bảo mật của hệ thống thử nghiệm theo tiêu chuẩn **OWASP IoT Security Verification Standard (ISVS)**.
- Ghi nhận kết quả thực nghiệm, giới hạn của mô hình và các khuyến nghị an toàn.

> ⚠️ Đây là đề tài mang tính học thuật/thực nghiệm trong môi trường lab, không nhằm mục đích khai thác hoặc tấn công hệ thống thực tế.

---

## 2. Nguồn đã sử dụng

| # | Thành phần | Vai trò trong đề tài |
|---|-----------|----------------------|
| 1 | **The Things Stack** (LoRaWAN Network Server) | Đóng vai trò Network Server/Application Server, xử lý join-request (OTAA), giải mã và định tuyến dữ liệu từ thiết bị. |
| 2 | **MCCI Arduino LoRaWAN** | Thư viện firmware chạy trên thiết bị đầu cuối (End Device), thực hiện mã hóa AES-128, quy trình OTAA gửi dữ liệu lên Network Server. |
| 3 | **OWASP ISVS** | Bộ tiêu chuẩn dùng làm khung tham chiếu để đánh giá, kiểm tra mức độ bảo mật của hệ thống thử nghiệm. |

*(Chi tiết trích dẫn đầy đủ ở mục [7. Tài liệu tham khảo](#7-tài-liệu-tham-khảo))*

---

## 3. Cách chạy (Setup & Run)

### 3.1. Phần cứng yêu cầu
- Board vi điều khiển hỗ trợ LoRa (ví dụ: Arduino MKR WAN 1300/1310, hoặc module LoRa + MCU tương thích).
- Gateway LoRaWAN (hoặc dùng gateway ảo/kết nối tới The Things Stack Sandbox).
- Máy tính có kết nối Internet để truy cập The Things Stack.

### 3.2. Cài đặt Network Server (The Things Stack)
```bash
# Clone kho mã nguồn (ghi rõ branch/tag đã dùng)
git clone --branch <TÊN_BRANCH_HOẶC_TAG> https://github.com/TheThingsNetwork/lorawan-stack.git
cd lorawan-stack

# Ví dụ chạy bằng Docker Compose theo hướng dẫn chính thức của dự án
docker compose up
```
> Thay `<TÊN_BRANCH_HOẶC_TAG>` bằng branch/tag cụ thể nhóm sử dụng (ví dụ: `v3.x.x`). Ghi rõ commit hash nếu có để đảm bảo khả năng tái lập.

Hoặc sử dụng **The Things Stack Sandbox** (bản cloud miễn phí) nếu không tự triển khai server.

### 3.3. Cấu hình thiết bị đầu cuối (Firmware)
```bash
# Clone thư viện MCCI Arduino LoRaWAN (ghi rõ branch/tag đã dùng)
git clone --branch <TÊN_BRANCH_HOẶC_TAG> https://github.com/mcci-catena/arduino-lorawan.git
```
1. Cài thư viện vào Arduino IDE (`Sketch > Include Library > Add .ZIP Library`).
2. Cấu hình `AppEUI`, `DevEUI`, `AppKey` lấy từ trang quản trị The Things Stack.
3. Nạp firmware mẫu (ví dụ `otaa-halfduplex` trong thư mục `examples/`) vào board.
4. Mở Serial Monitor để theo dõi quá trình Join (OTAA) và gửi dữ liệu.

### 3.4. Kiểm thử theo OWASP ISVS
- Tham chiếu checklist tại repo **OWASP ISVS** để rà soát các mục liên quan đến: quản lý khóa, xác thực thiết bị, mã hóa truyền tải, cập nhật firmware (OTA), lưu trữ thông tin nhạy cảm.

---

## 4. Kết quả

*(Điền kết quả thực nghiệm thực tế của nhóm vào phần này, ví dụ:)*

- Thiết bị Join thành công vào Network Server bằng OTAA, khóa phiên (`AppSKey`, `NwkSKey`) được sinh tự động sau handshake.
- Dữ liệu cảm biến được mã hóa AES-128 trước khi gửi qua sóng LoRa, xác minh bằng cách bắt gói tin và kiểm tra payload không đọc được ở dạng plaintext.
- Time-to-join trung bình: `... giây`.
- Tỷ lệ gói tin gửi thành công (Packet Delivery Ratio): `... %`.
- Phát hiện/ghi nhận (nếu có) các vấn đề như: replay attack thử nghiệm bị chặn nhờ frame counter, hoặc rủi ro khi tái sử dụng `DevNonce`.

---

## 5. Ảnh minh chứng

*(Chèn ảnh chụp màn hình thực tế của nhóm vào đây, ví dụ:)*

| Mô tả | Ảnh |
|-------|-----|
| Giao diện The Things Stack – thiết bị đã Join | `![Join thành công](images/join-success.png)` |
| Serial Monitor firmware Arduino | `![Serial log](images/serial-log.png)` |
| Gói tin bắt được (Wireshark/sniffer) cho thấy payload đã mã hóa | `![Packet capture](images/packet-encrypted.png)` |
| Kết quả rà soát theo checklist OWASP ISVS | `![ISVS checklist](images/isvs-checklist.png)` |

> Lưu ảnh minh chứng vào thư mục `images/` trong repo và cập nhật đường dẫn tương ứng.

---

## 6. Giới hạn an toàn (Safety & Ethical Limitations)

- Thử nghiệm được thực hiện **trong môi trường lab/cô lập**, không thực hiện tấn công lên hạ tầng LoRaWAN của bên thứ ba hoặc hệ thống sản xuất thực tế.
- Không thực hiện jamming, brute-force khóa, hoặc các hình thức tấn công gây gián đoạn dịch vụ (DoS) trên gateway/thiết bị không thuộc quyền sở hữu của nhóm.
- Không công bố `AppKey`, `DevEUI` hoặc bất kỳ thông tin định danh thiết bị thật nào của bên thứ ba.
- Kết quả đánh giá theo OWASP ISVS chỉ mang tính chất tham khảo học thuật, **không thay thế cho một audit bảo mật chính thức**.
- Mọi khóa mẫu (`AppKey`, `NwkKey`...) dùng trong tài liệu này đều là khóa giả lập, không dùng cho hệ thống thật.

---

## 7. Tài liệu tham khảo

1. **The Things Network / TheThingsNetwork.** *lorawan-stack – Open source LoRaWAN Network Server*. GitHub repository.
   URL: https://github.com/TheThingsNetwork/lorawan-stack
   Branch/Tag đã sử dụng: `<ghi rõ branch/tag hoặc commit hash>`
   Ngày truy cập: `<DD/MM/YYYY>`
   Phần đã sử dụng: Mã nguồn Network Server/Application Server dùng để triển khai môi trường thử nghiệm (Docker Compose setup, cấu hình OTAA, quản lý join-server và khóa AppKey/NwkKey).

2. **MCCI Corporation (mcci-catena).** *arduino-lorawan – Arduino LoRaWAN library*. GitHub repository.
   URL: https://github.com/mcci-catena/arduino-lorawan
   Branch/Tag đã sử dụng: `<ghi rõ branch/tag hoặc commit hash>`
   Ngày truy cập: `<DD/MM/YYYY>`
   Phần đã sử dụng: Thư viện firmware cho thiết bị đầu cuối, mã nguồn ví dụ (`examples/`) triển khai quy trình OTAA và mã hóa AES-128 khi gửi dữ liệu cảm biến.

3. **OWASP Foundation.** *IoT Security Verification Standard (ISVS)*. GitHub repository.
   URL: https://github.com/OWASP/IoT-Security-Verification-Standard-ISVS
   Branch/Tag đã sử dụng: `<ghi rõ branch/tag hoặc commit hash, nếu áp dụng>`
   Ngày truy cập: `<DD/MM/YYYY>`
   Phần đã sử dụng: Checklist các yêu cầu bảo mật (quản lý khóa, xác thực, mã hóa truyền tải, cập nhật firmware) dùng làm khung tham chiếu để đánh giá hệ thống thử nghiệm.

---

## 8. Cấu trúc thư mục đề xuất

```
.
├── README.md
├── firmware/           # Mã nguồn firmware cho thiết bị (dựa trên MCCI Arduino LoRaWAN)
├── server-config/      # File cấu hình/docker-compose cho The Things Stack
├── images/             # Ảnh minh chứng
└── docs/               # Ghi chú đánh giá theo OWASP ISVS, log thử nghiệm
```
