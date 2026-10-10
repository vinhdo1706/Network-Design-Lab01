# Giải thích lệnh CLI và cơ chế hoạt động

## 1. Các chế độ CLI

- `Router>`: **User EXEC**, hỗ trợ một tập lệnh kiểm tra cơ bản.
- `enable`: chuyển sang **Privileged EXEC** (`Router#`), cho phép sử dụng các lệnh kiểm tra và quản lý cấu hình.
- `configure terminal`: vào **Global Configuration** (`Router(config)#`).
- `interface fastethernet0/0` hoặc `interface serial0/0/0`: chuyển đến chế độ cấu hình cổng (`Router(config-if)#`).
- `exit`: quay về chế độ trước; `end`: trở về Privileged EXEC.

Lệnh phải được nhập trong chế độ phù hợp. Ví dụ, `ip address` và `no shutdown` được thực hiện tại chế độ cấu hình interface.

## 2. Hostname và cổng Ethernet

`hostname R1-ISP` hoặc `hostname R2-Central` đổi tên router, thể hiện trong dấu nhắc CLI và `show running-config`, không làm thay đổi địa chỉ mạng.

Lệnh `interface fastethernet0/0` chọn giao diện nối tới mạng Ethernet. Hai giá trị sử dụng trong mô hình:

- R1-ISP: `ip address 192.168.254.253 255.255.255.0` — gateway của mạng Eagle.
- R2-Central: `ip address 172.16.255.254 255.255.0.0` — gateway của LAN trung tâm.

Gán sai IP hoặc subnet mask có thể làm mất khả năng giao tiếp tới mạng dự kiến. Trạng thái cổng được xác minh bằng `show ip interface brief`.

## 3. Cổng Serial và clock rate

`interface serial0/0/0` chọn cổng WAN trên mô-đun WIC-2T. Hai đầu có địa chỉ:

- R1-ISP: `10.10.10.6 255.255.255.252`.
- R2-Central: `10.10.10.5 255.255.255.252`.

Hai IP cùng thuộc mạng `10.10.10.4/30`, với địa chỉ mạng `10.10.10.4`, hai địa chỉ host hợp lệ `10.10.10.5` và `10.10.10.6`, địa chỉ broadcast `10.10.10.7`.

`clock rate 2000000` được cấu hình trên **Serial0/0/0 của R1-ISP**, nơi đóng vai trò **DCE** trong mô hình. DCE cung cấp xung nhịp cho liên kết nối tiếp; phía DTE không cần cấu hình lệnh này. Vai trò DCE/DTE có thể kiểm tra qua kết nối vật lý hoặc `show controllers serial 0/0/0` nếu phiên bản IOS hỗ trợ. Giá trị `2000000` đã được đối chiếu với cấu hình đang chạy trên R1-ISP.

`no shutdown` bỏ trạng thái tắt quản trị của interface. Cổng chỉ đạt `up/up` khi kết nối vật lý, thiết bị đầu kia và giao thức tầng liên kết đều hoạt động. `administratively down/down` biểu thị cổng bị tắt bằng cấu hình; `up/down` cho biết lớp vật lý hoạt động nhưng line protocol chưa hoạt động.

## 4. Định tuyến tĩnh

**Trên R2-Central:**

```text
ip route 192.168.254.0 255.255.255.0 10.10.10.6
```

Router chuyển lưu lượng tới mạng Eagle (`192.168.254.0/24`) qua next hop Serial của R1-ISP (`10.10.10.6`).

**Trên R1-ISP:**

```text
ip route 172.16.0.0 255.255.0.0 10.10.10.5
```

Tuyến này tạo đường phản hồi tới LAN trung tâm (`172.16.0.0/16`) qua R2-Central (`10.10.10.5`). Thiếu tuyến chiều về có thể khiến Echo Request đến được đích nhưng Echo Reply không quay lại nguồn.

## 5. Các lệnh kiểm tra cấu hình

- `show ip interface brief`: hiển thị địa chỉ IP, trạng thái vật lý và line protocol của từng interface.
- `show ip route`: hiển thị bảng định tuyến. Ký hiệu `C` là **connected**, `L` là **local** (nếu phiên bản IOS hiển thị), `S` là **static**; `O` biểu thị tuyến OSPF nhưng mô hình này không sử dụng OSPF.
- `show running-config`: hiển thị cấu hình hiện hành trong RAM. Đây là bằng chứng về tham số cấu hình, không tự chứng minh liên lạc đầu cuối thành công.
- `copy running-config startup-config`: lưu cấu hình hiện hành vào NVRAM. Tệp Packet Tracer `.pkt` cũng cần được lưu để giữ toàn bộ mô hình.

Một tuyến được khai báo nhưng next hop hoặc cổng liên quan không khả dụng có thể không được cài vào bảng định tuyến. Do đó, cần kết hợp kiểm tra cấu hình, trạng thái interface và bảng định tuyến.

## 6. Các lệnh kiểm tra tại PC

- `ipconfig`: hiển thị IPv4, subnet mask và default gateway.
- `ping <địa_chỉ_IP>`: gửi ICMP Echo Request, đối chiếu số Echo Reply nhận được; timeout có thể do nhiều nguyên nhân khác nhau.
- `tracert <địa_chỉ_IP>`: thay đổi TTL để nhận diện các hop trung gian; thông thường hop hết TTL trả về ICMP Time Exceeded. Một dấu `*` chỉ cho biết probe không nhận phản hồi trong thời hạn quy định.

Trong công cụ `tracert` trên Windows, `-w 5` chỉ thời gian chờ **5 mili giây** cho mỗi phản hồi và `-h 4` giới hạn tối đa **4 hop**. Mức hỗ trợ tham số trong Command Prompt của Packet Tracer phụ thuộc phiên bản; các tùy chọn này chưa được xác nhận bằng phép thử riêng trong mô hình.


## 7. Bảng đối chiếu lệnh, trạng thái và cách kiểm chứng

Bảng sau hệ thống hóa từng nhóm lệnh sử dụng trong mô hình. Các hậu quả là **tình huống dự đoán nếu nhập thiếu/sai**, không phải những lỗi đã được chủ động gây ra trên mô hình. Phép kiểm tra chỉ được xem là đã thực hiện khi có kết quả trong `data/test_matrix.csv` hoặc ảnh tương ứng ở `data/evidence_manifest.csv`.

| Lệnh / thiết bị / chế độ | Tham số và thay đổi trạng thái | Cách kiểm chứng | Nếu bỏ hoặc cấu hình sai |
|---|---|---|---|
| `enable` — R1/R2, User EXEC | Sang Privileged EXEC (`#`); không đổi mạng. | Quan sát dấu nhắc `Router#`. | Không truy cập được một số lệnh quản trị. |
| `configure terminal` — R1/R2, Privileged EXEC | Sang Global Configuration (`(config)#`); chưa đổi interface. | Quan sát dấu nhắc `(config)#`. | Không cấu hình được IP, hostname hay route tại chế độ cần thiết. |
| `hostname R1-ISP` / `hostname R2-Central` — Global Configuration | Đặt tên thiết bị, không đổi địa chỉ IP. | Dấu nhắc CLI và `show running-config`. | Dễ nhầm thiết bị khi cấu hình hoặc thu thập bằng chứng. |
| `interface fastethernet0/0` — R1/R2, Global Configuration | Chọn cổng nối LAN; chưa tự gán IP. | Dấu nhắc `(config-if)#` và `show ip interface brief`. | Có thể gán IP vào nhầm cổng nếu chọn sai. |
| `ip address 192.168.254.253 255.255.255.0` — R1 Fa0/0, Interface Configuration | Đặt gateway của LAN Eagle thuộc `192.168.254.0/24`. | `show ip interface brief`, đối chiếu Eagle Server gateway. | Mất khả năng chuyển tiếp đúng qua mạng Eagle nếu IP/mask sai. |
| `ip address 172.16.255.254 255.255.0.0` — R2 Fa0/0, Interface Configuration | Đặt gateway của LAN trung tâm thuộc `172.16.0.0/16`. | `show ip interface brief`, T1–T2. | Host không liên lạc gateway hoặc không ra được mạng xa. |
| `interface serial0/0/0` — R1/R2, Global Configuration | Chọn cổng Serial WAN. | `show ip interface brief`; đối chiếu port mapping. | Dùng nhầm cổng khiến liên kết WAN dự kiến không hoạt động. |
| `ip address 10.10.10.6 255.255.255.252` — R1 S0/0/0, Interface Configuration | Gán IP đầu R1 trong `10.10.10.4/30`. | `show ip interface brief`, đối chiếu đầu R2. | Mạng WAN không được đánh địa chỉ nhất quán. |
| `ip address 10.10.10.5 255.255.255.252` — R2 S0/0/0, Interface Configuration | Gán IP đầu R2 trong cùng `/30`. | `show ip interface brief`, kiểm tra T3. | Không đảm bảo được liên lạc qua WAN. |
| `clock rate 2000000` — R1 S0/0/0, Interface Configuration (DCE) | Cung cấp định thời cho liên kết Serial; không đặt ở đầu DTE. | Kiểm tra DCE/DTE và `show running-config`. | Liên kết Serial có thể không hoạt động nếu phía DCE thiếu định thời. |
| `no shutdown` — các interface đang dùng, Interface Configuration | Bỏ trạng thái tắt quản trị; **không tự bảo đảm** cổng `up/up`. | `show ip interface brief`, trạng thái cáp và đầu bên kia. | Có thể xuất hiện `administratively down/down`. |
| `exit` / `end` — chế độ cấu hình | `exit` về một mức; `end` về Privileged EXEC. | Quan sát dấu nhắc CLI. | Nhập lệnh tại sai chế độ có thể bị từ chối. |
| `ip route 192.168.254.0 255.255.255.0 10.10.10.6` — R2, Global Configuration | Thêm tuyến tĩnh tới LAN Eagle qua R1. | `show ip route` tại R2, T7; thử T4/T5. | R2 có thể không tìm được tuyến hợp lệ tới LAN Eagle. |
| `ip route 172.16.0.0 255.255.0.0 10.10.10.5` — R1, Global Configuration | Thêm tuyến chiều về LAN trung tâm qua R2. | `show ip route` tại R1, T7; phản hồi ping T4. | Echo Reply có thể không về được host nguồn. |
| `show ip interface brief` — R1/R2, Privileged EXEC | Chỉ xem IPv4 và trạng thái interface; không sửa cấu hình. | Đối chiếu `up/up`, IP và ảnh E02/E03. | Bỏ bước này sẽ thiếu chứng cứ trạng thái cổng. |
| `show ip route` — R1/R2, Privileged EXEC | Chỉ xem mạng connected và route được cài; không tạo route. | Đối chiếu `C`, `S` và ảnh E10. | Chỉ nhìn cấu hình sẽ không biết tuyến có được cài hợp lệ hay không. |
| `show running-config` — R1/R2, Privileged EXEC | Hiển thị cấu hình RAM; không đo kết nối đầu cuối. | Đối chiếu IP, clock, route và ảnh E02/E03. | Không có bằng chứng trực tiếp về câu lệnh đã cấu hình. |
| `copy running-config startup-config` — R1/R2, Privileged EXEC | Ghi cấu hình hiện hành vào NVRAM nếu lệnh hoàn thành thành công. | Xem thông báo hoàn thành hoặc đối chiếu `show startup-config` khi được hỗ trợ. | Thay đổi có thể mất khi router khởi động lại. |
| `ipconfig` — hostPod, PC Command Prompt | Xem IP, mask và gateway; không thay đổi cấu hình. | T1, ảnh E04. | Thiếu dữ liệu để chẩn đoán host sai IP/gateway. |
| `ping <địa_chỉ_IP>` — PC Command Prompt | Gửi ICMP Echo Request để kiểm tra khả năng nhận Echo Reply. | T2–T4, T6; đối chiếu tỷ lệ nhận/mất. | Chỉ định sai đích làm sai kết luận kiểm thử. |
| `tracert <địa_chỉ_IP>` — PC Command Prompt | Dùng TTL tăng dần để quan sát các hop có phản hồi. | T5, ảnh E08. | Không có bằng chứng thứ tự hop; `*` không tự chứng minh mất kết nối. |
| `tracert -w 5 -h 4 <địa_chỉ_IP>` — Windows Command Prompt | `-w 5`: chờ 5 ms mỗi phản hồi; `-h 4`: tối đa 4 hop. | So sánh với traceroute mặc định **nếu** công cụ hỗ trợ; chưa có phép thử riêng trong Packet Tracer. | Giới hạn quá nhỏ có thể làm mất phản hồi quan sát hoặc dừng trước đích. |
