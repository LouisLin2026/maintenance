# EMS_PRODUCT_ARCHITECTURE.md

## BFY Engineering Management System — Product Architecture

**Work Order:** WO-EMS-003
**Version:** 1.0
**Status:** Specification — Awaiting Review
**Date:** 2026-06-30
**Branch:** feature/dashboard-refactor-v1

---

## 1. Product Vision

### 1.1 Why the Current Layout Is Insufficient

The current application (v11.x) is structured around **user roles as entry points** — Engineers enter via `#eng`, Admins enter via `#admin`. This creates three problems:

| Problem | Impact |
|---------|--------|
| Work is buried inside admin tabs | The most critical operational activity (dispatch management) requires 2+ taps to reach, and is hidden in an admin-only tab bar |
| No operational overview on entry | The landing page shows login buttons, not today's operational status — a manager cannot see if anything is on fire |
| Role walls fragment the same workflow | An engineer's `#pivot` (dispatch overview) and the admin's `dispatch` tab show the same workflow from different angles, creating duplication and inconsistency |

The current structure is: **"Where are you?" (role) → features**

The refactor changes it to: **"What are you doing?" (workflow) → sections**

### 1.2 Target Architecture: Workflow-Driven Navigation

The new architecture organises every screen around the **operational question it answers**:

| Section | Question Answered |
|---------|-----------------|
| Dashboard | What needs my attention right now? |
| Today's Tasks | What am I / my team working on today? |
| Dispatch | What work orders exist, and where are they in the workflow? |
| Equipment | What equipment do we have, and what is its status? |
| Engineers | Who is available, and what have they done? |
| Projects | What improvement or CAPEX projects are running? |
| Analytics | How are we performing over time? |
| Administration | How is the system configured? |

### 1.3 Scope of This Refactor

This refactor is **Information Architecture only**. All existing business logic, Firebase data paths, and storage structures remain unchanged. No new features are added.

---

## 2. Navigation Tree

```
EMS Application
│
├── Dashboard                         ← Default landing (replaces role-based landing)
│   ├── Today's Dispatch widget       (from: admDispatch live count)
│   ├── Waiting Assignment widget     (from: WO status=待派工)
│   ├── Overdue widget                (from: WO due < today)
│   ├── Critical Equipment widget     (from: equipment criticality flag)
│   ├── Today's PM widget             (from: plans nextDue = today)
│   ├── Waiting Parts widget          (from: requisitions unconfirmed)
│   ├── Weekly KPI widget             (from: reports aggregate)
│   └── Recent Activity widget        (from: audit log last 10)
│
├── Today's Tasks                     ← replaces #eng → #pivot
│   ├── My Work Orders (engineer view)
│   ├── Team Work Orders (manager view)
│   └── Quick Report button           (from: engScan flow)
│
├── Dispatch                          ← replaces admin dispatch tab + engineer pivot
│   ├── All Work Orders
│   │   ├── Pending Assignment        (status: 待派工)
│   │   ├── In Progress               (status: 已派工 / 進行中)
│   │   ├── Waiting Parts             (status: 待料)
│   │   └── Completed                 (status: 已完成)
│   ├── New Work Order form           (from: addWO)
│   └── Fault Reports                 (from: #report=, FR- prefix WOs)
│
├── Equipment                         ← Equipment Center (new unified view)
│   ├── Equipment Overview            (all equipment, status summary)
│   ├── Equipment Detail              (from: engScan + equipment master)
│   │   ├── PM Schedule               (from: admPlan / plans)
│   │   ├── History                   (from: admHist / reports by equipment)
│   │   ├── QR Code                   (from: admQR)
│   │   ├── Documents                 (spec only; future)
│   │   ├── Critical Points           (spec only; future)
│   │   ├── Spare Parts               (from: material + requisitions)
│   │   └── PLC Backup                (spec only; future)
│   └── Equipment Master Edit         (from: admMaster)
│
├── Engineers                         ← replaces name grid + #myperf
│   ├── Team Overview                 (all engineers, today's assignments)
│   ├── Engineer Profile              (individual work history)
│   │   ├── Today's Tasks
│   │   ├── Active Work Orders
│   │   ├── Daily Report
│   │   ├── Weekly Report
│   │   ├── Skills
│   │   ├── KPI
│   │   ├── Working Hours
│   │   └── History
│   └── Availability Status
│
├── Projects                          ← new section (currently no UI; future)
│   ├── Project Register
│   ├── Active Projects
│   └── Project Detail
│
├── Analytics                         ← replaces admin cost/perf tabs
│   ├── KPI Summary                   (from: admPerf)
│   ├── Cost Analysis                 (from: admCost)
│   ├── Equipment Performance         (from: admPerf equipment section)
│   ├── Engineer Performance          (from: admPerf engineer section)
│   └── PM Compliance                 (from: admPlan completion tracking)
│
└── Administration                    ← replaces admin area/config/audit tabs
    ├── Equipment Master              (from: admMaster)
    ├── Materials Master              (from: admMaster materials section)
    ├── Inventory Management          (from: #inventory)
    ├── Engineers & Roles             (from: REPORTERS config)
    ├── Area / QR Management          (from: admArea + admQR)
    ├── Requisitions                  (from: requisitions Firebase path)
    ├── System Config                 (from: config — laborRate, PIN)
    └── Audit Log                     (from: admAuditLog)
```

---

## 3. Screen Hierarchy

```
Level 0 — Application Shell
  └── Sticky header + bottom navigation bar (8 sections)

Level 1 — Section Home Screens
  ├── /dashboard            Summary cards + widget grid
  ├── /tasks                My tasks + team tasks toggle
  ├── /dispatch             Work order list with status filter tabs
  ├── /equipment            Equipment grid/list with status indicators
  ├── /engineers            Team roster with assignment status
  ├── /projects             Project register (future)
  ├── /analytics            KPI charts + performance tables
  └── /admin                Settings categories

Level 2 — Detail Screens
  ├── /dispatch/[wo-id]     Work order detail + action buttons
  ├── /equipment/[eq-id]    Equipment Center (tabbed)
  └── /engineers/[name]     Engineer workspace (tabbed)

Level 3 — Action Screens
  ├── /dispatch/new         New work order form
  ├── /dispatch/[id]/report Work report submission form
  └── /equipment/[id]/edit  Equipment master edit
```

---

## 4. User Journeys

### 4.1 Engineer Journey — Report Work on a Work Order

```
Open App
  │
  ▼
Dashboard — sees own open WO in "Today's Dispatch" widget
  │
  ▼
Today's Tasks — own work orders listed
  │
  ▼
Tap Work Order → Dispatch detail screen
  │
  ▼
Tap "Report Work" → Work report form (replaces engScan form)
  │
  ├── [needs parts] → Submit requisition → status: 待料
  │
  └── [work done] → Submit completion → status: 已完成
```

### 4.2 Engineer Journey — Fault Report from Floor

```
Open App
  │
  ▼
Scan QR Code → Equipment Detail screen
  │
  ▼
Tap "Report Fault" → Fault report form
  │
  ▼
Submit → creates FR- Work Order → status: 待派工
  │
  ▼
Admin notified → Dispatch section shows new fault report
```

### 4.3 Manager Journey — Morning Overview

```
Open App
  │
  ▼
Dashboard — scans all widgets:
  ├── 3 WOs waiting assignment → tap → Dispatch / Pending tab
  ├── 1 overdue WO → tap → Dispatch / Overdue view
  ├── 2 PM due today → tap → Equipment / PM section
  └── Weekly KPI → tap → Analytics
```

### 4.4 Manager Journey — Dispatch a Work Order

```
Dispatch section
  │
  ▼
Filter: Pending Assignment
  │
  ▼
Tap WO → WO detail
  │
  ▼
Assign engineer → WO status: 已派工
  │
  ▼
Engineer notified → appears in Today's Tasks for engineer
```

---

## 5. Workflow Mapping

### 5.1 Current Screens → New Navigation Sections

| Current Screen / Tab | New Section | New Sub-location |
|---------------------|-------------|-----------------|
| Landing page (Home) | Dashboard | Default entry |
| `#eng` engineer home | Today's Tasks | Section home |
| `#pivot` dispatch overview | Dispatch + Today's Tasks | Split by role |
| `#eng_pick` equipment select | Dispatch → New WO / Equipment | Context-driven |
| `#scan={id}` work report | Dispatch → WO detail → Report | Inline action |
| `#myperf` my performance | Engineers → Profile → KPI | Within Engineer workspace |
| `#report={code}` fault report | Equipment → Report Fault | From Equipment detail |
| Admin `dispatch` tab | Dispatch | Primary section |
| Admin `reports` tab | Analytics + Dispatch history | Split by use |
| Admin `master` tab (equip) | Administration → Equipment Master | Admin section |
| Admin `master` tab (materials) | Administration → Materials Master | Admin section |
| Admin `cost` tab | Analytics → Cost Analysis | Analytics section |
| Admin `perf` tab | Analytics → Engineer/Equip Performance | Analytics section |
| Admin `plan` tab | Equipment → PM Schedule | Within Equipment Center |
| Admin `hist` tab | Equipment → History | Within Equipment Center |
| Admin `area` tab | Administration → Area / QR | Admin section |
| Admin `qr` tab | Administration → Area / QR | Admin section |
| Admin `audit` tab | Administration → Audit Log | Admin section |
| `#inventory` | Administration → Inventory | Admin section |

### 5.2 New Sections Without Current Equivalent

| Section | Status |
|---------|--------|
| Dashboard | New — aggregates existing data into widgets |
| Engineers workspace | Partial — consolidates `#eng`, name grid, `#myperf` |
| Equipment Center tabs (Docs, Critical Points, PLC) | Specification only — future implementation |
| Projects | Specification only — future module |

---

## 6. Future Module Integration Points

The navigation architecture is designed so future modules plug in without restructuring:

| Future Module | Plug-in Point |
|--------------|---------------|
| People / HR | Engineers section expands to full People Management |
| Project Management | Projects section becomes full CAPEX + improvement tracking |
| Risk Management | Dashboard adds Risk widget; Equipment adds Risk tab |
| QA | New top-level section, or Equipment adds Quality tab |
| Inventory (full) | Administration → Inventory evolves to full Inventory section |
| Documents / SOP | Equipment → Documents tab activates |
| KPI Targets | Analytics → KPI section expands with target setting |
