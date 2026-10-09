# ND-Lab01 — Packet Tracer Network Implementation

Môn: Thiết kế mạng. Sinh viên: Đỗ Thành Vinh, MSSV 20110112.

Mô phỏng theo **NDLab01 – Requirement** và topology của *Cisco Lab 6.7.1: Ping and Traceroute*.
Quy trình: Predict → Build → Configure → Verify → Simulate → Explain.

## Phạm vi
- Cisco Packet Tracer 9.0.1, hai router Cisco 2811 với WIC-2T, S1-Central (2960), hai host và một PC đại diện Eagle Server.
- Pod# = 1 (giảng viên không giao số Pod riêng).
- Static routing hai chiều; không thêm OSPF, NAT, ACL, DHCP hay VLAN mở rộng.

## Cấu trúc repo
- `config/`: cấu hình mẫu cho hai router, cần áp dụng trong Packet Tracer và kiểm chứng.
- `data/`: bảng địa chỉ, port mapping, Test Matrix, danh mục evidence.
- `docs/`: hướng dẫn thao tác và các phần giải thích/bài giải kỹ thuật.
- `evidence/`: **chỉ chứa ảnh chụp thực nghiệm thật** từ Packet Tracer.
- `report/`: bản PDF từ khu vực `Reports/NDLab01/output/` được sao chép sau khi biên dịch.
- `20110112_DoThanhVinh_ND-Lab01.pkt`: **sẽ do Packet Tracer lưu trên máy**, chưa có trong gói P6.

## Trạng thái kiểm chứng
- [x] Kiểm tra môi trường PT 9.0.1, 2811/WIC-2T và các interface ở giai đoạn chuẩn bị.
- [ ] Dựng topology trong file `.pkt`.
- [ ] Chạy T1–T7, lưu ảnh có đối chiếu Expected/Actual.
- [ ] Mô phỏng ARP/ICMP và `tracert`.
- [ ] Kiểm tra 12 tình huống Problem Solving bằng evidence phù hợp.
- [ ] Đưa bản report có kết quả thật vào `report/`.
- [ ] Kiểm tra tối thiểu 3 commit có nội dung thật và push GitHub.

**Các kết quả thực nghiệm chỉ được công nhận khi đã chạy thực tế trong Packet Tracer.**

## Nguồn đề bài
1. `NDLab01 - Packet Tracer Introduction (Requirements)(1).docx`, mục 0–7.
2. `NDLab01 - Packet Tracer Introduction (Instruction)(1).docx`, dùng làm hướng dẫn workflow, không dùng topology R0–R3.
3. `lab.pdf`, Cisco Lab 6.7.1, trang 1, dùng topology và Addressing Table.

## Nộp bài (kiểm tra tại bước cuối)
Email: tuantran261083course@gmail.com; hạn: 20/10/2026.
Tên PDF dự kiến theo lựa chọn người học: `20110112_DoThanhVinh_ND-Lab01.pdf` (bỏ khoảng trắng thừa ở mẫu đề).
