# EMS Dashboard Refactor — Sprint WO-EMS-003

**Sprint:** WO-EMS-003
**Status:** Specification Complete — Awaiting Review
**Branch:** `feature/dashboard-refactor-v1`
**Date:** 2026-06-30

---

## Sprint Goal

Refactor the BFY 工務報工系統 navigation from a role-based structure (Engineer / Admin portals) to a Workflow-Driven Architecture (8 workflow-oriented sections). This sprint produces the **product specification only** — no implementation.

---

## Deliverables

| File | Description | Status |
|------|-------------|--------|
| [EMS_PRODUCT_ARCHITECTURE.md](EMS_PRODUCT_ARCHITECTURE.md) | Product vision, navigation tree, screen hierarchy, user journeys, workflow mapping | Complete |
| [EMS_NAVIGATION.md](EMS_NAVIGATION.md) | Screen-by-screen migration table: current → new, with risk ratings | Complete |
| [EMS_ENGINEER_WORKSPACE.md](EMS_ENGINEER_WORKSPACE.md) | Engineer workspace: Today's Tasks, team roster, individual profile tabs | Complete |
| [EMS_EQUIPMENT_WORKSPACE.md](EMS_EQUIPMENT_WORKSPACE.md) | Equipment Center: unified tabbed workspace per equipment unit | Complete |
| [EMS_DISPATCH_WORKFLOW.md](EMS_DISPATCH_WORKFLOW.md) | Full dispatch workflow spec: 10-stage lifecycle with screen mapping | Complete |
| [EMS_CRITICAL_EQUIPMENT.md](EMS_CRITICAL_EQUIPMENT.md) | Critical equipment classification, MTTR/MTBF, weak points, recovery SOP | Complete |
| [README.md](README.md) | This file — sprint overview | Complete |
| [CHANGELOG.md](CHANGELOG.md) | WO-EMS-003 change log entry | Complete |

---

## New Navigation Structure

```
EMS Application
├── Dashboard           ← New default landing
├── Today's Tasks       ← Replaces #eng + #pivot
├── Dispatch            ← Replaces Admin dispatch tab
├── Equipment           ← Equipment Center (unified)
├── Engineers           ← Team roster + profiles
├── Projects            ← Future module (spec only)
├── Analytics           ← Replaces Admin cost/perf tabs
└── Administration      ← Replaces Admin settings tabs
```

Full navigation tree: [EMS_PRODUCT_ARCHITECTURE.md § 2](EMS_PRODUCT_ARCHITECTURE.md)

---

## What This Sprint Does NOT Change

- Firebase data paths or data model (except additions noted in EMS_CRITICAL_EQUIPMENT.md § 6.1)
- Business logic in `index.html`
- Work order status values (待派工 / 已派工 / 進行中 / 待料 / 已完成)
- `service-worker.js` — unchanged per WO constraint
- Existing QR code links (`#report={code}`, `#scan={id}`) — must still resolve after refactor

---

## Key Decisions Requiring Product Owner Input

Before implementation begins, the following decisions must be confirmed:

| # | Decision | Options | Spec Default |
|---|---------|---------|-------------|
| 1 | Dispatch access control | Open to all users vs. maintain Admin PIN gate | Open (removes PIN from Dispatch) |
| 2 | Testing stage (Stage 7) | Include as a UI state vs. skip | Include (UI-only; no data change) |
| 3 | Verified stage (Stage 9) | Require admin verification before Close vs. auto-close on engineer completion | Include (admin must verify) |
| 4 | Stock quantity tracking | Add stock field to Firebase material records vs. infer from requisitions | Implementation decision |
| 5 | Engineer skills tab | Implement in v1 vs. placeholder | Placeholder in v1 |

---

## Repository

GitHub: `LouisLin2026/maintenance`
Branch: `feature/dashboard-refactor-v1`
App: BFY 工務報工系統 (PWA, Firebase Realtime Database)
