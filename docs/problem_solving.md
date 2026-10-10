# Phân tích 12 tình huống lỗi mạng

Các tình huống dưới đây được phân tích **theo giả định** để làm rõ triệu chứng, nguyên nhân có thể xảy ra, phép kiểm tra chẩn đoán và hướng khắc phục. Chưa có bằng chứng cho thấy cả 12 tình huống đã được chủ động tạo lỗi và kiểm thử trên mô hình. Kết quả dự kiến không được xem là kết quả thực nghiệm.

## 2.1 — Sai default gateway
**Điều kiện giả định:** hostPod1A đặt gateway 172.16.255.253 thay vì 172.16.255.254.
**Triệu chứng có thể xuất hiện:** Ping hostPod1B trong cùng LAN vẫn có thể thành công; ping Eagle Server sẽ thất bại nếu .253 không phải router hợp lệ.
**Cơ chế:** IP/mask của host có thể đúng nhưng next hop để đi ra ngoài mạng không tồn tại/ARP thất bại.
**Cách xác định:** `ipconfig` tại A → `ping 172.16.1.2` → `ping 172.16.255.254` → `ping 192.168.254.254`; quan sát ARP trong Simulation nếu cần.
**Hướng khắc phục:** Sửa default gateway của A về 172.16.255.254.
**Tiêu chí xác minh sau khắc phục:** ping gateway và Eagle thành công nếu không có lỗi khác; đối chiếu `ipconfig` và kết quả ping.

## 2.2 — Sai subnet mask
**Điều kiện giả định:** hostPod1B dùng 255.255.255.0 thay vì 255.255.0.0.
**Triệu chứng có thể xuất hiện:** Với IP 172.16.1.2/24, gateway 172.16.255.254 nằm ngoài subnet mà host nhận diện. Giao tiếp ra gateway/remote có thể lỗi, hoặc giao diện cấu hình từ chối gateway off-link; host cùng 172.16.1.0/24 vẫn có thể trao đổi.
**Cơ chế:** Mask /24 phân loại local/remote sai so với LAN thiết kế /16.
**Cách xác định:** `ipconfig` B → so sánh mạng của B/gateway → `ping 172.16.255.254` → kiểm tra A và R2.
**Hướng khắc phục:** Sửa mask của B thành 255.255.0.0; giữ gateway 172.16.255.254.
**Tiêu chí xác minh sau khắc phục:** Gateway trở thành on-link; ping gateway/remote có thể Pass nếu mạng còn lại đúng; đối chiếu trạng thái interface và kết quả ping.

## 2.3 — Interface administratively down
**Điều kiện giả định:** R2 FastEthernet0/0 chưa được `no shutdown`.
**Triệu chứng có thể xuất hiện:** R2 báo `administratively down/down`; hostPod1A và B vẫn có thể liên lạc qua S1; ping gateway hoặc server thất bại.
**Cơ chế:** Cổng router LAN bị khóa ở mức quản trị, không tham gia truyền gói.
**Cách xác định:** R2: `show ip interface brief` → `show running-config`; PC: `ping 172.16.1.2`, `ping 172.16.255.254`, `tracert 192.168.254.254`.
**Hướng khắc phục:** R2 `interface fastethernet0/0` → `no shutdown`.
**Tiêu chí xác minh sau khắc phục:** Sau khi link lên up/up, chạy lại ping gateway và server; so sánh trạng thái interface trước và sau điều chỉnh.

## 2.4 — Sai WAN mask
**Điều kiện giả định:** R1 Serial0/0/0 là /24 trong khi R2 là /30.
**Triệu chứng có thể xuất hiện:** Không bắt buộc tất cả ping thất bại: 10.10.10.5 và 10.10.10.6 vẫn thuộc cùng mạng theo cách tính của cả hai đầu. Tuy nhiên mạng connected trên R1 bị mở rộng thành 10.10.10.0/24, có nguy cơ định tuyến sai tới IP khác.
**Cơ chế:** Hai đầu WAN nhận diện phạm vi subnet khác nhau; có thể gây định tuyến bất nhất hoặc overlap.
**Cách xác định:** `show ip interface brief` → `show running-config` tại hai đầu → `show ip route` → ping các IP WAN và end-to-end.
**Hướng khắc phục:** R1 đặt lại `ip address 10.10.10.6 255.255.255.252`.
**Tiêu chí xác minh sau khắc phục:** Hai router thể hiện connected `10.10.10.4/30`, bảng định tuyến và phép ping WAN/đầu cuối được xác nhận lại.

## 2.5 — Thiếu clock rate
**Điều kiện giả định:** R1 (DCE) chưa được cấu hình clock rate.
**Triệu chứng có thể xuất hiện:** Liên kết serial có thể có vấn đề line protocol, không nhất thiết luôn cùng triệu chứng trong mọi mô phỏng.
**Cơ chế:** Phía DCE chịu trách nhiệm cung cấp clock; không có clock phù hợp thì liên kết đồng bộ có thể không hoạt động.
**Cách xác định:** Kiểm tra DCE/DTE qua `show controllers serial 0/0/0` nếu hỗ trợ; `show ip interface brief`, `show interfaces serial 0/0/0`; kiểm tra cáp, đầu đối diện.
**Hướng khắc phục:** Trên đúng interface DCE của R1 cấu hình `clock rate 2000000`, giữ `no shutdown`.
**Tiêu chí xác minh sau khắc phục:** R1/R2 serial đạt trạng thái phù hợp; kiểm tra lại ping WAN và ping đầu cuối.

## 2.6 — Thiếu return route
**Điều kiện giả định:** R2 có route tới Eagle nhưng R1 không có route về 172.16.0.0/16.
**Triệu chứng có thể xuất hiện:** ICMP Echo Request có thể tới server; Echo Reply không thể quay về host, dẫn tới timeout.
**Cơ chế:** Định tuyến IP yêu cầu cả chiều đi và chiều về hoạt động.
**Cách xác định:** R2 và R1: `show ip route`; kiểm tra `ping 192.168.254.254` và thử ping ngược từ Eagle; quan sát Simulation nếu có.
**Hướng khắc phục:** R1: `ip route 172.16.0.0 255.255.0.0 10.10.10.5`.
**Tiêu chí xác minh sau khắc phục:** R1 có static route đến LAN Pod và ping end-to-end thành công nếu không còn lỗi khác.

## 2.7 — Địa chỉ trùng lặp
**Điều kiện giả định:** hostPod1A và hostPod1B dùng cùng một IPv4.
**Triệu chứng có thể xuất hiện:** ARP có thể ánh xạ IP sang MAC không ổn định, gói gửi nhầm thiết bị hoặc mất liên lạc; kết quả phụ thuộc cách Packet Tracer mô phỏng duplicate IP.
**Cơ chế:** Hai NIC dùng trùng địa chỉ trong cùng broadcast domain.
**Cách xác định:** So sánh `ipconfig` của cả hai host → kiểm tra ARP cache/Simulation và thử ping → đối chiếu MAC của từng PC.
**Hướng khắc phục:** Trả B về 172.16.1.2 và A về 172.16.1.1; làm mới ARP nếu cần.
**Tiêu chí xác minh sau khắc phục:** Hai host có địa chỉ riêng, ping qua lại ổn định; kiểm chứng bằng `ipconfig` và `ping`.

## 2.8 — Ping được nhưng tracert timeout
**Điều kiện giả định:** Một hop không gửi phản hồi ICMP Time Exceeded.
**Triệu chứng có thể xuất hiện:** `ping` tới đích vẫn có thể thành công; `tracert` xuất hiện `*` ở một hay nhiều hop nhưng các hop sau vẫn có thể phản hồi.
**Cơ chế:** Một router có thể không phản hồi ICMP TTL exceeded hoặc phản hồi quá thời hạn, trong khi vẫn chuyển tiếp dữ liệu bình thường.
**Cách xác định:** Đối chiếu `ping` đích và `tracert` đích; kiểm tra interface/route; quan sát gói tin nếu hỗ trợ.
**Hướng khắc phục:** Chỉ sửa khi tìm thấy lỗi thật; nếu chỉ có hop không phản hồi thì ghi nhận giới hạn quan sát, không kết luận mất mạng.
**Tiêu chí xác minh sau khắc phục:** Ping vẫn Pass; tracert được diễn giải đúng và đối chiếu các hop được hiển thị nếu dấu `*` thực sự xuất hiện.

## 2.9 — Sai cáp/port
**Điều kiện giả định:** Cáp giữa S1 và R2 bị rút hoặc nối sai port.
**Triệu chứng có thể xuất hiện:** Có thể xuất hiện link down; host cùng switch vẫn có thể ping nhau, nhưng mất gateway/remote nếu uplink hỏng.
**Cơ chế:** Kết nối vật lý hoặc mapping cổng không khớp; switch/router thiếu link cần thiết.
**Cách xác định:** Quan sát dây/đèn link → R2 `show ip interface brief` → S1 `show interfaces status` nếu hỗ trợ → kiểm tra port mapping → ping local/gateway.
**Hướng khắc phục:** Cắm lại đúng R2 Fa0/0 ↔ S1 Fa0/24 và đúng loại cáp; chờ link lên.
**Tiêu chí xác minh sau khắc phục:** R2 Fa0/0 up/up và ping gateway thành công; trạng thái kết nối và thông tin CLI.

## 2.10 — Giới hạn traceroute
**Điều kiện giả định:** So sánh lệnh `tracert` thông thường và `tracert -w 5 -h 4`.
**Triệu chứng có thể xuất hiện:** `-h 4` giới hạn tối đa 4 hop; `-w 5` trên Windows chỉ chờ 5 mili giây mỗi lần, có thể tăng dấu `*` giả tạo.
**Cơ chế:** Thời hạn chờ quá ngắn hoặc TTL tối đa thấp làm traceroute kết thúc/timeout trước khi hiển thị đủ đường đi.
**Cách xác định:** Chạy `tracert 192.168.254.254`, sau đó thử `tracert -w 5 -h 4 192.168.254.254` nếu PT hỗ trợ; so sánh hop/timeout.
**Hướng khắc phục:** Sử dụng giới hạn phù hợp mục đích chẩn đoán; không cần sửa routing nếu ping vẫn hoạt động.
**Tiêu chí xác minh sau khắc phục:** Giải thích khác biệt thực tế; nếu Packet Tracer không hỗ trợ tùy chọn, kết quả được ghi nhận là giới hạn của công cụ, không suy ra đầu ra giả định.

## 2.11 — Server gateway sai
**Điều kiện giả định:** Eagle Server đổi gateway từ 192.168.254.253 sang 192.168.254.252.
**Triệu chứng có thể xuất hiện:** R1 có thể ping trực tiếp Eagle cùng LAN, nhưng Echo Reply về hostPod1A bị gửi tới gateway sai và thất bại.
**Cơ chế:** Server không có next hop hợp lệ để trở về 172.16.0.0/16.
**Cách xác định:** Server `ipconfig` → ping R1 Fa0/0 → ping hostPod1A; đối chiếu route và Simulation.
**Hướng khắc phục:** Khôi phục gateway của Eagle thành 192.168.254.253.
**Tiêu chí xác minh sau khắc phục:** Ping từ hostPod1A tới Eagle hoạt động lại khi toàn mạng đúng.

## 2.12 — Chẩn đoán nhiều tầng
**Điều kiện giả định:** PC không ping được Eagle Server, chưa rõ nguyên nhân.
**Triệu chứng có thể xuất hiện:** Ping đích timeout là triệu chứng tổng hợp, không chỉ ra ngay tầng lỗi.
**Cơ chế:** Có thể do PC IP/mask/GW, L1/L2, router LAN, WAN clock, static routing hoặc cấu hình server.
**Cách xác định:** Cây quyết định: (1) `ipconfig` kiểm IP/mask/GW → (2) ping host cùng LAN → (3) ping gateway R2 → (4) kiểm R2 Fa0/0 và Serial qua `show ip interface brief` → (5) ping 10.10.10.6 → (6) đối chiếu `show ip route` cả R2/R1 → (7) kiểm Eagle IP/GW và ping Eagle → (8) `tracert`/Simulation tìm hop cuối phản hồi.
**Hướng khắc phục:** Chỉ sửa nút thực sự sai sau khi xác định bằng phép kiểm tra; khôi phục địa chỉ/cáp/interface/route liên quan.
**Tiêu chí xác minh sau khắc phục:** Chạy lại T1–T7 để xác minh. Không nhận lỗi đơn lẻ làm nguyên nhân khi chưa kiểm tra các tầng khác.
