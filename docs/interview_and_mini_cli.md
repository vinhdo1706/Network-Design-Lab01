# 18 nội dung hỏi đáp về mô hình mạng và diễn giải CLI

Các nội dung được giải thích trên cơ sở mô hình hai router Cisco 2811, mạng LAN trung tâm sử dụng Pod 1 và định tuyến tĩnh. Phần mô tả cơ chế được phân biệt với kết quả kiểm thử thực tế.

## Q01 — Vai trò của router, switch và thiết bị đầu cuối
Router chuyển tiếp gói giữa các mạng IP và dùng bảng định tuyến; R1 kết nối với mạng Eagle và đường Serial WAN; R2 kết nối với LAN trung tâm và đường Serial WAN. Switch S1 chuyển frame Ethernet trong cùng LAN theo MAC. PC hostPod1A/B và Eagle là end device phát/nhận dữ liệu.

## Q02 — Lựa chọn default gateway của LAN trung tâm
Vì R2 Fa0/0 có địa chỉ 172.16.255.254/16 trong cùng mạng 172.16.0.0/16. Host gửi IP ngoài subnet cho router này để đi ra mạng khác.

## Q03 — Ý nghĩa subnet /30 trên đường Serial
Hai địa chỉ `10.10.10.5/30` và `10.10.10.6/30` **cùng thuộc một subnet**. Prefix /30 (`255.255.255.252`) xác định mạng `10.10.10.4/30`: network 10.10.10.4, host dùng được 10.10.10.5 và 10.10.10.6, broadcast 10.10.10.7.

## Q04 — Phân biệt DCE và DTE
DCE tạo hoặc cung cấp tín hiệu định thời cho serial synchronous; DTE nhận đồng hồ. `clock rate 2000000` được cấu hình ở Serial0/0/0 của R1-ISP (đầu DCE); cổng DTE trên R2-Central không cần cấu hình clock rate.

## Q05 — Đọc trạng thái interface
`up/up`: interface vật lý và line protocol hoạt động; `down/down`: không có kết nối vật lý thích hợp hoặc thiết bị/link đầu kia chưa sẵn sàng; `administratively down/down`: interface đang bị tắt bằng cấu hình; `up/down`: vật lý có tín hiệu nhưng giao thức lớp liên kết gặp vấn đề.

## Q06 — Điều kiện để liên lạc end-to-end
Còn cần đúng gateway, link/cáp, trạng thái interface, Serial DCE clock, route tới mạng đích và route chiều về, cũng như cấu hình server. Chỉ IP đúng không đủ chứng minh end-to-end.

## Q07 — Định tuyến tĩnh và OSPF
Static route được cấu hình thủ công; đơn giản và dễ kiểm soát trong topo 2 router. OSPF là giao thức định tuyến động trao đổi thông tin và tự cập nhật route khi topology thay đổi. Mô hình hai router sử dụng static routing, phù hợp với quy mô nhỏ và các tuyến đường đi đã xác định.

## Q08 — Đọc bảng định tuyến
`show ip route` hiển thị mạng đích, next hop/exit interface và nguồn tuyến. `C` = connected, `L` = local route đến IP của interface khi IOS hiển thị, `S` = static, `O` = OSPF (bài không dùng). Dòng route chỉ được cài khi điều kiện liên quan hợp lệ.

## Q09 — Các mức kiểm thử bằng ping
Ping local IP kiểm stack TCP/IP của máy, không chứng minh cáp hoạt động; ping gateway kiểm liên lạc trong LAN tới router; ping remote IP kiểm đường truyền qua nhiều router và cả đường về.

## Q10 — ICMP Echo Request và Echo Reply
ICMP Echo Request là gói yêu cầu phản hồi để thử kết nối. Đích có thể gửi Echo Reply nếu nhận và được phép phản hồi; timeout không tự chứng minh đích đã tắt vì còn nhiều nguyên nhân khác.

## Q11 — TTL và cơ chế traceroute
TTL giảm mỗi lần qua router nhằm tránh gói IP tồn tại mãi khi định tuyến vòng. Tracert tăng TTL dần để nhận ICMP Time Exceeded từ các hop trung gian, từ đó suy ra đường đi.

## Q12 — ICMP Time Exceeded và timeout
ICMP Time Exceeded là thông điệp router tạo khi TTL hết (giúp nhận diện hop). `Request timed out` là thông báo phía công cụ do hết thời gian chờ, không phải luôn là một gói ICMP có tên như vậy.

## Q13 — Ý nghĩa dấu sao trong traceroute
Dấu `*` biểu thị probe của traceroute không nhận được phản hồi trong thời hạn. Không thể kết luận chỉ vì `*` mà router ngừng chuyển tiếp hoặc đích không thể kết nối; cần xét các hop sau và ping đích.

## Q14 — Tham số giới hạn trong tracert
`-w 5` trên Windows đặt thời gian chờ mỗi phản hồi là 5 mili giây; `-h 4` giới hạn số hop tối đa là 4. Quá ít thời gian/hop có thể gây timeout hoặc kết thúc chưa tới đích. Việc hỗ trợ các tham số này trong Packet Tracer tùy thuộc phiên bản mô phỏng.

## Q15 — Quy trình khoanh vùng lỗi kết nối xa
Sau khi ping gateway thành công, kiểm tra R2 Serial, ping R1 `10.10.10.6`, `show ip route` tại R2/R1, ping hoặc IP/gateway của Eagle và route chiều về; dùng `tracert` để khoanh vùng.

## Q16 — Cổng bị tắt quản trị
`shutdown` chủ động tắt interface nên cho dù IP có đúng, interface vẫn `administratively down`. Trong `interface serial0/0/0`, sửa bằng `no shutdown`, đồng thời kiểm tra IP/mask và clock ở DCE; xác minh `show ip interface brief`.

## Q17 — Trạng thái up/down
`fa0/0 up/down` có nghĩa tín hiệu vật lý có nhưng line protocol không hoạt động. Kiểm tra trạng thái đầu bên kia, cáp/port, lỗi lớp liên kết và cấu hình tương thích; dùng `show interfaces fastethernet0/0` và quan sát kết nối. Không đồng nhất `up/down` với `administratively down`.

## Q18 — Đường đi của lưu lượng từ hostPod1A đến Eagle Server
Host nhận thấy IP Eagle khác subnet nên ARP lấy MAC gateway R2 nếu chưa lưu. Switch chuyển frame tới R2; R2 tra static route, đóng frame mới trên WAN và chuyển sang R1, R1 tới mạng server. Eagle có thể ARP MAC của R1 rồi gửi ICMP Reply về gateway; R1 dùng return route qua R2 để trả tới host. Simulation Mode cho phép theo dõi ARP/ICMP và từng hop; thứ tự có thể đổi do cache.

## Mini CLI Interpretation

Lệnh ở R2: `ip route 192.168.254.0 255.255.255.0 10.10.10.6` cấu hình mạng Eagle /24 qua next hop WAN của R1. Lệnh ở R1: `ip route 172.16.0.0 255.255.0.0 10.10.10.5` cấu hình đường về LAN Pod /16 qua R2.

Hai bảng định tuyến thực tế đã hiển thị tuyến tĩnh phù hợp. Kết quả từ hostPod1A tới Eagle Server cũng đã được xác nhận bằng `ping` và `tracert`. Thử ping chiều ngược từ Eagle đến `172.16.1.1` là phép kiểm tra bổ sung có thể sử dụng khi cần xác định tính hai chiều của đường đi. Việc đánh giá định tuyến dựa trên cả bảng định tuyến và khả năng chuyển tiếp gói, không chỉ dựa vào các lệnh cấu hình.


**Lệnh và bằng chứng đối chiếu hai chiều:** Trên R2-Central và R1-ISP dùng `show ip route` để xác minh lần lượt tuyến tĩnh đến `192.168.254.0/24` và tuyến về `172.16.0.0/16` (T7, ảnh E10). Từ `hostPod1A`, kết quả `ping 192.168.254.254` và `tracert 192.168.254.254` đã ghi nhận kết nối đầu cuối và đường đi (T4–T5, ảnh E07/E08). Việc nhận ICMP Echo Reply cho thấy lưu lượng phản hồi của **phép thử đó** đã về được nguồn; không thay thế một phép ping độc lập khởi tạo từ Eagle Server, vốn chưa được ghi nhận trong danh mục kiểm thử hiện tại.
