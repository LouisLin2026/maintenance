# EMS_IMPLEMENTATION_GATE.md

## BFY EMS — Implementation Gate

**Work Order:** WO-EMS-004A
**Date:** 2026-06-30
**Gate For:** WO-EMS-003 Specification + WO-EMS-004A Decisions

---

## ⚠ NOTICE

**Development may begin ONLY after Product Owner approval of this gate document.**

This document summarises the current state of all gate conditions. The verdict is issued here. Development must not start on any item marked NOT READY.

---

## 1. Gate Checklist

### 1.1 P0 Items (All must be RESOLVED)

| # | Item | Resolving Document | Status |
|---|------|--------------------|--------|
| P0-01 | Dispatch access control defined | EMS_ACCESS_CONTROL.md § 5 (DEC-001) | ✓ RESOLVED |
| P0-02 | Role model defined and stored | EMS_ROLE_MODEL.md (DEC-002) | ✓ RESOLVED |
| P0-03 | Material-equipment association defined | EMS_DATA_MODEL.md § 5.2, § 6 (DEC-003) | ✓ RESOLVED |

**P0 Gate: PASSED ✓**

---

### 1.2 P1 Items (All must be RESOLVED)

| # | Item | Resolving Document | Status |
|---|------|--------------------|--------|
| P1-01 | Verified stage writes audit record + WO fields | EMS_AUDIT_MODEL.md, EMS_DATA_MODEL.md § 4.1 (DEC-005) | ✓ RESOLVED |
| P1-02 | `finishAt` field formally defined in schema | EMS_DATA_MODEL.md § 4.1 (DEC-004) | ✓ RESOLVED |
| P1-03 | Emergency Repair widget specified | EMS_MVP_SCOPE.md § 2.2 (DEC-006) | ✓ RESOLVED |
| P1-04 | Overdue widget navigation destination defined | EMS_MVP_SCOPE.md § 2.2 (DEC-007) | ✓ RESOLVED |
| P1-05 | Notification mechanism specified | EMS_DECISION_LOG.md DEC-008 | ✓ RESOLVED |

**P1 Gate: PASSED ✓**

---

### 1.3 Architecture Freeze

| Condition | Document | Status |
|-----------|---------|--------|
| Navigation tree finalized (7 sections for MVP) | EMS_MVP_SCOPE.md § 2.1 | ✓ FROZEN |
| Dispatch 10-stage lifecycle confirmed | EMS_DISPATCH_WORKFLOW.md | ✓ FROZEN |
| Projects section excluded from MVP nav | EMS_DECISION_LOG.md DEC-010 | ✓ FROZEN |
| QR route backward compatibility required | EMS_NAVIGATION.md § 6 | ✓ FROZEN |
| No changes to `index.html`, `app.js`, `service-worker.js` during this review phase | WO-EMS-003, WO-EMS-004, WO-EMS-004A | ✓ CONFIRMED |

**Architecture Gate: FROZEN ✓**

---

### 1.4 Business Rules Freeze

| Rule | Document | Status |
|------|---------|--------|
| 10-stage WO lifecycle | EMS_DISPATCH_WORKFLOW.md | ✓ FROZEN |
| WO status values unchanged (待派工 / 已派工 / 進行中 / 待料 / 已完成) | EMS_DATA_MODEL.md § 3.1 | ✓ FROZEN |
| PM `lastDone` / `nextDue` updated at Stage 8 | EMS_DISPATCH_WORKFLOW.md Stage 8 | ✓ FROZEN |
| SLA targets (High: 2h dispatch; Medium: 1 day; Low: 3 days) | EMS_DISPATCH_WORKFLOW.md § 5 | ✓ FROZEN |
| WO re-open: ADM only | EMS_DECISION_LOG.md DEC-012 | ✓ FROZEN |
| WO delete: ADM only + audit record | EMS_DECISION_LOG.md DEC-013 | ✓ FROZEN |
| Fault reports (FR- prefix) created via QR or Dispatch | EMS_DISPATCH_WORKFLOW.md Stage 1 | ✓ FROZEN |
| Emergency = urgency == 緊急 AND FR- WO source | EMS_DECISION_LOG.md DEC-006 | ✓ FROZEN |
| PM overdue = informational; no auto-WO in MVP | EMS_DECISION_LOG.md DEC-016 | ✓ FROZEN |
| Parts timeout = informational; no auto-escalation in MVP | EMS_DECISION_LOG.md DEC-017 | ✓ FROZEN |

**Business Rules Gate: FROZEN ✓**

---

### 1.5 Data Model Freeze

| Condition | Document | Status |
|-----------|---------|--------|
| Current Firebase schema documented | EMS_DATA_MODEL.md § 3 | ✓ FROZEN |
| New WO fields: `finishAt`, `testingAt`, `verifiedAt`, `verifiedBy`, `closedAt`, `closedBy` | EMS_DATA_MODEL.md § 4.1 | ✓ FROZEN |
| New equipment fields: `criticality`, `criticalityReviewDate`, `criticalityReviewedBy`, `criticalityNote` | EMS_DATA_MODEL.md § 4.2 | ✓ FROZEN |
| New material fields: `stock`, `stockUpdatedAt`, `stockUpdatedBy` | EMS_DATA_MODEL.md § 7 | ✓ FROZEN |
| New path: `maintenance/roster` | EMS_DATA_MODEL.md § 5.1 | ✓ FROZEN |
| New path: `maintenance/equipmentParts` | EMS_DATA_MODEL.md § 5.2 | ✓ FROZEN |
| New path: `maintenance/auditLog` (structured) | EMS_AUDIT_MODEL.md | ✓ FROZEN |
| No existing fields renamed or deleted | EMS_DATA_MODEL.md § 3 | ✓ CONFIRMED |
| Backward compatibility preserved for all existing Firebase paths | All decision documents | ✓ CONFIRMED |

**Data Model Gate: FROZEN ✓**

---

### 1.6 Role Model Freeze

| Condition | Document | Status |
|-----------|---------|--------|
| 8 roles defined | EMS_ROLE_MODEL.md § 2 | ✓ FROZEN |
| MVP 3-tier model specified | EMS_ROLE_MODEL.md § 5 | ✓ FROZEN |
| Backward compatibility with REPORTERS constant | EMS_ROLE_MODEL.md § 4.2 | ✓ FROZEN |
| ADM via PIN (preserved) | EMS_ROLE_MODEL.md § 4.3 | ✓ FROZEN |
| Role defaults: no roster record → ENG | EMS_ROLE_MODEL.md § 6 | ✓ FROZEN |

**Role Model Gate: FROZEN ✓**

---

### 1.7 Permission Model Freeze

| Condition | Document | Status |
|-----------|---------|--------|
| WO action matrix (12 actions × 8 roles) | EMS_ACCESS_CONTROL.md § 3 | ✓ FROZEN |
| Section visibility matrix (8 sections × 8 roles) | EMS_ACCESS_CONTROL.md § 4 | ✓ FROZEN |
| Dispatch section detailed access rules | EMS_ACCESS_CONTROL.md § 5 | ✓ FROZEN |
| Inventory permission matrix | EMS_ACCESS_CONTROL.md § 6 | ✓ FROZEN |
| Critical Equipment permission matrix | EMS_ACCESS_CONTROL.md § 7 | ✓ FROZEN |
| Reports permission matrix | EMS_ACCESS_CONTROL.md § 8 | ✓ FROZEN |
| Client-side role resolution algorithm | EMS_ACCESS_CONTROL.md § 9.2 | ✓ FROZEN |
| Firebase security rules required (implementation responsibility) | EMS_ACCESS_CONTROL.md § 9.3 | ✓ NOTED |

**Permission Model Gate: FROZEN ✓**

---

## 2. Known Deferred Items (Not Blocking)

These items are documented, accepted, and deferred. They do NOT block the MVP implementation start.

| Item | Decision | Sprint |
|------|---------|--------|
| Full 8-role RBAC enforcement | DEC-002 | Sprint 2 |
| Push notifications | DEC-008 | Sprint 2 |
| Critical Equipment Dashboard widget | EMS_MVP_SCOPE.md § 3 | Sprint 2 |
| Multi-engineer WO assignment (full model) | DEC-014 | Sprint 2 |
| SLA auto-escalation | DEC-015 | Sprint 2 |
| PM auto-WO creation | DEC-016 | Sprint 2 |
| Parts request auto-timeout | DEC-017 | Sprint 2 |
| Projects section in nav | DEC-010 | Post-Sprint 2 |
| Analytics full charts | EMS_MVP_SCOPE.md § 3 | Sprint 2 |

---

## 3. Implementation Prerequisites

Before the first line of implementation code is written, the following must be prepared:

| Prerequisite | Owner | Notes |
|-------------|-------|-------|
| Firebase `maintenance/roster` path seeded with at minimum one ADM and one MGR record | Product Owner / Admin | Required for role resolution to function in dev/test |
| `maintenance/config/pin` confirmed unchanged | Product Owner | PIN must not change during refactor |
| At least one `maintenance/equipmentParts` record created as test data | Product Owner | Required to test Spare Parts tab |
| Git branch `feature/dashboard-refactor-v1` confirmed as the implementation branch | Engineering | No new branch; same branch |

---

## 4. Document Inventory

All documents that must be read before implementation begins:

| Document | Location | Purpose |
|---------|---------|---------|
| EMS_PRODUCT_ARCHITECTURE.md | docs/product/ | Navigation tree, user journeys |
| EMS_NAVIGATION.md | docs/product/ | Current → new screen mapping |
| EMS_ENGINEER_WORKSPACE.md | docs/product/ | Engineer workspace specification |
| EMS_EQUIPMENT_WORKSPACE.md | docs/product/ | Equipment Center specification |
| EMS_DISPATCH_WORKFLOW.md | docs/product/ | Full 10-stage dispatch lifecycle |
| EMS_CRITICAL_EQUIPMENT.md | docs/product/ | Critical equipment classification |
| EMS_ROLE_MODEL.md | docs/decision/ | Role definitions and storage |
| EMS_ACCESS_CONTROL.md | docs/decision/ | Permission matrix |
| EMS_DATA_MODEL.md | docs/decision/ | Official Firebase schema |
| EMS_AUDIT_MODEL.md | docs/decision/ | Audit event catalogue |
| EMS_MVP_SCOPE.md | docs/decision/ | Included / deferred / out of scope |
| EMS_DECISION_LOG.md | docs/decision/ | All 18 product decisions |
| EMS_IMPLEMENTATION_GATE.md | docs/decision/ | This document |

---

## 5. Verdict

```
┌────────────────────────────────────────────────────────────┐
│                                                            │
│                       VERDICT: GO                          │
│                                                            │
│  All gate conditions are satisfied.                        │
│                                                            │
│  P0 Items Resolved:      3 / 3   ✓                        │
│  P1 Items Resolved:      6 / 6   ✓                        │  
│  Architecture Frozen:    Yes     ✓                        │
│  Business Rules Frozen:  Yes     ✓                        │
│  Data Model Frozen:      Yes     ✓                        │
│  Role Model Frozen:      Yes     ✓                        │
│  Permission Model Frozen: Yes    ✓                        │
│                                                            │
│  Development may begin upon Product Owner approval         │
│  of this gate document.                                    │
│                                                            │
│  Next Work Order: WO-EMS-005 — Implementation Sprint 1    │
│                                                            │
└────────────────────────────────────────────────────────────┘
```

---

## 6. Product Owner Sign-Off

| Item | Status |
|------|--------|
| Gate document reviewed by Product Owner | **Pending** |
| WO-EMS-004A approved | **Pending** |
| Development authorized to begin | **Pending** |

**Development is NOT authorized until Product Owner explicitly approves this gate.**

---

*Gate prepared by: WO-EMS-004A Product Decision Resolution*
*Date: 2026-06-30*
*Branch: feature/dashboard-refactor-v1*
