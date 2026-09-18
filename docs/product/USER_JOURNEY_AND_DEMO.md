# Faroq — Kịch Bản Trải Nghiệm & Trình Diễn Sản Phẩm (Demo & User Journeys)

> **Tài liệu kịch bản trình diễn (Product Walkthrough)**: Mô tả 5 luồng trải nghiệm người dùng trọng tâm của Faroq, giúp nhà tuyển dụng và các bên liên quan nắm bắt được giải pháp và giá trị thực tế của ứng dụng chỉ trong 3 phút.

---

## 🎬 5 Luồng Nghiệp Vụ Trọng Tâm (Core User Journeys)

```mermaid
journey
    title Hành trình một ngày sống chung phòng trọ với Faroq
    section Buổi sáng: Khởi đầu ngày mới
      Xem việc nhà hôm nay: 5: Nam
      Check-in đổ rác nhận Streak: 5: Nam
    section Buổi chiều: Chi tiêu sinh hoạt
      Mua đồ siêu thị 450k: 4: Hải
      Tạo hóa đơn chia đều: 5: Hải
    section Buổi tối: Nấu ăn & Nghỉ ngơi
      Bận thi nhờ bạn làm thay: 4: Nam
      Đồng ý nhận ca Cover: 5: Quang
    section Cuối tháng: Quyết toán nợ
      Xem bảng cân đối nợ tối ưu: 5: Cả phòng
      Quét VietQR chuyển tiền sạch nợ: 5: Nam
```

---

### Luồng 1: Khởi Tạo Phòng & Mời Bạn Cùng Phòng An Toàn (Zero-Debt Trap Onboarding)
1. **Trưởng phòng tạo phòng**: Nhập tên phòng (VD: *"Phòng 402 KTX Bách Khoa"*), chọn ngày bắt đầu chu kỳ hóa đơn (mặc định ngày 1 hàng tháng).
2. **Khởi tạo danh sách phòng**: Thêm trước các thành viên ảo để ghi nhận tiền cọc/tiền phòng nếu các bạn chưa kịp tải app.
3. **Mời bạn vào phòng qua Email**:
   - Trưởng phòng nhập email của bạn cùng phòng. Nếu bạn đó thay thế cho một hồ sơ ảo cũ, chọn liên kết tiếp quản.
   - **Bảo vệ người được mời**: Người được mời mở app sẽ thấy popup **Bảng kê khai công nợ (Pre-claim Debt Statement)** ghi rõ từng khoản nợ mà hồ sơ cũ đang có. Sau khi kiểm tra minh bạch và bấm *"Xác nhận"*, tài khoản mới chính thức được gia nhập phòng.

---

### Luồng 2: Tạo Chi Tiêu Chung & Thuật Toán Chia Tiền 4 Phương Thức
1. **Nhập hóa đơn**: Người chi trả (VD: Hải) mua đồ gia dụng chung hết `450.000 đ`.
2. **Chọn 1 trong 4 cơ chế chia tiền linh hoạt**:
   - **Chia đều (Equal)**: Mỗi người tự động chia `150.000 đ`.
   - **Số tiền chính xác (Exact)**: Tự gõ số tiền cho từng thành viên.
   - **Phần trăm (Percentage)**: Người dùng nhiều trả 50%, người dùng ít trả 25%.
   - **Tỷ lệ phần chia (Shares)**: Phòng lớn trả 2 phần, phòng nhỏ trả 1 phần.
3. **Cập nhật số dư tức thì**: Hệ thống tự động ghi nhận vào sổ cái công nợ mà không phát sinh bất kỳ tranh cãi nào.

---

### Luồng 3: Quyết Toán Tối Ưu Hóa Giao Dịch & Quét Mã VietQR Napas 24/7
1. **Triệt tiêu nợ chéo**: Thay vì 4 bạn phải thực hiện 6 giao dịch lằng nhằng, thuật toán đồ thị Faroq rút gọn chỉ còn **2 giao dịch trực tiếp**.
2. **Thanh toán 1 chạm**: Người nợ bấm vào nút **"Trả nợ"**:
   - Thẻ **VietQR Card** hiển thị mã QR chuẩn Napas 24/7 tích hợp sẵn: *Số tài khoản ngân hàng, tên người nhận, số tiền nợ chính xác đến từng đồng và nội dung chuyển khoản tự động*.
3. **Quét mã & Ghi nhận**: Mở app ngân hàng quét mã ➔ Bấm xác nhận trên Faroq ➔ Công nợ sạch 100%.

---

### Luồng 4: Phân Công Việc Nhà Xoay Tua & Bảng Đo Công Bằng (Fairness Matrix)
1. **Tạo việc nhà mẫu**: Chọn các mẫu việc có sẵn (Rửa bát, Đổ rác, Lau sàn, Cọ toilet) với thang điểm công sức từ 5đ đến 25đ.
2. **Thuật toán xoay tua**: Hệ thống tự động phân công theo chu kỳ và nhận diện thông minh **Kỳ vắng mặt (Away Period)** của thành viên để không giao việc oan khi bạn về quê.
3. **Check-in & Điểm công bằng**:
   - Hoàn thành việc nhà kích hoạt hiệu ứng chúc mừng kèm rung xúc giác (Haptics) và chuỗi Streak 🔥.
   - Bảng **Fairness Matrix** hiển thị tỷ lệ đóng góp của từng người trong tuần/tháng, đảm bảo không ai phải gánh việc thay cho người lười biếng.

---

### Luồng 5: Nhờ Làm Hộ (Cover) & Đổi Ca Trực Nhật (Swap)
1. **Gửi yêu cầu**: Khi bận lịch thi hoặc ốm, thành viên chọn một lượt trực và bấm *"Nhờ làm hộ"* gửi cho bạn cùng phòng.
2. **Phản hồi 2 chiều**: Người được nhờ nhận thông báo, xem chi tiết điểm công sức của việc và bấm *"Đồng ý"* hoặc *"Từ chối"*.
3. **Điều chuyển công bằng**: Điểm công sức của ca trực tự động được cộng dồn cho người làm hộ trên bảng Fairness Score.
