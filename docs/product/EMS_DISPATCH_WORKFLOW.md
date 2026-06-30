# EMS_DISPATCH_WORKFLOW.md

## BFY EMS — Dispatch Workflow Specification

**Work Order:** WO-EMS-003
**Version:** 1.0
**Status:** Specification — Awaiting Review
**Date:** 2026-06-30

---

## 1. Workflow Overview

The Dispatch workflow is the core operational process of the EMS. It covers the complete lifecycle of a maintenance work order from initial request through verified closure.

### 1.1 Current vs. Proposed Status Model

**Current (app.js):**

```
待派工 → 已派工 → 進行中 → 待料 → 已完成
```

**Proposed (WO-EMS-003):**

```
Request → Review → Assign → Accepted → Working → Waiting Parts → Testing → Completed → Verified → Closed
```

**Mapping current statuses to new workflow stages:**

| Current Status | New Workflow Stage | Notes |
|---------------|-------------------|-------|
| 待派工 | Request → Review | Combined; split into submission + review |
| 已派工 | Assign → Accepted | Split: admin assigns; engineer accepts |
| 進行中 | Working | Direct mapping |
| 待料 | Waiting Parts | Direct mapping |
| (none) | Testing | New intermediate stage before completion |
| 已完成 | Completed → Verified → Closed | Split into 3 stages for quality gate |

No Firebase data changes are required for this specification. The new stage names map to the existing status field values during implementation.

---

## 2. Complete Dispatch Workflow

```
┌─────────────────────────────────────────────────────────────┐
│                    DISPATCH WORKFLOW                         │
│                                                             │
│  [1] REQUEST                                                │
│      Trigger: Engineer submits fault report via QR scan     │
│      OR: Admin creates work order manually                  │
│      OR: PM schedule triggers a planned work order          │
│      WO ID created: WO-{date}-{seq} or FR-{date}-{seq}      │
│              │                                              │
│              ▼                                              │
│  [2] REVIEW                                                 │
│      Admin reviews request — validity, priority, scope      │
│      Sets: Priority (高/中/低), Due date, Work scope        │
│      Current screen: Admin → dispatch tab                   │
│      New screen: Dispatch → Pending Assignment              │
│              │                                              │
│              ├──[Reject: duplicate/invalid]──► Closed       │
│              │                                              │
│              ▼                                              │
│  [3] ASSIGN                                                 │
│      Admin assigns engineer (setWOAssign)                   │
│      WO status: 已派工                                      │
│      Current screen: Admin dispatch tab → assignedTo field  │
│      New screen: Dispatch → WO detail → Assign button       │
│              │                                              │
│              ▼                                              │
│  [4] ACCEPTED                                               │
│      Engineer acknowledges assignment in Today's Tasks      │
│      Current screen: #pivot → engineer sees their WOs       │
│      New screen: Today's Tasks → My Work Orders → Accept    │
│              │                                              │
│              ▼                                              │
│  [5] WORKING                                                │
│      Engineer on-site; work in progress                     │
│      Current screen: #scan={id} → progress=進行中          │
│      New screen: Dispatch → WO detail → Update Progress     │
│              │                                              │
│              ├──[Parts needed]──────────────────────────┐  │
│              │                                          ▼  │
│              │                              [6] WAITING PARTS  │
│              │                              Requisition created  │
│              │                              (maintenance/requisitions)  │
│              │                              Current: submitReport → materials  │
│              │                              New: WO detail → Request Parts  │
│              │                              Inventory confirms → status clear  │
│              │                              └──────────────────────┘  │
│              │                                          │   │
│              ▼◄─────────────────────────────────────────┘  │
│  [7] TESTING                                                │
│      Repair complete; running verification test             │
│      Engineer documents test result                         │
│      Current: progress=進行中 (no testing stage)           │
│      New screen: WO detail → Mark as Testing                │
│              │                                              │
│              ├──[Test fails]──► back to [5] WORKING        │
│              │                                              │
│              ▼                                              │
│  [8] COMPLETED                                              │
│      Engineer submits work report                           │
│      Records: hours, materials used, root cause, note       │
│      Current screen: submitReport() → progress=已完成      │
│      New screen: WO detail → Submit Report form             │
│              │                                              │
│              ▼                                              │
│  [9] VERIFIED                                               │
│      Admin or senior engineer verifies completion           │
│      Checks: work report quality, materials recorded, RCA   │
│      Current: setWOStatus → 已完成 (no verify stage)       │
│      New screen: Dispatch → Completed tab → Verify          │
│              │                                              │
│              ├──[Verify fails]──► back to [7] TESTING      │
│              │                                              │
│              ▼                                              │
│  [10] CLOSED                                                │
│      WO permanently closed                                  │
│      Data feeds: Analytics, Equipment History, KPI          │
│      Current: already 已完成 (same as closed)              │
│      New screen: Read-only; appears in Equipment History    │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

---

## 3. Screen Mapping by Stage

### Stage 1 — Request

**Who triggers:**
- Engineer via QR code scan → fault report form (`#report={code}`)
- Admin via "New Work Order" form (Admin → dispatch tab)
- System via PM schedule trigger (planned work)

**Current screens used:**
- `#report={code}` → `faultView()` → fault form submission
- Admin `dispatch` tab → `admDispatch()` → New Work Order form

**New screen:** Dispatch → "+" New Work Order button (always accessible)

**WO record created (existing fields):**
```
id:          WO-{date}-{seq} | FR-{date}-{seq}
source:      故障叫修 | 保養 | 維修 | 優化
equip:       equipment ID
equipName:   equipment display name
item:        work description
priority:    高 / 中 / 低
status:      待派工
from:        engineer | fault | admin | plan
createdAt:   timestamp
```

---

### Stage 2 — Review

**Who triggers:** Admin / Manager

**Current screen:** Admin → `dispatch` tab — work order list is the review surface

**New screen:** Dispatch → Pending Assignment tab — same data, new position

**Actions available:**
- Set or adjust Priority
- Set Due date
- Edit work scope description
- Reject (close as invalid/duplicate)

---

### Stage 3 — Assign

**Who triggers:** Admin / Manager

**Current screen:** `setWOAssign(k, v)` inline in dispatch tab

**New screen:** Dispatch → WO detail → "Assign Engineer" dropdown → Confirm

**WO update:**
```
assignedTo:  engineer name
status:      已派工
```

---

### Stage 4 — Accepted

**Who triggers:** Engineer

**Current screen:** `#pivot` → engineer sees their WOs; no explicit accept action exists

**New screen:** Today's Tasks → My Work Orders → "Accept" button on WO card

**WO update:**
```
status:  進行中 (implicit accept = start working)
```

**Note:** Current app has no explicit Accepted state. This spec introduces it as a UI state only — no data model change required. The transition from 已派工 to 進行中 serves as the accept action.

---

### Stage 5 — Working

**Who triggers:** Engineer

**Current screen:** `#scan={id}` → `renderReportForm()` → progress = 進行中

**New screen:** Dispatch → WO detail → "Update Progress" inline; OR Today's Tasks → tap WO

**Actions available:**
- Add progress note
- Record materials used (partial)
- Request parts → moves to Stage 6

---

### Stage 6 — Waiting Parts

**Who triggers:** Engineer submits materials requisition

**Current screen:** `submitReport()` with materials → `ROOT.child("requisitions").push()` → status 待料

**New screen:** Dispatch → WO detail → "Request Parts" form

**Requisition record (existing fields):**
```
woKey:        work order key
reporter:     engineer name
materials:    [{id, name, qty}]
confirmedAt:  null (pending)
confirmedBy:  null (pending)
```

**Inventory confirmation screen:** Administration → Requisitions → Confirm

**After confirmation:** WO status reverts to Working; engineer sees confirmation banner

---

### Stage 7 — Testing

**Who triggers:** Engineer after repair complete but before report submission

**Current screen:** None — this stage does not exist in current app

**New screen:** Dispatch → WO detail → "Mark as Testing" button

**New UI state only:** This stage is captured as a UI progression indicator. Existing status field = 進行中 until Stage 8.

---

### Stage 8 — Completed

**Who triggers:** Engineer

**Current screen:** `submitReport()` → progress = 已完成 → creates report record

**New screen:** Dispatch → WO detail → "Submit Report" button → report form

**Report record created (existing fields):**
```
woKey:       work order key
equip:       equipment ID
equipName:   equipment name
reporter:    primary engineer
reporters:   all engineers on job
progress:    已完成
hours:       work hours
totalHours:  total hours (multi-engineer)
materials:   [{id, name, qty}]
note:        root cause / notes
source:      work type
createdAt:   timestamp
```

**WO update:**
```
status:    已完成
finishAt:  timestamp
```

**PM update (if planId exists):**
```
lastDone:  today
nextDue:   calculated from plan months
```

---

### Stage 9 — Verified

**Who triggers:** Admin / Senior Engineer

**Current screen:** None — completed WOs have no verification step

**New screen:** Dispatch → Completed tab → "Verify" button on WO card

**Verification checklist (UI only; no new data fields):**
- [ ] Work report submitted
- [ ] Hours recorded
- [ ] Root cause documented
- [ ] Materials recorded (if used)
- [ ] Equipment status confirmed

---

### Stage 10 — Closed

**Who triggers:** System (after verification) or Admin (manual close)

**Current screen:** 已完成 = closed (same state)

**New screen:** Read-only WO detail; visible in Equipment → History

**Data flows to:**
- Equipment → History tab (reports by equipment)
- Analytics → KPI calculations
- Analytics → Cost analysis
- Analytics → Engineer performance

---

## 4. Work Order Types

| Type | ID Prefix | Source | Priority Default |
|------|-----------|--------|-----------------|
| Planned Maintenance (PM) | WO- | PM schedule trigger | 中 |
| Admin-created maintenance | WO- | Admin manual creation | Per admin input |
| Engineer-initiated | WO- | Engineer from `#scan` | 中 |
| Fault Report | FR- | QR scan → fault form | 高 (if urgent) |

---

## 5. Priority Definitions

| Priority | Chinese | Response Target | Dispatch Target |
|----------|---------|----------------|----------------|
| 高 | High | Acknowledge within 30 min | Dispatch within 2 hours |
| 中 | Medium | Acknowledge within 4 hours | Dispatch within 1 working day |
| 低 | Low | Acknowledge within 1 day | Dispatch within 3 working days |

---

## 6. Dispatch Section UI Specification

### 6.1 Navigation Tabs

```
Dispatch Section
└── Tab bar:
    ├── All          — all open WOs
    ├── Pending      — status: 待派工
    ├── In Progress  — status: 已派工 / 進行中
    ├── Waiting Parts — status: 待料
    └── Completed    — status: 已完成 (unverified)
```

### 6.2 Work Order Card (List View)

```
┌──────────────────────────────────────────────┐
│ WO-20260630-1234          [HIGH] [待派工]     │
│ Equipment: Line A - Conveyor Motor (EQ-001)  │
│ Issue: Abnormal noise, vibration detected    │
│ Created: 2026-06-30 09:15 | Due: Today       │
│                            [Assign ▾] [View] │
└──────────────────────────────────────────────┘
```

### 6.3 Work Order Detail Screen

```
WO-20260630-1234
────────────────────────────────────────────────
Status:    [Working — Stage 5/10]
Equipment: Line A - Conveyor Motor
Priority:  HIGH
Assigned:  [Engineer Name]
Due:       2026-06-30

Description:
  Abnormal noise and vibration detected on startup.

Materials Requested: None

Work Reports: [1 report submitted]

Actions:
  [Request Parts]  [Submit Report]  [Mark Testing]
────────────────────────────────────────────────
History:
  09:15  WO created (fault report)
  09:22  Assigned to [Engineer]
  10:05  Engineer accepted
  10:30  Work in progress
```

---

## 7. Migration Risk Assessment

| Risk | Likelihood | Impact | Mitigation |
|------|-----------|--------|------------|
| Engineers used to entering via `#eng` directly | Medium | Medium | Bottom nav always visible; Today's Tasks replaces `#eng` home |
| Admin muscle memory for tab order | Low | Low | All existing admin tabs preserved in Administration section |
| "Testing" stage confusion (new stage) | Low | Low | UI-only state; no data model change; can be skipped if policy requires |
| Verified stage adds delay to close | Low | Medium | Verification is a UI step; admin can bulk-verify; bypass option for trusted engineers |
| Fault report QR flow change | Low | High | QR links point to `#report={code}`; Equipment detail wraps same form — no QR reprint needed |
