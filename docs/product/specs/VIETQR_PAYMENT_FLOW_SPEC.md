# FairRoom — Đặc Tả Luồng Trải Nghiệm Quyết Toán VietQR (VietQR Payment Flow Spec)

> **Tài liệu đặc tả luồng người dùng (User Journey & UX Specification)**  
> **Trạng thái:** ĐÃ TRIỂN KHAI HOÀN TẤT (COMPLETED - v1.0)  
> **Phạm vi:** Backend API (`/users/me/bank-account`) + Mobile App (`profile.tsx`, `settle.tsx`, `VietQRCard.tsx`).  
> **Giải quyết bài toán:** Làm thế nào để người dùng thanh toán nợ bằng mã QR ngay trên cùng một chiếc điện thoại một cách mượt mà và trực quan nhất.

---

## 1. Bài Toán UX: Thanh Toán QR Trên Cùng Một Thiết Bị (Single-Device Dilemma)

Khi mã QR hiển thị trên màn hình điện thoại, người dùng **không thể dùng camera của chính chiếc điện thoại đó để quét màn hình của mình**.

Do đó, các ứng dụng thương mại điện tử và tài chính hàng đầu tại Việt Nam (Shopee, TikTok Shop, Sổ Bán Hàng, MoMo) đều chuẩn hóa trải nghiệm thanh toán theo **3 kịch bản sử dụng (User Flows)** dưới đây. FairRoom sẽ hỗ trợ trọn vẹn cả 3 kịch bản này:

---

## 2. Ba Luồng Trải Nghiệm Người Dùng (Detailed User Flows)

```
                                  [MÀN HÌNH QUYẾT TOÁN SETTLE]
                                               │
                                               ▼
                             ┌──────────────────────────────────┐
                             │    Hiển thị Thẻ VietQR Động      │
                             │  (Mã QR + STK + Tên + Số tiền)   │
                             └─────────────────┬────────────────┘
                                               │
               ┌───────────────────────────────┼───────────────────────────────┐
               │                               │                               │
               ▼                               ▼                               ▼
      [KỊCH BẢN 1: QUÉT ẢNH]          [KỊCH BẢN 2: COPY STK]         [KỊCH BẢN 3: 2 MÁY]
   (Chụp màn hình / Lưu ảnh)          (Dành cho người quen dán)     (Ngồi cạnh nhau quét)
               │                               │                               │
               ▼                               ▼                               ▼
    1. Chụp màn hình / Bấm             1. Bấm [Sao chép STK]         1. Bạn A mở mã QR
       nút [Lưu ảnh QR]                   hoặc [Sao chép số tiền]    2. Bạn B cầm máy khác
    2. Mở App Ngân Hàng                2. Mở App Ngân Hàng              quét trực tiếp
    3. Bấm icon QR -> Chọn ảnh         3. Dán STK -> App tự nhận        camera vào màn hình
       từ Thư viện (Gallery)              tên chủ tài khoản             của bạn A
    4. App ngân hàng tự điền           4. Nhập số tiền & chuyển
       STK, số tiền & nội dung                         │                               │
               │                                       │                               │
               └───────────────────────┬───────────────┴───────────────────────────────┘
                                       │
                                       ▼
                       [XÁC NHẬN CHUYỂN TIỀN TRÊN APP NGÂN HÀNG]
                       (Xác thực FaceID / Vân tay / Smart OTP)
                                       │
                                       ▼
                       [QUAY LẠI FAIRROOM: BẤM XÁC NHẬN]
                       (Hệ thống xóa nợ & cập nhật số dư = 0 đ)
```

---

### Kịch bản 1: Quét Từ Ảnh Thư Viện (Luồng Phổ Biến Nhất — 85% Người Dùng VN Quen Thuộc)
* **Bước 1:** Thành viên A nợ B 120.000 đ -> A vào màn hình **Quyết toán**, chọn B và số tiền 120.000 đ.
* **Bước 2:** FairRoom tự sinh mã VietQR của B. A có thể:
  * **Chụp màn hình** chiếc điện thoại của mình.
  * Hoặc bấm nút **`[Lưu ảnh QR vào máy]`** trên thẻ VietQR.
* **Bước 3:** A mở app ngân hàng của mình (Vietcombank, MB, Techcombank, TPBank, MoMo...).
* **Bước 4:** Bấm vào biểu tượng **Quét QR** trên app ngân hàng -> Chọn icon **"Chọn ảnh từ Thư viện / Album"**.
* **Bước 5:** Chọn ảnh vừa chụp màn hình -> App ngân hàng **tự động quét sạch dữ liệu**:
  * Tự nhận diện đúng Ngân hàng & STK của B.
  * Tự điền đúng số tiền `120.000 đ`.
  * Tự điền nội dung chuyển khoản: `FairRoom A tra no B`.
* **Bước 6:** A quét FaceID/vân tay trên app ngân hàng để chuyển tiền -> Tiền về tài khoản B tức thì.
* **Bước 7:** A mở lại FairRoom, bấm **"Xác nhận đã chuyển tiền"** -> Hệ thống cập nhật công nợ hoàn tất!

---

### Kịch bản 2: Sao Chép Nhanh 1 Chạm (Dành Cho Người Quen Chuyển Khoản Truyền Thống)
* **Bước 1:** Thẻ VietQR trong FairRoom có hàng nút sao chép:
  * Nút **`[Sao chép STK]`** (Copy số tài khoản).
  * Nút **`[Sao chép số tiền]`**.
* **Bước 2:** A bấm `[Sao chép STK]` -> FairRoom rung nhẹ (Haptics) và hiện thông báo: *"Đã sao chép STK MBBank: 0987654321"*.
* **Bước 3:** A mở app ngân hàng -> Chọn "Chuyển tiền nhanh 24/7" -> Dán STK -> App ngân hàng tự tra cứu đúng tên của B.
* **Bước 4:** Chuyển tiền và quay lại FairRoom xác nhận.

---

### Kịch bản 3: Hai Bạn Cùng Phòng Ngồi Cạnh Nhau (Quét Trực Tiếp Phone-to-Phone)
* Nếu hai bạn đang ngồi cùng trong phòng trọ:
  * Bạn B mở màn hình nhận tiền (hoặc mở Profile hiển thị mã VietQR cá nhân).
  * Bạn A mở camera hoặc app ngân hàng trên máy mình quét trực diện vào màn hình máy bạn B -> Chuyển tiền trong 2 giây.

---

## 3. Thiết Kế Giao Diện Thẻ VietQR Card (UI Component Specs)

Thẻ VietQR trong FairRoom được thiết kế cao cấp, rõ ràng và đầy đủ tiện ích:

```
┌────────────────────────────────────────────────────────┐
│  🏦 MBBANK (Ngân hàng Quân Đội)                        │
│  TRAN VAN QUANG • 0987654321                           │
├────────────────────────────────────────────────────────┤
│                                                        │
│                 ┌────────────────────┐                 │
│                 │                    │                 │
│                 │     [ MÃ QR ]      │                 │
│                 │    VIETQR NAPAS    │                 │
│                 │                    │                 │
│                 └────────────────────┘                 │
│                                                        │
│          Số tiền chuyển: 120.000 đ                     │
│          Nội dung: FairRoom Minh tra no Quang          │
├────────────────────────────────────────────────────────┤
│  [📋 Sao chép STK]            [📸 Chụp/Lưu ảnh QR]     │
├────────────────────────────────────────────────────────┤
│  💡 Mẹo: Chụp màn hình rồi mở App Ngân Hàng chọn       │
│  "Quét mã từ Thư viện ảnh" để thanh toán tự động nhé!   │
└────────────────────────────────────────────────────────┘
```

---

## 4. Thiết Lập Tài Khoản Ngân Hàng Trong Profile (Setup Flow)

Để nhận được tiền qua VietQR, mỗi thành viên cài đặt 1 lần duy nhất trong trang Cá nhân:

1. Vào tab **Cá nhân (Profile)** -> Mục **"Tài khoản nhận tiền (VietQR)"**.
2. Bấm **"Thiết lập / Chỉnh sửa"** -> Modal hiện ra:
   * **Chọn Ngân hàng:** Danh sách hơn 50 ngân hàng Việt Nam (có logo + tên viết tắt: VCB, MB, TCB, VPB, ACB...).
   * **Số tài khoản:** Bàn phím số.
   * **Tên chủ tài khoản:** Tự động in hoa không dấu (VD: `NGUYEN VAN A`).
3. Bấm **[Lưu thông tin]** -> Hệ thống lưu vào cơ sở dữ liệu và hiển thị huy hiệu: *"Đã kết nối VietQR 🟢"*.

---

## 5. Các Bước Triển Khai Thực Hiện (Implementation Steps)

Để hiện thực hóa trọn vẹn luồng này, chúng ta sẽ thực hiện tuần tự 4 bước:

* [ ] **Bước 1 (Backend Database & API):**
  - Thêm 4 trường vào bảng `User` trong Prisma: `bankBin`, `bankShortName`, `bankAccountNumber`, `bankAccountName`.
  - Viết API `PUT /users/me/bank-account` và cập nhật API `GET /auth/me`.
* [ ] **Bước 2 (Danh mục Ngân hàng Việt Nam):**
  - Tạo file `apps/mobile/src/constants/vietnamBanks.ts` chứa danh sách mã BIN các ngân hàng lớn nhất Việt Nam.
* [ ] **Bước 3 (Giao diện Cài đặt Profile):**
  - Thêm form thiết lập tài khoản ngân hàng trong `profile.tsx`.
* [ ] **Bước 4 (Component VietQRCard & Tích hợp Settle):**
  - Xây dựng component `VietQRCard` có ảnh QR tự sinh từ URL chuẩn Napas247, nút copy STK và hướng dẫn quét ảnh.
  - Tích hợp vào màn hình `settle.tsx`.
