# Faroq — Kế Hoạch & Danh Sách Chuyển Đổi Thương Hiệu (Rebranding Checklist)

> **Tài liệu chiến lược thương hiệu**: Chuyển đổi định vị và nhận diện sản phẩm từ **FairRoom** sang **Faroq**.  
> **Nguyên tắc cốt lõi**: Tên Git repository vẫn giữ nguyên là `FairRoom`. Chỉ thay đổi các thành phần nhận diện sản phẩm, giao diện người dùng, cấu hình app và hạ tầng triển khai.

---

## 1. Ý Nghĩa Thương Hiệu & Câu Chuyện (Brand Story)

Tên gọi **Faroq** là một sự kết hợp độc đáo, mang tính cá nhân hóa cao và giàu ý nghĩa:
- **Fa** (trong *Fair*): Sự công bằng, minh bạch tuyệt đối trong chi tiêu và việc nhà.
- **Ro** (trong *Room*): Không gian sống chung, phòng trọ sinh viên, căn hộ coliving.
- **Q** (chữ cái đầu trong tên tác giả *Quang*): Dấu ấn người sáng lập, tạo nên sự khác biệt độc bản.

> 💡 **Phát âm**: *Fa-rok*  
> **Cảm nhận thương hiệu**: Trẻ trung, ngắn gọn, phong cách các ứng dụng công nghệ quốc tế (tương tự Slack, Figma, Revolut, Monzo), dễ tạo ấn tượng sâu đậm khi thuyết trình đồ án hoặc pitching khởi nghiệp.

---

## 2. Chiến Lược Thời Điểm: Nên Làm Liền Hay Làm Sau?

### 🎯 Lời Khuyên Chiến Lược (Senior Verdict):
**KHÔNG NÊN đổi đồng loạt 100% tất cả mọi thứ ngay bây giờ.** Hãy áp dụng lộ trình **2 giai đoạn (Phased Rollout)**:

1. **Giai đoạn 1 (NÊN LÀM LIỀN — Rủi ro = 0%, Hiệu quả nhận diện cao ngay lập tức)**:
   - Đổi tên hiển thị trên giao diện người dùng (App Name trên điện thoại, trang Web Landing Page, tiêu đề Email Resend, tiêu đề Swagger API).
   - Giúp bạn có ngay giao diện đẹp mắt, thương hiệu **Faroq** đồng bộ để quay video demo, chụp ảnh màn hình nộp báo cáo hoặc thuyết trình.
   - **Không làm ảnh hưởng đến mã nguồn logic hay cơ sở hạ tầng.**

2. **Giai đoạn 2 (LÀM SAU — Khi chuẩn bị Release Production / Mua tên miền / Đóng gói nộp bài cuối kỳ)**:
   - Đổi Package Name / Bundle ID (`com.tvquang.faroq`).
   - Đổi Deep Link Scheme (`faroq://`).
   - Cấu hình lại Google Cloud OAuth (vì đổi scheme hay domain sẽ cần cập nhật lại Client ID & Redirect URI trong Google Cloud Console).
   - Đổi tên miền triển khai chính thức (`faroq.vn` hoặc `faroq.app`).

---

## 3. Danh Sách Chi Tiết Các Thành Phần Cần Thay Đổi

### 📱 Phần 1: Mobile App (`apps/mobile`)

| Thành phần / File | Vị trí hiện tại (`FairRoom`) | Giá trị mới (`Faroq`) | Mức độ ưu tiên |
| :--- | :--- | :--- | :---: |
| [app.json](file:///d:/document/projects/start-up/FairRoom/apps/mobile/app.json) | `"name": "FairRoom"` | `"name": "Faroq"` | 🟢 Làm ngay |
| [app.json](file:///d:/document/projects/start-up/FairRoom/apps/mobile/app.json) | `"slug": "fairroom"` | `"slug": "faroq"` | 🟡 Giai đoạn 2 |
| [app.json](file:///d:/document/projects/start-up/FairRoom/apps/mobile/app.json) | `"scheme": "fairroom"` | `"scheme": "faroq"` | 🟡 Giai đoạn 2 |
| [app.json](file:///d:/document/projects/start-up/FairRoom/apps/mobile/app.json) | `"ios.bundleIdentifier": "com.tvquang.fairroom"` | `"ios.bundleIdentifier": "com.tvquang.faroq"` | 🟡 Giai đoạn 2 |
| [app.json](file:///d:/document/projects/start-up/FairRoom/apps/mobile/app.json) | `"android.package": "com.tvquang.fairroom"` | `"android.package": "com.tvquang.faroq"` | 🟡 Giai đoạn 2 |
| [login.tsx](file:///d:/document/projects/start-up/FairRoom/apps/mobile/src/app/(auth)/login.tsx) | `"Chào mừng đến với FairRoom"` | `"Chào mừng bạn đến với Faroq"` | 🟢 Làm ngay |
| [profile.tsx](file:///d:/document/projects/start-up/FairRoom/apps/mobile/src/app/(tabs)/profile.tsx) | `"FairRoom v1.0.0"` | `"Faroq v1.0.0"` | 🟢 Làm ngay |
| [VietQRCard.tsx](file:///d:/document/projects/start-up/FairRoom/apps/mobile/src/components/features/money/widgets/VietQRCard.tsx) | Cú pháp nội dung chuyển khoản: `FAIRROOM ...` | `FAROQ ...` | 🟢 Làm ngay |
| App Icons & Splash Screen | Logo chữ FairRoom | Logo biểu tượng chữ **F** hoặc **Faroq** cách điệu | 🟢 Làm ngay |

---

### 🌐 Phần 2: Web Landing Page (`apps/web`)

| Thành phần / File | Vị trí hiện tại (`FairRoom`) | Giá trị mới (`Faroq`) | Mức độ ưu tiên |
| :--- | :--- | :--- | :---: |
| [config.ts](file:///d:/document/projects/start-up/FairRoom/apps/web/src/lib/config.ts) | `siteName: 'FairRoom'`<br>`siteUrl: 'https://fairroom.app'` | `siteName: 'Faroq'`<br>`siteUrl: 'https://faroq.app'` (hoặc `faroq.vn`) | 🟢 Làm ngay |
| [layout.tsx](file:///d:/document/projects/start-up/FairRoom/apps/web/src/app/layout.tsx) | `<title>FairRoom - ...</title>` | `<title>Faroq - Quản lý phòng trọ minh bạch & văn minh</title>` | 🟢 Làm ngay |
| [Navbar.tsx](file:///d:/document/projects/start-up/FairRoom/apps/web/src/components/Navbar.tsx) | Logo text: `FairRoom` | Logo text: `Faroq` | 🟢 Làm ngay |
| [Footer.tsx](file:///d:/document/projects/start-up/FairRoom/apps/web/src/components/Footer.tsx) | `© 2026 FairRoom` | `© 2026 Faroq. Created by Quang.` | 🟢 Làm ngay |
| [page.tsx](file:///d:/document/projects/start-up/FairRoom/apps/web/src/app/page.tsx) | Tiêu đề Hero Banner, mô tả tính năng | Cập nhật toàn bộ các tiêu đề và slogan sang `Faroq` | 🟢 Làm ngay |
| [privacy/page.tsx](file:///d:/document/projects/start-up/FairRoom/apps/web/src/app/privacy/page.tsx)<br>[terms/page.tsx](file:///d:/document/projects/start-up/FairRoom/apps/web/src/app/terms/page.tsx) | Chính sách quyền riêng tư & điều khoản FairRoom | Đổi pháp nhân/dịch vụ sang `Faroq` | 🟢 Làm ngay |

---

### ⚙️ Phần 3: Backend API & Transactional Emails (`apps/api`)

| Thành phần / File | Vị trí hiện tại (`FairRoom`) | Giá trị mới (`Faroq`) | Mức độ ưu tiên |
| :--- | :--- | :--- | :---: |
| [main.ts](file:///d:/document/projects/start-up/FairRoom/apps/api/src/main.ts) | `FairRoom API — Hệ Thống...`<br>`FairRoom API Documentation` | `Faroq API — Hệ Thống Quản Lý Phòng Trọ & Chia Sẻ Chi Phí`<br>`Faroq API Documentation` | 🟢 Làm ngay |
| [typedoc.json](file:///d:/document/projects/start-up/FairRoom/apps/api/typedoc.json) | `"name": "FairRoom API Documentation"` | `"name": "Faroq API Documentation"` | 🟢 Làm ngay |
| [mail.service.ts](file:///d:/document/projects/start-up/FairRoom/apps/api/src/modules/mail/mail.service.ts) | Sender Name: `FairRoom Support` | `Faroq Team` hoặc `Faroq Support` | 🟢 Làm ngay |
| [welcome-email.template.ts](file:///d:/document/projects/start-up/FairRoom/apps/api/src/modules/mail/templates/welcome-email.template.ts) | "Chào mừng bạn đến với FairRoom!" | "Chào mừng bạn đến với Faroq!" | 🟢 Làm ngay |
| [verify-email.template.ts](file:///d:/document/projects/start-up/FairRoom/apps/api/src/modules/mail/templates/verify-email.template.ts) | Tiêu đề xác thực tài khoản FairRoom | Tiêu đề xác thực tài khoản Faroq | 🟢 Làm ngay |
| [password-reset.template.ts](file:///d:/document/projects/start-up/FairRoom/apps/api/src/modules/mail/templates/password-reset.template.ts) | Khôi phục mật khẩu FairRoom | Khôi phục mật khẩu Faroq | 🟢 Làm ngay |

---

### ☁️ Phần 4: Hạ Tầng Cloud, Tên Miền & Tích Hợp Thứ Ba

| Nền tảng | Hạng mục cần đổi | Chi tiết thực hiện | Mức độ ưu tiên |
| :--- | :--- | :--- | :---: |
| **Google Cloud Console** | OAuth 2.0 Consent Screen | Đổi App Name từ `FairRoom` sang `Faroq`. Thêm logo Faroq. | 🟡 Giai đoạn 2 |
| **Google Cloud Console** | Authorized Redirect URIs | Cập nhật URL callback theo domain mới (nếu có đổi domain landing page). | 🟡 Giai đoạn 2 |
| **Vercel** | Web Landing Page Domain | Đổi domain miễn phí `faroq.vercel.app` hoặc gắn custom domain (`faroq.vn`). | 🟡 Giai đoạn 2 |
| **Render** | Backend API Service | Tên service: `faroq-api.onrender.com`. | 🟡 Giai đoạn 2 |
| **Expo Application Services (EAS)** | EAS Project | Đổi slug dự án trên dashboard expo.dev thành `faroq`. | 🟡 Giai đoạn 2 |
| **Resend (Email Service)** | Domain gửi thư | Verify tên miền gửi thư chính thức (ví dụ: `notify@faroq.app`). | 🟡 Giai đoạn 2 |

---

## 4. Kế Hoạch Hành Động Khuyến Nghị Cho Bạn

1. **Bước 1 (Hôm nay)**:
   - Duyệt qua danh sách checklist trên.
   - Nếu bạn muốn, tôi có thể **thực hiện ngay Giai đoạn 1** (toàn bộ UI Mobile, Web Landing Page, Swagger API, và Email Templates) chỉ trong 5 phút. Rất an toàn và sạch sẽ!
2. **Bước 2 (Khi chuẩn bị nộp bài / demo lớn)**:
   - Tiến hành đổi tên app trong `app.json` và build lại bản preview APK mới với tên **Faroq**.
3. **Bước 3 (Khi triển khai thật)**:
   - Cân nhắc mua tên miền ngắn gọn như `faroq.vn` hoặc `faroq.app` để nâng tầm thương hiệu lên mức chuyên nghiệp cao nhất.
