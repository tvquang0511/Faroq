# FairRoom Mobile — iOS Dark Mode Design System
> Tài liệu chuẩn hóa giao diện phong cách **Apple Human Interface Guidelines (HIG) Dark Theme**, lấy cảm hứng từ các ứng dụng iOS cao cấp như **Amy** (Health & Nutrition), Apple Fitness, Apple Health.

---

## 1. Triết Lý Thiết Kế (Design Philosophy)

Ứng dụng hướng tới trải nghiệm **iOS-native 100%**:
1. **OLED Black / True Dark Depth**: Không sử dụng màu xám nhờ nhờ của web. Nền chính là đen sâu (`#000000` hoặc `#121214`), phân lớp bằng các thẻ xám đen nhạt (`#1c1c1e`) tạo chiều sâu thị giác tự nhiên.
2. **Inset Grouped Containers (Card bo góc lùi lề)**: Mọi nhóm thông tin, form cài đặt, danh sách đều được gom trong các khối card bo cong mạnh mẽ (`borderRadius: 16 - 20px`), có lề hai bên màn hình (16 - 20px).
3. **Capsule / Pill Elements**: Các nút lọc (Segmented Controls), nhãn badge, nút tác vụ đều sử dụng hình viên thuốc (Capsule) bo tròn hoàn toàn.
4. **Vibrant Accents on Dark**: Các màu điểm nhấn tương phản cao, phát sáng nhẹ trên nền tối (System Blue, Neon Indigo, Emerald Green, Flame Orange).
5. **Micro-Interactions & Haptics**: Phản hồi xúc giác (Taptic Engine) khi gạt công tắc, bấm checkbox xong việc nhà, chuyển tab.

---

## 2. Bảng Màu Chuẩn (Color Tokens)

| Token | Giá trị HEX | Mô tả & Ứng dụng |
| :--- | :--- | :--- |
| `bg-canvas` | `#121214` / `#000000` | Nền toàn ứng dụng (OLED Dark) |
| `bg-card` (Secondary) | `#1c1c1e` | Nền thẻ Grouped Card, Inset Section |
| `bg-pill` (Tertiary) | `#2c2c2e` | Nền nút viên thuốc, avatar placeholder, search/filter |
| `border-subtle` | `#2c2c2e` / `rgba(255,255,255,0.08)` | Đường viền siêu mảnh (Hairline border) ngăn cách các ô |
| `border-separator` | `#38383a` | Đường gạch ngang phân cách các hàng trong cùng một card |
| `text-primary` | `#ffffff` | Tiêu đề chính, văn bản nổi bật (White) |
| `text-secondary` | `#8e8e93` / `#a1a1aa` | Chú thích, label phụ, mô tả ngắn |
| `accent-blue` | `#0a84ff` | Màu iOS System Blue (Nút chính, link, tab đang chọn) |
| `accent-purple` | `#5e5ce6` | Màu tím neon (Thống kê, huy hiệu đặc biệt, nút VIP) |
| `accent-orange` | `#ff9f0a` | Màu lửa / Streak 🔥, cảnh báo vừa |
| `accent-green` | `#30d158` / `#10b981` | Màu hoàn thành việc nhà, Switch đang bật |
| `accent-red` | `#ff453a` | Quá hạn (Overdue), Đăng xuất, Thao tác nguy hiểm |

---

## 3. Thư Viện Cần Cài Đặt (Dependencies)

### ❌ KHÔNG CẦN tải các bộ UI Component cồng kềnh:
- Không dùng **NativeBase**, **React Native Paper**, **Tamagui**, **Gluestack**: Các thư viện này áp đặt phong cách Android/Material, bundle size nặng, animation không mượt mà và làm sai lệch cảm giác vuốt chạm tự nhiên của iOS.

###  CHỈ CẦN 2 thư viện Companion chính thức của Expo:
```bash
pnpm --filter mobile add expo-linear-gradient expo-haptics
```
1. **`expo-linear-gradient`**: 
   - Tạo thanh tiến độ / thanh chỉ số thống kê bo tròn có dải màu gradient mượt mà (như thanh Carbs/Protein/Fat của app Amy).
2. **`expo-haptics`**: 
   - Kích hoạt bộ rung phản hồi Taptic Engine của iPhone (`Haptics.impactAsync(Haptics.ImpactFeedbackStyle.Light)`) khi tick hoàn thành việc nhà, chuyển tab hoặc bật tắt switch.

---

## 4. Quy Chuẩn Thành Phần Giao Diện (Components Standard)

### 4.1. Thanh Menu Dưới Đáy (Bottom Tab Bar)
- Nền đen bán trong suốt / mờ ảo: `backgroundColor: '#161618'` hoặc `rgba(20, 20, 22, 0.94)`.
- Đường viền trên siêu mảnh: `borderTopColor: '#27272a'`.
- Màu tab đang chọn: `tabBarActiveTintColor: '#0a84ff'` hoặc `#ffffff`.
- Màu tab không chọn: `tabBarInactiveTintColor: '#8e8e93'`.
- Padding đáy: Tự động lùi an toàn cho phím Home Bar của iPhone (`useSafeAreaInsets().bottom`).

### 4.2. Khối Thẻ Inset Grouped (Giống Cài Đặt iOS / Amy)
```tsx
// Cấu trúc chuẩn 1 Inset Grouped Container
<View style={{ marginBottom: 24 }}>
  {/* Tiêu đề nhóm */}
  <Text style={{ 
    color: '#8e8e93', 
    fontSize: 13, 
    fontWeight: '600', 
    textTransform: 'uppercase', 
    letterSpacing: 0.5, 
    marginBottom: 8, 
    marginLeft: 16 
  }}>
    CÀI ĐẶT THIẾT BỊ
  </Text>

  {/* Khối card */}
  <View style={{ 
    backgroundColor: '#1c1c1e', 
    borderRadius: 18, 
    overflow: 'hidden',
    borderWidth: 1,
    borderColor: 'rgba(255,255,255,0.06)'
  }}>
    {/* Hàng 1 */}
    <View style={{ flexDirection: 'row', alignItems: 'center', padding: 16 }}>
      ...
    </View>
    {/* Đường gạch phân cách chừa lề trái 54px */}
    <View style={{ height: 1, backgroundColor: '#2c2c2e', marginLeft: 54 }} />
    {/* Hàng 2 */}
    <View style={{ flexDirection: 'row', alignItems: 'center', padding: 16 }}>
      ...
    </View>
  </View>
</View>
```

### 4.3. Nút Điều Khiển Phân Đoạn (Segmented Pill Control)
- Dùng cho các tab con (ví dụ: *Hôm nay* | *Sắp tới* | *Công bằng* | *Quản lý*).
- Container bọc ngoài: Hình capsule `borderRadius: 24`, nền `#1c1c1e` hoặc `#2c2c2e`, padding 4px.
- Nút con được chọn: Nền `#636366` hoặc `#3a3a3c`, chữ trắng bold. Nút chưa chọn: chữ `#8e8e93`.

---

## 5. Nguyên Tắc Tránh Lỗi Layout Giữa Web F12 và iPhone Thật (Yoga Layout Engine)

| Lỗi gặp phải trên iPhone | Nguyên nhân | Cách khắc phục chuẩn |
| :--- | :--- | :--- |
| **Nút nhảy lung tung, 1 cột méo mó** | Dùng `flexWrap: 'wrap'` + `width: '48%'` + `gap: 10`. Trên iOS Yoga Engine, làm tròn số khiến tổng chiều rộng $> 100\%$, thẻ thứ 2 bị rơi xuống dòng. | Chia layout thành các `<View style={{ flexDirection: 'row', gap: 10 }}>`, mỗi thẻ con có `style={{ flex: 1 }}`. |
| **Thứ trong tuần (T2..CN) bị rớt CN** | Đặt pixel cố định `width: 42`. $7 \times 42 = 294\text{px}$ cộng lề màn hình vượt quá bề ngang màn hình nhỏ. | Dùng `flex: 1` và `aspectRatio: 1` cho cả 7 nút, co giãn mượt trên mọi kích cỡ iPhone (SE đến Pro Max). |
| **Nội dung bị che dưới thanh Tab Bar** | ScrollView đặt `paddingBottom: 32` không đủ bù cho chiều cao Tab Bar + Home Indicator (~84pt). | Đặt `contentContainerStyle={{ paddingBottom: 100 }}` hoặc dùng `useSafeAreaInsets`. |
| **Bàn phím che / đẩy nút nhảy loạn** | `KeyboardAvoidingView` không có offset hoặc offset âm trên iOS. | Cấu hình `behavior={Platform.OS === 'ios' ? 'padding' : undefined}` và `keyboardVerticalOffset={Platform.OS === 'ios' ? 88 : 0}`. |
| **Tràn chữ / Chip đè lên chữ** | Thiếu `flexShrink: 1` trên cụm văn bản hoặc thiếu `numberOfLines={1}`. | Luôn gán `flexShrink: 1` cho Text/Cụm text nằm cạnh Badge/Status Chip. |

---

## 6. Phạm Vi Áp Dụng Cho FairRoom

1. **Thanh Menu (Tabs Layout)**: Chuyển sang Dark Tab Bar chuẩn iOS, làm mờ, icon tinh tế.
2. **Trang Chủ (Home)**: Nền tối `#121214`, thẻ phòng Inset Grouped, nút tạo phòng chuẩn iOS Sheet modal.
3. **Việc Nhà (Chores)**: Khắc phục triệt để lỗi nhảy layout trên iPhone, làm đẹp theo dạng Inset Cards, thanh tiến độ gradient, thanh điểm công bằng trực quan.
4. **Cá Nhân (Profile)**: Giao diện Inset Grouped Settings giống màn hình Amy (Cài đặt tài khoản, Tuỳ chọn giao diện, Bật/tắt thông báo, Rung phản hồi, Đăng xuất).
5. ⚠️ **Phần Chi Phí (Expenses)**: Giữ nguyên vẹn, không chỉnh sửa để đảm bảo tính ổn định hiện tại.
