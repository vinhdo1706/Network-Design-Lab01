# ND-Lab01 — Packet Tracer Network Implementation

**Môn học:** Thiết kế mạng  
**Sinh viên:** Đỗ Thành Vinh  
**MSSV:** 20110112

## Giới thiệu

Bài thực hành xây dựng và kiểm tra một mô hình mạng trong Cisco Packet Tracer 9.0.1, dựa trên sơ đồ và bảng địa chỉ của *Cisco Lab 6.7.1: Ping and Traceroute*. Mô hình gồm hai router Cisco 2811, một switch Cisco 2960-24TT, hai máy trạm thuộc LAN trung tâm và một PC đại diện cho Eagle Server.

Hai router kết nối qua đường Serial; các mạng ở hai đầu được liên thông bằng định tuyến tĩnh hai chiều. Hoạt động của mô hình được kiểm tra bằng các lệnh cấu hình, kiểm tra kết nối IP, truy vết đường đi và mô phỏng ICMP.

## Phạm vi và cấu trúc mạng

- Router: **R1-ISP**, **R2-Central** (Cisco 2811, mô-đun WIC-2T).
- Switch: **S1-Central** (Cisco 2960-24TT).
- Thiết bị đầu cuối: **hostPod1A**, **hostPod1B**, **Eagle Server** (PC-PT).
- Địa chỉ của hai máy trạm được tổ chức theo **Pod 1** (`Pod# = 1`), là phương án được Requirements cho phép khi chưa được giao số pod riêng.
- Định tuyến: **static routing**; không triển khai thêm OSPF, NAT, ACL, DHCP hoặc VLAN riêng.

| Phân đoạn | Mạng IPv4 | Giao diện định tuyến |
|---|---|---|
| LAN trung tâm | `172.16.0.0/16` | R2-Central Fa0/0: `172.16.255.254` |
| WAN Serial | `10.10.10.4/30` | R2-Central S0/0/0: `10.10.10.5`; R1-ISP S0/0/0: `10.10.10.6` |
| LAN Eagle | `192.168.254.0/24` | R1-ISP Fa0/0: `192.168.254.253` |

Địa chỉ từng thiết bị và sơ đồ cổng kết nối được liệt kê trong `data/addressing_table.csv` và `data/port_mapping.csv`.

## Cấu trúc repository

| Thành phần | Nội dung |
|---|---|
| `20110112_DoThanhVinh_ND-Lab01.pkt` | Mô hình mạng Cisco Packet Tracer |
| `config/` | Script cấu hình R1-ISP và R2-Central |
| `data/` | Bảng địa chỉ, ánh xạ cổng, kết quả kiểm thử và danh mục minh chứng |
| `docs/` | Thuyết minh triển khai, giải thích CLI, dự đoán, phân tích tình huống và xác minh |
| `evidence/` | Ảnh minh chứng cấu hình, kiểm thử, Simulation Mode và lịch sử Git/GitHub; chỉ mục đối chiếu nằm tại `data/evidence_manifest.csv` |

Ảnh dùng trong báo cáo được tổ chức riêng tại `Reports/NDLab01/figures/` trong workspace môn học; khu vực này nằm ngoài repository `Labs/NDLab01/`. Bản sao ảnh dùng để kiểm chứng và truy vết được lưu trong `evidence/`.

**GitHub repository (Public):** https://github.com/vinhdo1706/Network-Design-Lab01

## Kết quả kiểm thử

Bảy phép kiểm thử T1–T7 đã được thực hiện trong Packet Tracer và được ghi nhận **PASS** trong `data/test_matrix.csv`. Nội dung kiểm thử gồm cấu hình IP, liên lạc trong LAN, khả năng định tuyến giữa hai mạng, `tracert` và bảng định tuyến của hai router.

- Kết nối từ hostPod1A tới Eagle Server thành công. Phép ping đầu tiên nhận 3/4 phản hồi; lần kiểm tra lại nhận 4/4 phản hồi. Chưa có đủ bằng chứng để xác định nguyên nhân gói đầu bị mất.
- `tracert` từ hostPod1A tới Eagle Server hoàn tất qua ba điểm phản hồi: `172.16.255.254`, `10.10.10.6`, `192.168.254.254`.
- Chế độ Simulation đã ghi nhận quá trình chuyển tiếp ICMP thành công. Quan sát ARP chi tiết và bộ ảnh minh chứng được kiểm kê riêng trong `data/evidence_manifest.csv`.

Các tình huống trong `docs/problem_solving.md` là **phân tích lỗi giả định**; không đồng nghĩa với việc đã chủ động tạo và kiểm thử đủ 12 lỗi trên mô hình.

## Tài liệu

- `docs/build_walkthrough.md`: các thành phần, kết nối và cấu hình mô hình.
- `docs/cli_explanation.md`: ý nghĩa các lệnh Cisco IOS và lệnh kiểm thử.
- `docs/prediction.md`: dự đoán đường đi và phản hồi trước thực nghiệm.
- `docs/verification_and_simulation.md`: phép thử T1–T7 và quan sát Simulation.
- `docs/problem_solving.md`: phân tích 12 tình huống lỗi.
- `docs/interview_and_mini_cli.md`: 18 nội dung hỏi đáp và diễn giải các lệnh định tuyến.

## Tài liệu tham khảo

1. *NDLab01 — Packet Tracer Introduction (Requirements)*.
2. *NDLab01 — Packet Tracer Introduction (Instruction)*.
3. *Cisco Lab 6.7.1 — Ping and Traceroute*, sơ đồ mạng và bảng địa chỉ ở trang 1.
