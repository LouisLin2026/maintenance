# EMS_ROLE_MODEL.md

## BFY EMS — Official Role Model

**Work Order:** WO-EMS-004A
**Status:** FROZEN — Decision Resolved
**Date:** 2026-06-30
**Resolves:** WO-EMS-004 P0-02 — Role Model Is Not Defined

---

## 1. Decision

**Decision ID:** DEC-002
**Resolved:** Yes
**Verdict:** Eight operational roles are defined. MVP implementation uses a 3-tier simplified model (Admin / Manager / Engineer) mapped from the existing PIN system. Full 8-role RBAC is deferred to the first post-MVP sprint.

---

## 2. Role Definitions

### Role Registry

| Role ID | Role Name | Chinese | Description |
|---------|-----------|---------|-------------|
| ADM | Administrator | 系統管理員 | Full system control. Manages users, configuration, master data. Handles PIN-level access equivalent to current admin section. |
| MGR | Maintenance Manager | 維修主管 | Operational authority. Reviews, assigns, and verifies all WOs. Owns daily dispatch. Views all analytics and cost data. |
| ENG | Maintenance Engineer | 維修工程師 | Frontline maintenance. Accepts and executes assigned WOs. Submits work reports. Reports faults via QR. |
| PLR | Production Leader | 生產線長 | Production floor supervisor. Reports faults for their line. Verifies equipment is operational from the production perspective after repair. Cannot assign or close WOs. |
| WHS | Warehouse | 倉管 | Manages physical inventory. Confirms parts requisitions. No WO creation or dispatch authority. |
| EQO | Equipment Owner | 設備負責人 | Technical responsible for a specific equipment category or production line. Can request maintenance on owned equipment and verify recovery. |
| VND | Vendor / Contractor | 廠商/外包 | External contractor assigned to specific WOs. Accepts and reports work. Scoped view — sees only their own assignments. |
| RDO | Read Only | 唯讀 | Audit, compliance, or observer access. No write permissions. Sees all data. |

---

## 3. Role Hierarchy

```
ADM (Administrator)
  └── Full authority over all roles and data

MGR (Maintenance Manager)
  └── Operational authority: dispatch, verify, close, analytics

ENG (Maintenance Engineer)
  └── Execution authority: accept, work, report

PLR (Production Leader)
  └── Request authority: create fault WO, verify equipment OK

WHS (Warehouse)
  └── Inventory authority: confirm requisitions, manage stock

EQO (Equipment Owner)
  └── Equipment authority: request maintenance, verify own equipment

VND (Vendor)
  └── Contractor execution: accept assigned WO, submit report

RDO (Read Only)
  └── Observer: read all, write nothing
```

---

## 4. Role Storage — Firebase

### 4.1 New Path: `maintenance/roster`

The current REPORTERS constant is a flat string array. This is insufficient for role-based access control. A new Firebase path is required.

**Path:** `maintenance/roster/{autoId}`

**Record structure:**

```json
{
  "name":      "Engineer Display Name",
  "role":      "ENG",
  "active":    true,
  "addedAt":   1751299200000,
  "addedBy":   "admin"
}
```

| Field | Type | Description |
|-------|------|-------------|
| `name` | string | Display name; must match existing REPORTERS value for backward compatibility |
| `role` | string | Role ID: ADM / MGR / ENG / PLR / WHS / EQO / VND / RDO |
| `active` | boolean | Whether this person is currently active in the system |
| `addedAt` | number (ms timestamp) | When this record was created |
| `addedBy` | string | Name of admin who added this record |

### 4.2 Backward Compatibility

The existing REPORTERS constant in `index.html` is a flat array used to populate the engineer name picker. During the refactor:

- The roster Firebase path **supplements** REPORTERS; it does not replace it at launch.
- The name picker reads from REPORTERS (existing logic unchanged).
- Role resolution reads from `maintenance/roster` — if a name has no roster record, it defaults to the ENG role.
- Admin creates roster records via Administration → Engineers & Roles.

This means:
- Existing engineers work without migration (they default to ENG).
- Role upgrades (to MGR, ADM) are added explicitly.
- No breaking change to existing identity selection flow.

### 4.3 Admin Authentication

The current Admin PIN (`maintenance/config/pin`) remains unchanged. PIN entry grants the ADM session role for that device session. This is the authentication mechanism for the MVP.

For the full 8-role model (post-MVP), individual login/authentication is required. This is deferred.

---

## 5. MVP Role Simplification

The full 8-role model is fully specified for implementation. However, the **MVP release uses a 3-tier simplified model** while the roster Firebase path and full RBAC are built:

| MVP Tier | Maps To Full Roles | Mechanism |
|----------|--------------------|-----------|
| Admin | ADM | PIN entry (existing) |
| Manager | MGR + EQO (limited) | Roster record with role = MGR |
| Engineer (default) | ENG + PLR + WHS + VND + RDO | Roster record or REPORTERS default |

**Impact:** In the MVP, PLR, WHS, EQO, VND, and RDO roles are collapsed into the Engineer tier. The full 8-role access matrix is enforced from Sprint 2 onwards when the roster path is populated.

---

## 6. Role Assignment Rules

| Rule | Definition |
|------|-----------|
| Single role per person | Each person has exactly one role at a time |
| Role is global | Role applies across all sections; no per-equipment or per-project scoping in v1 |
| Admin can change any role | Only ADM can modify the roster |
| VND accounts are temporary | Vendor records should have an `expiresAt` field (future); for MVP they are active until manually deactivated |
| A name not in roster defaults to ENG | No hard failure if roster record is missing; safe degradation |

---

## 7. Role Display in UI

| Location | Display |
|---------|---------|
| Engineer Profile header | Role badge next to name (e.g., `[MGR]`) |
| Team Roster card | Role shown under name |
| WO detail — Assigned To | "(ENG)" suffix next to engineer name |
| Administration → Engineers & Roles | Full roster table with edit capability |
