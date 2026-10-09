# Build và Configure — Thao tác trong Cisco Packet Tracer 9.0.1

## 1. Tạo bản `.pkt` thực tế
- File → New; đặt hai **2811** (R1-ISP và R2-Central), mỗi router lắp **WIC-2T vào Slot 0** khi đã tắt nguồn, bật lại.
- Đặt một switch **2960-24TT** tên **S1-Central** và ba PC-PT: **hostPod1A**, **hostPod1B**, **Eagle Server**.
- Bật **Always Show Port Labels** trong Options → Preferences nếu có.
- Đặt thiết bị theo topology Cisco trang 1 (R1 nối trực tiếp Eagle; R1 nối serial R2; R2 nối S1; hai host nối S1).
- File → Save As vào `Network Design\Labs\NDLab01\20110112_DoThanhVinh_ND-Lab01.pkt`.
- Không dùng R0–R3 của Instruction. File `.pkt` chỉ có giá trị kiểm chứng sau khi được mở và chạy trên máy của bạn.

## 2. Kết nối cáp (đã lập kế hoạch, chưa có Actual)
1. Connections → **Serial DCE**, bấm **R1-ISP trước** và chọn Serial0/0/0, sau đó R2-Central → Serial0/0/0. Kiểm tra đầu DCE thực tế bằng CLI (`show controllers serial 0/0/0`) hoặc dấu nhận diện của Packet Tracer. Nếu DCE nằm ở R2, phải chỉnh lại cáp trước khi dán cấu hình.
2. R1-ISP Fa0/0 → Eagle Server FastEthernet0: Copper Cross-Over hoặc Auto.
3. R2-Central Fa0/0 → S1-Central Fa0/24: Copper Straight-Through.
4. S1-Central Fa0/1 → hostPod1A Fa0: Copper Straight-Through.
5. S1-Central Fa0/2 → hostPod1B Fa0: Copper Straight-Through.
6. Chụp topology với nhãn port khi đã nối và đặt tên đủ (E01).

## 3. Cấu hình router
- Mở R1-ISP → CLI. Nếu có `Would you like to enter...` trả lời `no`; vào `Router>`.
- Dán **từng khối lệnh phù hợp** từ `config/R1-ISP.txt` (dòng bắt đầu `!` là chú thích, có thể bỏ qua); xác minh `clock rate` chỉ trên DCE.
- Lặp lại trên R2 với `config/R2-Central.txt`, **không** chạy `clock rate` trên DTE.
- Đảm bảo interface đã có IP và `no shutdown`. Khi trạng thái chưa up/up, kiểm tra cáp, trạng thái đầu bên kia và Serial DCE.

## 4. Cấu hình 3 PC
Desktop → IP Configuration → Static. Nhập theo `data/addressing_table.csv`:
- hostPod1A: 172.16.1.1 / 255.255.0.0 / GW 172.16.255.254.
- hostPod1B: 172.16.1.2 / 255.255.0.0 / GW 172.16.255.254.
- Eagle Server (PC): 192.168.254.254 / 255.255.255.0 / GW 192.168.254.253.
- S1-Central không cần cấu hình IP quản trị (quyết định D04); giữ chuyển mạch mặc định.

## 5. Xác minh cấu hình và lưu lại
- R1, R2: `show ip interface brief`, `show ip route`, `show running-config`.
- Nếu route có vấn đề, kiểm tra R2 có static đến `192.168.254.0/24` và R1 có static đến `172.16.0.0/16`.
- Khi cấu hình đã đúng: `copy running-config startup-config` và chọn lưu file `.pkt` (`Ctrl+S`).
- Nếu ping đầu tiên timeout do ARP, thử lại và quan sát Simulation; không tự khẳng định Pass khi chưa thấy kết quả.

## 6. Git đúng tiến trình
- Sau khi giải nén pack trong `Network Design` kiểm tra README, config và docs là nội dung thực tế; `git init` tại `Labs/NDLab01` nếu chưa có repo.
- Commit 1 (plan): các file kế hoạch và cấu hình bản đầu (chưa nhận là đã triển khai).
- Commit 2 (configure): sau khi `.pkt` được tạo và có cấu hình đã xác minh.
- Commit 3 (verify/report): sau khi có evidence, Test Matrix Actual và PDF report thực tế.
- Remote GitHub phải kiểm tra repo hiện tồn tại hay chưa; không push nhầm repo của môn khác.
