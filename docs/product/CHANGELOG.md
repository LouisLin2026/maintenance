# CHANGELOG — EMS Product Specification

---

## [WO-EMS-003] — 2026-06-30

**Sprint:** WO-EMS-003 — EMS Dashboard Refactor
**Type:** Product Specification
**Branch:** `feature/dashboard-refactor-v1`
**Status:** Complete — Awaiting Review

### Added

- `docs/product/EMS_PRODUCT_ARCHITECTURE.md` — Product vision and complete navigation architecture
  - Workflow-Driven Architecture rationale (replaces role-based entry)
  - 8-section navigation tree with Firebase data source annotations
  - Screen hierarchy (Level 0–3)
  - 4 user journey flows (Engineer: work report, fault report; Manager: morning overview, dispatch)
  - Complete current-screen → new-section workflow mapping table
  - Future module integration points

- `docs/product/EMS_NAVIGATION.md` — Screen-by-screen migration table
  - 22 current screens/routes/tabs fully accounted for
  - New location, reason for move, and migration risk rating per screen
  - High / Medium / Low risk summary
  - 8 new navigation items with no current equivalent
  - Routes that must be preserved (QR code backward compatibility)

- `docs/product/EMS_ENGINEER_WORKSPACE.md` — Engineer workspace specification
  - Today's Tasks section: screen layout, data sources, identity selection, My/Team toggle
  - Engineers section home: team roster with availability indicators
  - Engineer Profile workspace: 8 tabs fully specified
    - Today's Tasks, Active Work Orders, Daily Report, Weekly Report
    - Skills (v1 placeholder), KPI, Working Hours, History
  - KPI calculation formulas from existing Firebase data
  - Access control matrix (engineer vs. admin)

- `docs/product/EMS_EQUIPMENT_WORKSPACE.md` — Equipment Center specification
  - Equipment Overview screen: list/grid with status indicators
  - Equipment detail header
  - 8 tabs specified: Overview, PM Schedule, History, QR Code, Documents, Critical Points, Spare Parts, PLC Backup
  - Documents / Critical Points / PLC tabs marked as v1 placeholder (future implementation)
  - Cross-equipment views: PM Calendar, History List
  - Note on stock quantity field gap in current Firebase schema
  - Access control matrix

- `docs/product/EMS_DISPATCH_WORKFLOW.md` — Dispatch workflow specification
  - 10-stage workflow: Request → Review → Assign → Accepted → Working → Waiting Parts → Testing → Completed → Verified → Closed
  - Current-to-new status mapping with implementation notes
  - Stage-by-stage screen mapping (current code reference → new screen)
  - Firebase records created/updated at each stage
  - Work order types and priority definitions
  - Dispatch section UI: tab bar, WO card, WO detail screen wireframes
  - Migration risk assessment for 5 key risks

- `docs/product/EMS_CRITICAL_EQUIPMENT.md` — Critical equipment classification specification
  - 3-tier classification: Critical (C) / Important (I) / Normal (N) with criteria
  - Critical Points tab: 7 sections (Classification, KPIs, Weak Points, Spare Parts, Recovery SOP, PLC Info, Visual Reference)
  - MTTR/MTBF calculation formulas from existing Firebase data
  - Dashboard Critical Equipment widget specification
  - New Firebase fields required for full implementation (12 new fields/sub-paths)
  - v1 minimum viable scope identified (criticality field + badge + widget + calculated KPIs)

- `docs/product/README.md` — Sprint overview
- `docs/product/CHANGELOG.md` — This file

### Not Changed

- `index.html` — No modifications to app logic, Firebase paths, or UI
- `service-worker.js` — No modifications (per WO constraint)
- `app.js` — No modifications
- Firebase data model — No structural changes (new fields identified; not yet applied)
- Existing QR code routes — Backward compatibility preserved in specification

### Scope Boundary

This sprint is **specification only**. No implementation has been performed. All documents in `docs/product/` are product requirements for the next implementation sprint.

---

*Prior changelog entries: None — this is the first `docs/product/` entry.*
