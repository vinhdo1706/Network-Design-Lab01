# Xác minh kết nối và mô phỏng gói tin

## 1. Các trường hợp kiểm thử T1–T7

Kết quả thực hiện được ghi trong `data/test_matrix.csv`. Các phép thử được tiến hành trên mô hình Cisco Packet Tracer với địa chỉ đã cấu hình trong `data/addressing_table.csv`.

| Mã | Thiết bị và phép thử | Kết quả quan sát | Đánh giá |
|---|---|---|---|
| T1 | hostPod1A: `ipconfig` | IPv4 `172.16.1.1`, mask `255.255.0.0`, gateway `172.16.255.254` | PASS |
| T2 | hostPod1A: `ping 172.16.255.254` | 4/4 phản hồi, mất 0% | PASS |
| T3 | hostPod1A: `ping 10.10.10.6` | 4/4 phản hồi, mất 0% | PASS |
| T4 | hostPod1A: `ping 192.168.254.254` | Lần đầu 3/4; lần thử lại 4/4 phản hồi | PASS ở lần thử lại |
| T5 | hostPod1A: `tracert 192.168.254.254` | Hoàn tất qua ba hop | PASS |
| T6 | hostPod1A: `ping 172.16.1.2` | 4/4 phản hồi, mất 0% | PASS |
| T7 | R1-ISP và R2-Central: `show ip route` | Hai router có mạng kết nối trực tiếp và tuyến tĩnh tới mạng còn lại | PASS |

Trên T5, ba địa chỉ xuất hiện lần lượt là `172.16.255.254`, `10.10.10.6` và `192.168.254.254`. Ở T7, R1 có tuyến đến `172.16.0.0/16` qua `10.10.10.5`, R2 có tuyến đến `192.168.254.0/24` qua `10.10.10.6`.

Phép T4 cho thấy một gói không nhận phản hồi trong lần thử đầu, nhưng tất cả gói đều nhận phản hồi ở lần thử kế tiếp. Chưa xác định được nguyên nhân mất gói ban đầu; không có cơ sở quy kết chắc chắn cho ARP hoặc lỗi mạng.

## 2. Quan sát Simulation Mode

Trong phép thử ICMP từ `hostPod1A` (`172.16.1.1`) đến `Eagle Server` (`192.168.254.254`), gói tin đi qua `S1-Central → R2-Central → R1-ISP` trước khi đến thiết bị đích; phản hồi được chuyển về theo chiều ngược lại. Kết quả **Simple PDU: Successful** đã được quan sát và lưu tại `evidence/E11_ICMP_roundtrip_success.png`. Các ảnh `E11_ICMP_R1_route_lookup.png`, `E11_R1_ARP_cache_encapsulation.png` và `E11_ICMP_Eagle_Server_request_reply.png` thể hiện những bước xử lý được kiểm tra tại router và thiết bị đích.

Trong **phép thử ARP riêng** từ `hostPod1B` (`172.16.1.2`) đến gateway `R2-Central` (`172.16.255.254`), đã ghi nhận bản tin **ARP Request (opcode 0x0001)** và **ARP Reply (opcode 0x0002)**. Sau phép thử, bảng `arp -a` tại `hostPod1B` có ánh xạ động `172.16.255.254 → 00e0.f932.9d01`; ping gateway nhận 4/4 phản hồi. Các ảnh tương ứng là `evidence/E11_ARP_request_hostPod1B_gateway.png`, `evidence/E11_ARP_reply_R2_gateway_hostPod1B.png` và `evidence/E11_ARP_cache_hostPod1B_after_resolution.png`.

Về cơ chế, khi host cần địa chỉ MAC của next hop trong mạng cục bộ mà chưa có ánh xạ, ARP hỗ trợ phân giải địa chỉ; switch chuyển tiếp frame theo địa chỉ MAC, còn router tra cứu bảng định tuyến và đóng gói lại frame trên giao diện ra. TTL của gói IPv4 giảm khi được router chuyển tiếp. Sự xuất hiện hay không xuất hiện ARP Request trong một lượt mô phỏng phụ thuộc trạng thái ARP cache và cách Packet Tracer xử lý các sự kiện.

**Giới hạn minh chứng:** Các ảnh ARP của `hostPod1B` và các ảnh ICMP của `hostPod1A` được thu thập qua **những phép thử riêng**, không phải một capture liên tục của cùng một gói tin. Bằng chứng đã thu thập hỗ trợ kết luận ARP hoạt động và ICMP khứ hồi thành công trong các phép thử tương ứng; không được suy diễn rằng đã quan sát đầy đủ mọi PDU hoặc một thứ tự ARP cố định trong lượt ICMP tới Eagle Server.

## 3. Cách tái hiện phép kiểm tra

Để kiểm tra kết nối, mở **Desktop → Command Prompt** tại PC tương ứng và chạy lệnh T1–T6; tại router, mở **CLI** để chạy T7. Trong **Simulation Mode**, chọn bộ lọc ARP/ICMP, khởi tạo lưu lượng từ `hostPod1A` và dùng **Capture/Forward** để xem thứ tự chuyển tiếp.

Khi phân tích `tracert`, dấu `*` chỉ là một lần thăm dò không nhận phản hồi đúng thời hạn, không tự chứng minh đích không thể liên lạc. Nếu cấu hình tạm thời được thay đổi để thử tình huống lỗi, trạng thái ban đầu cần được khôi phục trước khi lưu mô hình.

Đối chiếu đường dẫn ảnh minh chứng với `data/evidence_manifest.csv`. Các mục chưa có tệp tương ứng chưa được xác nhận ở cấp độ chứng cứ lưu trữ.
