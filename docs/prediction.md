# Dự đoán trước khi kiểm thử

## 1. Phân đoạn mạng

- **LAN trung tâm:** `172.16.0.0/16`, gateway R2-Central Fa0/0 là `172.16.255.254`. `hostPod1A` và `hostPod1B` cùng mạng.
- **WAN Serial:** `10.10.10.4/30`, hai địa chỉ host là `10.10.10.5` và `10.10.10.6`; broadcast là `10.10.10.7`.
- **LAN Eagle:** `192.168.254.0/24`, gateway R1-ISP Fa0/0 là `192.168.254.253`; Eagle Server mang IP `192.168.254.254`.

## 2. Dự đoán đường đi

**Từ hostPod1A đến hostPod1B:** Hai thiết bị cùng subnet, có thể giao tiếp trực tiếp qua S1-Central mà không đi qua router. Khi cần, hostPod1A dùng ARP để xác định MAC của hostPod1B.

**Từ hostPod1A đến Eagle Server:** hostPod1A xác định đích ở ngoài subnet và gửi frame tới MAC của default gateway R2-Central (sau khi ARP nếu chưa có cache). R2 chuyển gói qua liên kết Serial đến R1; R1 chuyển tiếp trên LAN Eagle. Echo Reply từ Eagle Server trở về qua R1 và R2 nhờ tuyến tĩnh chiều về.

**Traceroute:** Dự kiến có ba điểm phản hồi: R2-Central (`172.16.255.254`), R1-ISP (địa chỉ giao diện trả lời phụ thuộc IOS và mô phỏng), sau đó Eagle Server (`192.168.254.254`).

Hai tuyến tĩnh trên R1 và R2 là điều kiện để lưu lượng đi và về giữa hai mạng LAN. Thiếu một chiều có thể làm phép ping đầu cuối thất bại.

## 3. Đối chiếu với thực nghiệm

Kết quả `tracert 192.168.254.254` đã ghi nhận ba hop: `172.16.255.254 → 10.10.10.6 → 192.168.254.254`. Ping từ hostPod1A đến Eagle Server đạt 3/4 phản hồi ở lần đầu và 4/4 ở lần kiểm tra lại.

Đây là **kết quả quan sát sau triển khai**, tách biệt với các dự đoán ở mục 1–2. Một gói không phản hồi trong lần thử đầu chưa đủ để kết luận nguyên nhân là ARP; thứ tự sự kiện và thời gian chờ trong Packet Tracer còn phụ thuộc trạng thái mô phỏng và cache.
