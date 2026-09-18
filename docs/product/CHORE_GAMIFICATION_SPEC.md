# Đặc Tả Tính Năng Game Hóa Việc Nhà (Chore Gamification Spec)

> **Tài liệu tham khảo & Thiết kế kỹ thuật cho FairRoom**  
> *Phiên bản:* 1.0  
> *Trạng thái:* Thiết kế sẵn sàng để triển khai trong tương lai (Post-MVP)

---

## 1. Triết Lý & Mục Tiêu (Product Philosophy)

Việc nhà trong phòng trọ/nhà chung thường là nguyên nhân hàng đầu dẫn đến xích mích âm ỉ:
- **Hiện trạng:** Người chăm chỉ thì cảm thấy bị lợi dụng và ức chế, người lười thì hay viện cớ "quên" hoặc "bận". Lời nhắc nhở qua lại dễ trở nên gắt gỏng (passive-aggressive).
- **Mục tiêu của Gamification trong FairRoom:**
  - **Minh bạch hóa nỗ lực:** Mọi công sức (dù là đổ rác nhỏ hay cọ toilet lớn) đều được quy đổi ra điểm số định lượng công bằng.
  - **Tạo động lực tích cực thay vì trách phạt:** Thay vì chỉ trích người lười, ta tôn vinh người chăm và tạo cơ chế thưởng/phạt vui vẻ, lành mạnh.
  - **Gắn kết tình cảm bạn cùng phòng:** Biến trách nhiệm thành trò chơi tương tác hàng tuần/hàng tháng.

---

## 2. Hệ Thống Điểm Công Sức (Effort Points System)

Không phải việc nhà nào cũng tốn công sức như nhau. FairRoom phân cấp điểm số theo độ cực nhọc:

| Mức độ | Điểm | Loại công việc ví dụ | Thời gian ước tính |
| :--- | :---: | :--- | :--- |
| **Nhẹ (Tier 1)** | `5 pts` | Đổ rác, tưới cây, nhận bưu phẩm phòng | ~ 3 - 5 phút |
| **Trung bình (Tier 2)**| `10 pts` | Rửa bát, quét nhà, lau bàn ăn, thay bình nước | ~ 10 - 15 phút |
| **Nặng (Tier 3)** | `15 pts` | Lau sàn nhà, giặt & phơi quần áo chung, lau kính | ~ 20 - 30 phút |
| **Cực nhọc (Tier 4)** | `25 pts` | Cọ nhà vệ sinh (WC), tổng vệ sinh bếp, dọn tủ lạnh | ~ 45 - 60 phút |

### Hệ số nhân điểm (Point Multipliers):
- **Cứu bồ (Cover Hero) x 1.2:** Khi nhận làm hộ ca trực của bạn cùng phòng vì bạn bận đột xuất, người nhận làm được thưởng thêm **20% điểm**.
- **Siêu tốc (Early Bird) x 1.1:** Hoàn thành trước deadline từ 2 tiếng trở lên được cộng thêm **10% điểm**.
- **Trừ điểm phạt (Late Penalty):** Trễ hạn quá 24h mà không có lý do -> Bị trừ 50% số điểm của việc đó.

---

## 3. Bảng Xếp Hạng & Chỉ Số Công Bằng (Leaderboard & Fairness Index)

### 3.1. Điểm Công Bằng Tuần (Weekly Fairness Score - %)
Công thức tính toán độ cân bằng công việc trong phòng:
$$\text{Fairness Score} = 100 - \frac{\text{Độ lệch chuẩn điểm công sức của các thành viên}}{\text{Điểm công sức trung bình}} \times 50$$
- **80% - 100% (🟢 Siêu Cân Bằng):** Cả phòng cùng san sẻ việc nhà đều tay.
- **50% - 79% (🟡 Chấp Nhận Được):** Có 1-2 bạn đang gánh việc nhiều hơn một chút.
- **Dưới 50% (🔴 Mất Cân Bằng):** Phòng đang có hiện tượng "1 người làm - 3 người nhìn". Cần điều chỉnh lại phân công!

### 3.2. Bảng Phong Thần Tháng (Monthly Hall of Fame)
Vào ngày cuối cùng của tháng, app tự động tổng kết và vinh danh:
- 🥇 **Hạng 1 (The MVP / Gánh Đội):** Danh hiệu *"Chiến Thần Dọn Dẹp"*.
- 🥈 **Hạng 2:** *"Trợ Thủ Đắc Lực"*.
- 😴 **Hạng chót:** *"Vua Ngủ Kỹ"* (kèm thông báo hài hước, nhắc nhở tháng sau cố gắng).

---

## 4. Hệ Thống Huy Hiệu & Danh Hiệu (Badges & Achievements)

Mở khóa huy hiệu hiển thị trên trang cá nhân (Profile) và thẻ tên trong phòng:

### 🏅 Huy hiệu Chăm Chỉ:
- **🧽 Chúa Tể Bồn Cầu:** Cọ WC đủ 4 lần trong 1 tháng mà không bỏ ca nào.
- **🍽️ Vua Diệt Đĩa:** Hoàn thành 15 lần rửa bát trong tháng.
- **⚡ Thần Tốc:** Hoàn thành công việc trong vòng 15 phút kể từ khi đến giờ hẹn 5 lần liên tiếp.

### 🤝 Huy hiệu Tình Bạn & Hỗ Trợ:
- **🦸 Hiệp Sĩ Cứu Nguy:** Nhận làm hộ (Cover) 3 lần cho bạn cùng phòng.
- **🔄 Nhà Ngoại Giao:** Đổi ca (Swap) thành công 5 lần mà đôi bên đều vui vẻ.

### 🔥 Chuỗi Phong Độ (Streak Counter):
- **Chuỗi 7 ngày (7-Day Streak):** 7 ngày liên tiếp làm đúng hạn mọi việc được giao -> Tặng hiệu ứng vòng lửa 🔥 quanh avatar.
- **Chuỗi 30 ngày (30-Day Master):** Hoàn thành liên tục 1 tháng không trễ hẹn -> Mở khóa theme giao diện VIP riêng cho app.

---

## 5. Quy Đổi Quyền Lợi Thực Tế Trong Phòng (Room Perks & Rewards)

Gamification chỉ thực sự hiệu quả khi gắn liền với lợi ích đời thực của các bạn cùng phòng. FairRoom cung cấp tính năng cho phép phòng tự cài đặt quy ước:

### Gợi ý Phần Thưởng cho Người Nhất Tháng (The Winner):
1. **Trà sữa / Ăn uống:** Các thành viên còn lại chung tiền bao người nhất tháng 1 ly trà sữa hoặc 1 bữa ăn sáng.
2. **Miễn trực nhật:** Người nhất tháng được nhận 1 "Vé Miễn Trực Nhật" (Pass Ticket) trong 1 tuần của tháng sau.
3. **Quyền ưu tiên:** Ưu tiên chọn vị trí bàn học, chỗ để xe hoặc giường ngủ tốt hơn.

### Gợi ý "Hình Phạt" Vui Vẻ cho Người Ít Điểm Nhất:
1. **Đóng quỹ phòng:** Nộp 20.000đ - 50.000đ vào quỹ chung để mua nước rửa bát, nước lau sàn, giấy vệ sinh.
2. **Nấu ăn / Mua đồ:** Phụ trách đi chợ hoặc chuẩn bị đồ ăn cho bữa tiệc phòng cuối tháng.

---

## 6. Cơ Chế Chống Gian Lận (Anti-Cheat & Verification)

Để tránh trường hợp một bạn bấm nút "Hoàn thành" liên tục mà thực tế không hề làm việc:

1. **Minh chứng Ảnh chụp (Photo Proof - Tùy chọn bật/tắt):**
   - Khi cấu hình việc nhà (nhất là các việc nặng như cọ toilet, dọn bếp), chủ phòng có thể bật tùy chọn *"Yêu cầu chụp ảnh xác thực"*.
   - Khi bấm hoàn thành, camera mở ra chụp nhanh sàn nhà/bồn rửa bát sạch sẽ. Ảnh được lưu vào nhật ký hoạt động.
2. **Quyền Khiếu Nại (Dispute / "Chưa sạch đâu"):**
   - Sau khi thành viên A bấm hoàn thành, các thành viên khác có 12 tiếng để bấm nút *"Chưa làm / Chưa sạch"*.
   - Nếu có ít nhất 2 bạn cùng phòng xác nhận chưa làm, ca trực sẽ bị mở lại và điểm số tạm giữ sẽ không được cộng.

---

## 7. Kiến Trúc Dữ Liệu Dự Kiến (Database Schema Design)

Khi triển khai tính năng này, cần bổ sung vào Prisma Schema:

```prisma
// 1. Quản lý Huy hiệu
model Badge {
  id          String   @id // e.g. "CLEAN_HERO", "STREAK_7"
  title       String
  description String
  iconUrl     String
  pointsRequired Int   @default(0)
}

// 2. Huy hiệu thành viên đã đạt được
model MemberBadge {
  id        Int      @id @default(autoincrement())
  memberId  Int
  badgeId   String
  unlockedAt DateTime @default(now())

  member    RoomMember @relation(fields: [memberId], references: [id], onDelete: Cascade)
  badge     Badge      @relation(fields: [badgeId], references: [id], onDelete: Cascade)

  @@unique([memberId, badgeId])
}

// 3. Quy ước Thưởng/Phạt của phòng
model RoomRewardRule {
  id          Int      @id @default(autoincrement())
  roomId      Int
  title       String   // "Bao trà sữa người nhất tháng"
  rewardType  String   // PERK, PENALTY, FUND
  description String?
  
  room        Room     @relation(fields: [roomId], references: [id], onDelete: Cascade)
}
```

---

## 8. Lộ Trình Triển Khai (Roadmap)
- **Phase A (MVP - Hiện tại):** Đã có `effortPoints` (1-4), `FairnessSummaryCard` tính % công bằng theo điểm công sức.
- **Phase B (Kế tiếp):** Thêm Streak Counter (chuỗi ngày hoàn thành) + Bảng vinh danh Top 1 phòng trọ.
- **Phase C (Mở rộng):** Photo proof xác thực bằng hình ảnh + Đổi điểm thưởng quy ước phòng.
