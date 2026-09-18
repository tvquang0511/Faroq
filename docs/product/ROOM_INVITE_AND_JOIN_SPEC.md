# FairRoom — Đặc Tả Vòng Đời Thành Viên, Mời & Tham Gia Phòng (Room Lifecycle & Member Management Spec)

> **Tài liệu đặc tả kiến trúc, nghiệp vụ & kịch bản thực tế (Master Product & Technical Spec)**  
> **Phân hệ:** Quản lý Không gian sống, Thành viên & Phòng trọ (`Room & Member Lifecycle`)  
> **Trạng thái:** SẴN SÀNG TRIỂN KHAI (READY FOR IMPLEMENTATION)  
> **Mục tiêu:** Xây dựng luồng mời, tham gia phòng và quản lý thành viên thông minh, an toàn, giải quyết triệt để các rủi ro tài chính (bùng nợ khi rời phòng), mạo danh thành viên và bảo toàn lịch trực nhật.

---

## 🧭 1. Phân Tích Kịch Bản Thực Tế & Góc Khuất Nghiệp Vụ (Edge Cases Analysis)

Phòng trọ là một môi trường sống chung phức tạp với biến động nhân sự thường xuyên (người mới đến, người dọn đi, sinh viên về quê, tốt nghiệp). Thiết kế kỹ thuật bắt buộc phải bao quát 4 nhóm rủi ro:

### 🔴 Nhóm 1: Rủi Ro Tài Chính & Toàn Vẹn Sổ Nợ (Financial Integrity)
1. **Rời phòng khi đang còn nợ hoặc đang được nợ:**
   - *Tình huống:* Bạn A nợ cả phòng 450.000 đ tiền điện nước. Đến cuối tháng A tự ý bấm "Rời phòng". Nếu hệ thống cho phép xóa A, tổng số dư công nợ của phòng sẽ bị mất cân đối toán học ($\sum \text{balance} \neq 0$), con nợ biến mất và các bạn còn lại phải tự chịu thiệt.
   - *Quy tắc nghiệp vụ:* **TUYỆT ĐỐI CHẶN RỜI PHÒNG nếu số dư $\text{balance} \neq 0$**. Người dùng bắt buộc phải quyết toán hết nợ (hoặc nhận đủ tiền người khác trả) để số dư về đúng $0$ mới được bấm "Rời phòng".
2. **Trưởng phòng bắt buộc phải xóa thành viên (Người đó đã bùng nợ dọn đi):**
   - *Tình huống:* Người thuê trọ đã dọn đồ đi mất tích, không thể liên lạc để họ tự vào app bấm quyết toán. Trưởng phòng cần dọn dẹp danh sách phòng.
   - *Quy tắc nghiệp vụ:* **KHÔNG ĐƯỢC XÓA CỨNG (Hard Delete)** bản ghi `RoomMember` trong Database vì sẽ làm đứt gãy quan hệ dữ liệu của hàng chục hóa đơn chi tiêu trong quá khứ.
   - *Giải pháp:* Chuyển trạng thái sang **`isArchived = true` (Thành viên cũ / Đã rời phòng)**. Khi lưu trữ, hệ thống cung cấp 2 tùy chọn xử lý khoản nợ tồn:
     - *Tùy chọn A (Ghi nhận thu tiền ngoài đời):* Tạo 1 giao dịch Settlement tự động đưa số dư về 0.
     - *Tùy chọn B (Phòng gánh nợ chung):* Tạo 1 khoản Expense "Nợ xấu phòng gánh" chia đều cho các thành viên còn lại.

---

### 🟡 Nhóm 2: Nhận Nhầm & Mạo Danh "Thành Viên Ảo" (Virtual Member Claiming)
3. **Mạo danh hoặc chọn nhầm tên:**
   - *Tình huống:* Trưởng phòng đã lập sẵn thành viên ảo tên "Nam" (đang gánh 200.000 đ tiền cọc). Bạn "Cường" mới vào phòng bấm nhầm hoặc cố tình chọn *"Tôi là Nam"*.
   - *Giải pháp:*
     - Cảnh báo rõ ràng: *"Bạn đang liên kết tài khoản với [Nam]. Toàn bộ số nợ/dư và lịch trực nhật của [Nam] sẽ thuộc về tài khoản của bạn."*
     - Bắn thông báo (Activity Log) cho cả phòng: *"Nguyễn Văn Cường đã nhận danh tính thành viên [Nam]"*.
     - **Trưởng phòng có quyền "Hủy liên kết (Unlink Identity)":** Đưa thành viên đó trở lại thành thành viên ảo không có tài khoản, nếu phát hiện có sự nhầm lẫn.
4. **Biệt danh trong phòng khác với Tên tài khoản:**
   - Trưởng phòng đặt tên thân mật là *"Bé Mèo"*, nhưng tài khoản thật là *"Vũ Quang"*. Cho phép thành viên chọn: Giữ biệt danh trong phòng (`displayName: "Bé Mèo"`) hoặc đồng bộ theo tên thật.

---

### 🔵 Nhóm 3: Bảo Mật Không Gian Phòng & Lộ Mã Mời
5. **Lộ mã phòng hoặc link mời bị phát tán:**
   - Mã phòng `FR-8899` để lâu ngày hoặc lỡ gửi vào nhóm chat lớp, người lạ tò mò bấm vào xem trộm số tiền chi tiêu của phòng.
   - *Giải pháp:* Trưởng phòng có tính năng **"Làm mới mã phòng (Regenerate Invite Code)"**. Mã cũ lập tức bị vô hiệu hóa.
6. **Trưởng phòng chuyển trọ (Bàn giao quyền quản lý):**
   - Người tạo phòng ra trường/chuyển đi, các bạn khóa dưới ở lại. Trưởng phòng không thể xóa phòng.
   - *Giải pháp:* **"Chuyển quyền Trưởng phòng (Transfer Ownership)"** cho một thành viên khác trước khi rời đi.

---

### 🟢 Nhóm 4: Bảo Toàn Phân Công Việc Nhà (Chores Continuity)
7. **Xử lý ca trực khi có người vào / ra:**
   - *Người rời phòng:* Tự động gỡ tên người đó khỏi danh sách xoay vòng ca trực nhật (`rotationMemberIds`), tránh việc đến ngày đến giờ bị trống lịch không ai làm.
   - *Người mới vào:* Đưa ra tùy chọn cho trưởng phòng: Có muốn bổ sung bạn mới vào danh sách trực nhật các công việc hiện có hay không.

---

### 🔵 Nhóm 5: Nghiên Cứu Đối Chiếu Splitwise & Bằng Chứng "Chấp Nhận Rủi Ro"
8. **Lỗ hổng "Bắt cóc con tin nợ" (Hostage Debt Trap) trên Splitwise:**
   - Trên Splitwise, bất kỳ ai biết email của bạn đều có thể add bạn vào group ngay lập tức và gán hóa đơn cho bạn mà không cần bạn đồng ý.
   - Khi bị gán nợ, người dùng **không thể tự rời group** vì điều kiện tiên quyết của Splitwise: *"Số dư của bạn phải bằng 0 mới được rời nhóm"*.
   - **Bằng chứng từ tài liệu chính thức của Splitwise (Splitwise Help Center):**
     > *"Splitwise currently does not have a global setting to prevent people from adding you to groups. The platform is designed for shared expense tracking, which typically assumes a level of trust between users."*  
     > *(Nguồn: [Splitwise Help — Someone added me to a group without permission](https://feedback.splitwise.com))*
     >  
     > *"Splitwise does not support group admin roles or specific permissions to prevent members from adding expenses or editing the group. The system is designed with the philosophy that all members should be able to correct mistakes and manage shared debts."*  
     > *(Nguồn: [Splitwise Help — Can I set admin permissions on a group?](https://feedback.splitwise.com))*
   - **Cách giải quyết bị động của Splitwise:** Nạn nhân phải tự bấm "Ghi nhận trả tiền mặt ngoài đời" (Record cash payment) để số dư về 0 ảo rồi mới rời nhóm được, hoặc dùng tính năng "Block User".
   - **Quy tắc vượt trội của FairRoom:** Không phụ thuộc vào "niềm tin mù quáng" (trust-based). Bắt buộc người dùng phải có quyền bấm **[Chấp nhận]** hoặc **[Từ chối]** khi được mời vào phòng, và khi chưa chấp nhận thì **KHÔNG AI ĐƯỢC PHÉP CHIA TIỀN CHO NGƯỜI ĐÓ**.

---

## 🏛️ 2. Kiến Trúc Dữ Liệu (Database Schema Design)

### 2.1. Cập nhật `model Room` trong `schema.prisma`:
```prisma
model Room {
  id   Int @id @default(autoincrement())
  name String

  // Mã mời tham gia phòng (6 ký tự viết hoa không trùng, VD: FR8291)
  inviteCode String? @unique

  // Cấu hình phòng
  billingCycleStartDay Int @default(1)

  // ... các quan hệ hiện có
  invitations RoomInvitation[]
}
```

### 2.2. Bổ sung `model RoomInvitation` trong `schema.prisma`:
```prisma
enum InvitationStatus {
  PENDING
  ACCEPTED
  DECLINED
  CANCELLED
}

model RoomInvitation {
  id Int @id @default(autoincrement())

  roomId Int
  room   Room @relation(fields: [roomId], references: [id], onDelete: Cascade)

  inviterId Int
  inviter   User @relation("SentInvitations", fields: [inviterId], references: [id], onDelete: Cascade)

  inviteeEmail String
  inviteeId    Int?
  invitee      User?   @relation("ReceivedInvitations", fields: [inviteeId], references: [id], onDelete: SetNull)

  // Nếu muốn mời người này để nhận lại vị trí của 1 thành viên ảo đang có nợ
  claimMemberId Int?

  status InvitationStatus @default(PENDING)

  createdAt DateTime @default(now())
  updatedAt DateTime @updatedAt

  @@index([inviteeEmail, status])
  @@index([roomId, status])
}
```

### 2.3. Cập nhật `model RoomMember` trong `schema.prisma`:
```prisma
model RoomMember {
  id Int @id @default(autoincrement())

  roomId Int
  room   Room   @relation(fields: [roomId], references: [id], onDelete: Cascade)

  // Null = thành viên ảo (trưởng phòng tạo hộ khi bạn cùng phòng chưa có app)
  userId Int?
  user   User?   @relation(fields: [userId], references: [id], onDelete: SetNull)

  displayName String
  avatarUrl   String?
  role        RoomRole @default(MEMBER)

  // Đánh dấu thành viên đã dọn đi (tránh xóa cứng làm hỏng dữ liệu lịch sử)
  isArchived  Boolean  @default(false)
  archivedAt  DateTime?

  // ... các quan hệ Expense, Chore, Settlement
}
```

---

## 🌐 3. Thiết Kế API Backend (NestJS Endpoints)

### 3.1. Nhóm API Mời Phòng
#### `GET /rooms/:id/invite`
- **Mục đích:** Lấy mã mời và link chia sẻ Zalo/Messenger (Chỉ thành viên trong phòng mới gọi được).
- **Logic:** Nếu phòng chưa có `inviteCode`, backend tự động sinh 1 mã mới duy nhất.
- **Response:**
  ```json
  {
    "roomId": 1,
    "roomName": "Phòng 302 - KTX",
    "inviteCode": "FR8291",
    "inviteUrl": "https://fairroom.app/join/FR8291",
    "shareMessage": "Tham gia phòng \"Phòng 302 - KTX\" cùng mình trên FairRoom nhé! Mã phòng: FR8291 hoặc bấm vào link: https://fairroom.app/join/FR8291"
  }
  ```

#### `POST /rooms/:id/regenerate-invite`
- **Mục đích:** Trưởng phòng đổi mã phòng mới khi mã cũ bị lộ.
- **Quyền hạn:** Chỉ `OWNER` hoặc `ADMIN`.
- **Response:** Trả về mã mời mới.

---

### 3.2. Nhóm API Tham Gia Phòng & Nhận Danh Tính
#### `POST /rooms/join/preview`
- **Mục đích:** Người chuẩn bị vào phòng nhập mã `FR8291`, app gọi API này để xem trước thông tin phòng và kiểm tra xem có thành viên ảo nào trùng với mình không.
- **Body:** `{ "inviteCode": "FR8291" }`
- **Response:**
  ```json
  {
    "roomId": 1,
    "roomName": "Phòng 302 - KTX",
    "memberCount": 3,
    "isAlreadyMember": false,
    "claimableMembers": [
      { "id": 12, "displayName": "Tuấn Anh" },
      { "id": 15, "displayName": "Bé Mèo" }
    ]
  }
  ```

#### `POST /rooms/join`
- **Mục đích:** Thực hiện gia nhập phòng chính thức.
- **Body:**
  ```json
  {
    "inviteCode": "FR8291",
    "claimMemberId": 12, // null nếu là người mới tinh
    "customDisplayName": "Tuấn Anh" // optional
  }
  ```
- **Logic xử lý trong Transaction:**
  1. Kiểm tra `currentUser` đã là thành viên trong phòng chưa. Nếu đã là thành viên ➔ Trả về thành công luôn (tránh tạo trùng lặp).
  2. Nếu có `claimMemberId`:
     - Kiểm tra `RoomMember` có `id = claimMemberId` thuộc phòng này và `userId === null` (chưa bị ai nhận trước).
     - Cập nhật `userId = currentUser.id`.
     - Cập nhật avatar theo tài khoản thật.
  3. Nếu không có `claimMemberId`:
     - Tạo mới `RoomMember` với `role: MEMBER`, `userId = currentUser.id`.
  4. Nếu `currentUser.primaryRoomId` đang `null` ➔ Tự động đặt phòng này làm phòng chính.

#### `POST /rooms/:id/members/:memberId/unlink`
- **Mục đích:** Trưởng phòng sửa sai nếu có ai đó nhận nhầm danh tính thành viên ảo.
- **Quyền hạn:** Chỉ `OWNER`.
- **Logic:** Đặt lại `userId = null` cho `RoomMember` đó.

---

### 3.3. Nhóm API Lời Mời Trực Tiếp Trong App (In-App Direct Invitations by Email)

#### `POST /rooms/:id/invitations`
- **Mục đích:** Trưởng phòng gửi lời mời qua email cho bạn cùng phòng.
- **Body:**
  ```json
  {
    "email": "nam@gmail.com",
    "claimMemberId": 12 // optional: nếu muốn mời Nam nhận lại vị trí của thành viên ảo
  }
  ```
- **Logic:**
  1. Kiểm tra email hợp lệ.
  2. Tạo bản ghi `RoomInvitation` với `status: PENDING`.
  3. Không thêm người này vào `RoomMember` ngay, **người này chưa có mặt trong danh sách chia tiền** cho đến khi tự bấm chấp nhận!

#### `GET /rooms/invitations/me`
- **Mục đích:** Khi người dùng mở app, lấy danh sách các lời mời vào phòng đang chờ duyệt.
- **Response:**
  ```json
  [
    {
      "id": 1,
      "roomId": 10,
      "roomName": "Phòng 302 - KTX",
      "inviter": {
        "displayName": "Vũ Quang",
        "avatarUrl": "..."
      },
      "claimMember": {
        "id": 12,
        "displayName": "Nam",
        "currentBalance": -250000
      },
      "createdAt": "2026-09-18T10:00:00Z"
    }
  ]
  ```

#### `POST /rooms/invitations/:id/accept`
- **Mục đích:** Người dùng đồng ý gia nhập phòng (và tiếp quản nợ nếu có `claimMemberId`).
- **Logic:**
  1. Đổi `status` thành `ACCEPTED`.
  2. Thêm vào `RoomMember` (hoặc liên kết vào `claimMemberId`).
  3. Tự động gán làm `primaryRoomId` nếu chưa có.
  4. Trả về thông tin phòng.

#### `POST /rooms/invitations/:id/decline`
- **Mục đích:** Người dùng từ chối lời mời (tránh bị gán nợ oan hoặc add nhầm).
- **Logic:** Đổi `status` thành `DECLINED`. Lời mời biến mất khỏi app.

---

### 3.4. Nhóm API Rời Phòng & Bảo Toàn Sổ Nợ
#### `POST /rooms/:id/leave`
- **Mục đích:** Thành viên tự nguyện rời phòng.
- **Ràng buộc kiểm tra nghiêm ngặt:**
  1. Nếu người rời là `OWNER` duy nhất và phòng còn thành viên khác ➔ Báo lỗi: *"Bạn là Trưởng phòng. Vui lòng chuyển quyền Trưởng phòng cho người khác trước khi rời đi!"*
  2. Tính toán số dư công nợ của người này:
     - Nếu `balance < 0`: Báo lỗi: *"Bạn đang nợ cả phòng ${Math.abs(balance).toLocaleString()} đ. Vui lòng thanh toán hết nợ trước khi rời phòng!"*
     - Nếu `balance > 0`: Báo lỗi: *"Phòng đang nợ bạn ${balance.toLocaleString()} đ. Vui lòng nhắc các bạn thanh toán hết cho bạn trước khi rời phòng!"*
  3. Nếu `balance === 0`:
     - Chuyển `isArchived = true`, `archivedAt = now()`.
     - Gỡ người này khỏi tất cả các ca phân công trực nhật tương lai.
     - Nếu phòng này đang là `primaryRoomId` của user ➔ Đặt lại `primaryRoomId = null`.

#### `POST /rooms/:id/transfer-owner`
- **Mục đích:** Bàn giao vai trò Trưởng phòng.
- **Body:** `{ "newOwnerMemberId": 15 }`
- **Logic:** Đổi role của mình thành `ADMIN` hoặc `MEMBER`, đổi role của người được chỉ định thành `OWNER`.

---

## 📱 4. Thiết Kế Giao Diện Người Dùng (Mobile Wireframes & UX)

### 4.1. Thẻ Mời Bạn Cùng Phòng Tại `room/[id]/members.tsx`
Nằm nổi bật ở đầu danh sách thành viên:
```
┌────────────────────────────────────────────────────────┐
│  👥 Mời bạn cùng phòng vào nhóm                        │
│  Mã phòng: [ FR-8291 ]  [📋 Sao chép]                  │
│                                                        │
│  ┌─────────────────────────┐  ┌─────────────────────┐  │
│  │ 💬 Gửi link qua Zalo   │  │ 📷 Mã QR tại phòng  │  │
│  └─────────────────────────┘  └─────────────────────┘  │
└────────────────────────────────────────────────────────┘
```
- Khi bấm **[Gửi link qua Zalo]**: Mở hộp thoại chia sẻ của điện thoại (iOS / Android Share Sheet) với tin nhắn định dạng chuẩn.
- Khi bấm **[Mã QR tại phòng]**: Mở modal phóng to mã QR để bạn cùng phòng quét bằng Camera điện thoại.

---

### 4.2. Modal Tham Gia Phòng Tại Trang Chủ (`JoinRoomModal.tsx`)
Tại Trang chủ, bấm nút **[+ Vào phòng]** mở modal:

- **Bước 1: Nhập mã phòng**
  - Ô nhập chữ to nổi bật, tự động in hoa và loại bỏ khoảng trắng.
  - Nút "Kiểm tra phòng".

- **Bước 2: Xem trước & Xác nhận danh tính**
  - Hiển thị: *"Phòng 302 - KTX (3 thành viên)"*.
  - Nếu phòng có thành viên ảo chưa có tài khoản:
    ```
    Trưởng phòng đã thêm bạn trước đó chưa?
    ⚪ Tuấn Anh (Đang ghi nhận 2 khoản chi)
    ⚪ Bé Mèo
    🔘 Tôi là thành viên mới
    ```
  - Nút **"Tham gia phòng ngay"** (Màu xanh iOS `#0a84ff`) ➔ Rung phản hồi Haptic Success ➔ Tự động mở vào phòng!

---

## 🗓️ 5. Phân Kỳ Triển Khai (Phased Rollout)

- **Giai đoạn P0 (Làm ngay đợt này):**
  1. Bổ sung `inviteCode` vào Database và tự động sinh mã khi tạo phòng.
  2. API `GET /rooms/:id/invite`, `POST /rooms/join/preview`, `POST /rooms/join`.
  3. Thẻ chia sẻ mã phòng & gửi link Zalo trong màn hình Thành viên.
  4. Modal tham gia phòng 2 bước tại Trang chủ (Nhập mã + Nhận danh tính thành viên ảo).
  5. Chặn tham gia trùng lặp.
  6. Rào chắn chặn rời phòng nếu còn nợ (`POST /rooms/:id/leave`).

- **Giai đoạn P1 (Kế tiếp sau P0):**
  7. Tính năng Đổi trưởng phòng (`transfer-owner`).
  8. Tính năng Làm mới mã mời (`regenerate-invite`).
  9. Xóa thành viên sang trạng thái `Archived` kèm tùy chọn xử lý nợ.
