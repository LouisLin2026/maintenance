# EMS_NAVIGATION.md

## BFY EMS — Navigation Migration Map

**Work Order:** WO-EMS-003
**Version:** 1.0
**Status:** Specification — Awaiting Review
**Date:** 2026-06-30

---

## 1. Purpose

This document maps every current screen, route, and admin tab in the existing BFY 工務報工系統 to its new location in the Workflow-Driven Architecture. It is the authoritative source for implementation: every existing screen must be accounted for before refactor work begins.

---

## 2. Current Navigation Structure

```
app entry
│
├── Home (landing)
│   ├── [Engineer Login Button]
│   └── [Admin Login Button]
│
├── #eng (Engineer Section)
│   ├── Name selection grid
│   ├── #pivot  (my dispatch overview)
│   ├── #eng_pick (equipment selection for manual WO)
│   └── #scan={eqid}  (work report form per equipment)
│       └── submitReport()
│
├── #admin (Admin Section — PIN protected)
│   ├── Tab: dispatch
│   │   ├── WO list (待派工 / 已派工 / 進行中 / 待料 / 已完成)
│   │   ├── New WO form (addWO)
│   │   └── Assign engineer (setWOAssign)
│   ├── Tab: reports
│   │   └── Work report viewer (by WO, by engineer, by equipment)
│   ├── Tab: master
│   │   ├── Equipment master (add/edit/delete equipment)
│   │   └── Materials master (add/edit/delete materials)
│   ├── Tab: cost
│   │   └── Cost summary (labor hours × rate + materials)
│   ├── Tab: perf
│   │   ├── Engineer performance
│   │   └── Equipment performance (fault frequency, downtime)
│   ├── Tab: plan
│   │   ├── PM plan viewer
│   │   └── PM completion tracking
│   ├── Tab: hist
│   │   └── Equipment history (work orders by equipment)
│   ├── Tab: area
│   │   └── Area / zone management
│   ├── Tab: qr
│   │   └── QR code generator per equipment
│   └── Tab: audit
│       └── Audit log (system event log)
│
├── #inventory (Inventory Section)
│   ├── Material stock list
│   ├── Stock level indicators
│   └── Requisition requests
│
└── #report={code} (Fault Report — QR entry)
    └── Fault report form (faultView)
```

---

## 3. Full Screen-by-Screen Migration Table

| # | Current Screen | Current Route / Tab | New Section | New Location | Reason for Move | Migration Risk |
|---|---------------|---------------------|-------------|--------------|----------------|----------------|
| 1 | Home / Landing | `/` (default) | Dashboard | Default entry point | Landing should show operational status, not login buttons | Low — default route still loads; display changes |
| 2 | Engineer Login Button | `/` → `#eng` | Today's Tasks | Bottom nav item 2 | Role-wall removed; engineers land in Tasks not a role portal | Medium — engineers expect `#eng` as entry; Today's Tasks replaces it |
| 3 | Admin Login Button | `/` → `#admin` | Administration | Bottom nav item 8 | PIN protected settings belong in Admin section | Low — admin accessed via nav; PIN flow preserved |
| 4 | Engineer name grid | `#eng` picker | Today's Tasks → name/identity picker | Same content; new context | Name selection feeds task filtering | Low — same logic; new container |
| 5 | My Dispatch Overview | `#pivot` | Today's Tasks (engineer) + Dispatch (all) | Split by scope | `#pivot` was role-specific; replicated in both sections | Medium — two entry points replace one |
| 6 | Equipment Select for WO | `#eng_pick` | Dispatch → New WO → Equipment picker | Inline in WO creation | Equipment selection is part of WO creation flow | Low — same UI component; different parent |
| 7 | Work Report Form | `#scan={eqid}` | Dispatch → WO detail → Submit Report | Report is an action on a WO, not a scan | Decouples report from physical QR scanning; enables reporting on any WO | Medium — current flow is QR-driven; new flow is WO-driven |
| 8 | My Performance | `#myperf` | Engineers → Profile → KPI tab | Performance is an engineer workspace view | KPI belongs alongside history, not as a standalone route | Low — same data; nested within engineer profile |
| 9 | Fault Report Form | `#report={code}` | Equipment → detail → Report Fault | Fault report is about an equipment unit | Context is the machine, not the route code | Low — QR still resolves to equipment; wraps same form |
| 10 | Admin: Dispatch tab | `#admin` → dispatch | Dispatch | Primary top-level section | Core operational workflow; should not require admin login to access | High — removes PIN gate from dispatch view; access control policy decision needed |
| 11 | Admin: Reports tab | `#admin` → reports | Analytics + Dispatch history | Split: historical data → Analytics; active WO reports → Dispatch | Reports have two purposes: operational review (Dispatch) and trend analysis (Analytics) | Medium — current tab does both; needs split |
| 12 | Admin: Master tab (Equipment) | `#admin` → master → equipment | Administration → Equipment Master | Equipment configuration is admin-only | No change in access level; new location in Admin section | Low — same tab; relocated under Admin |
| 13 | Admin: Master tab (Materials) | `#admin` → master → materials | Administration → Materials Master | Materials config is admin-only | No change in access level | Low — same tab; relocated |
| 14 | Admin: Cost tab | `#admin` → cost | Analytics → Cost Analysis | Cost is an analytical view, not a live operation screen | Cost belongs with performance data in Analytics | Low — data unchanged; section changes |
| 15 | Admin: Perf tab (Engineers) | `#admin` → perf → engineers | Analytics → Engineer Performance | Performance analytics belong in Analytics section | Unified analytics view; no access change | Low |
| 16 | Admin: Perf tab (Equipment) | `#admin` → perf → equipment | Analytics → Equipment Performance | Equipment analytics belong alongside engineer analytics | Unified analytics view | Low |
| 17 | Admin: Plan tab | `#admin` → plan | Equipment → Equipment detail → PM Schedule | PM is about a specific machine | PM context belongs on the equipment it applies to | Medium — current plan tab shows all equipment; new view is per-equipment (needs list view too) |
| 18 | Admin: Hist tab | `#admin` → hist | Equipment → Equipment detail → History | History is per-equipment | Same reasoning as PM plan | Medium — current hist tab shows cross-equipment; needs Equipment History list view |
| 19 | Admin: Area tab | `#admin` → area | Administration → Area / QR | Admin configuration | No change in function; relocated to Admin section | Low |
| 20 | Admin: QR tab | `#admin` → qr | Administration → Area / QR | QR management is admin config | Merged with Area since they are related (QR codes are per area/equipment) | Low |
| 21 | Admin: Audit tab | `#admin` → audit | Administration → Audit Log | System log is admin-only | No change; relocated to Admin | Low |
| 22 | Inventory section | `#inventory` | Administration → Inventory Management | Inventory config and control is admin-managed | Access pattern unchanged; relocated under Admin | Low |

---

## 4. Risk Summary

### 4.1 High Risk Items

| Item | Risk | Decision Required |
|------|------|-------------------|
| Admin dispatch tab (row 10) | Removing PIN gate from Dispatch section changes who can see all work orders | Product owner must decide: open Dispatch to all users, or maintain role-based access in new UI |

### 4.2 Medium Risk Items

| Item | Risk | Mitigation |
|------|------|------------|
| `#eng` / `#pivot` split (rows 2, 5) | Engineers used to entering at `#eng` and navigating to `#pivot` as their workflow entry | Today's Tasks section should open to My Work Orders by default; prominent "My WOs" tab on first load |
| Work report flow change (row 7) | Current flow: scan QR → see equipment → fill report. New flow: find WO → tap report. QR scanning becomes a navigation shortcut, not the only path | QR links still work — they resolve to Equipment detail which surfaces active WOs |
| PM plan view (row 17) | Current plan tab shows PM schedule for all equipment in one table. New view is per-equipment | Need an Equipment → PM List screen (cross-equipment PM calendar view) in addition to per-equipment PM tab |
| History view (row 18) | Current hist tab allows cross-equipment history filter. New model is per-equipment | Need Equipment → History list screen (search/filter across equipment) in addition to per-equipment History tab |

### 4.3 Low Risk Items

All other migrations (rows 1, 4, 6, 8, 9, 12–16, 19–22) involve relocating existing screens to new sections with no functional changes.

---

## 5. New Navigation Items (No Current Equivalent)

| New Item | Section | Notes |
|---------|---------|-------|
| Dashboard — summary widgets | Dashboard | Aggregates existing data; no new Firebase paths needed |
| Today's Tasks home | Today's Tasks | Filters existing WO data by engineer identity |
| Equipment Center — unified tabbed view | Equipment | New container for existing equipment tabs |
| Engineers team roster | Engineers | New container using existing REPORTERS config + report data |
| Engineer Profile workspace | Engineers | New container; history data from reports Firebase path |
| Projects section | Projects | Specification only; no implementation in this sprint |
| Analytics section home | Analytics | New container aggregating existing cost/perf data |
| Administration home | Administration | New container for existing admin tabs |

---

## 6. Routes That Must Be Preserved

The following routes are referenced by printed QR codes or external links and must continue to function after the refactor. They may redirect to new screens but must not return 404 or blank pages.

| Preserved Route | Resolves To (New) |
|----------------|------------------|
| `#report={code}` | Equipment detail → Report Fault (QR printed on equipment) |
| `#scan={eqid}` | Equipment detail → Work Report form (direct QR links) |
| `#admin` | Administration section (PIN flow preserved) |
| `#inventory` | Administration → Inventory Management |
