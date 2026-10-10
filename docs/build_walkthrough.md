# Triển khai mô hình mạng trên Cisco Packet Tracer

## 1. Thiết bị và bố cục

Mô hình được xây dựng bằng Cisco Packet Tracer 9.0.1 với hai router **Cisco 2811**, một switch **2960-24TT** và ba thiết bị **PC-PT**. Hai router được đặt tên `R1-ISP`, `R2-Central`, có mô-đun `WIC-2T` tại Slot 0 để sử dụng cổng Serial. Switch mang tên `S1-Central`; ba PC có tên `hostPod1A`, `hostPod1B` và `Eagle Server`.

Bố cục chính: `hostPod1A` và `hostPod1B` nối tới `S1-Central`; switch nối với `R2-Central`; `R2-Central` nối Serial tới `R1-ISP`; `R1-ISP` nối Ethernet trực tiếp tới `Eagle Server`. Tên thiết bị và tên cổng được hiển thị trong Logical Workspace để thuận tiện đối chiếu.

Tệp mô phỏng: `20110112_DoThanhVinh_ND-Lab01.pkt`.

## 2. Kết nối vật lý

| Kết nối | Cổng đầu A | Cổng đầu B | Loại cáp |
|---|---|---|---|
| R1-ISP — R2-Central | S0/0/0 | S0/0/0 | Serial DCE, đầu DCE tại R1-ISP |
| R1-ISP — Eagle Server | Fa0/0 | Fa0 | Copper Cross-Over |
| R2-Central — S1-Central | Fa0/0 | Fa0/24 | Copper Straight-Through |
| S1-Central — hostPod1A | Fa0/1 | Fa0 | Copper Straight-Through |
| S1-Central — hostPod1B | Fa0/2 | Fa0 | Copper Straight-Through |

Cổng Serial sử dụng là `Serial0/0/0` trên cả hai router. Đầu DCE cung cấp xung nhịp; cổng DTE nhận xung nhịp từ phía đối diện. Đầu DCE/DTE được nhận diện bằng thông tin cáp hoặc lệnh `show controllers serial 0/0/0` khi IOS hỗ trợ.

## 3. Cấu hình hai router

Các script cấu hình tương ứng nằm trong `config/R1-ISP.txt` và `config/R2-Central.txt`. Sau khi đặt hostname, cấu hình IPv4 trên cổng Ethernet và Serial, các cổng sử dụng được kích hoạt bằng `no shutdown`.

**R1-ISP**

- `FastEthernet0/0`: `192.168.254.253 255.255.255.0`.
- `Serial0/0/0`: `10.10.10.6 255.255.255.252`.
- `clock rate 2000000` trên cổng Serial DCE.
- Tuyến tĩnh về LAN trung tâm:

```text
ip route 172.16.0.0 255.255.0.0 10.10.10.5
```

**R2-Central**

- `FastEthernet0/0`: `172.16.255.254 255.255.0.0`.
- `Serial0/0/0`: `10.10.10.5 255.255.255.252`.
- Tuyến tĩnh tới mạng Eagle Server:

```text
ip route 192.168.254.0 255.255.255.0 10.10.10.6
```

Cổng Serial DTE trên R2-Central không cần cấu hình `clock rate`. Hai tuyến tĩnh bảo đảm đường chuyển tiếp và đường phản hồi giữa các mạng không kết nối trực tiếp. Việc nhập lệnh và cấu hình thực tế được xác minh bằng `show running-config`, `show ip interface brief` và `show ip route`.

## 4. Địa chỉ IPv4 của các PC

Tại **Desktop → IP Configuration**, cả ba PC dùng cấu hình **Static**.

| Thiết bị | IPv4 | Subnet mask | Default gateway |
|---|---|---|---|
| hostPod1A | `172.16.1.1` | `255.255.0.0` | `172.16.255.254` |
| hostPod1B | `172.16.1.2` | `255.255.0.0` | `172.16.255.254` |
| Eagle Server | `192.168.254.254` | `255.255.255.0` | `192.168.254.253` |

`S1-Central` hoạt động với chức năng chuyển mạch Ethernet cơ bản, không sử dụng địa chỉ IP quản trị trong mô hình này.

## 5. Kiểm chứng và lưu cấu hình

Cấu hình router được kiểm tra bằng các lệnh:

```text
show ip interface brief
show ip route
show running-config
```

Hai cổng Ethernet đang dùng và cặp Serial WAN đã được kiểm tra ở trạng thái `up/up`. Bảng định tuyến chứa các mạng kết nối trực tiếp và tuyến tĩnh tới mạng ở phía router còn lại. Kết quả liên lạc giữa các thiết bị được tổng hợp trong `data/test_matrix.csv` và `docs/verification_and_simulation.md`.

Để lưu cấu hình router từ RAM vào NVRAM, sử dụng `copy running-config startup-config`. Lệnh này cần được xác minh riêng nếu muốn khẳng định cấu hình khởi động đã được ghi; `show running-config` chỉ phản ánh cấu hình đang chạy. Mô hình Packet Tracer được lưu vào tệp `.pkt`. Đối với phép ping đầu tiên tới Eagle Server, một gói không nhận phản hồi, nhưng lần thử tiếp theo đạt 4/4. Kết quả này được ghi nhận mà không quy nguyên nhân khi chưa đủ bằng chứng.
