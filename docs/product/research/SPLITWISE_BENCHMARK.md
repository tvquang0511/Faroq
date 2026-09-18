# Splitwise Features — Research & FairRoom Reference

> Purpose: use Splitwise as the primary reference model for FairRoom's Money / Expense module.
>
> This document separates **observed Splitwise capabilities** from **FairRoom implementation decisions**. It is a product-research specification, not a copy of Splitwise's proprietary implementation.

---

## 1. Product Model

Splitwise is fundamentally an expense-sharing and balance-tracking system:

```text
Users / Friends / Groups
        ↓
     Expenses
        ↓
   Expense Splits
        ↓
      Balances
        ↓
     Settle Up
        ↓
 Optional Debt Simplification
```

For FairRoom, the closest context is a persistent `Room` representing an apartment / shared home.

### Core objects

```text
User
Friendship
Group
Expense
ExpenseSplit
RecurringExpense
Settlement
Balance
Category
Currency
```

FairRoom adaptation:

```text
Room
RoomMember
Expense
ExpenseSplit
RecurringExpenseRule
Settlement
Balance (derived)
Category
```

---

# 2. Groups

## 2.1 Create Group

Splitwise supports groups for ongoing shared expenses such as an apartment, trip, family, etc.

Group types exposed by the public API include:

- home
- trip
- couple
- other
- apartment
- house

For FairRoom:

```text
Room
├── name
├── owner
├── members
└── money settings
```

### Test

1. Create a group.
2. Add 3–4 people.
3. Verify every member can see the group.
4. Add an expense.
5. Verify balances are group-specific.

---

# 3. Friends / Non-group Expenses

Splitwise also supports one-off expenses outside a group.

Example:

```text
Quang ↔ Minh

Dinner
300,000₫
```

This is useful when a shared expense does not belong to a persistent room/group.

### FairRoom decision

MVP can omit friend-only expenses because the core domain is a Room.

---

# 4. Add Expense

A Splitwise expense contains concepts such as:

```text
Description
Amount
Paid by
Participants
Date
Notes/details
Category
Currency
Group
Split method
Recurrence
```

The public API also exposes:

```text
description
cost
details
date
repeat_interval
currency_code
category_id
group_id
split_equally
```

### FairRoom MVP

```text
Title
Amount
Paid by
Split between
Split method
Category
Expense date
Note
```

---

# 5. Who Paid vs Who Owes

These are separate concepts.

Example:

```text
Paid by:
Quang

Split between:
Quang
Minh
Nam
An
```

Quang may pay 450,000₫ while only being responsible for 112,500₫.

Therefore:

```text
payer != participant
```

This must be reflected in the domain model.

---

# 6. Split Methods

Splitwise currently documents these methods:

1. Equally
2. Exact amounts
3. Percentage
4. Shares
5. Adjustment
6. Reimbursement
7. Itemized

---

## 6.1 Equal Split

Default method.

Example:

```text
450,000₫ / 4

Quang 112,500
Minh  112,500
Nam   112,500
An    112,500
```

Users can remove people from the split; the remaining amount is divided among the remaining participants.

### FairRoom

**MVP: YES**

---

## 6.2 Exact Amount

Each participant receives an explicit amount.

Example:

```text
450,000₫

Quang 100,000
Minh 120,000
Nam  130,000
An   100,000
```

Invariant:

```text
sum(splits) == expense.amount
```

### Validation

Reject:

```text
100 + 100 + 100 + 50 != 450
```

### FairRoom

**MVP: YES**

---

## 6.3 Percentage

Example:

```text
500,000₫

Quang 40%
Minh  30%
Nam   20%
An    10%
```

Validation:

```text
sum(percentages) == 100%
```

### FairRoom

V2.

---

## 6.4 Shares

Shares express relative responsibility.

Example:

```text
500,000₫

Quang 2 shares
Minh  1 share
Nam   1 share
An    1 share
```

Total shares:

```text
5
```

Result:

```text
Quang 200,000
Minh  100,000
Nam   100,000
An    100,000
```

### FairRoom

V2.

---

## 6.5 Adjustment

Adjustment changes one person's share and distributes the remainder equally among the others.

Example:

```text
600,000₫

Base equal split:
150,000 each

Quang adjustment:
+50,000

Quang:
200,000

Remaining:
400,000
÷ 3
```

### FairRoom

V2/V3.

---

## 6.6 Reimbursement

Used when a refund/reimbursement needs to be distributed.

Example:

```text
Refund:
200,000₫

Recipient:
Quang

Original participants:
Quang / Minh / Nam
```

The reimbursement should affect balances appropriately without being confused with an ordinary expense.

### FairRoom

V2.

---

## 6.7 Itemized Expense

Individual items can be assigned to specific people.

Example:

```text
Milk             30k → Quang
Chicken          80k → Quang + Minh
Vegetables       40k → Everyone
```

Additional components can include:

```text
Tax
Tip
Discount
```

Receipt scanning/itemization is associated with Splitwise Pro.

### FairRoom

V3.

Do not build OCR first.

---

# 7. Expense Date

Expense date is not necessarily the same as creation date.

Example:

```text
expenseDate = Sep 1
createdAt   = Sep 3
```

This matters for monthly reports.

FairRoom should store both.

---

# 8. Categories

Splitwise supports categorizing expenses.

Typical FairRoom categories:

```text
🏠 Rent
⚡ Electricity
💧 Water
🌐 Internet
🛒 Groceries
🥤 Drinks
🧻 Shared supplies
🧹 Cleaning
🔧 Repair
📦 Other
```

Allow custom categories later.

---

# 9. Notes / Details

An expense can contain extra details.

Examples:

```text
"Electricity for August"
"Bought at Coopmart"
"Includes delivery"
```

FairRoom:

```text
note: String?
```

---

# 10. Recurring Expenses

Splitwise supports recurring expenses.

Documented intervals:

```text
Weekly
Fortnightly
Monthly
Yearly
```

Recurring expenses continue until stopped or otherwise invalidated.

The latest recurring expense can be edited; changes to the amount apply to future occurrences but not past expenses.

Users can cancel future recurrences without deleting historical occurrences.

### FairRoom

Use:

```text
RecurringExpenseRule
```

rather than mutating historical expenses.

Example:

```text
Internet
300,000₫
Monthly
Day 5
Paid by Quang
Split: current room members
```

Each generated occurrence should snapshot its participants.

---

# 11. Balance Engine

For every member:

```text
balance =
    totalPaid
  - totalShare
  - settlementsPaid
  + settlementsReceived
```

Convention:

```text
positive = others owe this member
negative = this member owes others
```

Invariant:

```text
sum(all balances) == 0
```

Example:

```text
Quang +175,000
Minh  -25,000
Nam   +75,000
An   -225,000
```

---

# 12. Settle Up

A settlement is separate from an expense.

Example:

```text
An → Quang
150,000₫
```

Domain:

```text
Settlement
├── fromMemberId
├── toMemberId
├── amount
├── status
├── createdAt
└── settledAt
```

Suggested status:

```text
PENDING
CONFIRMED
CANCELLED
```

A settlement must not delete or rewrite the original expense.

---

# 13. Record External Payment

A user may settle using:

```text
Cash
Bank transfer
Other external method
```

FairRoom MVP can simply provide:

```text
[Mark as paid]
```

Later:

```text
Payment method
Payment reference
Payment date
```

---

# 14. Debt Simplification

Splitwise's Simplify Debts feature reduces the number of payments without changing anyone's total balance.

Example:

```text
Before:
8 payments

After:
3 payments
```

Important:

```text
balances stay identical
payment paths change
```

FairRoom should treat simplification as a settlement suggestion algorithm.

### Example

```text
Creditors:
Quang +175
Nam    +75

Debtors:
An   -225
Minh  -25
```

Possible result:

```text
An   → Quang 175
An   → Nam    50
Minh → Nam    25
```

---

# 15. Simplification Algorithm

Use a greedy debtor/creditor matcher for MVP.

Pseudo:

```text
while debtors and creditors:
    debtor = first debtor
    creditor = first creditor

    amount = min(
        abs(debtor.balance),
        creditor.balance
    )

    create transfer(debtor, creditor, amount)

    update both balances

    remove any balance that reaches zero
```

Do not mutate expenses.

---

# 16. Rounding

Money must not use floating-point arithmetic.

For VND:

```text
Int64
```

Example:

```text
100,000 / 3

33,334
33,333
33,333
```

Invariant:

```text
sum(splits) == expense.amount
```

---

# 17. Multi-currency

Splitwise supports many currencies and currency conversion.

FairRoom:

### MVP

```text
VND only
```

But keep:

```text
currencyCode
```

in the model.

Later:

```text
USD
JPY
EUR
...
```

Never sum balances across currencies without conversion.

---

# 18. Expense Editing

Editing an expense must recalculate:

```text
ExpenseSplit
↓
Balances
↓
Settlement suggestions
```

Test:

```text
Original:
400k

Change:
450k
```

All affected balances should update.

---

# 19. Expense Deletion

Deleting an expense must remove its contribution to balances.

Historical settlements should not silently disappear.

For production, consider:

```text
deletedAt
```

instead of hard deletion.

---

# 20. Member Changes

A member with a non-zero balance should not simply disappear from accounting.

Important cases:

```text
Member joins
Member leaves
Member owes money
Member is owed money
Member participates in recurring expense
```

A recurring rule should not rewrite old expenses.

---

# 21. Permissions

Splitwise's current help material indicates that groups do not have a traditional admin/permission system for preventing members from editing/deleting expenses.

FairRoom can intentionally differ:

```text
Room owner
Member
```

but expense editing should remain collaborative enough to fix mistakes.

---

# 22. Dashboard

Useful financial information:

```text
Total room spending
Your spending
Your share
You owe
You are owed
Recent expenses
```

Example:

```text
SEPTEMBER

Room spending
4,820,000₫

You paid
1,450,000₫

Your share
1,210,000₫

Balance
+240,000₫
```

---

# 23. Expense History

Must support:

```text
Chronological list
Category
Payer
Participants
Amount
Date
Settlement relationship
```

Useful filters:

```text
All
Mine
Paid by me
Owed by me
Category
Date
```

---

# 24. Search

Splitwise Pro includes expense search.

FairRoom can later support:

```text
Search:
"electricity"

Filter:
September
```

---

# 25. Reports / Charts

Splitwise Pro includes charts and graphs.

FairRoom can later expose:

```text
Monthly spending
Category spending
Member contribution
Your share
Paid vs owed
```

Do not confuse contribution charts with fairness scoring.

---

# 26. Save Default Splits

Splitwise Pro includes saving default splits.

For FairRoom this is highly valuable for recurring household expenses.

Example:

```text
Internet

Default split:
Quang 25%
Minh  25%
Nam   25%
An    25%
```

Or:

```text
Electricity

Default:
Quang 2 shares
Minh  1 share
Nam   1 share
An    1 share
```

---

# 27. Offline Mode / Sync

Splitwise advertises offline mode and cloud sync.

FairRoom iOS architecture should support:

```text
Local-first
    ↓
Repository
    ↓
Sync engine
    ↓
Backend
```

Do not make the expense UI depend on a network round trip.

---

# 28. Notifications

Relevant money notifications:

```text
Expense added
Expense edited
You owe someone
Someone owes you
Recurring expense posted
Settlement recorded
```

FairRoom should avoid notification spam.

---

# 29. Payment Integrations

Splitwise supports payment integrations in supported markets.

Examples documented by Splitwise include:

```text
Venmo
PayPal
Splitwise Pay
Pay by Bank
```

These are not necessary for FairRoom MVP.

Vietnam-first future options could include local payment flows, but do not couple the accounting engine to a payment provider.

---

# 30. API

Splitwise has an official public API.

The API exposes resources including:

```text
Users
Groups
Expenses
Currencies
Categories
```

The create-expense API supports equal group splits or explicit shares and recurring intervals.

FairRoom should treat Splitwise's API as a **behavioral reference**, not as source code.

---

# 31. FairRoom MVP Feature Set

Implement first:

```text
[ ] Create Room
[ ] Add members
[ ] Add expense
[ ] Edit expense
[ ] Delete expense

[ ] Equal split
[ ] Exact split

[ ] Choose payer
[ ] Choose participants
[ ] Categories
[ ] Expense date
[ ] Notes

[ ] Balance calculation

[ ] Settle up
[ ] Mark as paid

[ ] Basic settlement suggestions
```

---

# 32. FairRoom V2

```text
[ ] Recurring expenses
[ ] Shares
[ ] Percentage
[ ] Debt simplification
[ ] Expense search
[ ] Filters
[ ] Monthly reports
[ ] Default splits
[ ] Transfer / reimbursement
[ ] Advanced settlement history
```

---

# 33. FairRoom V3

```text
[ ] Itemized expenses
[ ] Receipt scanning
[ ] OCR
[ ] Tax
[ ] Tip
[ ] Discount
[ ] Multi-currency
[ ] Currency conversion
[ ] Advanced charts
[ ] Payment integrations
```

---

# 34. Splitwise Test Plan

Create a test group:

```text
ROOM 302

Quang
Minh
Nam
An
```

Use realistic amounts in VND if the UI supports it, otherwise use USD.

---

## TEST 01 — Equal split

Create:

```text
Electricity
400
Paid by Quang
Split equally among 4
```

Expected:

```text
Each = 100
Quang balance = +300
Minh = -100
Nam  = -100
An   = -100
```

---

## TEST 02 — Exclude participant

Create:

```text
Internet
300

Quang
Minh
Nam
```

Exclude An.

Expected:

```text
100 each
An unaffected
```

---

## TEST 03 — Exact split

```text
450

Quang 100
Minh 120
Nam  130
An   100
```

Verify total = 450.

---

## TEST 04 — Percentage

```text
500

40%
30%
20%
10%
```

Verify:

```text
200
150
100
50
```

---

## TEST 05 — Shares

```text
500

Quang 2
Minh 1
Nam  1
An   1
```

Expected:

```text
200
100
100
100
```

---

## TEST 06 — Adjustment

Test:

```text
600

Equal base
Quang +50 adjustment
```

Verify Quang receives the adjustment and the remaining amount is rebalanced.

---

## TEST 07 — Rounding

Create:

```text
100
3 people
equal
```

Check how Splitwise distributes the rounding remainder.

Record the exact behavior for FairRoom tests.

---

## TEST 08 — Different payer

Create:

```text
300
Paid by Minh
Split:
Quang / Minh / Nam
```

Expected:

```text
Minh paid 300
Each owes 100
Minh net +200
Quang -100
Nam -100
```

---

## TEST 09 — Multiple expenses

Create:

```text
Expense A
400
Quang pays

Expense B
200
Minh pays

Expense C
300
Nam pays
```

All split equally among 4.

Record the resulting balances.

---

## TEST 10 — Simplify debts OFF

Create several expenses that create circular obligations.

Record:

```text
who owes whom
number of payments
```

---

## TEST 11 — Simplify debts ON

Enable Simplify Debts.

Repeat TEST 10.

Verify:

```text
number of payment paths decreases
total balance does not change
```

---

## TEST 12 — Settlement

Record:

```text
An → Quang
100
```

Verify:

```text
An balance increases by 100
Quang balance decreases by 100
```

---

## TEST 13 — External payment

Record a payment as cash/bank transfer if available.

Verify that it changes the accounting record without creating a normal expense.

---

## TEST 14 — Edit expense

Create:

```text
400
```

then change:

```text
450
```

Verify every affected balance updates.

---

## TEST 15 — Delete expense

Create an expense.

Record balances.

Delete it.

Verify the expense contribution disappears.

---

## TEST 16 — Recurring expense

Create:

```text
Internet
300
Monthly
```

Verify:

```text
next occurrence
repeat frequency
future instance
```

Then edit the latest occurrence and verify whether the change applies to future occurrences while historical instances remain unchanged.

---

## TEST 17 — Cancel recurring

Cancel future recurrence.

Verify:

```text
past instances remain
future instances stop
```

---

## TEST 18 — Add member

Add:

```text
Huy
```

Verify whether existing expenses change.

They should not silently change.

---

## TEST 19 — Remove member

Try removing a member with:

```text
balance = 0
```

Then try:

```text
balance != 0
```

Observe the restriction and document the exact behavior.

---

## TEST 20 — Non-group expense

Create a one-off expense between two friends outside the group.

Verify:

```text
group balance
vs
overall balance
```

are distinguishable.

---

## TEST 21 — Categories

Create expenses in different categories.

Verify category appears in:

```text
expense detail
history
reports
```

---

## TEST 22 — Date

Create an expense with:

```text
expense date = previous month
created today
```

Check how dashboard/history/reporting handles it.

---

## TEST 23 — Notes

Add details/notes.

Edit them.

Verify persistence.

---

## TEST 24 — Currency

Create expenses in different currencies if your account supports them.

Check whether balances are kept separately or converted.

Do not assume conversion behavior; record what the current product actually does.

---

## TEST 25 — Search

Create:

```text
Electricity September
Water September
Internet September
```

Search:

```text
electricity
```

Verify results.

---

## TEST 26 — Itemization

Create:

```text
Milk
Chicken
Water
```

Assign items to different people.

Add:

```text
tax
tip
discount
```

Verify the final total and participant shares.

---

## TEST 27 — Default split

If available in your current plan:

Create a reusable/default split and use it on another expense.

---

## TEST 28 — Offline

Disable network.

Try:

```text
open group
add expense
edit expense
```

Reconnect and verify sync.

---

## TEST 29 — Concurrent editing

Use two devices/accounts.

```text
Device A:
add expense

Device B:
open group
```

Then reverse the order.

Record conflict behavior.

This is especially valuable for FairRoom's future sync engine.

---

## TEST 30 — Permission behavior

Have:

```text
Quang creates expense
Minh edits it
Nam attempts deletion
```

Record exactly who can edit/delete what.

---

# 35. Research Log

When testing Splitwise, maintain:

```text
Feature
Observed behavior
Plan required
Platform
Expected
Actual
FairRoom decision
Screenshot
```

Example:

```text
Feature:
Exact split

Observed:
Can assign exact amount to each participant.

Platform:
iOS / Web

Plan:
Free

FairRoom:
MVP
```

This prevents assumptions from becoming product requirements.

---

# 36. What NOT to Copy

Do not copy Splitwise's:

```text
UI
Brand
Icons
Text
Screenshots
Source code
Proprietary algorithms
```

Use it as a behavioral/product reference.

FairRoom should have its own:

```text
UX
Domain model
Visual language
Vietnamese categories
Room workflows
Fairness system
```

---

# 37. Recommended FairRoom Architecture

```text
Money
│
├── Expense
│   ├── Create
│   ├── Edit
│   ├── Delete
│   └── Split
│       ├── Equal
│       ├── Exact
│       ├── Shares
│       ├── Percentage
│       └── Itemized
│
├── Recurring
│
├── Balance
│
├── Settlement
│
├── Simplification
│
└── Reports
```

Source of truth:

```text
Expenses
ExpenseSplits
Settlements
```

Derived:

```text
Balances
Settlement suggestions
Reports
Fairness metrics
```

---

# 38. Core Invariants

Every implementation should enforce:

```text
sum(ExpenseSplit.amount)
    == Expense.amount
```

and:

```text
sum(RoomMember.balance)
    == 0
```

and after a complete settlement:

```text
all balances == 0
```

and:

```text
editing an expense
    never silently changes historical expenses
```

and:

```text
settlement
    never modifies the original expense
```

---

# 39. Final Priority

For FairRoom:

### Must understand deeply

```text
★★★★★ Expense model
★★★★★ Split methods
★★★★★ Balance calculation
★★★★★ Settlement
★★★★★ Debt simplification
★★★★★ Recurring expenses
★★★★★ Rounding
★★★★★ Editing/deleting
```

### Study but implement later

```text
★★★★ Percentage
★★★★ Shares
★★★★ Adjustment
★★★★ Reimbursement
★★★ Itemization
★★★ Currency conversion
★★★ Reports
★★★ Receipt scanning
```

### Ignore initially

```text
Payment integrations
Advanced subscription features
Social features
```

---

# 40. Primary Research Sources

Official Splitwise Help Center:
https://kb.splitwise.com/

Official Splitwise API:
https://dev.splitwise.com/

Official Splitwise GitHub organization:
https://github.com/splitwise

These should be treated as the primary sources when researching Splitwise behavior.
