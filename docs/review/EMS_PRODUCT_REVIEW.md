# EMS_PRODUCT_REVIEW.md

## BFY EMS — Product Owner Review
### WO-EMS-004 — Product Review & Decision Approval

**Review Of:** WO-EMS-003 Specification (6 documents)
**Branch:** `feature/dashboard-refactor-v1`
**Review Date:** 2026-06-30
**Reviewer:** Product Owner Review (WO-EMS-004)

---

## 1. Executive Summary

The WO-EMS-003 specification establishes a sound Workflow-Driven Architecture for the BFY EMS navigation refactor. The core direction is correct: moving from role-based portals to workflow-oriented sections will significantly improve operational usability for both engineers and managers.

**All 22 existing screens are accounted for and mapped.** The navigation tree is complete. The dispatch lifecycle (10 stages) is a meaningful improvement over the current 5-state model.

However, **7 implementation blockers** were identified that must be resolved before development begins. These are not architectural flaws — they are unresolved design decisions, data model gaps, and missing business rules that will cause implementation work to stall or produce incorrect behavior.

**Verdict: GO WITH CHANGES** — see Section 9 for the full decision matrix.

---

## 2. Strengths

### 2.1 Architecture

| Strength | Evidence |
|----------|---------|
| All existing screens fully accounted for | EMS_NAVIGATION.md § 3 — 22 screens mapped; no orphans |
| No data migration risk for core refactor | EMS_DISPATCH_WORKFLOW.md § 1.1 — new stages map to existing status values |
| QR backward compatibility preserved | EMS_NAVIGATION.md § 6 — `#report={code}` and `#scan={eqid}` explicitly preserved |
| PM context correctly relocated | Equipment → PM tab is the right model; cross-equipment view also specified |
| Future modules have clear plug-in points | EMS_PRODUCT_ARCHITECTURE.md § 6 — 7 future modules mapped |

### 2.2 Workflow

| Strength | Evidence |
|----------|---------|
| 10-stage lifecycle adds quality gates | Testing (Stage 7) and Verified (Stage 9) close the quality gap in the current 5-state model |
| PM update on WO completion is specified | EMS_DISPATCH_WORKFLOW.md Stage 8 — `lastDone` and `nextDue` update on report submit |
| Requisition flow is fully mapped | Stage 6 Waiting Parts → Administration → Requisitions → confirm → engineer notified |
| All WO types covered | 4 WO types (PM, Admin-created, Engineer-initiated, Fault Report) with ID prefix rules |

### 2.3 Data

| Strength | Evidence |
|----------|---------|
| MTTR formula is correct and implementable | `SUM(reports.hours) / COUNT(completed WOs)` — uses existing Firebase data |
| MTBF formula is correct | `Total days in period / COUNT(fault WOs)` — uses existing FR- WO count |
| New Firebase fields are explicitly listed | EMS_CRITICAL_EQUIPMENT.md § 6.1 — 12 fields documented |
| Critical Equipment v1 scope is appropriately scoped | § 6.2 limits v1 to 4 items using only existing data |

---

## 3. Weaknesses

### 3.1 Role Model Is Not Defined (Blocker)

**Finding:** The specifications reference 4 distinct actors — Engineer, Senior Engineer, Manager, and Admin — but no document defines what distinguishes them or how the app determines which role a user holds.

| Actor | Mentioned In | Specific Permissions |
|-------|-------------|----------------------|
| Engineer | All documents | Basic; own WOs only |
| Senior Engineer | EMS_DISPATCH_WORKFLOW.md Stage 9 | Can verify completions |
| Manager | EMS_ENGINEER_WORKSPACE.md § 2.5 | Team View; can assign |
| Admin | All documents | Full access; PIN-protected |

**Problem:** The current app identifies users by name only (REPORTERS config — a flat string list). There is no `role` field. When the spec says "Admin or senior engineer can verify" — the app has no mechanism to determine if the current user is a Senior Engineer.

**Impact:** Stage 9 Verified, Team View toggle, and Admin-only actions (PM schedule edit, Equipment Master edit) cannot be correctly gated without role data.

---

### 3.2 Testing and Verified Stages Have No Data Persistence (Blocker)

**Finding:** Stages 7 (Testing) and 9 (Verified) are defined as "UI-only states" with no corresponding Firebase field changes.

- **Stage 7 Testing:** Firebase status remains `進行中`. On app reload, the Testing state is lost.
- **Stage 9 Verified:** No `verifiedBy` or `verifiedAt` field. There is no audit record of who verified a WO or when.

**Impact 1:** If a user closes the app during Testing, the WO reverts to "Working" appearance. Engineers who are testing cannot prove they completed a test before submitting the report.

**Impact 2:** Verification has no accountability. The quality gate (Stage 9) has no data footprint, making it impossible to report on verification compliance or investigate disputes.

---

### 3.3 Material-to-Equipment Association Is Unresolved (Blocker)

**Finding:** EMS_EQUIPMENT_WORKSPACE.md § 3.8 specifies an Equipment → Spare Parts tab showing materials associated with a specific equipment unit. However:

- The current `maintenance/material` schema has no `equipId` or `category` field linking a material to equipment.
- The spec says "association is by category or explicit link — implementation decision."
- EMS_CRITICAL_EQUIPMENT.md § 3 (Section 4) references a `criticalParts` sub-path but makes the same deferral.

**Impact:** Without a defined association mechanism, the Spare Parts tab cannot be built. An implementation team will make an ad-hoc decision here, potentially creating a schema that conflicts with the Critical Equipment model.

---

### 3.4 Emergency Repair Is Not Surfaced on Dashboard (Gap)

**Finding:** The WO review scope (WO-EMS-004) requires the Dashboard to support "Emergency Repairs." The current Dashboard specification has:

- Waiting Assignment widget (status: 待派工)
- Overdue widget (due < today)
- Critical Equipment widget (C/I equipment with alerts)

None of these capture **active emergency breakdown repairs** — a HIGH priority FR- work order that was dispatched 30 minutes ago and is actively in progress. This is arguably the most important piece of operational visibility for a manager.

**Impact:** A manager opening the app during a critical breakdown will not see the emergency on the Dashboard. They must navigate to Dispatch → In Progress and manually scan the list.

---

### 3.5 `finishAt` Field Is Not in the Current Schema (Data Gap)

**Finding:** EMS_DISPATCH_WORKFLOW.md Stage 8 writes `finishAt: timestamp` to the WO record. EMS_ENGINEER_WORKSPACE.md § 2.3 uses `finishAt` for the "Completed today" filter.

The original Firebase schema for workorders (as analyzed from `index.html`) is:
```
id, source, equip, equipName, item, priority, assignedTo, status,
createdAt, due, from, symptom, urgency, reporter, area, dept
```

`finishAt` is **not in this schema.** It must be formally added before the Completed today filter and WO close timestamp can function.

**Impact:** The "Completed today" display in Today's Tasks and the WO close timestamp in Analytics will fail silently if this field does not exist.

---

### 3.6 Dashboard Widget Content Is Underspecified (Gap)

**Finding:** EMS_PRODUCT_ARCHITECTURE.md § 2 lists 8 Dashboard widgets with `(from: ...)` source annotations but no document specifies:

- How many items appear per widget (before "View All")
- What date range "Weekly KPI" uses (calendar week vs rolling 7 days)
- Where tapping each widget navigates to
- Whether MTTR and MTBF appear on the Dashboard (they appear in Critical Equipment widget and Critical Points tab, but not explicitly in the Weekly KPI widget)

**Impact:** Each widget's navigation destination is undefined. Tapping "Overdue widget" — does it go to Dispatch filtered by overdue? Which view exactly? An implementation team will make different assumptions per widget without this spec.

---

### 3.7 Notification Mechanism Is Not Specified (Gap)

**Finding:** Two user journeys reference notifications:

- User Journey 4.2: "Submit → creates FR- Work Order → **Admin notified** → Dispatch section shows new fault report"
- User Journey 4.4: "Assign engineer → WO status: 已派工 → **Engineer notified** → appears in Today's Tasks"

No document specifies **how** notification happens. Options — none of which are specified:
- Push notification (requires PWA notification API + permission)
- In-app badge/counter on navigation
- Email notification
- No notification; users must refresh

**Impact:** User Journey 4.2 and 4.4 cannot be implemented as drawn without this decision.

---

## 4. Missing Features

### 4.1 Missing: Overdue Tab in Dispatch

**Location:** EMS_PRODUCT_ARCHITECTURE.md § 4.3 references "Dispatch / Overdue view" as a navigation destination from the Dashboard Overdue widget.

**Problem:** EMS_DISPATCH_WORKFLOW.md § 6.1 specifies Dispatch tabs as: `All | Pending | In Progress | Waiting Parts | Completed`. There is no "Overdue" tab.

**Resolution needed:** Either add an Overdue tab to Dispatch, or specify that the Overdue widget navigates to the In Progress tab with a date filter pre-applied.

---

### 4.2 Missing: Emergency Repair Widget and Fast Path

**What is needed:** A Dashboard widget that shows active fault reports (FR- WOs) with HIGH priority or urgency, regardless of whether they are overdue. Current widgets cover "waiting assignment" and "overdue" but not "active emergency in progress."

**Suggested widget:**
```
Active Emergencies: [N]
⚠ FR-20260630-2345  [HIGH] [進行中]
  Line A Conveyor — Motor fault
  Dispatched: 10 min ago → [Name]
```

---

### 4.3 Missing: Multi-Engineer WO Assignment Flow

**Finding:** Firebase `workorders.assignedTo` is a single string field. Firebase `reports.reporters` is an array. The spec does not explain:

- How a manager assigns a second engineer to an existing WO
- How the WO appears in Today's Tasks for multiple engineers simultaneously
- How hours are split when multiple engineers report on the same WO

This is a real operational scenario in the current app (`totalHours` field exists). The refactor spec ignores it.

---

### 4.4 Missing: Closed WO Historical Search

**Finding:** Dispatch section tabs show `All | Pending | In Progress | Waiting Parts | Completed`. Closed WOs (Stage 10) are read-only and appear in Equipment → History and Engineer → History by individual context. But there is no cross-equipment, cross-engineer historical search at the Dispatch level.

If an admin needs to find WO-20250101-0001 from 18 months ago, there is no specified path to locate it.

---

### 4.5 Missing: Business Rules for Edge Cases

The following operational rules are undefined:

| Rule | Why Needed |
|------|-----------|
| WO re-open after Close | Equipment fails again the same day as a verified close — is the old WO re-opened or a new one created? |
| WO delete / cancel | Who can delete a WO? No delete permission is specified in any access control table |
| SLA escalation | If a HIGH priority WO is not dispatched within 2 hours, what triggers? |
| PM overdue handling | If PM is overdue (nextDue < today) and no WO exists, what alert fires? |
| Parts request timeout | If a requisition is unconfirmed for N days, what happens to the WO? |
| Engineer unavailability | If an assigned engineer is absent, who re-assigns? What happens to their WOs? |

---

## 5. Risks

| Risk | Severity | Probability | Details |
|------|---------|------------|---------|
| R-01: Dispatch access control not decided | **High** | Certain | Every engineer can see all WOs or the dispatch tab remains PIN-gated — this is the most consequential security decision in the spec. Must be resolved before implementation begins. |
| R-02: Testing/Verified stages cause confusion | **Medium** | High | Engineers testing equipment see status as "Working" on every other device. If an admin sees a WO in 進行中 for 6 hours, they cannot tell if work is happening or if it's being tested. |
| R-03: Role model requires Firebase schema change | **Medium** | High | Adding roles requires either a new Firebase path or modifying REPORTERS config to be a structured object instead of a string array. Either change must be backward-compatible. |
| R-04: Spare Parts tab has no implementable data source | **Medium** | Certain | The material-equipment association gap means this tab will show either all materials (unhelpful) or nothing (broken) until the association method is decided. |
| R-05: `finishAt` missing from WO schema | **Medium** | Certain | The "Completed today" filter silently shows nothing if this field is absent. |
| R-06: 8-item bottom nav is at the mobile usability limit | **Low** | Medium | Mobile PWA bottom navigation bar best practice is 3–5 items. 8 items (Dashboard, Tasks, Dispatch, Equipment, Engineers, Projects, Analytics, Administration) will be cramped on small screens. Projects (future) may need to be excluded from v1 nav. |
| R-07: localStorage identity is shared on shared devices | **Low** | Medium | Engineers who share a device (common in shop floor environments) will see each other's tasks if one engineer forgets to switch identity. The spec preserves the current behavior but doesn't flag this as a known limitation. |

---

## 6. Product Decisions Required

The following 7 decisions are **blocking** — implementation cannot begin without them. Each is either a security/access decision, an unresolvable implementation ambiguity, or a policy question.

### Decision 1 — Dispatch Access Control

**Decision:** Is the Dispatch section open to all users (no PIN) or does it require the Admin PIN?

| Option A | Option B |
|---------|---------|
| Open to all users — no PIN to access Dispatch | Admin PIN required to view or act on Dispatch |
| All engineers can see all WOs and their status | Only Admin/Manager can dispatch and assign |
| Engineers can create WOs without PIN | Engineers cannot create WOs without Admin approval |
| Matches the spec default | Matches current app behavior |

**Impact:** Determines whether the PIN gate moves to action-level (assign, approve) rather than page-level.

---

### Decision 2 — Role Model Implementation

**Decision:** How are user roles (Engineer / Senior Engineer / Manager / Admin) determined?

| Option A | Option B | Option C |
|---------|---------|---------|
| Flat model: Engineer vs Admin (PIN) only | Role field added to REPORTERS config | New Firebase path `maintenance/engineers` with role field |
| No new data needed | Config change only | Firebase schema change |
| Senior Engineer = Admin for verification purposes | Senior Engineer is a named role | Full RBAC |

**Impact:** Stage 9 Verified permissions, Team View access, and PM schedule edit permissions depend on this.

---

### Decision 3 — Testing Stage Persistence

**Decision:** Should the Testing stage (Stage 7) be persisted in Firebase?

| Option A | Option B |
|---------|---------|
| UI-only (as specced) | Add `status: 測試中` value to Firebase |
| No data model change | Minor data model change; preserves state across reloads |
| Testing state lost on reload | Full auditability of test phase |

---

### Decision 4 — Verified Stage Audit Trail

**Decision:** Should WO verification (Stage 9) create a Firebase record?

| Option A | Option B |
|---------|---------|
| UI-only (as specced) | Add `verifiedBy: string` and `verifiedAt: timestamp` to WO record |
| No data model change; no audit trail | Full verification audit trail; requires data model change |
| No accountability if dispute arises | Enables verification compliance reporting |

**Recommendation:** Option B. Verification without an audit trail undermines the purpose of the quality gate.

---

### Decision 5 — Material-to-Equipment Association

**Decision:** How are spare parts associated with a specific equipment unit?

| Option A | Option B | Option C |
|---------|---------|---------|
| By equipment category tag on material record | Explicit link table `equipment/{id}/parts = [matId...]` | By admin tagging at Equipment → Spare Parts |
| Low precision; shows all materials in same category | Explicit but requires manual linking by admin | Most accurate; most maintenance overhead |

**Impact:** Equipment → Spare Parts tab and Critical Equipment → Section 4 cannot be built without this decision.

---

### Decision 6 — Notification Mechanism

**Decision:** How are engineers notified of new WO assignments, and how are admins notified of new fault reports?

| Option A | Option B | Option C |
|---------|---------|---------|
| No notification — users refresh manually | In-app badge count on nav icons | PWA push notification |
| Zero implementation cost | Low cost; works in browser | Higher cost; requires notification permission |

**Impact:** User journeys 4.2 and 4.4 are broken without a decision here.

---

### Decision 7 — Bottom Navigation: Include Projects in v1?

**Decision:** The Projects section has no current implementation and no Firebase data. Should it appear in the v1 bottom navigation bar?

| Option A | Option B |
|---------|---------|
| Include (greyed out / Coming Soon) | Exclude from v1 navigation entirely |
| Preserves the 8-section architecture in v1 | Reduces nav to 7 items; better mobile usability |
| Engineers see a non-functional tab | Projects can be added when ready |

---

## 7. Cross-Document Consistency Findings

### 7.1 Naming Inconsistencies

| Term | Inconsistent Usage | Document(s) |
|------|--------------------|------------|
| "Admin" vs "Administrator" | Section name is "Administration"; actor is "Admin"; sometimes "Admin / Manager" | All |
| "Today's Tasks" | Both a top-level section AND a tab within Engineer Profile | ARCH, ENGINEER |
| "Fault Report" | A form, a WO type (FR-), and a sub-section of Dispatch | ARCH, DISPATCH, EQUIP |
| "Verified" | Stage 9 label; also used generically ("admin verifies") | DISPATCH |

### 7.2 Contradictions

| Contradiction | Location A | Location B |
|--------------|-----------|-----------|
| Dispatch access: "should not require admin login" | EMS_PRODUCT_ARCHITECTURE.md § 1.1 | EMS_NAVIGATION.md Row 10: "High Risk — access control policy decision needed" |
| Stage 9 actor: "Admin or senior engineer" | EMS_DISPATCH_WORKFLOW.md Stage 9 | No "Senior Engineer" role defined in any access control table |
| Availability "within work hours" | EMS_ENGINEER_WORKSPACE.md § 5.1 | No shift/schedule data exists in Firebase or config |
| `finishAt` written at Stage 8 | EMS_DISPATCH_WORKFLOW.md Stage 8 | Not present in the workorders schema documented in the session |

### 7.3 Orphaned References

| Reference | In Document | Missing Destination |
|-----------|------------|-------------------|
| "Dispatch / Overdue view" | EMS_PRODUCT_ARCHITECTURE.md § 4.3 | No "Overdue" tab in Dispatch tab bar spec |
| "Admin notified" | EMS_PRODUCT_ARCHITECTURE.md § 4.2 | No notification mechanism specified anywhere |
| "bulk-verify" for admin | EMS_DISPATCH_WORKFLOW.md § 7 | No bulk verification screen specified |

---

## 8. Dashboard Completeness Matrix

| Required (WO-EMS-004) | Specified | Widget/Screen | Gap |
|----------------------|-----------|--------------|-----|
| Today's Tasks | ✓ | Today's Dispatch widget + Waiting Assignment widget | Widget content not fully detailed |
| Overdue Work | ✓ | Overdue widget | Navigate destination undefined |
| Critical Equipment | ✓ | Critical Equipment widget (EMS_CRITICAL_EQUIPMENT.md § 4) | Depends on new `criticality` field |
| Emergency Repairs | **✗** | Not specified | No active-emergency widget; HIGH priority active WOs not surfaced |
| MTTR | Partial | Critical Equipment widget only | Not on main Dashboard for all equipment |
| MTBF | Partial | Critical Equipment widget only | Not on main Dashboard; not in Weekly KPI widget |
| KPIs | ✓ | Weekly KPI widget | Content of widget not detailed |

---

## 9. Priority Matrix

### P0 — Must Resolve Before Implementation Starts

| # | Issue | Type | Document |
|---|-------|------|---------|
| P0-01 | Decision 1: Dispatch access control | Security policy decision | NAVIGATION row 10 |
| P0-02 | Decision 2: Role model implementation | Data model decision | All access control tables |
| P0-03 | Decision 5: Material-equipment association | Implementation blocker | EQUIPMENT § 3.8 |

### P1 — Must Resolve Before First Release

| # | Issue | Type | Document |
|---|-------|------|---------|
| P1-01 | Decision 4: Verified stage audit trail | Quality gate integrity | DISPATCH Stage 9 |
| P1-02 | `finishAt` field formally added to WO schema | Data model gap | DISPATCH Stage 8 |
| P1-03 | Emergency Repair widget added to Dashboard | Missing feature | ARCHITECTURE § 2 |
| P1-04 | Overdue tab / widget navigation destination defined | Missing specification | ARCHITECTURE § 4.3 |
| P1-05 | Decision 6: Notification mechanism | UX decision | ARCHITECTURE § 4.2, § 4.4 |

### P2 — Document Before Subsequent Sprints

| # | Issue | Type | Document |
|---|-------|------|---------|
| P2-01 | Decision 3: Testing stage persistence | Optional quality improvement | DISPATCH Stage 7 |
| P2-02 | Multi-engineer WO assignment flow | Missing feature | DISPATCH Stage 3 |
| P2-03 | Closed WO historical search in Dispatch | Missing feature | DISPATCH § 6.1 |
| P2-04 | Dashboard widget content detail (items per widget, date ranges, tap targets) | Underspecification | ARCHITECTURE § 2 |
| P2-05 | Business rules: WO re-open, delete, SLA escalation, PM overdue, parts timeout, engineer absent | Missing rules | None |
| P2-06 | Decision 7: Projects section in v1 nav | UX decision | ARCHITECTURE § 2 |
| P2-07 | Shared-device identity risk documented | Known limitation | ENGINEER § 2.4 |

---

## 10. Recommendation

### Overall Assessment: GO WITH CHANGES

| Dimension | Assessment |
|-----------|-----------|
| Architecture direction | ✓ Sound — workflow-driven is the correct approach |
| Screen coverage | ✓ Complete — all 22 existing screens accounted for |
| Dispatch lifecycle | ✓ Improved — 10-stage lifecycle is a meaningful upgrade |
| Data model | ⚠ Mostly preserved — `finishAt` and role fields need addition |
| Dashboard | ⚠ Mostly complete — emergency repair widget missing |
| Role/permission model | ✗ Undefined — blocking for any permission-gated feature |
| Notifications | ✗ Not specified — blocking for user journey accuracy |
| Material-equipment link | ✗ Not specified — blocking for Spare Parts tab |

### Conditions for GO

The specification may proceed to implementation under these conditions:

1. **P0-01, P0-02, P0-03 resolved first** — Dispatch access control, role model, and material-equipment association decisions must be documented as amendments to the relevant spec files before a single line of implementation code is written.

2. **P1-01 through P1-05 resolved in Sprint WO-EMS-005** — These items do not block the start of implementation but must be resolved before the first public release.

3. **P2 items documented in backlog** — These are known gaps. They should be recorded as product backlog items, not deferred silently.

### Suggested Next Steps

| Step | Work Order | Owner |
|------|-----------|-------|
| Resolve P0 decisions (3 items) | WO-EMS-005: Decision Log | Product Owner |
| Amend spec docs with P0 decisions | WO-EMS-005: Spec Amendment | Engineering |
| Add Emergency Repair widget to Dashboard spec | WO-EMS-005: Spec Amendment | Engineering |
| Begin implementation | WO-EMS-006: Implementation Sprint 1 | Engineering |

---

## 11. Document Versions Reviewed

| Document | Version | Review Status |
|---------|---------|--------------|
| EMS_PRODUCT_ARCHITECTURE.md | 1.0 | Reviewed |
| EMS_NAVIGATION.md | 1.0 | Reviewed |
| EMS_DISPATCH_WORKFLOW.md | 1.0 | Reviewed |
| EMS_ENGINEER_WORKSPACE.md | 1.0 | Reviewed |
| EMS_EQUIPMENT_WORKSPACE.md | 1.0 | Reviewed |
| EMS_CRITICAL_EQUIPMENT.md | 1.0 | Reviewed |
| docs/product/README.md | 1.0 | Reviewed |
