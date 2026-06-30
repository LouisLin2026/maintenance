# EMS_DECISION_LOG.md

## BFY EMS — Product Decision Log

**Work Order:** WO-EMS-004A
**Status:** All P0 and P1 items RESOLVED
**Date:** 2026-06-30

---

## Purpose

This document is the canonical record of every product decision made during WO-EMS-004A. Each entry records the question, options considered, the chosen option, the rationale, and which document implements it.

Decisions in this log are **frozen**. They may only be changed via a new Work Order with explicit Product Owner approval.

---

## Decision Index

| ID | Category | Question | Priority | Status |
|----|----------|---------|---------|--------|
| DEC-001 | Access Control | Dispatch section access model | P0 | ✓ RESOLVED |
| DEC-002 | Role Model | How roles are defined and stored | P0 | ✓ RESOLVED |
| DEC-003 | Data Model | Material-equipment association method | P0 | ✓ RESOLVED |
| DEC-004 | Data Model | `finishAt` field formal definition | P1 | ✓ RESOLVED |
| DEC-005 | Audit | Audit trail model and verification record | P1 | ✓ RESOLVED |
| DEC-006 | Dashboard | Emergency Repair widget | P1 | ✓ RESOLVED |
| DEC-007 | Navigation | Overdue widget tap destination | P1 | ✓ RESOLVED |
| DEC-008 | Notification | Notification mechanism for MVP | P1 | ✓ RESOLVED |
| DEC-009 | Workflow | Testing stage (Stage 7) persistence | P2→P1 | ✓ RESOLVED |
| DEC-010 | MVP | Projects section in v1 navigation | P2 | ✓ RESOLVED |
| DEC-011 | Data Model | Stock quantity tracking method | P1 | ✓ RESOLVED |
| DEC-012 | Workflow | WO re-open rule | P2 | ✓ RESOLVED |
| DEC-013 | Workflow | WO delete permission | P2 | ✓ RESOLVED |
| DEC-014 | Workflow | Multi-engineer WO assignment | P2 | ✓ RESOLVED |
| DEC-015 | Business Rule | SLA escalation rule | P2 | ✓ RESOLVED |
| DEC-016 | Business Rule | PM overdue handling | P2 | ✓ RESOLVED |
| DEC-017 | Business Rule | Parts request timeout | P2 | ✓ RESOLVED |

---

## P0 Decisions

---

### DEC-001 — Dispatch Access Control

**Question:** Should the Dispatch section require the Admin PIN, or should it be visible to all users?

**Options Considered:**

| Option | Description |
|--------|-------------|
| A — Open (no PIN) | Dispatch visible to all authenticated users; write actions role-gated |
| B — Admin PIN required | All of Dispatch remains behind PIN gate; current behavior |
| C — Hybrid | Dispatch visible to all; specific tabs (assign, verify) require PIN |

**Decision:** Option A — Open to all authenticated users.

**Rationale:**
- Engineers need situational awareness of all WOs to coordinate without double-dispatching.
- The PIN was protecting operational visibility, not configuration or sensitive data. Configuration remains PIN-protected (Administration section).
- Role-gated write actions (assign, verify, close, delete) provide the correct security boundary — restricting by action, not by page.
- Option C (hybrid) creates confusing UX: a page appears accessible but then demands a PIN mid-flow.

**Implemented in:** [EMS_ACCESS_CONTROL.md § 5](EMS_ACCESS_CONTROL.md)

---

### DEC-002 — Role Model

**Question:** How are user roles defined, stored, and resolved in the application?

**Options Considered:**

| Option | Description |
|--------|-------------|
| A — 2-tier: Engineer / Admin (PIN) | Minimal; no role data model change |
| B — 3-tier MVP: ENG / MGR / ADM | Adds MGR via roster; ADM via PIN |
| C — Full 8-role RBAC from day 1 | All roles, full roster, complete access gating |

**Decision:** Option B (3-tier MVP) with full 8-role model specified for Sprint 2.

**Rationale:**
- Option A (2-tier) is insufficient — managers need dispatch and verification rights without sharing the Admin PIN.
- Option C (full RBAC from day 1) requires all engineers to have roster records before launch, creating a blocking migration task.
- Option B launches with functional role separation (Admin / Manager / Engineer-default) immediately. Sprint 2 populates the full roster and enforces the remaining 5 roles.

**Full role definitions:** [EMS_ROLE_MODEL.md](EMS_ROLE_MODEL.md)
**Permission matrix:** [EMS_ACCESS_CONTROL.md](EMS_ACCESS_CONTROL.md)

---

### DEC-003 — Material-Equipment Association

**Question:** What is the official data model for associating spare parts with equipment?

**Options Considered:**

| Option | Description |
|--------|-------------|
| A — Sub-collection on equipment | `equipment/{id}/parts/{id}` |
| B — Junction path | `maintenance/equipmentParts/{id}` with `equipId` and `matId` |
| C — Category tag on material | `material/{id}/cat` used as implicit equipment association |

**Decision:** Option B — many-to-many via `maintenance/equipmentParts` junction path.

**Rationale:**
- Option A: Firebase reads the entire node; embedding parts in equipment means every equipment fetch pulls all parts data even when not needed. Reverse query (which equipment uses this part?) requires a full scan.
- Option B: Junction records allow efficient queries in both directions (`orderByChild("equipId")` and `orderByChild("matId")`). Clean separation of concerns.
- Option C: Category association is too coarse — the same bearing type (category: Bearings) may be used in 40 machines; the tab would show all bearings, not the specific ones for this machine.

**Implemented in:** [EMS_DATA_MODEL.md § 5.2, § 6](EMS_DATA_MODEL.md)

---

## P1 Decisions

---

### DEC-004 — `finishAt` Field Definition

**Question:** Is `finishAt` an existing field in the Firebase WO schema, and if not, how is it formally added?

**Finding:** Field does not exist in the current `workorders` schema (confirmed from `index.html` analysis). It was referenced in two spec documents without formal definition.

**Decision:** `finishAt` is a **new field** added to `workorders/{id}`. Type: number (Unix ms timestamp). Written by the report submission handler (`submitReport`) at Stage 8 when `progress = 已完成`.

**Implemented in:** [EMS_DATA_MODEL.md § 4.1](EMS_DATA_MODEL.md)

---

### DEC-005 — Audit Trail Model

**Question:** What events require an audit record, and what is the structure of those records?

**Decision:** Structured audit records are written to `maintenance/auditLog` for all state-changing actions. The existing unstructured audit log entries remain; new entries use the defined schema. Audit log is append-only; no deletion.

Specifically for Stage 9 Verified: `verifiedBy` and `verifiedAt` are written to the WO record directly (for query efficiency) AND a `WO_VERIFIED` audit event is written to the audit log (for accountability).

**Implemented in:** [EMS_AUDIT_MODEL.md](EMS_AUDIT_MODEL.md) and [EMS_DATA_MODEL.md § 4.1](EMS_DATA_MODEL.md)

---

### DEC-006 — Emergency Repair Widget

**Question:** How does the Dashboard surface active emergency / breakdown repairs?

**Decision:** Add an **Emergency Repair widget** to the Dashboard (9th widget). It shows WOs where `urgency == 緊急 AND status IN (已派工, 進行中)` — meaning: dispatched emergency repairs that are currently active, not just waiting.

**Widget content:**
```
Emergency Repairs: [N active]
⚠ FR-20260630-2345  [HIGH] [進行中]
  Line A Conveyor — Motor fault — 35 min ago
  Assigned: [Name]
[View All]
```

**Tap destination:** Dispatch → In Progress tab, filter pre-applied for `urgency == 緊急`.

**Implemented in:** [EMS_MVP_SCOPE.md § 2.2](EMS_MVP_SCOPE.md)

---

### DEC-007 — Overdue Widget Navigation

**Question:** Where does tapping the Dashboard Overdue widget navigate?

**Finding from WO-EMS-004 review:** The spec referenced "Dispatch / Overdue view" but no Overdue tab exists in the Dispatch tab bar.

**Decision:** The Overdue widget navigates to **Dispatch → All tab** with a pre-applied filter for `due < today AND status != 已完成`. There is no separate Overdue tab. The filter state is passed as a navigation parameter.

**No new Dispatch tab is added.** The existing All tab handles filtered views via query parameters.

**Implemented in:** [EMS_MVP_SCOPE.md § 2.2](EMS_MVP_SCOPE.md)

---

### DEC-008 — Notification Mechanism

**Question:** How are engineers notified of new WO assignments, and how are managers notified of new fault reports?

**Options Considered:**

| Option | Description | Cost |
|--------|-------------|------|
| A — No notification; refresh manually | Zero implementation | Poor UX for urgent dispatches |
| B — In-app badge counters on nav icons | Badge count on Dispatch / Today's Tasks icons | Low |
| C — PWA push notification | OS-level notification when app is backgrounded | Medium-high; requires permission flow |

**Decision:** Option B (in-app badge counters) for MVP. Option C (push notifications) deferred to Sprint 2.

**MVP notification behavior:**
- Dispatch nav icon: shows badge count of WOs in 待派工 status (unassigned)
- Today's Tasks nav icon: shows badge count of WOs assigned to current user that are in 已派工 status (awaiting acceptance)
- Badges update in real-time via Firebase listeners
- No OS-level push notifications in MVP

**"Admin notified" and "Engineer notified" in user journeys mean:** Badge count increments on the relevant nav icon on next Firebase listener fire. No pop-up, no push.

**Implemented in:** [EMS_MVP_SCOPE.md](EMS_MVP_SCOPE.md)

---

### DEC-009 — Testing Stage Persistence (Promoted from P2)

**Question:** Should Stage 7 (Testing) be persisted in Firebase, or remain UI-only as specified?

**Decision:** Add `testingAt` (timestamp) to the WO record. Firebase status field remains `進行中` (no new status enum value). The `testingAt` field proves the testing phase occurred and when it started, without requiring a new status value.

**Rationale:** Upgrading from UI-only to lightweight persistence costs one Firebase write. It provides evidence that the engineer marked equipment as testing before submitting the completion report — defensible in a dispute.

**Implemented in:** [EMS_DATA_MODEL.md § 4.1](EMS_DATA_MODEL.md)

---

## P2 Decisions

---

### DEC-010 — Projects Section in v1 Navigation

**Question:** Should the Projects section appear in the bottom navigation bar at MVP launch?

**Decision:** **Excluded from MVP navigation.** The bottom nav has 7 items: Dashboard, Today's Tasks, Dispatch, Equipment, Engineers, Analytics, Administration.

**Rationale:** Projects has no current implementation and no Firebase data. Showing a "Coming Soon" tab in the nav bar creates confusion about the app's capabilities. Projects will be added to the nav when it is implemented.

**Implemented in:** [EMS_MVP_SCOPE.md § 2.1](EMS_MVP_SCOPE.md)

---

### DEC-011 — Stock Quantity Tracking

**Question:** Should stock quantity be tracked by adding a `stock` field to `material` records, or inferred from requisition history?

**Decision:** Add a `stock` field directly to `maintenance/material/{id}`.

**Rationale:** Requisition history is incomplete — it records what was requested and confirmed, but not physical stock received, consumed outside a WO, or counted in a stocktake. A direct field edited by WHS is accurate and simple.

**Implemented in:** [EMS_DATA_MODEL.md § 7](EMS_DATA_MODEL.md)

---

### DEC-012 — WO Re-open Rule

**Question:** Can a closed WO be re-opened, and under what conditions?

**Decision:** Only ADM can re-open a closed WO. Re-open creates a `WO_REOPENED` audit event and resets status to `進行中`. No data is lost on re-open.

**Business rule:** Re-open is reserved for cases where verification was performed in error or where the same equipment failure recurs within 24 hours of closing. It is not a routine operation.

---

### DEC-013 — WO Delete Permission

**Question:** Who can delete a work order?

**Decision:** Only ADM can delete a WO. A `WO_DELETED` audit event is written containing the full WO summary before deletion. The WO record is removed from Firebase. Associated reports are not deleted.

**Rationale:** Delete is irreversible. This is a high-risk action appropriate only for administrators (e.g., test data, duplicate entries). MGR can Reject (Stage 2) which closes the WO without deleting it.

---

### DEC-014 — Multi-Engineer WO Assignment

**Question:** How are WOs with multiple engineers managed?

**Decision for MVP:** `assignedTo` remains a single string (primary engineer). Additional engineers are recorded in the `report.reporters` array when they submit a report on the WO. Today's Tasks shows a WO to an engineer if they appear in any report's `reporters` array, in addition to `assignedTo`.

**Sprint 2 full model:** `assignedTo` will support an array, and Today's Tasks will filter by the full assignment array. For MVP, the current single-field model is preserved.

---

### DEC-015 — SLA Escalation Rule

**Decision:** For MVP, SLA is **informational only** — the Overdue widget and badge counters surface breaches. No automated escalation.

Full SLA escalation rules (auto-notify, auto-reassign) are deferred to Sprint 2 and require Firebase Functions (server-side triggers). The priority definitions from EMS_DISPATCH_WORKFLOW.md § 5 are the reference SLA targets.

---

### DEC-016 — PM Overdue Handling

**Decision:** When `plans/{planId}/nextDue < today` and no WO exists for this plan, the Equipment section and Dashboard "Today's PM" widget flag the equipment as PM overdue. No automatic WO is created in MVP. Admin/MGR must manually create the PM WO from Equipment → PM Schedule tab.

Automatic PM WO creation is deferred to Sprint 2 (requires scheduled Firebase Function).

---

### DEC-017 — Parts Request Timeout

**Decision:** For MVP, there is no automatic timeout on unconfirmed requisitions. A requisition in `待料` status remains until WHS confirms it or Admin manually resets the WO status.

The Dashboard "Waiting Parts" widget surfaces unconfirmed requisitions as a visibility mechanism. Automated escalation after N days is deferred to Sprint 2.

---

## Decision Summary

| P-Level | Count | All Resolved? |
|---------|-------|--------------|
| P0 | 3 | ✓ Yes |
| P1 | 6 | ✓ Yes |
| P2 | 9 | ✓ Yes |
| **Total** | **18** | ✓ **All Resolved** |
