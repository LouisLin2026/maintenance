# EMS_ACCESS_CONTROL.md

## BFY EMS — Access Control & Permission Matrix

**Work Order:** WO-EMS-004A
**Status:** FROZEN — Decision Resolved
**Date:** 2026-06-30
**Resolves:** WO-EMS-004 P0-01 — Dispatch Access Control; WO-EMS-004 P0-02 (Role Model)

---

## 1. Decision

**Decision ID:** DEC-001
**Resolved:** Yes

**Dispatch Access:** The Dispatch section is **accessible to all authenticated users** (Engineer tier and above) for viewing work order status. **Write actions within Dispatch are role-gated.** The Admin PIN is no longer required to view work orders.

**Rationale:** Engineers on the floor need to see the full dispatch board to coordinate work — knowing what their colleagues are working on prevents duplicated effort and wasted travel. Restricting the view to Admin-PIN holders created an operational bottleneck with no security benefit (all engineers already knew which WOs existed via conversation). Write actions (assign, verify, close) are protected by role, not by a shared PIN.

---

## 2. Permission Scope Definitions

| Permission Level | Symbol | Meaning |
|-----------------|--------|---------|
| Full | **Y** | Can read and write; all sub-actions allowed |
| Read | **R** | Can view; cannot create, edit, or delete |
| Limited | **L** | Partial access — details defined per cell |
| None | **N** | Not accessible |

---

## 3. Work Order Action Matrix

| Action | ADM | MGR | ENG | PLR | WHS | EQO | VND | RDO |
|--------|-----|-----|-----|-----|-----|-----|-----|-----|
| Create WO (any type) | Y | Y | Y | L¹ | N | L² | N | N |
| View all WOs | Y | Y | Y | Y | N | L³ | L⁴ | Y |
| Assign WO to engineer | Y | Y | N | N | N | N | N | N |
| Accept WO (own) | Y | Y | Y | N | N | N | Y | N |
| Update progress / add note | Y | Y | Y | N | N | N | Y | N |
| Submit work report | Y | Y | Y | N | N | N | Y | N |
| Request parts (Stage 6) | Y | Y | Y | N | N | N | Y | N |
| Mark as Testing (Stage 7) | Y | Y | Y | N | N | N | Y | N |
| Verify completion (Stage 9) | Y | Y | N | L⁵ | N | L⁵ | N | N |
| Close WO (Stage 10) | Y | Y | N | N | N | N | N | N |
| Reject WO | Y | Y | N | N | N | N | N | N |
| Delete WO | Y | N | N | N | N | N | N | N |
| Re-open closed WO | Y | N | N | N | N | N | N | N |

**Notes:**
- ¹ PLR can create **Fault Reports only** (FR- prefix; source = 故障叫修). Cannot create PM or admin WOs.
- ² EQO can create WOs **for their designated equipment only**.
- ³ EQO sees all WOs but Dashboard and view defaults filter to their equipment.
- ⁴ VND sees only WOs assigned to them.
- ⁵ PLR and EQO can add a **production/equipment-ready confirmation** (a separate field from the maintenance verification). This is not the same as Stage 9 Verified — it is an additional stakeholder sign-off that occurs after Stage 9.

---

## 4. Section Visibility Matrix

| Section | ADM | MGR | ENG | PLR | WHS | EQO | VND | RDO |
|---------|-----|-----|-----|-----|-----|-----|-----|-----|
| Dashboard | Y | Y | Y | L⁶ | L⁷ | L⁸ | L⁹ | Y |
| Today's Tasks | Y | Y | Y | N | N | N | Y | N |
| Dispatch | Y | Y | Y | R | N | R | L⁴ | R |
| Equipment | Y | Y | Y | R | R | Y | R | R |
| Engineers | Y | Y | R¹⁰ | N | N | N | N | R |
| Projects | Y | Y | R | N | N | N | N | R |
| Analytics | Y | Y | R¹¹ | N | N | R¹² | N | R |
| Administration | Y | L¹³ | N | N | L¹⁴ | N | N | N |

**Notes:**
- ⁶ PLR Dashboard: sees only widgets for their production line equipment (PM, faults, critical equipment). No cost, no engineer performance.
- ⁷ WHS Dashboard: sees only Waiting Parts widget and Inventory summary. No dispatch or engineer data.
- ⁸ EQO Dashboard: sees their equipment's critical status, open WOs, and PM schedule.
- ⁹ VND Dashboard: sees only their own assigned WOs summary.
- ¹⁰ ENG can view own profile and team roster (read). Cannot view other engineers' KPI financial details.
- ¹¹ ENG Analytics: read access to own performance KPIs only. No cost view.
- ¹² EQO Analytics: read access to their equipment's performance data only.
- ¹³ MGR Administration: can access Equipment Master (edit), Materials Master (read), Engineers & Roles (read), Requisitions (approve), Audit Log (read). Cannot change System Config or PIN.
- ¹⁴ WHS Administration: Inventory Management only.

---

## 5. Dispatch Section — Detailed Access

This resolves the highest-risk item from WO-EMS-004 review (Row 10, EMS_NAVIGATION.md).

### 5.1 Who Can Access Dispatch

| User Tier | Can Access Dispatch | What They See |
|-----------|--------------------|--------------| 
| No identity selected | No | Redirect to identity selection |
| ENG (default) | Yes | All WOs visible; own WOs highlighted |
| PLR | Yes (read only) | All WOs visible; no action buttons shown |
| MGR | Yes | All WOs; assign + verify buttons visible |
| ADM | Yes | All WOs; all actions + delete |
| WHS | No | Dispatch is not in navigation for WHS |
| VND | Yes (limited) | Only WOs assigned to this vendor |
| RDO | Yes (read) | All WOs; no action buttons |

### 5.2 Dispatch Action Gating

The Dispatch section renders differently based on the current user's role. Action buttons appear or are hidden based on role, not PIN entry.

| Action Button | Visible To | Hidden From |
|--------------|-----------|------------|
| [Assign Engineer] | ADM, MGR | ENG, PLR, EQO, WHS, VND, RDO |
| [Accept] | ENG, VND, ADM, MGR | PLR, WHS, EQO, RDO |
| [Submit Report] | ENG, VND, ADM, MGR | PLR, WHS, EQO, RDO |
| [Request Parts] | ENG, VND, ADM, MGR | PLR, WHS, EQO, RDO |
| [Mark Testing] | ENG, VND, ADM, MGR | PLR, WHS, EQO, RDO |
| [Verify] | ADM, MGR | All others |
| [Close] | ADM, MGR | All others |
| [Reject] | ADM, MGR | All others |
| [Delete] | ADM only | All others |

### 5.3 PIN Gate Removal — Scope

The Admin PIN is **removed as the gatekeeper for Dispatch viewing.** It is **retained** for:
- Administration section (Equipment Master edit, System Config, Engineer & Roles management)
- Any ADM-level action (Delete WO, change PIN, bulk operations)

This preserves the existing security boundary for configuration and destructive actions while opening the operational workflow to the team.

---

## 6. Inventory Permission Matrix

| Action | ADM | MGR | ENG | PLR | WHS | EQO | VND | RDO |
|--------|-----|-----|-----|-----|-----|-----|-----|-----|
| View material list | Y | Y | Y | R | Y | R | R | R |
| View stock levels | Y | Y | Y | R | Y | R | R | R |
| Edit material master | Y | Y | N | N | Y | N | N | N |
| Add new material | Y | Y | N | N | Y | N | N | N |
| Submit parts requisition | Y | Y | Y | N | N | N | Y | N |
| Approve / confirm requisition | Y | Y | N | N | Y | N | N | N |
| View requisition history | Y | Y | Y | N | Y | N | R¹⁵ | R |
| Adjust stock quantity | Y | N | N | N | Y | N | N | N |

**Note:**
- ¹⁵ VND can view requisitions linked to their own WOs only.

---

## 7. Critical Equipment Permission Matrix

| Action | ADM | MGR | ENG | PLR | WHS | EQO | VND | RDO |
|--------|-----|-----|-----|-----|-----|-----|-----|-----|
| View criticality classification | Y | Y | Y | Y | R | Y | R | Y |
| Edit criticality level | Y | Y | N | N | N | L¹⁶ | N | N |
| View MTTR / MTBF | Y | Y | Y | R | N | Y | R | Y |
| Add weak point observation | Y | Y | Y | N | N | Y | N | N |
| Edit / close weak point | Y | Y | N | N | N | N | N | N |
| View Recovery SOP | Y | Y | Y | Y | N | Y | Y | Y |
| Edit Recovery SOP | Y | Y | N | N | N | N | N | N |
| View PLC Backup info | Y | Y | Y | N | N | Y | N | N |
| Upload PLC Backup | Y | Y | Y | N | N | N | N | N |
| View Visual Reference | Y | Y | Y | Y | N | Y | Y | Y |
| Upload photos / videos | Y | Y | Y | N | N | N | N | N |

**Note:**
- ¹⁶ EQO can **propose** a criticality level change. ADM or MGR must confirm.

---

## 8. Reports Permission Matrix

| Action | ADM | MGR | ENG | PLR | WHS | EQO | VND | RDO |
|--------|-----|-----|-----|-----|-----|-----|-----|-----|
| View own work reports | Y | Y | Y | N | N | N | Y | N |
| View all work reports | Y | Y | R¹⁷ | N | N | R¹⁸ | N | Y |
| Submit work report | Y | Y | Y | N | N | N | Y | N |
| Edit submitted report | Y | Y | L¹⁹ | N | N | N | L¹⁹ | N |
| Delete report | Y | N | N | N | N | N | N | N |
| Export reports | Y | Y | N | N | N | N | N | Y |
| View cost breakdown | Y | Y | N | N | N | N | N | N |
| View engineer performance | Y | Y | R²⁰ | N | N | N | N | R |

**Notes:**
- ¹⁷ ENG can read all reports in read-only mode (useful for checking past work on equipment). Can only write their own.
- ¹⁸ EQO can read reports for their equipment only.
- ¹⁹ ENG / VND can edit their own report within 24 hours of submission only.
- ²⁰ ENG can see their own performance KPIs. Cannot see cost data or other engineers' hourly rate.

---

## 9. Access Control Implementation Notes

### 9.1 MVP Implementation (3-Tier)

For the MVP, implement 3 tiers only:

| MVP Tier | Full Roles | Access Level |
|---------|-----------|-------------|
| ADM | ADM | PIN entry grants ADM tier for session |
| MGR | MGR | Roster record role == MGR |
| Default | ENG + all others | No roster record, or role == ENG/PLR/WHS/EQO/VND/RDO |

The full 8-role matrix is implemented in Sprint 2 after the `maintenance/roster` path is populated.

### 9.2 Client-Side Role Resolution

```
1. On identity selection:
   a. Look up name in maintenance/roster
   b. If found: sessionRole = roster.role
   c. If not found: sessionRole = "ENG"
   d. Store sessionRole in memory (not localStorage — security)

2. On Admin PIN entry:
   a. Successful PIN: sessionRole = "ADM" for this session
   b. PIN override applies until page reload or explicit logout

3. UI rendering:
   a. Action buttons: check sessionRole against permission matrix
   b. Section visibility: check sessionRole against section matrix
   c. Default to most restrictive if role cannot be resolved
```

### 9.3 Server-Side Note

Firebase Realtime Database security rules must enforce the same permission matrix. Client-side role resolution is UX only — it is not a security boundary. Firebase rules are the enforcement layer.

Firebase security rules implementation is deferred to the implementation sprint. The permission matrix in this document is the authoritative source for rule authoring.
