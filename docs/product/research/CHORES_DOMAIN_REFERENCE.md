# FairRoom --- CHORES_SPLIT_FEATURES

> Product specification for the **Split Chores / Fair Chores** module of
> FairRoom.

**Status:** Draft for implementation\
**Target:** FairRoom mobile app --- React Native + Expo\
**Primary context:** One shared rental room/household, normally 2--6
members\
**Product principle:** Make household work visible, coordinated, and
fair.

------------------------------------------------------------------------

## 1. Purpose

This document defines the functional, domain, UX, business-rule, API,
data-model, and implementation requirements for FairRoom's chore module.

It is intentionally designed around FairRoom's product context rather
than copying a multi-room cleaning application.

FairRoom's existing product philosophy is:

> **Your shared home, kept fair.**

The application is designed for students and people living together,
including a solo-manager mode where only one person has installed the
app. The README already identifies chores such as taking out trash,
cleaning, reminders, rotation, cover, swap, history, and fairness as
core concepts. The chore system must therefore work both when one person
manages the whole room and when multiple members are connected.

Reference: `README.md`, especially the product overview and Solo →
Connected principle.

------------------------------------------------------------------------

# 2. Product Definition

## 2.1 What FairRoom Chores is

FairRoom Chores is a shared responsibility system that answers four
questions:

1.  **What needs to be done?**
2.  **When does it need to be done?**
3.  **Who is responsible for it?**
4.  **Is the workload reasonably fair over time?**

It is not primarily a house-cleaning database.

The core object is a **chore responsibility**, not a physical room.

------------------------------------------------------------------------

## 2.2 What FairRoom Chores is not

Do not turn the module into:

-   a detailed cleaning-management system for dozens of rooms;
-   a professional housekeeping application;
-   a project-management board;
-   a social feed;
-   a gamified competition between roommates;
-   an automatic judge that labels people as lazy or responsible.

The product must remain lightweight.

------------------------------------------------------------------------

# 3. Product Philosophy

## 3.1 Household work should become visible

A lot of household work is invisible until someone stops doing it.

Examples:

-   taking out trash;
-   washing dishes;
-   cleaning the bathroom;
-   sweeping/mopping;
-   buying water;
-   cleaning the kitchen;
-   washing shared towels;
-   refilling shared supplies.

FairRoom records these responsibilities without turning them into
accusations.

Bad UX:

> "Nam is lazy."

Good UX:

> "Nam completed 2 points this week. The room average is 6 points."

The system reports activity, not character.

------------------------------------------------------------------------

## 3.2 Fairness is not equality

Four chores do not necessarily mean four equal workloads.

Example:

``` text
Take out trash       1 point
Wash dishes          2 points
Mop floor            2 points
Clean bathroom       3 points
```

A member completing two heavy chores may contribute more than another
member completing four light chores.

Therefore FairRoom must use **relative effort points** rather than raw
task count as the primary chore workload metric.

------------------------------------------------------------------------

## 3.3 Solo → Connected

The existing FairRoom architecture explicitly supports:

> One person can use the app and manage the room; the experience becomes
> stronger when other members join.

The chore module must preserve this rule.

A solo manager must be able to:

-   create members;
-   create chores;
-   assign chores;
-   edit schedules;
-   mark chores completed;
-   record that another member completed a chore;
-   view history;
-   view basic fairness.

When another member joins, they gain:

-   personal assignments;
-   reminders;
-   completion actions;
-   cover requests;
-   swap requests;
-   fairness visibility.

------------------------------------------------------------------------

# 4. Core Domain Model

The conceptual hierarchy should be:

``` text
Room
│
├── RoomMember
│
├── ChoreArea (optional)
│
└── Chore
     │
     ├── Schedule / Recurrence
     │
     ├── Assignment
     │      │
     │      ├── Cover
     │      └── Swap
     │
     └── Completion
```

The most important distinction is:

``` text
Chore
    = reusable definition

ChoreOccurrence
    = one concrete instance that must be completed

ChoreAssignment
    = who is responsible for that occurrence

ChoreCompletion
    = who actually completed it
```

This distinction should be preserved in the backend.

------------------------------------------------------------------------

# 5. Area Model

## 5.1 Do not model the home as multiple rooms

Tody/Sweepy-style applications commonly organize the system around
physical rooms.

FairRoom should not require:

``` text
House
├── Bedroom
├── Kitchen
├── Bathroom
├── Living Room
└── Laundry Room
```

because a Vietnamese student rental room may be:

``` text
One room
+ one small kitchen
+ one bathroom
```

or even a single shared space.

------------------------------------------------------------------------

## 5.2 Use optional `ChoreArea`

An area is only a label used to organize chores.

Example:

``` text
Shared Space
Kitchen
Bathroom
Laundry
Entrance
Other
```

A chore may have no area.

Examples:

``` text
Take out trash
Area: Shared

Wash dishes
Area: Kitchen

Clean toilet
Area: Bathroom
```

Area must never be required before creating a chore.

------------------------------------------------------------------------

## 5.3 Default areas

Recommended system presets:

-   🏠 Shared
-   🍳 Kitchen
-   🚿 Bathroom
-   🧺 Laundry
-   🛋️ Living area
-   📦 Other

Room owners may create custom areas later.

Custom areas are not required for MVP.

------------------------------------------------------------------------

# 6. Chore Entity

A Chore is the reusable definition of a recurring household
responsibility.

## 6.1 Required fields

``` text
id
roomId
title
effortPoints
estimatedMinutes
scheduleType
active
createdAt
updatedAt
```

## 6.2 Optional fields

``` text
description
areaId
icon
color
```

## 6.3 Example

``` text
Title:
Take out trash

Area:
Shared

Estimated effort:
1 point

Estimated time:
10 minutes

Schedule:
Monday, Wednesday, Friday

Assignment:
Rotation

Active:
true
```

------------------------------------------------------------------------

# 7. Effort Points

## 7.1 Purpose

Effort points represent the **relative workload** of a chore.

They are not intended to measure exact labor.

The README defines the principle:

``` text
Light   → 1
Medium  → 2
Heavy   → 3
```

It also gives examples up to 4 points for very heavy chores.

Recommended MVP range:

``` text
1–4 points
```

------------------------------------------------------------------------

## 7.2 Recommended defaults

  Chore                  Default points
  -------------------- ----------------
  Take out trash                      1
  Buy shared water                    1
  Wash dishes                         2
  Sweep floor                         1
  Mop floor                           2
  Clean kitchen                       2
  Clean bathroom                      3
  Deep clean room                     4
  Wash shared towels                  2

These are defaults only.

The room manager must be able to change the value.

------------------------------------------------------------------------

## 7.3 Estimated minutes

`estimatedMinutes` is useful for explaining effort and future analytics.

Example:

``` text
Take out trash
1 point
10 minutes

Clean bathroom
3 points
35 minutes
```

Do not calculate fairness solely from minutes in MVP.

Use effort points as the canonical fairness workload value.

------------------------------------------------------------------------

# 8. Scheduling

The system must support both simple schedules and recurring schedules.

## 8.1 Schedule types

MVP:

``` text
ONCE
DAILY
WEEKLY
CUSTOM_WEEKDAYS
```

Future:

``` text
INTERVAL
MONTHLY
CUSTOM_CRON
```

Do not implement a general-purpose cron engine in MVP.

------------------------------------------------------------------------

# 9. Schedule Examples

## 9.1 Once

``` text
Clean the room
Due:
September 15
```

------------------------------------------------------------------------

## 9.2 Daily

``` text
Wash dishes
Every day
Due:
20:00
```

------------------------------------------------------------------------

## 9.3 Weekly

``` text
Mop floor
Every Saturday
Due:
18:00
```

------------------------------------------------------------------------

## 9.4 Multiple weekdays

``` text
Take out trash
Monday
Wednesday
Friday
Due:
19:00
```

This corresponds directly to the existing FairRoom README example.

------------------------------------------------------------------------

# 10. Scheduling Principle

The system should distinguish:

``` text
Due date
Due time
Completion time
```

Do not store only a date.

Example:

``` text
scheduledDate = 2026-09-12
dueTime = 19:00
completedAt = 2026-09-12 18:35
```

This allows:

-   reminders;
-   overdue state;
-   completion history;
-   future analytics.

------------------------------------------------------------------------

# 11. Chore Occurrence

A recurring Chore should generate concrete occurrences.

Example:

``` text
Chore:
Take out trash

Recurrence:
Mon / Wed / Fri
```

generates:

``` text
Sep 14 → occurrence A
Sep 16 → occurrence B
Sep 18 → occurrence C
```

Each occurrence can have a different assignment.

This is essential for rotation.

Do not simply update one `assignedTo` field on the parent Chore.

------------------------------------------------------------------------

# 12. Assignment Modes

FairRoom should support four assignment modes.

## 12.1 Fixed assignment

One member is responsible for all occurrences.

``` text
Bathroom cleaning
→ Minh
```

Use when roommates already have an agreement.

------------------------------------------------------------------------

## 12.2 Rotation

Members take turns.

Example:

``` text
Week 1 → Quang
Week 2 → Minh
Week 3 → Nam
Week 4 → An
```

or:

``` text
Mon → Quang
Wed → Minh
Fri → Nam
```

The rotation must be deterministic and persisted.

------------------------------------------------------------------------

## 12.3 Custom assignment

The room manager explicitly chooses each occurrence.

``` text
Monday → Quang
Wednesday → Minh
Friday → Nam
```

Use for irregular arrangements.

------------------------------------------------------------------------

## 12.4 Unassigned

A chore may exist without an assignee.

``` text
Take out recycling
Assigned:
Anyone
```

This is useful in solo-manager mode and for rooms that decide
responsibility informally.

------------------------------------------------------------------------

# 13. Rotation Rules

## 13.1 Basic rotation

Given:

``` text
Members:
A, B, C
```

and four weekly occurrences:

``` text
Occurrence 1 → A
Occurrence 2 → B
Occurrence 3 → C
Occurrence 4 → A
```

Use a persistent rotation cursor.

Do not derive rotation from current member list position every time.

------------------------------------------------------------------------

## 13.2 Member changes

If a member leaves:

``` text
A, B, C
```

and B leaves:

``` text
A, C
```

Future occurrences should use:

``` text
A → C → A → C
```

Past assignments must not be rewritten.

Historical records are immutable.

------------------------------------------------------------------------

## 13.3 New member joins

A new member should not automatically receive an unfair amount of
existing workload.

Recommended behavior:

-   do not modify past assignments;
-   allow the manager to add the member to future rotation;
-   provide an option to "start rotation from next cycle."

Example:

``` text
Current:
A → B → C

D joins.

Next cycle:
A → B → C → D
```

------------------------------------------------------------------------

# 14. Fairness-Aware Assignment

This should be a future enhancement, not MVP automation.

The system may calculate:

``` text
Quang: 10 points
Minh:   7 points
Nam:   11 points
```

and recommend:

> "Minh currently has the lowest chore workload. Assigning this 2-point
> chore to Minh would improve balance."

Important:

**Recommend, do not silently auto-assign.**

The product must not make roommates feel that an algorithm controls
their household.

------------------------------------------------------------------------

# 15. Chore Lifecycle

Recommended lifecycle:

``` text
DRAFT
  ↓
ACTIVE
  ↓
SCHEDULED
  ↓
DUE
  ↓
COMPLETED
```

Alternative states:

``` text
SKIPPED
CANCELLED
OVERDUE
COVER_REQUESTED
COVERED
SWAP_PENDING
```

Do not mix all of these into one status field if doing so creates
contradictory states.

Prefer deriving some UI states from:

-   scheduled date;
-   due time;
-   completion;
-   assignment;
-   cover status;
-   swap status.

------------------------------------------------------------------------

# 16. Completion

## 16.1 Completing an assigned chore

Example:

``` text
🧹 Take out trash

Assigned to:
Quang

Due:
Today, 19:00

[Complete]
```

When Quang taps Complete:

``` text
completedAt = now
completedBy = Quang
```

The occurrence becomes completed.

------------------------------------------------------------------------

## 16.2 Someone else completes it

Example:

``` text
Assigned:
Quang

Actually completed:
Minh
```

Store both values.

This distinction is required for fairness and coverage history.

------------------------------------------------------------------------

# 17. Completion Rules

When completing a chore:

1.  Verify the user belongs to the room.
2.  Verify the occurrence is not cancelled.
3.  Record completion time.
4.  Record actual completer.
5.  Calculate contribution using the occurrence's effort points.
6.  Update fairness data.
7.  Notify relevant members if appropriate.
8.  Preserve the original assignment.

Never rewrite:

``` text
assignedMemberId = Minh
```

just because Minh completed it.

Instead:

``` text
assignedMemberId = Quang
completedByMemberId = Minh
```

------------------------------------------------------------------------

# 18. Skip

A member may need to skip a chore.

Example:

``` text
🧹 Mop floor

[Skip]
```

The app should ask for a reason only when useful.

Suggested reasons:

``` text
I'm away
Not needed this time
Someone else handled it
Other
```

Do not require a long explanation.

------------------------------------------------------------------------

# 19. Postpone

A chore may be postponed.

Example:

``` text
Due:
Today 19:00

Postpone:
Tomorrow 19:00
```

Important business rule:

**Postponing should not silently change responsibility.**

The same member remains responsible unless the user explicitly
transfers/asks for cover.

------------------------------------------------------------------------

# 20. Overdue

An occurrence becomes overdue when:

``` text
now > dueAt
AND completedAt IS NULL
AND status != CANCELLED
```

UI:

``` text
⚠️ Overdue

Take out trash
Was due 2 hours ago
Assigned to Quang
```

Overdue is a state, not necessarily a punishment.

Do not automatically subtract points.

------------------------------------------------------------------------

# 21. Cover

Cover means:

> Another member temporarily performs a responsibility on behalf of the
> originally assigned member.

Example:

``` text
Quang
→ away this weekend

Take out trash
→ asks Minh to cover
```

If Minh accepts:

``` text
Original:
Quang

Covered by:
Minh
```

------------------------------------------------------------------------

# 22. Cover Flow

``` text
Assigned member
      ↓
Ask for cover
      ↓
Choose member
      ↓
Cover request
      ↓
Accept / Decline
      ↓
If accepted
      ↓
Occurrence covered
      ↓
Actual completion
```

------------------------------------------------------------------------

# 23. Cover Data Rules

A cover request should contain:

``` text
assignmentId
requesterMemberId
coveringMemberId
status
message
createdAt
respondedAt
```

Recommended statuses:

``` text
PENDING
ACCEPTED
DECLINED
CANCELLED
EXPIRED
```

------------------------------------------------------------------------

# 24. Who gets the effort points?

For fairness, the **actual completed work** should normally count toward
the member who performed it.

Example:

``` text
Chore:
Clean bathroom
3 points

Assigned:
Quang

Covered by:
Minh

Completed by:
Minh
```

Contribution:

``` text
Minh +3
```

However, the system should also preserve:

``` text
Quang was originally responsible.
```

This allows future fairness logic to distinguish:

-   assigned responsibility;
-   actual work;
-   helping others.

Do not delete the original assignment.

------------------------------------------------------------------------

# 25. Swap

Swap is different from Cover.

### Cover

``` text
"Can you do my chore this time?"
```

### Swap

``` text
"Let's exchange our responsibilities."
```

Example:

``` text
Quang:
Trash — Monday

Minh:
Kitchen — Wednesday
```

They agree to exchange them.

------------------------------------------------------------------------

# 26. Swap Flow

``` text
Quang selects assignment
        ↓
Swap
        ↓
Choose Minh's assignment
        ↓
Send request
        ↓
Minh accepts
        ↓
Assignments are exchanged
```

A swap should be atomic.

If one side fails, neither assignment should change.

------------------------------------------------------------------------

# 27. Away Mode

Away mode is a key FairRoom feature because it solves a common
shared-living problem.

Example:

``` text
I'm away
Sep 15 → Sep 20
```

Options:

``` text
Pause my chores
Ask others to cover
Keep reminders
```

------------------------------------------------------------------------

# 28. Away Mode Rules

When a member activates Away Mode:

1.  Find future assignments during the away period.
2.  Do not modify completed/past occurrences.
3.  Show affected chores.
4.  Let the member choose:
    -   request cover;
    -   reassign;
    -   leave unassigned.
5.  Notify affected members after changes.

Example:

``` text
3 chores affected

🧹 Trash — Sep 16
🧽 Kitchen — Sep 17
🚿 Bathroom — Sep 19
```

------------------------------------------------------------------------

# 29. Automatic Redistribution

MVP:

**Do not automatically redistribute chores without user confirmation.**

Instead:

``` text
3 chores affected

[Find someone to cover]
[Reassign manually]
[Leave as is]
```

Future:

``` text
[Suggest fair redistribution]
```

The algorithm can recommend members based on:

-   current workload;
-   availability;
-   previous assignments;
-   effort points.

------------------------------------------------------------------------

# 30. Fairness Model

## 30.1 Goal

The fairness system answers:

> "How evenly is household chore responsibility distributed?"

It must not answer:

> "Who is a good/bad roommate?"

------------------------------------------------------------------------

## 30.2 Basic contribution

For each member:

``` text
completedEffort =
sum(effortPoints of chores actually completed)
```

Example:

``` text
Quang:
3 + 2 + 1 = 6

Minh:
2 + 2 = 4

Nam:
3 + 1 + 2 = 6
```

------------------------------------------------------------------------

# 31. Fairness Percentage

For MVP, use a simple distribution metric.

Let:

``` text
totalEffort = sum(memberEffort)
expectedEffort = totalEffort / activeMembers
```

For each member:

``` text
deviation = abs(memberEffort - expectedEffort)
```

A simple room-level fairness score can be based on normalized deviation.

One practical MVP formulation:

``` text
fairness =
100 × (1 - totalAbsoluteDeviation / (2 × totalEffort))
```

Clamp the result to:

``` text
0–100
```

Example:

``` text
Members:
Quang = 10
Minh  = 10
Nam   = 10
```

Fairness:

``` text
100
```

Example:

``` text
Quang = 15
Minh  = 5
Nam   = 10
```

Fairness is lower.

The exact formula can be changed later without changing the domain
model.

------------------------------------------------------------------------

# 32. Important Fairness Caveat

Effort points are estimates.

Therefore:

> Fairness is an indicator, not an objective truth.

The UI should use language such as:

``` text
Looking balanced
Slightly uneven
Needs attention
```

rather than:

``` text
FAIR
UNFAIR
```

especially in early versions.

------------------------------------------------------------------------

# 33. Fairness Time Window

MVP:

``` text
This week
```

Future:

``` text
This month
Last 30 days
Custom range
```

Do not compare all-time totals for normal fairness UX.

A person who did many chores three months ago should not permanently
receive credit against someone who is currently overloaded.

------------------------------------------------------------------------

# 34. Rolling Fairness

Future enhancement:

Use a rolling window such as:

``` text
Last 28 days
```

This helps smooth irregular schedules.

Example:

``` text
Bathroom cleaning:
every 7 days

Trash:
every 2 days
```

A 28-day window provides enough observations without relying on lifetime
history.

------------------------------------------------------------------------

# 35. Assigned vs Actual Contribution

Track two dimensions.

### Responsibility

``` text
Who was assigned?
```

### Work

``` text
Who actually completed it?
```

Example:

``` text
Assigned:
Quang

Completed:
Minh
```

This gives FairRoom richer information than a normal checklist.

------------------------------------------------------------------------

# 36. Coverage Metric

The README identifies:

``` text
Coverage / Help
```

as a separate fairness dimension.

Track:

``` text
coverRequestsSent
coverRequestsAccepted
coverCompleted
```

and:

``` text
coveredForOthersEffort
receivedCoverEffort
```

Do not merge these directly into money or chore fairness.

------------------------------------------------------------------------

# 37. Fairness Dashboard

The dashboard should be understandable in seconds.

Example:

``` text
        ⚖️

        92
      FAIRNESS

This week

Quang     10 pts
Minh       9 pts
Nam       11 pts
```

Then:

``` text
Chore balance
███████████░ 92%
```

------------------------------------------------------------------------

# 38. Two-Member Visualization

For two members:

``` text
       ⚖️

Quang       Minh
 48%         52%

████████|████████

Fairness 96
```

Do not use complex charts.

A simple balance visualization is more consistent with the FairRoom
concept.

------------------------------------------------------------------------

# 39. Three-to-Six Members

For 3--6 members, use:

-   horizontal contribution bars;
-   percentage labels;
-   effort points;
-   small balance indicator.

Example:

``` text
Quang   ██████████ 10
Minh    ████████    8
Nam     ██████████ 10
An      █████████    9
```

Avoid pie charts for small differences because they are harder to
compare.

------------------------------------------------------------------------

# 40. Chore List UX

Primary Chores screen:

``` text
Chores

TODAY

🗑️ Take out trash
Quang · 1 pt
Due 19:00

🍽️ Wash dishes
Minh · 2 pts
Due 20:00


UPCOMING

🧹 Mop floor
Nam · 2 pts
Tomorrow
```

Primary action:

``` text
+ Add chore
```

------------------------------------------------------------------------

# 41. Personal View

A connected user should have:

``` text
My chores

Today
1 task

Upcoming
3 tasks

Completed
12

Helped others
2
```

This should make the app immediately useful even if the room has many
chores.

------------------------------------------------------------------------

# 42. Room View

Room manager:

``` text
All chores

Today
Quang → Trash
Minh  → Dishes
Nam   → Kitchen

Upcoming
...
```

The manager should be able to filter:

``` text
All
Mine
Member
Overdue
Unassigned
```

------------------------------------------------------------------------

# 43. Chore Detail Screen

Recommended structure:

``` text
🧹 Take out trash

Shared

1 point
~10 min

Every Mon / Wed / Fri
19:00

Assigned to
Quang

Next:
Wed, Sep 16

────────────────

History

✓ Sep 14 — Quang
✓ Sep 12 — Minh
✓ Sep 10 — Quang

────────────────

[Complete]
[Ask for cover]
[Swap]
[More]
```

------------------------------------------------------------------------

# 44. Create Chore UX

Do not expose every advanced option on the first screen.

Step 1:

``` text
What needs to be done?

[ Take out trash ]
```

Step 2:

``` text
How often?

○ Once
○ Every day
○ Every week
○ Custom days
```

Step 3:

``` text
Who does it?

○ Me
○ Assign member
○ Rotate
○ Anyone
```

Step 4:

``` text
How much work?

Light
Medium
Heavy

Estimated time:
10 min
```

Advanced settings can come afterward.

------------------------------------------------------------------------

# 45. Suggested Quick Chores

Provide Vietnam-oriented templates.

Examples:

``` text
🗑️ Take out trash
🍽️ Wash dishes
🧹 Sweep floor
🧽 Mop floor
🚿 Clean bathroom
🍳 Clean kitchen
🧺 Wash shared towels
🛒 Buy water
🧻 Refill toilet paper
🧴 Refill dish soap
```

Templates should be suggestions, not mandatory categories.

------------------------------------------------------------------------

# 46. Notifications

Notifications solve the "I forgot" problem.

Core notification:

``` text
🧹 Chore reminder

Take out trash in 2 hours.

You're responsible today.
```

Overdue:

``` text
⚠️ Chore overdue

Take out trash
Due 2 hours ago.
```

Cover:

``` text
🤝 Cover request

Minh asked you to cover
Take out trash — today.
```

Swap:

``` text
🔄 Swap request

Quang wants to swap
Take out trash for Kitchen cleaning.
```

------------------------------------------------------------------------

# 47. Notification Rules

Never spam.

Recommended MVP:

-   one reminder before due;
-   one overdue reminder;
-   cover request notification;
-   swap request notification.

Allow users to configure reminder timing.

Possible settings:

``` text
1 day before
2 hours before
30 minutes before
```

------------------------------------------------------------------------

# 48. Quiet Hours

Support:

``` text
Quiet hours
22:00 → 07:00
```

Notifications should be deferred where practical.

Do not block urgent interactive requests indefinitely.

------------------------------------------------------------------------

# 49. Calendar Integration

Calendar is a shared FairRoom module, not necessarily part of the chore
domain.

Chores should expose events to the shared calendar:

``` text
Sep 16
19:00
🧹 Trash — Quang
```

Do not duplicate calendar ownership logic inside Chores.

------------------------------------------------------------------------

# 50. Widget

The first widget should answer:

> "What do I need to do?"

Example:

``` text
┌────────────────────┐
│ 🏠 ROOM 302        │
│                    │
│ 🧹 Your task       │
│ Take out trash     │
│ Today · 19:00      │
└────────────────────┘
```

Future widget:

``` text
TODAY

🧹 Quang → Trash
🍽️ Minh → Dishes
🚿 Nam → Bathroom

⚖️ Fairness 92
```

------------------------------------------------------------------------

# 51. Activity Feed

Record important events:

``` text
✓ Minh completed Kitchen
🔄 Quang swapped Trash with An
🤝 Nam accepted cover request
🧹 An completed Bathroom
```

Avoid recording every tiny UI action.

The feed exists for visibility and coordination.

------------------------------------------------------------------------

# 52. Permissions

Use the existing `RoomRole`:

``` text
OWNER
ADMIN
MEMBER
```

## OWNER / ADMIN

Can:

-   create chore;
-   edit chore;
-   delete/archive chore;
-   configure rotation;
-   assign members;
-   manage areas;
-   view room-wide history;
-   manage schedules.

## MEMBER

Can:

-   view chores;
-   complete assigned chores;
-   request cover;
-   accept/decline cover;
-   request swap;
-   accept/decline swap;
-   view fairness;
-   manage their own reminders.

A member should not be able to modify another member's historical
completion.

------------------------------------------------------------------------

# 53. Solo Manager Mode

If a RoomMember has no User account:

``` text
RoomMember.userId = null
```

the member can still appear in:

``` text
assignments
completions
fairness
```

This matches the existing FairRoom data model where a room member may be
a guest without an account.

The backend must therefore use `RoomMember.id` for room-level
relationships, not `User.id`.

------------------------------------------------------------------------

# 54. Existing Schema Compatibility

The current FairRoom schema already has:

``` text
User
Room
RoomMember
RoomRole
```

and `RoomMember` supports:

``` text
userId Int?
```

for guest/non-account members.

The existing schema also uses `RoomMember` as the participant identity
for expenses:

``` text
Expense.payerId
ExpenseSplit.memberId
Settlement.payerId
Settlement.receiverId
```

Chores should follow the same pattern.

Do not make Chores depend directly on `User.id`.

Recommended:

``` text
ChoreAssignment.memberId → RoomMember.id
ChoreCompletion.completedByMemberId → RoomMember.id
```

------------------------------------------------------------------------

# 55. Proposed Prisma Models

The following is a proposed direction. It should be reviewed before
migration.

``` prisma
enum ChoreScheduleType {
  ONCE
  DAILY
  WEEKLY
  CUSTOM_WEEKDAYS
}

enum ChoreAssignmentType {
  FIXED
  ROTATION
  CUSTOM
  UNASSIGNED
}

enum ChoreOccurrenceStatus {
  SCHEDULED
  COMPLETED
  SKIPPED
  CANCELLED
}

enum CoverRequestStatus {
  PENDING
  ACCEPTED
  DECLINED
  CANCELLED
  EXPIRED
}

enum SwapRequestStatus {
  PENDING
  ACCEPTED
  DECLINED
  CANCELLED
  EXPIRED
}

model ChoreArea {
  id Int @id @default(autoincrement())

  roomId Int
  room Room @relation(fields: [roomId], references: [id], onDelete: Cascade)

  name String
  icon String?
  color String?

  chores Chore[]

  createdAt DateTime @default(now())
  updatedAt DateTime @updatedAt

  @@index([roomId])
}

model Chore {
  id Int @id @default(autoincrement())

  roomId Int
  room Room @relation(fields: [roomId], references: [id], onDelete: Cascade)

  areaId Int?
  area ChoreArea? @relation(fields: [areaId], references: [id], onDelete: SetNull)

  title String
  description String?

  effortPoints Int @default(1)
  estimatedMinutes Int?

  scheduleType ChoreScheduleType
  assignmentType ChoreAssignmentType

  dueTimeMinutes Int?

  active Boolean @default(true)

  occurrences ChoreOccurrence[]

  createdAt DateTime @default(now())
  updatedAt DateTime @updatedAt

  @@index([roomId, active])
  @@index([areaId])
}

model ChoreOccurrence {
  id Int @id @default(autoincrement())

  choreId Int
  chore Chore @relation(fields: [choreId], references: [id], onDelete: Cascade)

  scheduledDate DateTime
  dueAt DateTime?

  status ChoreOccurrenceStatus @default(SCHEDULED)

  assignments ChoreAssignment[]
  completion ChoreCompletion?

  createdAt DateTime @default(now())
  updatedAt DateTime @updatedAt

  @@unique([choreId, scheduledDate])
  @@index([scheduledDate, status])
}

model ChoreAssignment {
  id Int @id @default(autoincrement())

  occurrenceId Int
  occurrence ChoreOccurrence @relation(fields: [occurrenceId], references: [id], onDelete: Cascade)

  memberId Int
  member RoomMember @relation(fields: [memberId], references: [id])

  isOriginal Boolean @default(true)

  coverRequests CoverRequest[] @relation("CoverAssignment")

  createdAt DateTime @default(now())

  @@index([occurrenceId])
  @@index([memberId])
}

model ChoreCompletion {
  id Int @id @default(autoincrement())

  occurrenceId Int @unique
  occurrence ChoreOccurrence @relation(fields: [occurrenceId], references: [id], onDelete: Cascade)

  completedByMemberId Int
  completedByMember RoomMember @relation(fields: [completedByMemberId], references: [id])

  completedAt DateTime

  note String?

  createdAt DateTime @default(now())

  @@index([completedByMemberId, completedAt])
}

model CoverRequest {
  id Int @id @default(autoincrement())

  assignmentId Int
  assignment ChoreAssignment @relation("CoverAssignment", fields: [assignmentId], references: [id], onDelete: Cascade)

  requesterMemberId Int
  requesterMember RoomMember @relation("CoverRequester", fields: [requesterMemberId], references: [id])

  coveringMemberId Int
  coveringMember RoomMember @relation("CoverMember", fields: [coveringMemberId], references: [id])

  status CoverRequestStatus @default(PENDING)

  message String?

  createdAt DateTime @default(now())
  respondedAt DateTime?

  @@index([assignmentId])
  @@index([coveringMemberId, status])
}

model SwapRequest {
  id Int @id @default(autoincrement())

  assignmentAId Int
  assignmentBId Int

  requesterMemberId Int
  requesterMember RoomMember @relation("SwapRequester", fields: [requesterMemberId], references: [id])

  status SwapRequestStatus @default(PENDING)

  createdAt DateTime @default(now())
  respondedAt DateTime?

  @@index([requesterMemberId, status])
}
```

This is a starting point, not a migration to paste blindly.

Before migration, Prisma relation names should be finalized because
several relations target `RoomMember`.

------------------------------------------------------------------------

# 56. Recommended Additional Fields

For a production implementation, consider:

``` text
Chore:
  createdByMemberId
  archivedAt

ChoreOccurrence:
  generatedAt
  originalDueAt

ChoreAssignment:
  assignedAt
  assignedByMemberId

ChoreCompletion:
  completedByUserId is NOT required
```

Use `RoomMember` as the primary domain identity.

------------------------------------------------------------------------

# 57. Do Not Delete Historical Chores

When a recurring chore is no longer needed:

``` text
active = false
```

or:

``` text
archivedAt = now
```

Do not delete all occurrences.

Historical fairness depends on them.

------------------------------------------------------------------------

# 58. API Structure

Recommended REST-style structure.

## Chores

``` http
GET    /rooms/:roomId/chores
POST   /rooms/:roomId/chores
GET    /rooms/:roomId/chores/:choreId
PATCH  /rooms/:roomId/chores/:choreId
DELETE /rooms/:roomId/chores/:choreId
```

------------------------------------------------------------------------

## Occurrences

``` http
GET /rooms/:roomId/chore-occurrences
GET /rooms/:roomId/chore-occurrences/today
GET /rooms/:roomId/chore-occurrences/upcoming
```

------------------------------------------------------------------------

## Completion

``` http
POST /rooms/:roomId/chore-occurrences/:occurrenceId/complete
POST /rooms/:roomId/chore-occurrences/:occurrenceId/skip
POST /rooms/:roomId/chore-occurrences/:occurrenceId/postpone
```

------------------------------------------------------------------------

## Cover

``` http
POST   /rooms/:roomId/chore-assignments/:assignmentId/cover-requests
PATCH  /rooms/:roomId/cover-requests/:requestId
DELETE /rooms/:roomId/cover-requests/:requestId
```

------------------------------------------------------------------------

## Swap

``` http
POST  /rooms/:roomId/swap-requests
PATCH /rooms/:roomId/swap-requests/:requestId
```

------------------------------------------------------------------------

## Fairness

``` http
GET /rooms/:roomId/fairness/chores
GET /rooms/:roomId/fairness/chores?from=...&to=...
```

------------------------------------------------------------------------

# 59. API Business Rules

Every chore endpoint must verify:

``` text
authenticated user
        ↓
Room membership
        ↓
permission
        ↓
resource belongs to room
```

Never trust:

``` text
roomId
memberId
occurrenceId
```

from the client without validating their relationships.

------------------------------------------------------------------------

# 60. Transaction Requirements

The following operations should be transactional.

## Complete chore

``` text
validate occurrence
→ create completion
→ update occurrence status
→ create activity event
→ trigger notification
```

The database state must not end up with:

``` text
status = COMPLETED
completion = null
```

or:

``` text
completion exists
status = SCHEDULED
```

------------------------------------------------------------------------

## Accept cover

``` text
validate request
→ validate occurrence
→ update cover request
→ preserve original assignment
→ create effective coverage record
```

Use a transaction.

------------------------------------------------------------------------

## Accept swap

``` text
validate request
→ lock relevant assignments
→ verify both are still swappable
→ exchange responsibility
→ mark request accepted
```

Use a transaction.

------------------------------------------------------------------------

# 61. Concurrency / Race Conditions

Important examples:

### Two users complete the same chore

Only one completion should succeed.

Use:

``` text
unique occurrenceId
```

and/or transaction/row locking.

### Two people accept the same cover

Only one request should become effective.

### Two swap requests target the same assignment

Only one can succeed.

These cases are rare in a small room but must not corrupt the data.

------------------------------------------------------------------------

# 62. Fairness Calculation Service

Do not put all fairness calculations inside controllers.

Recommended:

``` text
ChoreFairnessService
```

Responsibilities:

``` text
getMemberContribution()
getRoomContribution()
calculateFairnessScore()
getDistribution()
getTrends()
```

Example:

``` text
ChoreFairnessService
├── calculateContribution()
├── calculateDistribution()
├── calculateFairnessScore()
└── getPeriodSummary()
```

------------------------------------------------------------------------

# 63. Avoid Premature Precomputed Scores

For MVP, fairness data volume will be small.

Prefer calculating from occurrence/completion records.

Do not immediately create:

``` text
FairnessSnapshot
```

unless performance requires it.

If the app later has many rooms and long histories, introduce
snapshots/caching.

------------------------------------------------------------------------

# 64. Query Examples

## Today's chores

``` text
WHERE
  roomId = ?
  AND scheduledDate = today
  AND status != CANCELLED
```

Order by:

``` text
dueAt ASC
```

------------------------------------------------------------------------

## My chores

``` text
occurrence
JOIN assignment
WHERE assignment.memberId = currentRoomMember
```

------------------------------------------------------------------------

## Overdue

``` text
dueAt < now
AND status = SCHEDULED
```

------------------------------------------------------------------------

# 65. Frontend Feature Structure

Recommended React Native feature structure:

``` text
features/
└── chores/
    ├── api/
    │   ├── chores.api.ts
    │   ├── occurrences.api.ts
    │   ├── covers.api.ts
    │   └── swaps.api.ts
    │
    ├── components/
    │   ├── ChoreCard.tsx
    │   ├── ChoreList.tsx
    │   ├── ChoreStatusBadge.tsx
    │   ├── EffortBadge.tsx
    │   ├── AssignmentRow.tsx
    │   └── FairnessBar.tsx
    │
    ├── screens/
    │   ├── ChoresScreen.tsx
    │   ├── ChoreDetailScreen.tsx
    │   ├── CreateChoreScreen.tsx
    │   ├── EditChoreScreen.tsx
    │   ├── ChoreScheduleScreen.tsx
    │   └── ChoreFairnessScreen.tsx
    │
    ├── hooks/
    │   ├── useChores.ts
    │   ├── useTodayChores.ts
    │   ├── useCompleteChore.ts
    │   └── useChoreFairness.ts
    │
    ├── types/
    └── utils/
```

Adapt naming to your existing FairRoom architecture.

------------------------------------------------------------------------

# 66. React Query Strategy

The app already uses server-state patterns elsewhere.

Recommended query keys:

``` text
['rooms', roomId, 'chores']
['rooms', roomId, 'chores', choreId]
['rooms', roomId, 'chore-occurrences', date]
['rooms', roomId, 'chore-occurrences', 'today']
['rooms', roomId, 'chore-fairness', period]
```

After completion:

Invalidate:

``` text
today occurrences
chore detail
fairness
activity
```

Do not invalidate the entire room unnecessarily.

------------------------------------------------------------------------

# 67. Optimistic Completion

Completing a chore is a good candidate for optimistic UI.

Flow:

``` text
tap Complete
    ↓
UI immediately shows completed
    ↓
API request
    ↓
success → keep
failure → rollback + show error
```

Do this only after the basic implementation is stable.

------------------------------------------------------------------------

# 68. iOS UX Direction

The FairRoom app is being developed with React Native + Expo and aims
for an iOS-like visual language.

Chores should feel:

``` text
Clean
Friendly
Playful
Calm
Modern
```

Avoid:

``` text
Excel-like tables
Dense admin dashboards
Corporate task-management UI
Excessive badges
Too many numbers
```

Use:

-   cards;
-   grouped lists;
-   subtle status indicators;
-   haptics for completion;
-   native-feeling sheets;
-   swipe actions where appropriate.

------------------------------------------------------------------------

# 69. Swipe Actions

Useful iOS-style interactions:

``` text
Chore row
→ swipe right
→ Complete
```

or:

``` text
swipe left
→ More
```

Do not hide essential actions exclusively behind gestures.

There must always be a visible action path.

------------------------------------------------------------------------

# 70. Chore Status Language

Prefer friendly language.

Instead of:

``` text
TASK FAILED
```

use:

``` text
Overdue
```

Instead of:

``` text
USER DID NOT COMPLETE
```

use:

``` text
Still waiting
```

Instead of:

``` text
PENALTY
```

use:

``` text
Workload balance
```

------------------------------------------------------------------------

# 71. Empty States

No chores:

``` text
🧹

Nothing scheduled yet.

Add your first shared chore
and let FairRoom handle the rotation.

[Add chore]
```

No chores today:

``` text
✨ All clear!

No chores for you today.
```

Overdue:

``` text
No overdue chores 🎉
```

------------------------------------------------------------------------

# 72. Onboarding

Do not force users to create a complex chore system.

Suggested first-time flow:

``` text
Room created
    ↓
"Want to add some chores?"
    ↓
Quick templates
    ↓
Choose 1–3 chores
    ↓
Assign / Rotate
    ↓
Done
```

Example:

``` text
☑ Take out trash
☑ Mop floor
☑ Clean bathroom
```

------------------------------------------------------------------------

# 73. Quick Setup

For a new room:

``` text
Choose chores

☑ Trash
☑ Dishes
☑ Floor
☑ Bathroom
☐ Kitchen
☐ Laundry
```

Then:

``` text
How should we divide them?

○ I'll assign
● Rotate fairly
○ Leave unassigned
```

This should get a room from zero to usable in under a minute.

------------------------------------------------------------------------

# 74. MVP Scope

The first implementation should include only:

## Chore management

-   [ ] Create chore
-   [ ] Edit chore
-   [ ] Archive chore
-   [ ] Optional area
-   [ ] Effort points
-   [ ] Estimated minutes

## Scheduling

-   [ ] Once
-   [ ] Daily
-   [ ] Weekly
-   [ ] Custom weekdays
-   [ ] Due time

## Assignment

-   [ ] Fixed
-   [ ] Rotation
-   [ ] Unassigned
-   [ ] Basic custom assignment

## Execution

-   [ ] Today list
-   [ ] Upcoming list
-   [ ] Complete
-   [ ] Skip
-   [ ] Overdue

## Fairness

-   [ ] Effort contribution
-   [ ] Weekly distribution
-   [ ] Basic fairness score
-   [ ] Member comparison

## Notifications

-   [ ] Reminder
-   [ ] Overdue notification

------------------------------------------------------------------------

# 75. Phase 2

After MVP is stable:

-   [ ] Cover request
-   [ ] Accept/decline cover
-   [ ] Away mode
-   [ ] Swap
-   [ ] Activity feed
-   [ ] Chore history
-   [ ] Advanced notification settings
-   [ ] Calendar integration
-   [ ] Widget

------------------------------------------------------------------------

# 76. Phase 3

Only after real-room validation:

-   [ ] Fairness-aware assignment suggestions
-   [ ] Rolling fairness
-   [ ] Advanced trends
-   [ ] House-level fairness
-   [ ] Contribution insights
-   [ ] House XP
-   [ ] Streaks
-   [ ] Achievements
-   [ ] Virtual home/pet

------------------------------------------------------------------------

# 77. Explicitly Out of MVP

Do not implement:

-   [ ] AI chore planner
-   [ ] AI roommate judge
-   [ ] automatic punishment
-   [ ] complex gamification
-   [ ] multiple-house management
-   [ ] professional cleaning workflows
-   [ ] complex calendar engine
-   [ ] arbitrary cron schedules
-   [ ] marketplace
-   [ ] social network
-   [ ] chat
-   [ ] paid premium chore analytics

------------------------------------------------------------------------

# 78. User Stories

## US-01 --- Create chore

> As a room manager, I want to create a chore so that everyone knows
> what needs to be done.

Acceptance criteria:

-   title is required;
-   effort is required;
-   schedule is required;
-   room is inferred from current room context;
-   creator must have permission.

------------------------------------------------------------------------

## US-02 --- Assign chore

> As a room manager, I want to assign a chore to a member.

Acceptance:

-   selected member must belong to room;
-   assignment applies to future occurrence(s);
-   past occurrences are not modified.

------------------------------------------------------------------------

## US-03 --- Rotate chore

> As a room manager, I want to rotate a chore between members.

Acceptance:

-   rotation order is stored;
-   future occurrences follow order;
-   removing a member does not corrupt history.

------------------------------------------------------------------------

## US-04 --- Complete chore

> As a member, I want to mark my chore as complete.

Acceptance:

-   occurrence becomes completed;
-   completion time is recorded;
-   actual completer is recorded;
-   fairness contribution updates.

------------------------------------------------------------------------

## US-05 --- Complete someone else's chore

> As a room member, I want to record that I completed another person's
> chore.

Acceptance:

-   original assignment remains;
-   actual completer is recorded;
-   contribution goes to actual completer;
-   history makes the distinction visible.

------------------------------------------------------------------------

## US-06 --- Cover

> As a member who is away, I want another member to cover my chore.

Acceptance:

-   request is sent;
-   receiver can accept/decline;
-   accepted request becomes effective;
-   original responsibility remains visible.

------------------------------------------------------------------------

## US-07 --- Swap

> As a member, I want to swap a chore with another member.

Acceptance:

-   both assignments belong to same room;
-   other member must accept;
-   swap is atomic;
-   history remains correct.

------------------------------------------------------------------------

## US-08 --- Fairness

> As a roommate, I want to know whether chore work is reasonably
> balanced.

Acceptance:

-   show period;
-   show effort by member;
-   show room fairness score;
-   do not shame members;
-   explain that points are estimates.

------------------------------------------------------------------------

# 79. Edge Cases

## Member leaves room

-   future assignments must be handled;
-   past history remains;
-   user should be prompted to reassign future chores.

------------------------------------------------------------------------

## Member joins room

-   no retroactive chores;
-   can be added to future rotation.

------------------------------------------------------------------------

## Chore deleted

Archive instead of destructive deletion when history exists.

------------------------------------------------------------------------

## Chore schedule changed

Existing historical occurrences remain unchanged.

Future occurrences follow new schedule.

------------------------------------------------------------------------

## Effort changed

Recommended rule:

-   existing occurrences keep their historical effort;
-   future occurrences use the new effort.

This prevents historical fairness from changing unexpectedly.

------------------------------------------------------------------------

## Due time changed

Historical records remain unchanged.

Future occurrences use new due time.

------------------------------------------------------------------------

## Room becomes empty of active users

Keep data.

Do not automatically delete chores.

------------------------------------------------------------------------

# 80. Data Integrity Rules

The backend must guarantee:

1.  Every Chore belongs to exactly one Room.
2.  Every ChoreArea belongs to exactly one Room.
3.  Every occurrence belongs to exactly one Chore.
4.  Every assignment belongs to exactly one occurrence.
5.  Every assignment member belongs to the same Room.
6.  Every completion member belongs to the same Room.
7.  Every cover participant belongs to the same Room.
8.  Every swap assignment belongs to the same Room.
9.  A completed occurrence has at most one completion.
10. Historical occurrences are not rewritten by future configuration
    changes.

------------------------------------------------------------------------

# 81. Important Domain Decision

Do not use:

``` text
Chore.assignedMemberId
```

as the only assignment mechanism.

This seems simple initially:

``` text
Chore
  assignedMemberId
```

but breaks when you need:

-   rotation;
-   history;
-   cover;
-   swap;
-   recurring occurrences;
-   actual completer;
-   fairness.

Instead:

``` text
Chore
  ↓
Occurrence
  ↓
Assignment
  ↓
Completion
```

This is one of the most important architectural decisions in this
module.

------------------------------------------------------------------------

# 82. Suggested Implementation Order

## Step 1 --- Domain

Create:

``` text
Chore
ChoreArea
ChoreOccurrence
ChoreAssignment
ChoreCompletion
```

------------------------------------------------------------------------

## Step 2 --- Basic CRUD

Implement:

``` text
Create
Read
Update
Archive
```

------------------------------------------------------------------------

## Step 3 --- Occurrence generation

Implement:

``` text
ONCE
DAILY
WEEKLY
CUSTOM_WEEKDAYS
```

Generate future occurrences.

Do not generate years of occurrences.

Recommended initial horizon:

``` text
30–60 days
```

Generate further when needed.

------------------------------------------------------------------------

## Step 4 --- Assignment

Implement:

``` text
FIXED
ROTATION
UNASSIGNED
```

Then custom assignment.

------------------------------------------------------------------------

## Step 5 --- Completion

Implement:

``` text
Complete
Skip
Overdue
History
```

------------------------------------------------------------------------

## Step 6 --- Fairness

Implement:

``` text
Member effort
Room distribution
Basic score
```

------------------------------------------------------------------------

## Step 7 --- Notifications

Implement:

``` text
Reminder
Overdue
```

------------------------------------------------------------------------

## Step 8 --- Coordination

Implement:

``` text
Cover
Away
Swap
```

------------------------------------------------------------------------

## Step 9 --- iOS Experience

Implement:

``` text
Haptics
Swipe actions
Widgets
Deep links
Calendar
```

------------------------------------------------------------------------

# 83. Suggested Backend Modules

If the FairRoom backend uses a modular architecture:

``` text
modules/
└── chores/
    ├── chore.controller
    ├── chore.service
    ├── chore.repository
    ├── occurrence.service
    ├── assignment.service
    ├── completion.service
    ├── rotation.service
    ├── cover.service
    ├── swap.service
    ├── fairness.service
    └── dto/
```

Keep fairness calculations separate from basic CRUD.

------------------------------------------------------------------------

# 84. Suggested Background Jobs

Recurring chores and notifications are good candidates for background
jobs.

Potential jobs:

``` text
generate-chore-occurrences
send-chore-reminders
mark-overdue-chore
expire-cover-request
expire-swap-request
```

Do not rely on the mobile app being open.

The server should be the source of truth.

------------------------------------------------------------------------

# 85. Recurrence Strategy

Recommended MVP strategy:

``` text
Chore = recurrence rule
Occurrence = generated instance
```

Do not create an infinite number of occurrences.

Generate a rolling window.

Example:

``` text
Today + next 30 days
```

When the window gets low:

``` text
generate next 30 days
```

This avoids unnecessary database growth.

------------------------------------------------------------------------

# 86. Time Zone

Room schedules should use the room's local time.

For FairRoom's current target market:

``` text
Asia/Ho_Chi_Minh
```

However, avoid hardcoding timezone into every chore.

Store dates consistently and convert for presentation/scheduling.

If the Room model later supports timezone:

``` text
Room.timezone
```

use that as the source of truth.

------------------------------------------------------------------------

# 87. Testing Strategy

## Unit tests

Test:

-   recurrence generation;
-   rotation;
-   effort calculation;
-   fairness;
-   overdue calculation;
-   cover rules;
-   swap rules.

------------------------------------------------------------------------

## Integration tests

Test:

``` text
create chore
→ generate occurrence
→ assign
→ complete
→ fairness
```

------------------------------------------------------------------------

## Important test

``` text
Assigned = Quang
Completed by = Minh
```

Expected:

``` text
assignment.member = Quang
completion.completedBy = Minh
Minh receives effort contribution
```

------------------------------------------------------------------------

# 88. Rotation Test Cases

### 3 members

``` text
A B C
```

Expected:

``` text
A B C A B C
```

### Remove B

Future:

``` text
A C A C
```

### Add D

Future after next cycle:

``` text
A B C D
```

depending on configured rotation start.

------------------------------------------------------------------------

# 89. Fairness Test Cases

### Equal

``` text
10 / 10 / 10
```

Expected:

``` text
100
```

### Slightly uneven

``` text
10 / 9 / 11
```

Expected:

``` text
high fairness
```

### Very uneven

``` text
15 / 5 / 10
```

Expected:

``` text
lower fairness
```

Do not hardcode expected exact percentages until the final formula is
frozen.

Test the properties:

``` text
equal distribution → maximum score
greater imbalance → lower score
```

------------------------------------------------------------------------

# 90. Security

Never allow a client to:

-   complete a chore for a member outside the room;
-   assign a chore to another room's member;
-   create a cover request across rooms;
-   swap assignments across rooms;
-   access another room's fairness data.

All relationships must be verified server-side.

------------------------------------------------------------------------

# 91. Analytics

Track product-level events, not invasive personal data.

Recommended events:

``` text
chore_created
chore_completed
chore_skipped
chore_postponed
cover_requested
cover_accepted
cover_declined
swap_requested
swap_accepted
fairness_viewed
reminder_opened
```

Useful metrics:

``` text
chores completed / room / week
completion rate
overdue rate
cover usage
swap usage
rooms with 2+ active members
weekly active rooms
```

------------------------------------------------------------------------

# 92. Product Metrics

The README already emphasizes room retention over individual retention.

Important chore metrics:

### Activation

``` text
Room created
→ First chore created
→ First chore completed
```

### Engagement

``` text
Completed chores / week
```

### Collaboration

``` text
Rooms with 2+ active users
Cover requests
Swap requests
```

### Retention

``` text
D7 room retention
D30 room retention
```

The most important validation question is:

> Does a real room continue using the chore system after the novelty
> disappears?

------------------------------------------------------------------------

# 93. UX Principles

## Rule 1

The user should understand today's responsibility immediately.

## Rule 2

Completing a chore should take one tap.

## Rule 3

Asking someone for cover should take seconds.

## Rule 4

Fairness should be understandable without reading documentation.

## Rule 5

Do not shame.

## Rule 6

Do not force all roommates to install the app.

## Rule 7

Do not make configuration more difficult than using Zalo.

------------------------------------------------------------------------

# 94. Example End-to-End Scenario

Room:

``` text
Room 302

Quang
Minh
Nam
An
```

Chore:

``` text
Take out trash
1 point
Mon / Wed / Fri
19:00
Rotation
```

Generated:

``` text
Sep 14 → Quang
Sep 16 → Minh
Sep 18 → Nam
Sep 21 → An
```

Sep 16:

``` text
Minh is away.
```

Minh:

``` text
Ask Quang to cover.
```

Quang accepts.

Occurrence:

``` text
Assigned:
Minh

Covered by:
Quang

Completed by:
Quang
```

Contribution:

``` text
Quang +1
```

History:

``` text
Sep 16
Minh → assigned
Quang → covered & completed
```

Fairness:

``` text
Quang's actual workload increases by 1.
```

The next rotation still follows the configured rotation sequence unless
the room explicitly changes it.

------------------------------------------------------------------------

# 95. Example Fairness Scenario

Four members:

``` text
Quang  8
Minh   9
Nam   10
An     9
```

UI:

``` text
⚖️ HOUSE FAIRNESS

      94

Quang  ████████   8
Minh   █████████  9
Nam    ██████████ 10
An     █████████  9

Looks balanced.
```

If distribution becomes:

``` text
Quang  14
Minh    4
Nam    10
An      8
```

UI should say:

``` text
⚖️ HOUSE FAIRNESS

      78

Workload is becoming uneven.

Quang has completed more effort
than the room average this period.

Consider rotating the next heavy chore.
```

No accusation.

------------------------------------------------------------------------

# 96. Relationship With Expenses

FairRoom has two major contribution systems:

``` text
Money contribution
+
Chore contribution
```

They should remain separate.

Do not create:

``` text
Quang paid more money
therefore Quang should do fewer chores
```

automatically in MVP.

This is too subjective.

Instead show:

``` text
💰 Money
Chore
🤝 Coverage
```

as separate dimensions.

------------------------------------------------------------------------

# 97. Long-Term Fairness Model

Future Fairness may combine:

``` text
Chore contribution
+ Coverage/help
+ Expense contribution
```

but only at the UX level.

Do not create a universal numeric score such as:

``` text
Quang = 83.42 overall roommate score
```

This would be misleading.

Keep dimensions explainable.

------------------------------------------------------------------------

# 98. Recommended Fairness Screen Structure

``` text
Fairness

This week

⚖️ 92
Looks balanced

────────────────

CHORES

Quang   10 pts
Minh     9 pts
Nam     11 pts
An      10 pts

────────────────

HELPING OTHERS

Quang   Covered 2 times
Minh    Covered 1 time
Nam     Covered 0 times
An      Covered 1 time

────────────────

Tip

Nam has slightly more workload
this week. The next rotation may
naturally balance it.
```

------------------------------------------------------------------------

# 99. What Makes FairRoom Different

The module should not compete with Tody/Sweepy by having more cleaning
features.

Its differentiation is:

``` text
Tody/Sweepy:
"What chores exist in my home?"

FairRoom:
"How should the people sharing this home
divide the responsibility fairly?"
```

That difference should influence every major design decision.

------------------------------------------------------------------------

# 100. Final Architecture

The recommended conceptual architecture is:

``` text
                         FAIRROOM
                            │
                       Shared Room
                            │
                 ┌──────────┴──────────┐
                 │                     │
              MONEY                  CHORES
                 │                     │
             Expenses                Chore
                 │                     │
             Balance             Occurrence
                                       │
                                  Assignment
                                       │
                         ┌─────────────┼─────────────┐
                         │             │             │
                       Fixed        Rotation       Custom
                         │             │             │
                         └─────────────┼─────────────┘
                                       │
                                  Completion
                                       │
                          ┌────────────┴────────────┐
                          │                         │
                       Effort                   Coverage
                          │                         │
                          └────────────┬────────────┘
                                       │
                                  FAIRNESS
                                       │
                                  Dashboard
```

------------------------------------------------------------------------

# 101. Final MVP Definition

The Split Chores MVP is complete when a real room can do this:

``` text
Create Room
     ↓
Add Members
     ↓
Create Chore
     ↓
Choose Frequency
     ↓
Assign / Rotate
     ↓
Generate Occurrences
     ↓
See Today's Chores
     ↓
Receive Reminder
     ↓
Complete Chore
     ↓
Record Actual Contributor
     ↓
Calculate Effort
     ↓
View Fairness
```

And the system correctly handles:

``` text
Solo manager
Guest members
Recurring chores
Rotation
Completion
Overdue
History
```

Cover, Away, Swap, Widgets, and advanced fairness should come after this
core loop is stable.

------------------------------------------------------------------------

# 102. Implementation Checklist

## Domain

-   [ ] Chore
-   [ ] ChoreArea
-   [ ] ChoreOccurrence
-   [ ] ChoreAssignment
-   [ ] ChoreCompletion

## Backend

-   [ ] CRUD
-   [ ] occurrence generation
-   [ ] rotation service
-   [ ] completion service
-   [ ] fairness service
-   [ ] authorization
-   [ ] transactions
-   [ ] tests

## Frontend

-   [ ] Chores home
-   [ ] Today
-   [ ] Upcoming
-   [ ] Create chore
-   [ ] Edit chore
-   [ ] Chore detail
-   [ ] Completion
-   [ ] Fairness

## Notifications

-   [ ] reminder
-   [ ] overdue

## Phase 2

-   [ ] cover
-   [ ] away
-   [ ] swap
-   [ ] history
-   [ ] activity
-   [ ] widget
-   [ ] calendar

------------------------------------------------------------------------

# 103. Product Decision Summary

The following decisions should be treated as the default implementation
direction:

  Decision                Direction
  ----------------------- -------------------------------------------------------------
  Home structure          One shared room/household
  Physical organization   Optional `ChoreArea`
  Core object             Chore responsibility
  Recurring work          Chore → Occurrence
  Assignment              Separate Assignment entity
  Actual work             Separate Completion entity
  Assignment modes        Fixed / Rotation / Custom / Unassigned
  Effort                  1--4 relative points
  Fairness                Based primarily on completed effort
  Responsibility          Preserve original assignment
  Cover                   Temporary responsibility transfer
  Swap                    Mutual assignment exchange
  Away                    Future occurrence coordination
  History                 Never rewrite historical records
  Solo mode               Fully supported
  Guest member            Use `RoomMember`, not `User`
  Notifications           Useful, configurable, low volume
  Fairness language       Neutral, non-judgmental
  MVP                     Chore + schedule + assignment + completion + basic fairness
  Future                  Cover + Away + Swap + widgets + advanced fairness

------------------------------------------------------------------------

# 104. Final Product Principle

FairRoom should not try to answer:

> "Who is the worst roommate?"

It should answer:

> **"What needs to be done, who is responsible, and how can we keep the
> shared workload balanced?"**

The ultimate experience is:

``` text
Share the home.
Share the work.
Keep it fair.
```
