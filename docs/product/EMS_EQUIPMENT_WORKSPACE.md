# EMS_EQUIPMENT_WORKSPACE.md

## BFY EMS — Equipment Center Specification

**Work Order:** WO-EMS-003
**Version:** 1.0
**Status:** Specification — Awaiting Review
**Date:** 2026-06-30

---

## 1. Overview

The Equipment Center is a new unified view that consolidates every equipment-related screen from the current application. It replaces and reorganises:

| Current Screen / Tab | Replaced By |
|---------------------|-------------|
| `#scan={id}` equipment context | Equipment detail — work report entry point |
| `#report={code}` fault entry | Equipment detail → Report Fault |
| Admin `master` tab (equipment) | Administration → Equipment Master (edit) + Equipment detail (view) |
| Admin `plan` tab | Equipment detail → PM Schedule tab |
| Admin `hist` tab | Equipment detail → History tab |
| Admin `qr` tab | Administration → Area / QR (generation) + Equipment detail → QR tab (view) |

The Equipment Center has two levels:
1. **Equipment Overview** — list of all equipment with status summary
2. **Equipment Detail** — tabbed workspace for a single piece of equipment

---

## 2. Equipment Section Home

### 2.1 Equipment Overview Screen

```
┌──────────────────────────────────────────────────────┐
│  Equipment                         [List | Grid] [⌕] │
├──────────────────────────────────────────────────────┤
│  Filter: [All ▾]  Line: [All ▾]  Status: [All ▾]    │
├──────────────────────────────────────────────────────┤
│                                                      │
│  ● EQ-001  Line A - Conveyor Motor                   │
│    Status: Active │ Open WOs: 1 (HIGH) │ PM: OK      │
│                                                      │
│  ⚠ EQ-002  Line B - Hydraulic Pump                  │
│    Status: Under Maintenance │ Open WOs: 2 │ PM: Due │
│                                                      │
│  ● EQ-003  Cooling Tower Fan Unit 1                  │
│    Status: Active │ Open WOs: 0 │ PM: 15 days        │
│                                                      │
│  ○ EQ-004  Air Compressor Unit 2                     │
│    Status: Standby │ Open WOs: 0 │ PM: OK            │
│                                                      │
└──────────────────────────────────────────────────────┘
```

**Status indicators:**
- `●` Green — Active, no open faults
- `⚠` Yellow — Under Maintenance or open HIGH priority WO
- `○` Grey — Standby / inactive
- `✕` Red — Critical fault / Decommissioned

### 2.2 Data Sources

| Display Element | Firebase Path | Calculation |
|----------------|--------------|-------------|
| Equipment list | `maintenance/equipment` | All records |
| Equipment name / line | `equipment/{id}/name`, `equipment/{id}/line` | — |
| Equipment status | `equipment/{id}/status` | Active values: normal (active) / inactive |
| Open WOs | `maintenance/workorders` | Count where `equip == id AND status != 已完成` |
| Highest priority WO | `workorders` filtered by equip | Max priority of open WOs |
| PM status | `maintenance/plans/{planId}/nextDue` | Days until next PM due; "Due" if today or overdue |

---

## 3. Equipment Detail Screen

### 3.1 Header

```
┌──────────────────────────────────────────────────────┐
│  ← Equipment    EQ-002 — Line B Hydraulic Pump       │
│  Status: ⚠ Under Maintenance │ Line: B               │
├──────────────────────────────────────────────────────┤
│  [Overview][PM][History][QR][Docs][Critical][Parts][PLC] │
└──────────────────────────────────────────────────────┘
```

**Tab visibility:**
- Overview, PM, History, QR, Spare Parts — available in this sprint
- Documents, Critical Points, PLC Backup — specification only; visible but show "Coming Soon" placeholder in v1

---

### 3.2 Tab: Overview

**Content:** Equipment identity, current status, open work orders, and quick actions.

```
┌──────────────────────────────────────────────────────┐
│  Overview                                            │
├──────────────────────────────────────────────────────┤
│  Equipment ID:   EQ-002                              │
│  Name:           Line B Hydraulic Pump               │
│  Line / Area:    Line B                              │
│  Status:         Under Maintenance                   │
│  ─────────────────────────────────────────────────   │
│  Open Work Orders:   2                               │
│  ┌─────────────────────────────────────────────┐    │
│  │ WO-20260628-0891  [HIGH] [進行中]            │    │
│  │ Seal replacement                             │    │
│  └─────────────────────────────────────────────┘    │
│  ┌─────────────────────────────────────────────┐    │
│  │ WO-20260627-0834  [MED] [待料]              │    │
│  │ Filter element replacement                   │    │
│  └─────────────────────────────────────────────┘    │
│  ─────────────────────────────────────────────────   │
│  PM Due:         2026-07-15 (15 days)                │
│  Last PM:        2026-06-15                          │
│  ─────────────────────────────────────────────────   │
│  [Report Fault]       [Request Maintenance]          │
└──────────────────────────────────────────────────────┘
```

**Quick actions:**
- **Report Fault** — opens fault report form; same as `#report={code}` flow. Creates FR- work order.
- **Request Maintenance** — opens new WO form pre-filled with this equipment.

**Data sources:**
- Equipment record: `maintenance/equipment/{id}`
- Open WOs: `maintenance/workorders` filtered by `equip == id AND status != 已完成`
- PM data: `maintenance/plans` filtered by `planId` linked to equipment

---

### 3.3 Tab: PM Schedule

**Content:** Replaces Admin → `plan` tab for this equipment. Shows PM history, upcoming schedule, and PM work order records.

```
┌──────────────────────────────────────────────────────┐
│  PM Schedule                                         │
├──────────────────────────────────────────────────────┤
│  Plan ID:     PM-EQ002-001                           │
│  Interval:    Every 3 months                         │
│  Next Due:    2026-07-15                             │
│  Last Done:   2026-06-15                             │
│  ─────────────────────────────────────────────────   │
│  PM History:                                         │
│  ✓ 2026-06-15  WO-20260615-0445  Completed 3.0h     │
│  ✓ 2026-03-14  WO-20260314-0211  Completed 2.5h     │
│  ✓ 2025-12-10  WO-20251210-0098  Completed 3.5h     │
│  ─────────────────────────────────────────────────   │
│  [Create PM Work Order]  [Edit Schedule]             │
└──────────────────────────────────────────────────────┘
```

**Data sources:**
- Plan record: `maintenance/plans/{planId}` — fields: `months[]`, `lastDone`, `nextDue`, `planId`
- PM work orders: `maintenance/workorders` filtered by `equip == id AND source == 保養`

**Actions (Admin only):**
- Create PM Work Order — generates WO with `source = 保養`, links `planId`
- Edit Schedule — modifies `plans/{planId}/months`

---

### 3.4 Tab: History

**Content:** Replaces Admin → `hist` tab for this equipment. Complete repair and maintenance history.

```
┌──────────────────────────────────────────────────────┐
│  History                                [Filter] [⌕] │
├──────────────────────────────────────────────────────┤
│  2026-06-28  WO-20260628-0891                        │
│    Seal replacement — 進行中                         │
│    Engineer: [Name]  |  Hours: 1.5h (partial)        │
│                                                      │
│  2026-06-15  WO-20260615-0445  (PM)                  │
│    Quarterly preventive maintenance                  │
│    Engineer: [Name]  |  Hours: 3.0h                  │
│    Note: Replaced filter, checked seals, lubricated  │
│                                                      │
│  2026-05-03  WO-20260503-0312                        │
│    Pressure loss investigation                       │
│    Engineer: [Name]  |  Hours: 4.5h                  │
│    Root cause: Worn seal on inlet valve              │
└──────────────────────────────────────────────────────┘
```

**Data sources:**
- `maintenance/workorders` filtered by `equip == id`, ordered by `createdAt` descending
- `maintenance/reports` joined by `woKey`

---

### 3.5 Tab: QR Code

**Content:** QR code for this equipment. Replaces Admin → `qr` tab (per-equipment view).

```
┌──────────────────────────────────────────────────────┐
│  QR Code                                             │
├──────────────────────────────────────────────────────┤
│                                                      │
│       ┌─────────────────────────┐                   │
│       │  [QR CODE IMAGE]        │                   │
│       │                         │                   │
│       │  EQ-002                 │                   │
│       │  Line B Hydraulic Pump  │                   │
│       └─────────────────────────┘                   │
│                                                      │
│  Scan target:  #report=EQ-002                        │
│                                                      │
│  [Print]   [Download PNG]                           │
└──────────────────────────────────────────────────────┘
```

**Data source:**
- Equipment ID and name: `maintenance/equipment/{id}`
- QR encodes: `{appUrl}#report={equipId}`

---

### 3.6 Tab: Documents

**Specification — v1 shows placeholder. Not implemented in this sprint.**

Intended content:
- Equipment manuals (PDF links or uploaded files)
- SOPs (Standard Operating Procedures)
- Wiring diagrams
- Inspection checklists

**Placeholder v1:**
```
┌──────────────────────────────────────────────────────┐
│  Documents                                           │
├──────────────────────────────────────────────────────┤
│                                                      │
│  Document management is not yet available.           │
│  This feature is planned for a future release.       │
│                                                      │
└──────────────────────────────────────────────────────┘
```

---

### 3.7 Tab: Critical Points

**Specification — v1 shows placeholder. See [EMS_CRITICAL_EQUIPMENT.md](EMS_CRITICAL_EQUIPMENT.md) for full spec.**

Intended content:
- Critical Level rating
- MTTR / MTBF indicators
- Known weak points
- Recovery SOP link

---

### 3.8 Tab: Spare Parts

**Content:** Materials and spare parts associated with this equipment. Sourced from `maintenance/material` and linked by equipment category or explicit association.

```
┌──────────────────────────────────────────────────────┐
│  Spare Parts                                         │
├──────────────────────────────────────────────────────┤
│  Associated Materials:                               │
│                                                      │
│  MAT-0023  Bearing #6205                             │
│    Brand: NSK  │  Unit: pcs  │  In Stock: 4         │
│                                                      │
│  MAT-0047  Hydraulic Seal Kit (80mm)                 │
│    Brand: Parker  │  Unit: set  │  In Stock: 1       │
│    ⚠ Low stock — reorder point: 2                   │
│                                                      │
│  MAT-0089  Filter Element HY-150                     │
│    Brand: Donaldson  │  Unit: pcs  │  In Stock: 6    │
│  ─────────────────────────────────────────────────   │
│  [Request Parts]                                     │
└──────────────────────────────────────────────────────┘
```

**Data sources:**
- Materials: `maintenance/material` (all materials; association is by category or explicit link — implementation decision)
- Stock levels: `maintenance/material/{id}` — requires stock quantity field (not currently in Firebase schema; implementation must add or approximate from requisitions)

**Note on stock levels:** The current `maintenance/material` schema has `id, name, cat, brand, model, unit, price` but no `stock` field. Implementing stock levels requires either a new `stock` field per material or inference from requisition history. This is flagged as an implementation decision, not a specification change.

---

### 3.9 Tab: PLC Backup

**Specification — v1 shows placeholder. Not implemented in this sprint.**

Intended content:
- PLC program version
- Last backup date and responsible engineer
- Download link for backup file
- Program notes / change history

---

## 4. Equipment Master Edit (Admin Only)

Accessible from:
- Equipment detail → "Edit" button (Admin role only)
- Administration → Equipment Master

```
┌──────────────────────────────────────────────────────┐
│  Edit Equipment — EQ-002                             │
├──────────────────────────────────────────────────────┤
│  ID:       EQ-002              (read-only)           │
│  Name:     [Line B Hydraulic Pump        ]           │
│  Line:     [Line B                       ]           │
│  Status:   [Active ▾]                                │
│  ─────────────────────────────────────────────────   │
│  [Cancel]                              [Save]        │
└──────────────────────────────────────────────────────┘
```

**Data written:** `maintenance/equipment/{id}` — `name`, `line`, `status`

---

## 5. Cross-Equipment Views

### 5.1 PM Calendar (Equipment List View)

The current Admin → `plan` tab shows PM schedules for all equipment in one table. The new Equipment detail → PM tab shows only one equipment's PM. A cross-equipment PM view is needed.

**Location:** Equipment section home → toggle "PM View"

```
┌──────────────────────────────────────────────────────┐
│  Equipment                              [PM View ▼]  │
├──────────────────────────────────────────────────────┤
│  Due This Week:                                      │
│  ⚠ EQ-002  Line B Hydraulic Pump   Due: 2026-07-01  │
│  ⚠ EQ-007  Cooling Tower Unit 2    Due: 2026-07-02  │
│                                                      │
│  Due Next 30 Days:                                   │
│  ● EQ-001  Line A Conveyor          Due: 2026-07-15  │
│  ● EQ-004  Air Compressor Unit 2    Due: 2026-07-20  │
└──────────────────────────────────────────────────────┘
```

### 5.2 Equipment History List (Cross-Equipment Search)

**Location:** Equipment section home → search / filter → "History" toggle

Allows filtering work history across all equipment by date range, work type, or engineer. This preserves the cross-equipment query capability of the current Admin → `hist` tab.

---

## 6. Access Control

| Feature | Engineer | Admin / Manager |
|---------|----------|----------------|
| View equipment list | Read | Read |
| View equipment detail | Read | Read |
| Report Fault / Request Maintenance | Write | Write |
| PM tab — view | Read | Read |
| PM tab — create WO, edit schedule | — | Write |
| History tab | Read | Read |
| QR tab — view | Read | Read |
| QR tab — print/download | Read | Read |
| Spare Parts — view | Read | Read |
| Spare Parts — request | Write | Write |
| Equipment Master edit | — | Write |
| Documents, Critical, PLC tabs | — (future) | Write (future) |
