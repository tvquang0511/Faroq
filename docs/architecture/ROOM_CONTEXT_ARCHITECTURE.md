# FairRoom — Kiến Trúc Room Context & Primary Room

> **Tài liệu kiến trúc hệ thống (System Architecture Specification)**  
> **Trạng thái:** ĐÃ TRIỂN KHAI HOÀN TẤT (IMPLEMENTED - v1.0)  
> **Mục tiêu:** Quản lý không gian sống linh hoạt khi một người dùng tham gia nhiều phòng (Căn hộ chính, Nhóm du lịch, Phòng trọ cũ), luôn có một Phòng Chính (`primaryRoomId`) làm bối cảnh mặc định.  
> **Phạm vi áp dụng:** Database (PostgreSQL) + Backend API (NestJS) + Frontend App (React Native Expo).

---

## 1. Bối cảnh

FairRoom hiện lấy `Room` làm context chính cho Expenses, Chores, Members, Settlements... Đây là hướng phù hợp với product vision: FairRoom là một **Shared Living OS**, trong đó Money và Chores xoay quanh một căn phòng/nhóm người sống chung.

Tuy nhiên, về UX có một vấn đề:

- Một user có thể tham gia nhiều Room.
- Không phải Room nào cũng có mức độ sử dụng giống nhau.
- Room chính thường là căn phòng đang ở.
- Một Room khác có thể chỉ là một nhóm tạm thời, ví dụ `Đà Lạt 2026`, chủ yếu để lưu expense.
- Nếu bắt user chọn Room mỗi lần vào Money hoặc Chores, navigation sẽ trở nên rối và tăng thao tác.

### Đề xuất

Tách rõ ba khái niệm:

```text
User
 │
 ├── Primary Room        ← preference lâu dài của user
 │
 ├── Current Context     ← context UI hiện tại
 │
 └── Memberships         ← tất cả Room user tham gia
```

Trong đó:

- **Primary Room:** Room được user ưu tiên sử dụng.
- **Current Context:** Room mà màn hình hiện tại đang hiển thị.
- **Membership:** danh sách tất cả Room user có quyền truy cập.

> `Primary Room` là mặc định, **không phải Room duy nhất**.

---

# 2. Product / UX Principle

## 2.1. Không bắt user chọn Room liên tục

Thay vì:

```text
Money
 ↓
Chọn Room?
 ↓
Room 302
 ↓
Xem expense
```

nên là:

```text
Money
 ↓
⭐ Room 302
 ↓
Xem expense ngay
```

Nếu user muốn xem Room khác:

```text
⭐ Room 302 ▼
       ↓
 ┌─────────────────┐
 │ ⭐ Room 302      │
 │ 🏖 Đà Lạt 2026   │
 │ 🏠 Nhà cũ        │
 └─────────────────┘
```

## 2.2. Primary Room xuất hiện nổi bật

Ở các module thường xuyên dùng:

- Home
- Money
- Chores
- Calendar

Primary Room nên là context mặc định.

Ví dụ:

```text
MONEY

┌─────────────────────────────┐
│ ⭐ ROOM 302                 │
│                             │
│ Balance       +320.000 ₫    │
│ Spending      4.820.000 ₫   │
│                             │
│ [ + Thêm chi tiêu ]         │
└─────────────────────────────┘

Other rooms
─────────────────────────────
Đà Lạt 2026        1.240.000 ₫
Nhà cũ             0 ₫
```

Không nên hiển thị tất cả Room với mức độ quan trọng ngang nhau.

---

# 3. Database

## 3.1. Hiện trạng

Schema hiện tại đã có quan hệ:

```text
User
 ↓
RoomMember
 ↓
Room
```

và các domain object như `Expense`, `Settlement`, `Chore` đều gắn với `Room`.

`User` hiện chưa có field thể hiện Room được ưu tiên.

## 3.2. Thêm Primary Room vào User

Đề xuất thêm:

```prisma
model User {
  id          Int     @id @default(autoincrement())
  email       String  @unique
  displayName String
  avatarUrl   String?
  emailVerifiedAt DateTime?

  primaryRoomId Int?

  primaryRoom Room? @relation(
    "UserPrimaryRoom",
    fields: [primaryRoomId],
    references: [id],
    onDelete: SetNull
  )

  createdAt DateTime @default(now())
  updatedAt DateTime @updatedAt

  memberships RoomMember[]
  authIdentities AuthIdentity[]
  sessions Session[]
  passwordResetTokens PasswordResetToken[]
  emailVerificationTokens EmailVerificationToken[]
}
```

Trong `Room`:

```prisma
model Room {
  id   Int @id @default(autoincrement())
  name String

  primaryForUsers User[] @relation("UserPrimaryRoom")

  billingCycleStartDay Int @default(1)

  members          RoomMember[]
  expenses         Expense[]
  settlements      Settlement[]
  categories       ExpenseCategory[]
  presets          ExpensePreset[]
  incomes          Income[]
  incomeCategories IncomeCategory[]
  chores           Chore[]
  coverRequests    CoverRequest[]
  swapRequests     SwapRequest[]
  awayPeriods      AwayPeriod[]
  choreActivities  ChoreActivity[]

  createdAt DateTime @default(now())
  updatedAt DateTime @updatedAt
}
```

## 3.3. Vì sao đặt `primaryRoomId` ở User?

Không nên đặt:

```prisma
Room {
  isPrimary Boolean
}
```

vì Primary Room là **primary đối với từng user**, không phải thuộc tính chung của Room.

Ví dụ:

```text
Room 302

Quang → Primary
Minh  → Primary
Nam   → không Primary
```

Hoàn toàn hợp lệ.

Một Room có thể là Primary Room của nhiều user.

---

# 4. Có nên thêm RoomType?

## Đề xuất: Có, nhưng không bắt buộc cho Phase 1

Có thể thêm:

```prisma
enum RoomType {
  LIVING
  TRIP
  OTHER
}
```

và:

```prisma
model Room {
  id   Int @id @default(autoincrement())
  name String

  type RoomType @default(LIVING)

  ...
}
```

### Ý nghĩa

```text
LIVING
  → căn phòng / căn nhà đang sống
  → Money + Chores + Calendar + Fairness

TRIP
  → chuyến đi
  → Money + Members + Trip Summary
  → không cần Chores

OTHER
  → context khác
```

Ví dụ:

```text
⭐ Room 302
type = LIVING

🏖 Đà Lạt 2026
type = TRIP
```

### Lưu ý

Không nên dùng `RoomType` để quyết định quyền truy cập.

Nó chỉ là **semantic/product metadata** để frontend biết nên ưu tiên module nào.

---

# 5. Primary Room và Current Room phải khác nhau

Đây là điểm quan trọng.

## Primary Room

Lưu trong database:

```text
user.primaryRoomId
```

Ví dụ:

```text
Quang
primaryRoomId = Room 302
```

## Current Room

Không cần lưu database.

Đây là state của frontend:

```text
currentRoomId = 17
```

Ví dụ:

```text
User mở Money
→ currentRoomId = 17

User chuyển sang Đà Lạt
→ currentRoomId = 25

User thoát màn hình
→ currentRoomId có thể reset về primaryRoomId
```

Không nên lưu `currentRoomId` vào User vì nó là trạng thái UI tạm thời.

---

# 6. Quy tắc chọn Primary Room

## 6.1. User chưa có Primary Room

Khi user mới tạo/join Room đầu tiên:

```text
User
 ↓
Room 302
 ↓
primaryRoomId = 302
```

Có thể tự động set Room đầu tiên thành Primary.

## 6.2. User có nhiều Room

Không tự động đổi Primary chỉ vì user vừa join Room mới.

Ví dụ:

```text
Primary:
⭐ Room 302

New:
Đà Lạt 2026
```

Primary vẫn là Room 302.

## 6.3. User chủ động đổi Primary

Trong Room Switcher hoặc Room Settings:

```text
Room 302
⭐ Primary Room

Đà Lạt 2026
[Set as Primary]
```

Sau khi chọn:

```text
Đà Lạt 2026
⭐ Primary Room
```

## 6.4. Primary Room bị xoá

Database:

```text
onDelete: SetNull
```

Khi Room bị xoá:

```text
primaryRoomId = null
```

Backend/frontend phải chọn fallback.

Fallback đề xuất:

```text
1. Room membership gần nhất
2. Nếu không còn Room → empty state
```

Không nên âm thầm chọn một Room khác làm Primary nếu user chưa biết.

---

# 7. Backend Architecture

Backend cần coi `primaryRoomId` là **user preference**, nhưng vẫn phải kiểm tra quyền.

## 7.1. Set Primary Room

API đề xuất:

```http
PUT /users/me/preferences/primary-room
```

Body:

```json
{
  "roomId": 17
}
```

Backend flow:

```text
Request
 ↓
Authenticate user
 ↓
Get roomId
 ↓
Check Room exists
 ↓
Check RoomMember(userId, roomId)
 ↓
Update User.primaryRoomId
 ↓
Return updated preference
```

### Quan trọng

Không được chỉ làm:

```sql
UPDATE users
SET primary_room_id = 17
WHERE id = currentUser
```

mà không kiểm tra membership.

Nếu không, user có thể trỏ `primaryRoomId` tới Room mà họ không thuộc.

---

# 8. Backend Service

Có thể tạo service:

```text
UserPreferenceService
```

Ví dụ:

```text
setPrimaryRoom(userId, roomId)
getPrimaryRoom(userId)
clearPrimaryRoom(userId)
```

Pseudo-code:

```ts
async function setPrimaryRoom(userId: number, roomId: number) {
  const membership = await prisma.roomMember.findUnique({
    where: {
      roomId_userId: {
        roomId,
        userId,
      },
    },
  });

  if (!membership) {
    throw new ForbiddenError(
      "You are not a member of this room"
    );
  }

  return prisma.user.update({
    where: { id: userId },
    data: {
      primaryRoomId: roomId,
    },
    include: {
      primaryRoom: true,
    },
  });
}
```

---

# 9. API cho App Bootstrap

Đề xuất API hiện tại/user session trả về:

```json
{
  "user": {
    "id": 1,
    "displayName": "Quang"
  },
  "primaryRoom": {
    "id": 17,
    "name": "Room 302",
    "type": "LIVING"
  },
  "rooms": [
    {
      "id": 17,
      "name": "Room 302",
      "type": "LIVING"
    },
    {
      "id": 25,
      "name": "Đà Lạt 2026",
      "type": "TRIP"
    }
  ]
}
```

Frontend không phải gọi nhiều API chỉ để xác định context ban đầu.

---

# 10. Room Context API

Nên có một abstraction ở backend:

```text
RoomContext
```

Ví dụ:

```ts
{
  roomId,
  room,
  membership,
  isPrimary,
  roomType
}
```

Các API Money/Chores vẫn nhận `roomId` rõ ràng:

```http
GET /rooms/:roomId/expenses
GET /rooms/:roomId/balance
GET /rooms/:roomId/chores
```

Không nên làm tất cả API phụ thuộc trực tiếp vào `primaryRoomId`.

### Vì sao?

Vì user vẫn phải xem được Room khác:

```text
Primary Room
     ↓
default context

Other Room
     ↓
explicit context
```

---

# 11. Backend Authorization

Mọi endpoint Room-specific vẫn phải check:

```text
currentUser
   ↓
RoomMember
   ↓
roomId
```

Ví dụ:

```text
GET /rooms/25/expenses
```

phải kiểm tra:

```text
User có phải member của Room 25 không?
```

Không được suy luận:

```text
25 == primaryRoomId
```

vì Primary Room chỉ là preference.

---

# 12. Frontend Architecture

Frontend nên có một `RoomContext`.

Ví dụ SwiftUI:

```text
AppState
 ├── currentUser
 ├── rooms
 ├── primaryRoom
 └── currentRoom
```

Hoặc tách:

```text
UserPreferencesStore
RoomContextStore
```

### RoomContextStore

```text
primaryRoomId
currentRoomId
rooms
```

Logic:

```text
App launch
 ↓
currentRoomId = primaryRoomId
```

Khi user chọn Room khác:

```text
currentRoomId = selectedRoomId
```

Khi cần reset:

```text
currentRoomId = primaryRoomId
```

---

# 13. Room Switcher

Đây nên là component dùng chung.

Ví dụ:

```text
┌──────────────────────────┐
│ ⭐ Room 302          ˅   │
└──────────────────────────┘
```

Tap:

```text
┌──────────────────────────┐
│ YOUR ROOMS               │
│                          │
│ ⭐ Room 302              │
│   4 members              │
│                          │
│ 🏖 Đà Lạt 2026           │
│   4 members              │
│                          │
│ + Create Room             │
└──────────────────────────┘
```

Component này nên xuất hiện ở:

- Home
- Money
- Chores
- Calendar
- Fairness nếu màn hình này có Room context

Không cần xuất hiện ở:

- Login
- Register
- Global settings
- Profile cá nhân

---

# 14. Home UX

Home nên ưu tiên Primary Room.

```text
Good morning, Quang 👋

⭐ ROOM 302
────────────────────

💰 Money
You are owed 320.000 ₫

🧹 Today
Đổ rác · 19:00

⚖️ Fairness
93

[ View Room ]

────────────────────

Other Rooms
Đà Lạt 2026
```

Mục tiêu:

> Khi mở app, user biết ngay hôm nay trong Room chính có chuyện gì.

Không biến Home thành danh sách database của tất cả Room.

---

# 15. Money UX

## Primary Room

Khi user vào Money:

```text
Money

⭐ Room 302 ▼

┌───────────────────────────┐
│ Balance                   │
│                           │
│ +320.000 ₫                │
│ You are owed              │
│                           │
│ [ + Add Expense ]         │
└───────────────────────────┘

Recent
───────────────────────────

⚡ Electricity
450.000 ₫
Quang paid

🛒 Groceries
280.000 ₫
Minh paid
```

Đây là khu vực lớn nhất.

## Other Rooms

Có thể hiển thị nhỏ hơn:

```text
Other rooms

🏖 Đà Lạt 2026
1.240.000 ₫ spent
```

Nếu user switch:

```text
Money
↓
Room Switcher
↓
Đà Lạt 2026
```

Toàn bộ content chuyển sang context của Room đó.

---

# 16. Chores UX

Chores cần ưu tiên Primary Room mạnh hơn Money.

```text
Chores

⭐ Room 302 ▼

TODAY

┌─────────────────────────────┐
│ 🧹 Đổ rác                  │
│ Today · 19:00              │
│ Assigned to you            │
│                             │
│              [ Complete ]   │
└─────────────────────────────┘

UPCOMING

🧽 Lau bếp · Tomorrow
🚿 Dọn phòng tắm · Fri
```

Nếu user switch sang:

```text
🏖 Đà Lạt 2026
```

và Room này không có chore:

```text
Chores

🏖 Đà Lạt 2026

No chores in this room.

This room looks like a trip room.
Try Money or Trip Summary.
```

Không nên để:

```text
No chores
```

một cách vô nghĩa.

---

# 17. Room Type ảnh hưởng UX

Nếu:

```text
Room.type = LIVING
```

Bottom navigation / actions ưu tiên:

```text
Home
Money
Chores
More
```

Nếu:

```text
Room.type = TRIP
```

có thể ưu tiên:

```text
Overview
Money
Members
More
```

Tuy nhiên **không nên implement navigation động quá sớm**.

Phase 1 chỉ cần:

```text
RoomType = metadata
```

Phase sau mới dùng để customize experience.

---

# 18. Bottom Navigation đề xuất

Không nên đưa tất cả module vào bottom bar.

Đề xuất:

```text
┌────────────────────────────────┐
│ Home │ Money │ Chores │ More  │
└────────────────────────────────┘
```

Trong `More`:

```text
Calendar
Fairness
Members
Room Settings
History
```

Lý do:

- Money là action thường xuyên.
- Chores là action thường xuyên.
- Home là context tổng quan.
- Các chức năng còn lại ít cần truy cập liên tục.

---

# 19. Room Management

Tạo một màn hình:

```text
My Rooms
```

Ví dụ:

```text
MY ROOMS

⭐ Room 302
   Living · 4 members
   Primary Room

🏖 Đà Lạt 2026
   Trip · 4 members

+ Create Room
+ Join Room
```

Tap Room:

```text
Room 302

[Set as Primary]

Members
Money
Chores
Calendar
Settings
```

---

# 20. Set Primary Room UX

Có hai nơi nên cho phép:

### A. Room Switcher

```text
Room 302
⭐ Primary

Đà Lạt 2026
[Set as Primary]
```

### B. Room Settings

```text
Room Settings

Name
Room 302

Type
Living

⭐ Use as my Primary Room
```

Không cần tạo một màn hình riêng chỉ để chọn Primary Room.

---

# 21. Onboarding

Khi user tạo Room đầu tiên:

```text
Create your room

Room name:
[ Room 302 ]

[ Create Room ]
```

Sau khi tạo:

```text
Room 302 is ready 🎉

We'll use this as your Primary Room.
```

Nếu user join Room thứ hai:

```text
You've joined Đà Lạt 2026

Keep Room 302 as Primary?

[ Keep Room 302 ]
[ Make Đà Lạt Primary ]
```

Không cần hỏi lại nếu user chỉ join một Room phụ.

---

# 22. Data Loading Strategy

Khi app mở:

```text
1. Load User
2. Load Rooms
3. Resolve Primary Room
4. Set Current Room
5. Load primary Room dashboard
```

Ưu tiên:

```text
Primary Room data
        ↓
UI usable
        ↓
Other Room data
        ↓
Background refresh
```

Không nên chặn toàn bộ Home chỉ vì một Room phụ chưa load xong.

---

# 23. Cache / Local State

Vì FairRoom hướng tới trải nghiệm nhanh và về lâu dài có local-first/sync, Room context nên được cache ở client.

Ví dụ:

```text
RoomContextStore
 ├── primaryRoomId
 ├── currentRoomId
 ├── rooms[]
 └── lastSelectedRoomId
```

Có thể lưu:

```text
primaryRoomId
```

server-side là source of truth.

Client chỉ cache để UX nhanh hơn.

Không nên để local storage là nguồn sự thật duy nhất.

---

# 24. Migration Database

Migration nên thực hiện theo thứ tự:

```text
1. Add RoomType enum (nếu triển khai ngay)
2. Add Room.type
3. Add User.primaryRoomId
4. Add relation User ↔ Room
5. Deploy migration
6. Backfill primaryRoomId
7. Update backend
8. Update frontend
```

## Backfill

Với user đã có Room:

```text
primaryRoomId = first valid membership
```

Ưu tiên Room mà user tham gia sớm nhất.

Sau đó user có thể thay đổi Primary Room thủ công.

---

# 25. Backward Compatibility

Trong giai đoạn migration:

```text
primaryRoomId = nullable
```

để hỗ trợ:

```text
User chưa có Room
```

hoặc:

```text
User cũ chưa được backfill
```

Backend có thể resolve:

```text
if primaryRoomId exists
    use primary
else
    fallback to first membership
```

Sau khi migration hoàn tất có thể cân nhắc giữ nullable vì user hoàn toàn có thể không còn Room.

---

# 26. Không thay đổi domain Money / Chores

Proposal này **không yêu cầu** thay đổi cách Expense hoặc Chore hoạt động.

Expense vẫn:

```text
Expense
 ↓
roomId
 ↓
Room
```

Chore vẫn:

```text
Chore
 ↓
roomId
 ↓
Room
```

Balance vẫn tính theo Room.

Chore history vẫn theo Room.

Settlement vẫn theo Room.

Điều thay đổi chủ yếu là:

```text
User
 ↓
⭐ primaryRoomId
 ↓
default context
```

Điều này giúp giảm coupling.

---

# 27. Validation Rules

Backend cần đảm bảo:

### Primary Room

```text
primaryRoomId == null
OR
User is member of Room(primaryRoomId)
```

### Room context

```text
currentRoomId
→ must belong to user's memberships
```

### Expense

```text
expense.roomId
→ Room exists
→ current user has membership
```

### Chore

```text
chore.roomId
→ Room exists
→ current user has membership
```

Primary Room **không thay thế authorization**.

---

# 28. Testing

## Database

### Test 01

```text
User A
Room 1
Room 2

primaryRoomId = 1
```

Expected:

```text
Room 1 = Primary
```

### Test 02

```text
User A
tries to set Room 3 as primary

Room 3 ∉ memberships
```

Expected:

```text
403 Forbidden
```

### Test 03

```text
Primary Room deleted
```

Expected:

```text
primaryRoomId = null
```

### Test 04

```text
User has no Room
```

Expected:

```text
primaryRoomId = null
```

---

# 29. Backend API Tests

```text
GET  /users/me
PUT  /users/me/preferences/primary-room
GET  /rooms
GET  /rooms/:roomId
GET  /rooms/:roomId/expenses
GET  /rooms/:roomId/chores
```

Test:

```text
authenticated user
unauthenticated user
member
non-member
deleted room
invalid roomId
```

---

# 30. Frontend Tests

## Test 01 — First Room

```text
Create Room 302
```

Expected:

```text
Room 302 automatically becomes Primary
```

## Test 02 — Multiple Rooms

```text
Room 302 = Primary
Đà Lạt 2026 = Secondary
```

Enter Money:

```text
Room 302 displayed by default
```

## Test 03 — Switch Room

```text
Money
→ Room Switcher
→ Đà Lạt 2026
```

Expected:

```text
Expenses belong to Đà Lạt 2026
```

No Room 302 expense appears.

## Test 04 — Chores

```text
Primary Room = Room 302
```

Chores opens:

```text
Room 302
```

## Test 05 — Trip

```text
Current Room = Đà Lạt 2026
type = TRIP
```

Expected:

```text
No misleading household chore dashboard
```

---

# 31. Suggested Implementation Order

## Phase 1 — Database

```text
[ ] Add User.primaryRoomId
[ ] Add User ↔ Room relation
[ ] Migration
[ ] Backfill existing users
```

## Phase 2 — Backend

```text
[ ] GET current user + primary room
[ ] GET user's rooms
[ ] Set primary room API
[ ] Membership validation
[ ] Room authorization helper
```

## Phase 3 — Frontend foundation

```text
[ ] RoomContextStore
[ ] Primary Room state
[ ] Current Room state
[ ] Room Switcher
```

## Phase 4 — Screens

```text
[ ] Home
[ ] Money
[ ] Chores
[ ] More / Room Management
```

## Phase 5 — Room Type

```text
[ ] Add RoomType
[ ] LIVING
[ ] TRIP
[ ] OTHER
[ ] Adapt UI only where useful
```

---

# 32. Recommended Final Architecture

```text
                         USER
                           │
             ┌─────────────┴─────────────┐
             │                           │
      primaryRoomId                 memberships
             │                           │
             ↓                           ↓
       ⭐ PRIMARY ROOM              Room 1
             │                      Room 2
             │                      Room 3
             ↓
       CURRENT CONTEXT
             │
      ┌──────┼────────┐
      ↓      ↓        ↓
    Home   Money    Chores
      │      │        │
      └──────┼────────┘
             ↓
        Room Switcher
             │
      ┌──────┴─────────┐
      ↓                ↓
 Room 302         Đà Lạt 2026
 LIVING              TRIP
```

---

# 33. Quyết định kiến trúc quan trọng

| Vấn đề | Quyết định |
|---|---|
| Primary Room lưu ở đâu? | `User.primaryRoomId` |
| Room có `isPrimary` không? | Không |
| Primary có phải Room duy nhất? | Không |
| Current Room lưu DB? | Không |
| Current Room là gì? | Frontend/UI state |
| RoomType cần không? | Nên có, nhưng có thể Phase 2 |
| Money mặc định mở Room nào? | Primary Room |
| Chores mặc định mở Room nào? | Primary Room |
| User xem Room khác được không? | Có |
| Có cần chọn Room mỗi lần vào feature? | Không |
| Có kiểm tra membership khi set Primary? | Bắt buộc |
| Xoá Primary Room? | `SetNull` + fallback |
| Expense/Chore có đổi model không? | Không cần |
| Authorization có dựa vào Primary không? | Không |

---

# 34. Kết luận

Đây nên được xem là một **Room Context Architecture**, không đơn thuần là thêm một nút "Favorite Room".

Mô hình cuối cùng:

```text
                  USER
                   │
          ┌────────┴────────┐
          ↓                 ↓
   Primary Room       All Memberships
          │                 │
          ↓                 ↓
   Default Context     Other Rooms
          │
    ┌─────┼─────┐
    ↓     ↓     ↓
  Home  Money  Chores
          │
          ↓
    Room Switcher
          │
          ↓
   Explicit Context
```

Điểm quan trọng nhất là **Primary Room chỉ quyết định Room nào được ưu tiên trong UX; nó không quyết định quyền truy cập và cũng không làm thay đổi domain accounting/chore hiện tại.**

Điều này giữ được kiến trúc Room-centric hiện tại của FairRoom, đồng thời giải quyết vấn đề UX khi user bắt đầu có nhiều Room.

---

## 35. MVP Recommendation

Nếu muốn triển khai nhanh, **không cần làm tất cả proposal cùng lúc**.

Nên làm đúng 4 thứ đầu tiên:

```text
1. User.primaryRoomId
        ↓
2. Room Switcher
        ↓
3. Money/Chores mặc định dùng Primary Room
        ↓
4. Set Primary Room
```

Sau khi flow này ổn định mới thêm:

```text
RoomType
Trip UX
Room-specific dashboard
Advanced multi-room experience
```

Như vậy FairRoom vẫn giữ nguyên core loop:

```text
Room
 ↓
Money + Chores
 ↓
Coordination
 ↓
Fairness
```

nhưng người dùng không còn phải liên tục suy nghĩ:

> "Mình đang muốn xem Room nào?"

