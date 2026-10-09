# Predict — Thiết kế và dự đoán trước thực nghiệm
- LAN Pod: 172.16.0.0/16; gateway R2 Fa0/0 = 172.16.255.254; hostPod1A và hostPod1B cùng mạng.
- Serial WAN: 10.10.10.4/30, network=10.10.10.4, usable=.5/.6, broadcast=.7.
- LAN Eagle: 192.168.254.0/24; gateway R1 Fa0/0 = .253; Eagle PC = .254.
- A → B: cùng subnet, không đi router; có thể ARP lấy MAC của B.
- A → Eagle: A ARP gateway R2, đi R2 → Serial WAN → R1 → Eagle; phản hồi từ Eagle đi R1 → R2 → A.
- Hai static route giúp đi và về; thiếu một chiều sẽ làm ping end-to-end thất bại.
- Tracert dự kiến hop R2 (172.16.255.254), R1 (thường IP cổng WAN hoặc interface khác tùy hành vi trả lời), rồi Eagle 192.168.254.254. Không tự gán IP hop R1 khi chưa chạy thực nghiệm.
- Dự đoán này không được nhập vào cột Actual. Giới hạn: Packet Tracer có thể mô phỏng timeout/hop/ARP khác tùy phiên bản và trạng thái cache.
