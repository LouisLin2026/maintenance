# EMS_ENGINEER_WORKSPACE.md

## BFY EMS — Engineer Workspace Specification

**Work Order:** WO-EMS-003
**Version:** 1.0
**Status:** Specification — Awaiting Review
**Date:** 2026-06-30

---

## 1. Overview

The Engineer Workspace consolidates all engineer-facing screens from the current application into a single, coherent workspace. It replaces:

| Current Screen | Replaced By |
|---------------|-------------|
| `#eng` name picker → name grid | Engineers section home → Identity selection |
| `#pivot` dispatch overview | Today's Tasks → My Work Orders |
| `#eng_pick` equipment picker | Dispatch → New WO (context-driven) |
| `#scan={id}` work report form | Dispatch → WO detail → Submit Report |
| `#myperf` my performance | Engineers → Profile → KPI tab |

The workspace has two entry contexts:
- **Today's Tasks** — operational context (what to work on today)
- **Engineers** — team context (who the team is, individual profiles, KPIs)

---

## 2. Today's Tasks Section

### 2.1 Purpose

The operational home for engineers. Replaces `#eng` + `#pivot` as the primary daily entry point. Engineers open this to see what they are working on today and take action directly.

### 2.2 Screen: Today's Tasks Home

```
┌──────────────────────────────────────────────────────┐
│  Today's Tasks                      [My Work | Team] │
├──────────────────────────────────────────────────────┤
│  [Engineer: ________________]  (identity selection)  │
├──────────────────────────────────────────────────────┤
│                                                      │
│  ● WO-20260630-1234  [HIGH] [進行中]                 │
│    Line A - Conveyor Motor                           │
│    Abnormal noise and vibration detected             │
│    Due: Today  │  Assigned: 09:22                   │
│    [View Details]  [Submit Report]                   │
│                                                      │
│  ● WO-20260628-0891  [MED] [待料]                   │
│    Pump Station 3 - Hydraulic Pump                   │
│    Seal replacement — waiting for parts              │
│    Due: Tomorrow  │  Parts requested: Yesterday      │
│    [View Details]                                    │
│                                                      │
│  ── Completed Today ──────────────────────────────── │
│  ✓ WO-20260629-0756  [MED] [已完成]                 │
│    Electrical panel lubrication                      │
│    Completed at 14:30                                │
│                                                      │
└──────────────────────────────────────────────────────┘
```

### 2.3 Data Sources

| Display Element | Firebase Path | Filter |
|----------------|--------------|--------|
| My Work Orders | `maintenance/workorders` | `assignedTo == engineer name` |
| WO status badges | `workorders/{id}/status` | — |
| Due date indicator | `workorders/{id}/due` | — |
| Completed today | `workorders/{id}/status == 已完成`, `finishAt` == today | — |

### 2.4 Identity Selection

The current app uses a name grid at `#eng` entry. The new workspace preserves this:
- Engineer selects their name on first load (or if identity not set)
- Identity persists in `localStorage` for session continuity
- "Switch Identity" button always visible in header
- Names sourced from `REPORTERS` config constant (same as current app)

### 2.5 Toggle: My Work | Team View

| View | Content | Who Sees It |
|------|---------|-------------|
| My Work | Only WOs assigned to the selected engineer | All engineers |
| Team | All open WOs for all engineers today | Manager / senior engineer |

### 2.6 Quick Actions from Today's Tasks

| Action | Trigger | Result |
|--------|---------|--------|
| Submit Report | Button on WO card | Opens work report form (inline or detail screen) |
| View Details | Button on WO card | Opens Dispatch → WO detail |
| New Fault Report | FAB (floating action button) | Opens fault report form; pre-fills engineer identity |

---

## 3. Engineers Section

### 3.1 Purpose

Team-level view of all engineers: who is available, what everyone is working on, and individual performance history. Replaces the name grid as a roster view and consolidates `#myperf` into individual profiles.

### 3.2 Screen: Engineers Section Home (Team Roster)

```
┌──────────────────────────────────────────────────────┐
│  Engineers                               [Today | Week] │
├──────────────────────────────────────────────────────┤
│                                                      │
│  ┌──────────────┐  ┌──────────────┐                 │
│  │ [Name A]     │  │ [Name B]     │                 │
│  │ ● 2 active   │  │ ○ Available  │                 │
│  │ 1 waiting    │  │ 0 open WOs   │                 │
│  │ [View]       │  │ [View]       │                 │
│  └──────────────┘  └──────────────┘                 │
│                                                      │
│  ┌──────────────┐  ┌──────────────┐                 │
│  │ [Name C]     │  │ [Name D]     │                 │
│  │ ● 1 active   │  │ ✓ Done today │                 │
│  │ 0 waiting    │  │ 3 completed  │                 │
│  │ [View]       │  │ [View]       │                 │
│  └──────────────┘  └──────────────┘                 │
│                                                      │
└──────────────────────────────────────────────────────┘
```

**Status indicators:**
- `●` Green — has active work orders
- `○` Grey — no active WOs (available)
- `✓` Blue — all WOs completed today

### 3.3 Data Sources for Team Roster

| Display Element | Firebase Path | Calculation |
|----------------|--------------|-------------|
| Engineer names | `REPORTERS` config constant | Static list from app config |
| Active WO count | `maintenance/workorders` | Count where `assignedTo == name AND status IN (已派工, 進行中)` |
| Waiting count | `maintenance/workorders` | Count where `assignedTo == name AND status == 待料` |
| Completed today | `maintenance/workorders` | Count where `assignedTo == name AND status == 已完成 AND finishAt >= today` |

---

## 4. Engineer Profile Workspace

### 4.1 Purpose

Each engineer has a personal workspace screen accessible from the team roster. It is a tabbed interface showing all information about that engineer's work, performance, and history.

### 4.2 Profile Header

```
┌──────────────────────────────────────────────────────┐
│  ← Engineers     [Name]           Engineer           │
│  Active WOs: 2 │ Today's Hours: 4.5h │ This Week: 22h │
└──────────────────────────────────────────────────────┘
```

### 4.3 Tab: Today's Tasks

**Content:** Same as Today's Tasks home filtered to this engineer.

**Actions:**
- Submit Report — opens report form for selected WO
- View Details — navigates to Dispatch → WO detail

---

### 4.4 Tab: Active Work Orders

**Content:** All open work orders assigned to this engineer (all statuses except 已完成 / Closed).

```
┌──────────────────────────────────────────────────────┐
│  Active Work Orders                           [2]    │
├──────────────────────────────────────────────────────┤
│  WO-20260630-1234  [HIGH] [進行中]                   │
│  Line A - Conveyor Motor                             │
│  Due: Today                                          │
│  ─────────────────────────────────────────────────   │
│  WO-20260628-0891  [MED] [待料]                     │
│  Pump Station 3 - Hydraulic Pump                     │
│  Parts pending since 2026-06-28                      │
└──────────────────────────────────────────────────────┘
```

**Data source:** `maintenance/workorders` filtered by `assignedTo`

---

### 4.5 Tab: Daily Report

**Content:** Summary of today's work — WOs worked on, hours reported, materials used.

```
┌──────────────────────────────────────────────────────┐
│  Daily Report — 2026-06-30                  [←][→]  │
├──────────────────────────────────────────────────────┤
│  Work Reports Submitted Today:                       │
│                                                      │
│  09:00–11:30 (2.5h) — WO-20260630-1234              │
│    Line A Conveyor — Bearing replacement             │
│    Materials: Bearing #6205 × 2                      │
│                                                      │
│  13:00–15:00 (2.0h) — WO-20260629-0982              │
│    Cooling Tower Fan — Belt inspection               │
│    No materials used                                 │
│  ─────────────────────────────────────────────────   │
│  Total: 4.5h  │  WOs: 2  │  Materials: 2 items      │
└──────────────────────────────────────────────────────┘
```

**Navigation:** Previous day / next day arrows.

**Data source:** `maintenance/reports` filtered by `reporter == name AND createdAt >= today`

---

### 4.6 Tab: Weekly Report

**Content:** 7-day rolling summary with daily breakdown.

```
┌──────────────────────────────────────────────────────┐
│  Weekly Report — 2026-06-23 to 2026-06-29   [←][→]  │
├──────────────────────────────────────────────────────┤
│  Mon 23  ████████  8.0h   4 WOs                      │
│  Tue 24  ████░░░░  4.5h   2 WOs                      │
│  Wed 25  ██████░░  6.0h   3 WOs                      │
│  Thu 26  ████████  8.5h   5 WOs                      │
│  Fri 27  ████░░░░  4.0h   2 WOs                      │
│  ─────────────────────────────────────────────────   │
│  Total:  31.0h   │  16 WOs   │  8 materials used     │
└──────────────────────────────────────────────────────┘
```

**Data source:** `maintenance/reports` filtered by `reporter == name` and date range

---

### 4.7 Tab: Skills

**Content:** Static skill profile for this engineer.

**Data source:** `REPORTERS` config or a new `engineers` Firebase path (implementation decision). For the specification, skills are defined as:
- Equipment categories the engineer is qualified for (Electrical / Mechanical / HVAC / PLC / Civil)
- Certifications (free text)
- Date last updated

**Note:** In current app, engineer skills are not tracked in Firebase. This tab is a specification for future implementation. In v1 of the refactor, this tab may show a placeholder or be omitted.

---

### 4.8 Tab: KPI

**Content:** Replaces `#myperf`. Individual performance indicators for this engineer.

```
┌──────────────────────────────────────────────────────┐
│  KPI — [Name]                [This Month | This Year] │
├──────────────────────────────────────────────────────┤
│  WOs Completed     18      Target: 20 / month        │
│  ─────────────────────────────────────────────────   │
│  Avg Resolution    3.2h    (MTTR per WO)             │
│  ─────────────────────────────────────────────────   │
│  PM Completion     91%     (PM WOs completed on time)│
│  ─────────────────────────────────────────────────   │
│  Report Rate       100%    (reports filed per WO)    │
│  ─────────────────────────────────────────────────   │
│  Hours Logged      152h    (this month)              │
└──────────────────────────────────────────────────────┘
```

**Metrics calculated from:**

| KPI | Calculation | Data Source |
|-----|-------------|------------|
| WOs Completed | Count of `已完成` WOs assigned to engineer | `maintenance/workorders` |
| Avg Resolution (MTTR) | `SUM(hours)` / count of completed WOs | `maintenance/reports` |
| PM Completion Rate | PM WOs completed by due date / all PM WOs assigned | `maintenance/workorders` filtered by `source == 保養` |
| Report Rate | Reports filed / WOs assigned and completed | `maintenance/reports` vs `workorders` |
| Hours Logged | `SUM(hours)` from reports | `maintenance/reports` |

---

### 4.9 Tab: Working Hours

**Content:** Calendar or table view of hours logged per day, week, month.

```
┌──────────────────────────────────────────────────────┐
│  Working Hours — June 2026                  [←][→]  │
├──────────────────────────────────────────────────────┤
│  Week 1 (Jun 2–8)     38.5h   ██████████ ░          │
│  Week 2 (Jun 9–15)    40.0h   ██████████ █          │
│  Week 3 (Jun 16–22)   35.5h   █████████ ░           │
│  Week 4 (Jun 23–29)   31.0h   ████████ ░░           │
│  ─────────────────────────────────────────────────   │
│  Monthly Total:       145h   │  Target: 160h        │
└──────────────────────────────────────────────────────┘
```

**Data source:** `maintenance/reports` — aggregate `hours` field by date range

---

### 4.10 Tab: History

**Content:** Full work history for this engineer — all submitted reports, all WOs, sortable and filterable.

```
┌──────────────────────────────────────────────────────┐
│  History                    [Filter: All ▾] [Search] │
├──────────────────────────────────────────────────────┤
│  2026-06-30  WO-20260630-1234  2.5h                  │
│    Line A Conveyor — Bearing replacement             │
│                                                      │
│  2026-06-29  WO-20260629-0982  2.0h                  │
│    Cooling Tower — Belt inspection                   │
│                                                      │
│  2026-06-28  WO-20260628-0891  1.5h                  │
│    Pump Station 3 — Seal check (waiting parts)       │
│  ─────────────────────────────────────────────────   │
│  Showing 3 of 148 records                            │
└──────────────────────────────────────────────────────┘
```

**Data source:** `maintenance/reports` filtered by `reporter == name`, ordered by `createdAt` descending

---

## 5. Availability Status

### 5.1 Status Logic

| Status | Condition | Display |
|--------|-----------|---------|
| On Site | Has 進行中 WO today | Green dot |
| Available | No active WOs; within work hours | Grey dot |
| Waiting Parts | All open WOs are in 待料 status | Yellow dot |
| Off / Unknown | No reports in last 24h | Dashed circle |

**Note:** Availability is derived from WO status and report timestamps — not from a dedicated presence/attendance system. The current app does not track check-in/check-out; this is best-effort inference.

---

## 6. Access Control

| Feature | All Engineers | Admin / Manager | Notes |
|---------|--------------|----------------|-------|
| Today's Tasks (own) | Read + Write | Read + Write | Engineers only see their own tasks by default |
| Team View | Read | Read + Write | Managers can assign and edit; engineers view only |
| Engineer Profile (own) | Read + Write (reports) | Read + Write | Engineers submit reports on their own WOs |
| Engineer Profile (others) | Read | Read + Write | Engineers can view others' KPIs; managers can edit |
| KPI data | Read | Read | Calculated; no direct edit |
| Skills tab | — (future) | Write | Admin configures skills profiles |
