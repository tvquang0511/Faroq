# FairRoom — Đặc Tả Hệ Thống Thống Kê, Bảng Vinh Danh & Cơ Chế Tính Điểm Công Bằng (Fairness & Analytics Specification)

> **Tài liệu đặc tả nghiệp vụ sản phẩm (Product Specification)**  
> **Mục tiêu:** Định lượng hóa và trực quan hóa tính "Công bằng" trong phòng trọ, xóa bỏ mọi tranh cãi về việc nhà và tiền bạc giữa các thành viên.

---

## 1. Triết Lý Công Bằng Của FairRoom (Core Philosophy)

> *"Fairness is not identical equality — Công bằng không phải là cào bằng máy móc, mà là sự cân bằng tương đối giữa công sức và trách nhiệm."*

Trong một phòng trọ sinh viên hoặc người đi làm trẻ:
1. **Khác biệt về độ cực nhọc:** Không thể coi việc "Đổ rác" (mất 2 phút) tương đương với "Cọ nhà vệ sinh" (mất 30 phút).
2. **Khác biệt về thời gian biểu:** Có người học sáng, có người làm thêm ca tối, có người cuối tuần về quê (`AwayPeriod`).
3. **Mục tiêu của thống kê:** Không phải để phán xét ai lười biếng hay tạo xung đột, mà là **gương soi khách quan** giúp cả phòng nhìn thấy sự thật bằng số liệu, từ đó tự giác điều chỉnh trong hòa khí.

---

## 2. Cơ Chế Thang Điểm Công Sức Việc Nhà (Effort Points System)

### 2.1. Bảng Điểm Chuẩn (Effort Points Benchmark)
Mỗi công việc nhà trong FairRoom bắt buộc gắn với một mức điểm công sức:

| Mức độ | Điểm | Loại công việc mẫu | Thời gian ước tính |
| :--- | :---: | :--- | :--- |
| **Nhẹ (Light)** | **5đ** | Đổ rác, tưới cây, mua đồ lặt vặt chung | 2 - 5 phút |
| **Vừa (Medium)** | **10đ** | Rửa bát sau bữa ăn, quét nhà, lau bàn ăn | 10 - 15 phút |
| **Nặng (Heavy)** | **15đ** | Lau sàn nhà, cọ bếp ga/bếp từ, giặt & phơi đồ | 20 - 30 phút |
| **Cực nhọc (Hardcore)**| **25đ** | Cọ bồn cầu & nhà vệ sinh, tổng vệ sinh phòng cuối tuần | 30 - 60 phút |

### 2.2. Quy Tắc Chuyển Nhượng & Thưởng Phạt Điểm
1. **Làm thay bạn (`Cover Request`):**
   - Thành viên A nhờ B làm hộ việc cọ WC (25đ).
   - Khi B bấm hoàn thành: **B được cộng +25đ**, hệ thống ghi nhận B đã hỗ trợ A.
2. **Đổi ca (`Swap Request`):**
   - Hai thành viên hoán đổi ngày trực nhật, điểm công sức tính cho người thực tế hoàn thành.
3. **Chuỗi ngày chăm chỉ (`Streak 🔥`):**
   - Hoàn thành việc đúng hạn liên tiếp 3 ca: Thưởng huy hiệu Streak + 10% điểm thưởng.
4. **Quá hạn / Trốn việc (`Overdue`):**
   - Quá 24h không hoàn thành mà không báo lý do/không nhờ hộ: Giảm điểm đóng góp tuần, hệ thống gợi ý gán thêm ca nhẹ vào tuần sau để bù lại.

---

## 3. Thuật Toán Tính Chỉ Số Công Bằng Tuần (Weekly Fairness Score)

Hệ thống tính toán lại chỉ số công bằng vào cuối mỗi tuần hoặc theo thời gian thực (Real-time).

### 3.1. Công thức toán học (Độ lệch chuẩn chuẩn hóa):

$$E_{\text{expected}} = \frac{\sum_{i=1}^{N} E_i}{N}$$

$$\text{Deviation}_{\text{total}} = \sum_{i=1}^{N} |E_i - E_{\text{expected}}|$$

$$\text{Fairness Score} = 100 \times \left(1 - \frac{\text{Deviation}_{\text{total}}}{2 \times \sum_{i=1}^{N} E_i}\right)$$

*Trong đó:*
* $N$: Số lượng thành viên hoạt động trong phòng ($N > 1$).
* $E_i$: Tổng điểm công sức thành viên thứ $i$ đã tích lũy trong tuần.
* Giá trị trả về: Được chặn từ $0\%$ đến $100\%$.

### 3.2. Bảng Phân Loại Trạng Thái (Status Tier):

| Chỉ số Fairness | Nhãn hiển thị | Màu sắc | Ý nghĩa đời sống phòng trọ |
| :---: | :--- | :--- | :--- |
| **80% — 100%** | **Rất cân bằng** | 🟢 `#30d158` | Mọi người chia sẻ công việc đều đặn, không ai bị quá tải. |
| **50% — 79%** | **Hơi lệch** | 🟠 `#ff9f0a` | Có 1 người đang làm nhiều hơn bình thường, các thành viên khác nên chủ động làm giúp. |
| **0% — 49%** | **Cần chú ý** | 🔴 `#ff453a` | Có sự mất cân bằng nghiêm trọng (1 người gánh toàn bộ phòng hoặc nhiều việc bị bỏ bê). |

---

## 4. Bảng Xếp Hạng Phòng Trọ (Room Chore Leaderboard)

### 4.1. Cấu Trúc Bảng Vinh Danh
Hiển thị định kỳ theo tuần (Thứ 2 đến Chủ nhật) và theo tháng:
* **Top 1 🥇 (Chiến thần dọn dẹp):** Người có tổng điểm công sức cao nhất.
* **Top 2 🥈 (Bạn cùng phòng gương mẫu):** Người hoàn thành việc đều đặn.
* **Cần nỗ lực 🥉:** Người có điểm công sức thấp nhất tuần.

### 4.2. Thanh Tỷ Lệ Đóng Góp (% Contribution Bar)
Biểu đồ thanh trực quan thể hiện miếng bánh công việc trong phòng:
* Minh: `45%` (135đ)
* Quang: `35%` (105đ)
* An: `20%` (60đ)

### 4.3. Hệ Thống Huy Hiệu Vui Vẻ (Fun Badges)
| Huy hiệu | Tên gọi | Điều kiện đạt được |
| :---: | :--- | :--- |
| 👑 | **Chiến thần diệt rác** | Đổ rác đúng giờ 5 lần liên tiếp trong tháng |
| 🧽 | **Dũng sĩ bồn cầu** | Xung phong cọ nhà vệ sinh $\ge 3$ lần/tháng |
| 🤝 | **Cứu cánh tình bạn** | Nhận làm thay (Cover) cho bạn cùng phòng $\ge 3$ lần |
| 🔥 | **Chuỗi bất bại** | Đạt Streak hoàn thành việc nhà 7 ngày liên tục |
| 👻 | **Bậc thầy né việc** | Tỷ lệ trễ hạn > 40% (Nhắc khéo để cải thiện) |

---

## 5. Thống Kê Tài Chính & Độ Sòng Phẳng (Financial Analytics)

Bên cạnh việc nhà, sự công bằng về tiền bạc được phản ánh qua:

### 5.1. Cơ Cấu Chi Tiêu Phòng Trọ (Expense Breakdown by Category)
Biểu đồ phân bổ tỷ lệ dòng tiền trong chu kỳ thanh toán:
* 🛒 **Ăn uống & Đi chợ:** `42%`
* ⚡ **Điện & Nước:** `30%`
* 🏠 **Tiền phòng:** `20%`
* 🧻 **Đồ dùng chung (Nước giặt, gia vị...):** `8%`

### 5.2. Chỉ Số Sòng Phẳng Cá Nhân (Financial Punctuality)
* **Ai là người hay ứng tiền trước:** Thống kê tổng số tiền từng thành viên đã bỏ ra để chi trả cho phòng.
* **Tốc độ thanh toán nợ (Settlement Speed):** Đánh giá thời gian từ khi có thông báo nợ đến khi bấm thanh toán (Trả ngay trong ngày / Trả sau 3 ngày / Nợ dai).

---

## 6. Điểm Sức Khỏe Phòng Trọ (Overall Room Harmony Score)

Chỉ số tổng hợp duy nhất đại diện cho độ hòa thuận và văn minh của cả căn phòng:

$$\text{Room Harmony} = (0.5 \times \text{Chore Fairness}) + (0.5 \times \text{Financial Health})$$

* **Chore Fairness:** Điểm công bằng việc nhà (0 - 100).
* **Financial Health:** Điểm tài chính (100 điểm trừ đi số ngày nợ tồn đọng chưa quyết toán).
* **Hiển thị:** Huy hiệu sao vàng trên Trang chủ:
  > *"Điểm hòa thuận phòng trọ: **94/100** 🎉 Phòng bạn là phòng trọ kiểu mẫu!"*

---

## 7. Giao Diện & Kế Hoạch Triển Khai Vào Code (Implementation Blueprint)

### 7.1. Tận Dụng Sẵn API Backend:
- Gọi API: `GET /chores/room/:roomId/fairness?date=YYYY-MM-DD`
- Đã trả về đầy đủ: `fairnessScore`, `statusLabelVi`, `totalEffort`, `members[]` (`totalEffort`, `completedCount`, `contributionPercent`).

### 7.2. Các Màn Hình Cần Tích Hợp:
1. **Trang Chủ (`(tabs)/index.tsx`):** Thêm widget nhỏ gọn hiển thị `Fairness Score (%)` và Streak 🔥 của bạn.
2. **Màn Hình Việc Nhà (`room/[id]/chores.tsx`):** Kích hoạt component `FairnessSummaryCard` vào Sub-tab Nhật ký hoặc Lịch trình.
3. **Màn Hình Thống Kê Riêng Biệt (`room/[id]/analytics.tsx`):** Màn hình chuyên sâu tổng hợp Bảng xếp hạng điểm, biểu đồ đóng góp và biểu đồ chi tiêu.
