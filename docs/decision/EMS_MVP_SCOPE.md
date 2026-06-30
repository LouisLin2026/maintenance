# EMS_MVP_SCOPE.md

## BFY EMS — MVP Feature Freeze

**Work Order:** WO-EMS-004A
**Status:** FROZEN
**Date:** 2026-06-30
**Resolves:** WO-EMS-004 — MVP Scope Definition

---

## 1. MVP Definition

The MVP (Minimum Viable Product) for the EMS Dashboard Refactor is the **smallest releasable version that delivers the Workflow-Driven Architecture** to all users. It must:

1. Replace the current role-based landing with the 8-section navigation.
2. Preserve all existing functionality — no feature regression.
3. Resolve all P0 and P1 items from WO-EMS-004.
4. Not require a data migration or Firebase downtime.

Everything else is deferred.

---

## 2. Included in MVP

### 2.1 Navigation Shell

| Feature | Description |
|---------|-------------|
| Bottom navigation bar | 7 items: Dashboard, Today's Tasks, Dispatch, Equipment, Engineers, Analytics, Administration. Projects excluded (see § 4). |
| Sticky header | App title + current identity indicator |
| Identity selection flow | Name picker from REPORTERS + roster role lookup; persists in memory |
| QR route preservation | `#report={code}` and `#scan={eqid}` continue to resolve |
| `#admin` route preservation | Navigates to Administration with PIN prompt |
| `#inventory` route preservation | Navigates to Administration → Inventory Management |

---

### 2.2 Dashboard

| Widget | Source | Included |
|--------|--------|----------|
| Today's Dispatch | Count of all open WOs assigned today | ✓ |
| Waiting Assignment | Count of WOs with status 待派工 | ✓ |
| Overdue | Count of WOs with `due < today AND status != 已完成` | ✓ |
| Today's PM | Count of `plans` with `nextDue == today` | ✓ |
| Waiting Parts | Count of unconfirmed `requisitions` | ✓ |
| Weekly KPI | WOs completed this week, total hours, avg MTTR | ✓ |
| Recent Activity | Last 10 audit log entries | ✓ |
| **Emergency Repair** | Active FR- WOs with urgency == 緊急 | ✓ (P1-03) |
| Critical Equipment | Equipment with criticality == C and open fault WO | Deferred — requires `criticality` field population |

**Dashboard widget tapping destinations (P1-04):**

| Widget | Tap Destination |
|--------|----------------|
| Today's Dispatch | Dispatch → In Progress tab |
| Waiting Assignment | Dispatch → Pending tab |
| Overdue | Dispatch → All tab, filter: `due < today` |
| Today's PM | Equipment → PM Calendar view |
| Waiting Parts | Administration → Requisitions |
| Weekly KPI | Analytics → KPI Summary |
| Recent Activity | Administration → Audit Log |
| Emergency Repair | Dispatch → In Progress tab, filter: urgency == 緊急 |

---

### 2.3 Today's Tasks

| Feature | Included |
|---------|----------|
| My Work Orders list | ✓ |
| Team toggle (all engineers' WOs) | ✓ (MGR tier only) |
| WO status badges | ✓ |
| Submit Report button on card | ✓ |
| View Details button on card | ✓ |
| Completed today section | ✓ (uses new `finishAt` field) |
| New Fault Report FAB | ✓ |
| Identity selection / switch | ✓ |

---

### 2.4 Dispatch

| Feature | Included |
|---------|----------|
| WO list with status filter tabs | ✓ (All, Pending, In Progress, Waiting Parts, Completed) |
| Overdue filter (pre-applied from Dashboard tap) | ✓ |
| New Work Order form | ✓ |
| WO detail screen | ✓ |
| Assign engineer (MGR/ADM) | ✓ |
| Accept WO (ENG) | ✓ |
| Request Parts | ✓ |
| Mark as Testing | ✓ |
| Submit Report | ✓ |
| Verify (MGR/ADM) | ✓ |
| Close (MGR/ADM) | ✓ |
| Delete (ADM only) | ✓ |
| WO history timeline on detail | ✓ |
| Fault Report sub-view (FR- prefix filter) | ✓ |
| Closed WO historical search | Deferred |
| Bulk verify | Deferred |
| Emergency Repair fast-path highlight | ✓ (urgency == 緊急 badge on card) |

---

### 2.5 Equipment

| Feature | Included |
|---------|----------|
| Equipment list with status indicators | ✓ |
| Filter by line / status | ✓ |
| PM Calendar view (cross-equipment) | ✓ |
| Equipment detail — Overview tab | ✓ |
| Equipment detail — PM Schedule tab | ✓ |
| Equipment detail — History tab | ✓ |
| Equipment detail — QR Code tab | ✓ |
| Equipment detail — Spare Parts tab | ✓ (shows associated parts from `equipmentParts`; placeholder if none associated) |
| Equipment detail — Documents tab | Placeholder ("Coming Soon") |
| Equipment detail — Critical Points tab | MVP-partial: criticality badge + MTTR/MTBF display only |
| Equipment detail — PLC Backup tab | Placeholder ("Coming Soon") |
| Equipment Master edit (ADM/MGR) | ✓ |
| Report Fault button on Equipment detail | ✓ |
| Request Maintenance button on Equipment detail | ✓ |

---

### 2.6 Engineers

| Feature | Included |
|---------|----------|
| Team roster with availability indicators | ✓ |
| Engineer profile — Today's Tasks tab | ✓ |
| Engineer profile — Active Work Orders tab | ✓ |
| Engineer profile — KPI tab | ✓ (WOs completed, MTTR, PM completion rate, report rate, hours) |
| Engineer profile — Working Hours tab | ✓ |
| Engineer profile — History tab | ✓ |
| Engineer profile — Daily Report tab | ✓ |
| Engineer profile — Weekly Report tab | ✓ |
| Engineer profile — Skills tab | Placeholder ("Not yet configured") |
| Role badge display | ✓ |

---

### 2.7 Analytics

| Feature | Included |
|---------|----------|
| KPI Summary (WOs, MTTR, PM compliance) | ✓ |
| Engineer Performance table | ✓ |
| Equipment Performance table | ✓ |
| Cost Analysis | ✓ (existing logic from admCost tab) |
| PM Compliance chart | ✓ |
| Date range filter | ✓ |

---

### 2.8 Administration

| Feature | Included |
|---------|----------|
| Equipment Master (view + edit) | ✓ |
| Materials Master (view + edit) | ✓ |
| Inventory Management (stock view + adjust) | ✓ (includes new `stock` field) |
| Engineers & Roles (roster management) | ✓ |
| Equipment-Parts Association editor | ✓ (manages `equipmentParts` records) |
| Area / QR Management | ✓ |
| Requisitions (confirm/view) | ✓ |
| System Config (laborRate, PIN) | ✓ (ADM only) |
| Audit Log viewer | ✓ |

---

### 2.9 Role Model (MVP Tier)

| Feature | Included |
|---------|----------|
| 3-tier MVP model (ADM / MGR / ENG-default) | ✓ |
| Roster Firebase path (`maintenance/roster`) | ✓ |
| Role resolution on identity selection | ✓ |
| ADM tier via PIN (existing behavior) | ✓ |
| Full 8-role RBAC enforcement | Deferred to Sprint 2 |

---

## 3. Deferred (Sprint 2)

These are specified but not built in MVP. They require either new Firebase data (that won't exist at launch) or additional design work.

| Feature | Reason for Deferral | Sprint Target |
|---------|--------------------|--------------------|
| Critical Equipment widget on Dashboard | Requires `criticality` field to be populated in all equipment records | Sprint 2 |
| Critical Points tab — full implementation (Weak Points, Recovery SOP, PLC, Visual Reference) | New Firebase sub-paths; requires content creation by team | Sprint 2 |
| Full 8-role RBAC | Requires roster records to be created for all team members | Sprint 2 |
| Push notifications | Requires PWA notification permission flow and backend trigger logic | Sprint 2 |
| Closed WO historical search | Low usage; not needed for daily operations | Sprint 2 |
| Bulk WO verify | Admin convenience; not blocking | Sprint 2 |
| Production Leader (PLR) dashboard scoping | Requires production line association data | Sprint 2 |

---

## 4. Future (Post-Sprint 2)

These require new modules, new Firebase paths with significant data, or separate design work.

| Feature | Description |
|---------|-------------|
| Projects section | Requires full CAPEX/improvement project data model |
| Equipment → Documents tab | Requires file storage (Firebase Storage or external) |
| Equipment → PLC Backup tab | Requires file storage + version control |
| Vendor portal (VND role, scoped view) | Requires external-facing auth |
| MTBF trending alert | Requires time-series analysis across months of data |
| SLA escalation automation | Requires trigger functions (Firebase Functions) |
| QA module | New Business module |
| HR / People module | New Business module |
| Production module | New Business module |
| Mobile native app | Separate project from PWA |

---

## 5. Out of Scope — This Refactor

These will NOT be built in the current refactor under any circumstances.

| Item | Why Out of Scope |
|------|----------------|
| Changes to existing Firebase data paths | No migration; backward compatibility required |
| Modifications to `index.html` business logic | Refactor is navigation/UI only |
| Modifications to `service-worker.js` | Explicitly excluded by WO-EMS-003 |
| New Firebase Cloud Functions | No server-side code in this refactor |
| Changes to `maintenance/config/pin` mechanism | Security — existing PIN preserved |
| External API integrations | No ERP, no email service, no third-party APIs |
| Authentication system (login/logout) | Current identity selection model preserved |
| Multi-language support | App is zh-TW; no i18n in scope |
| Reporting export (PDF/Excel) | Export feature is future |

---

## 6. Feature Flag Conventions

Features deferred to Sprint 2 should be built with a feature flag so that the MVP UI can show placeholders without dead code. Suggested flag prefix: `EMS_FEATURE_` (e.g., `EMS_FEATURE_CRITICAL_WIDGET`, `EMS_FEATURE_FULL_RBAC`).

This is a recommendation for the implementation team, not a specification requirement.
