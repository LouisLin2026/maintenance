# EMS_CRITICAL_EQUIPMENT.md

## BFY EMS — Critical Equipment Specification

**Work Order:** WO-EMS-003
**Version:** 1.0
**Status:** Specification — Awaiting Review
**Date:** 2026-06-30

---

## 1. Purpose

Critical Equipment is a classification layer within the Equipment module that identifies machinery whose failure causes disproportionate production or safety impact. This specification defines how critical equipment is identified, what additional data is captured for it, and how the UI surfaces criticality information.

This specification covers the **Equipment → Critical Points tab** and the Dashboard **Critical Equipment widget**, both flagged as future implementation in [EMS_EQUIPMENT_WORKSPACE.md](EMS_EQUIPMENT_WORKSPACE.md).

---

## 2. Critical Level Classification

### 2.1 Levels

| Level | Code | Definition | Example Impact |
|-------|------|-----------|----------------|
| Critical | C | Failure stops an entire production line or creates safety hazard | Main conveyor motor; primary hydraulic unit |
| Important | I | Failure degrades production but does not stop it | Secondary pump; standby compressor |
| Normal | N | Failure has no immediate production impact | Support equipment; tools |

### 2.2 Classification Criteria

Equipment is classified as **Critical (C)** if any of the following are true:
- Failure stops ≥1 production line
- Failure creates a direct safety risk (pressure vessel, lifting equipment, electrical panels)
- No standby unit exists and MTTR > 4 hours
- Lead time for critical spare parts > 30 days

Equipment is classified as **Important (I)** if:
- Failure reduces production capacity by ≥20%
- Standby unit exists but switchover requires >2 hours
- MTTR is consistently >8 hours

All other equipment is **Normal (N)** by default.

### 2.3 Data Field

**Firebase path:** `maintenance/equipment/{id}/criticality`
**Values:** `C` | `I` | `N`

This field does not currently exist in the Firebase schema. Implementation of the Critical Equipment feature requires adding this field to the equipment master.

---

## 3. Critical Points Tab — Data Specification

The Critical Points tab appears in the Equipment detail screen for all equipment, but its content is most relevant for Critical and Important equipment. For Normal equipment, the tab shows a brief statement confirming Normal classification.

### 3.1 Tab Sections

**Section 1: Classification**

| Field | Description | Data Source |
|-------|-------------|------------|
| Critical Level | C / I / N | `equipment/{id}/criticality` |
| Classification Date | When criticality was last reviewed | `equipment/{id}/criticalityReviewDate` |
| Reviewed By | Person who last reviewed classification | `equipment/{id}/criticalityReviewedBy` |
| Justification | Why this level was assigned | `equipment/{id}/criticalityNote` |

---

**Section 2: Performance Indicators**

| Field | Description | Calculation |
|-------|-------------|------------|
| MTTR (Mean Time to Repair) | Average repair hours for this equipment | `SUM(reports.hours WHERE equip == id) / COUNT(completed WOs)` |
| MTBF (Mean Time Between Failures) | Average days between fault occurrences | `Total days in period / COUNT(fault WOs WHERE equip == id)` |
| Fault Frequency | Number of fault WOs in last 12 months | Count of FR- WOs for this equipment |
| PM Compliance | PM WOs completed on schedule | PM WOs completed on/before due date / total PM WOs |
| Downtime (last 12 months) | Total hours equipment was under repair | `SUM(hours)` from reports where equip == id and WO is not PM |

**Data sources:**
- `maintenance/workorders` filtered by `equip == id`
- `maintenance/reports` filtered by `equip == id`

---

**Section 3: Known Weak Points**

Free-text entries documenting recurring failure modes or structural vulnerabilities observed by engineers.

| Field | Description |
|-------|-------------|
| Weak Point Description | What fails, where, and why |
| First Observed | Date this weak point was documented |
| Frequency | How often this failure mode occurs |
| Recommended Action | Preventive measure or design change |
| Status | Open / In Progress / Resolved |

**Firebase path:** `maintenance/equipment/{id}/weakPoints/{id}` (new sub-path; not in current schema)

---

**Section 4: Spare Parts Inventory**

Critical and Important equipment must maintain minimum spare parts stock. This section shows whether minimum stock levels are met.

| Field | Description |
|-------|-------------|
| Part Name | Material name from `maintenance/material` |
| Part ID | Material ID |
| Min Required | Minimum quantity to maintain |
| Lead Time | Days to procure if stock is zero |
| MOQ | Minimum order quantity from supplier |
| Alternative Parts | Compatible substitute parts |
| Current Stock | Current quantity on hand |
| Status | OK / Low / Critical |

**Firebase paths:**
- Materials: `maintenance/material/{id}`
- Min stock / lead time / MOQ: new fields to be added to `maintenance/material/{id}` or a new `maintenance/equipment/{id}/criticalParts` sub-path
- Current stock: implementation decision — see EMS_EQUIPMENT_WORKSPACE.md note on stock levels

---

**Section 5: Recovery SOP**

Standard procedure for rapid recovery of this equipment after failure.

| Field | Description |
|-------|-------------|
| SOP Title | Name of this recovery procedure |
| Steps | Numbered steps for recovery |
| Estimated Recovery Time | Expected time to restore to operation |
| Required Tools | Tools and test equipment needed |
| Safety Precautions | Safety steps before and during repair |
| Escalation Contact | Who to call if standard SOP fails |
| Last Updated | Date of last revision |
| Updated By | Person responsible for SOP content |

**Firebase path:** `maintenance/equipment/{id}/recoverySOP` (new sub-path; not in current schema)

---

**Section 6: PLC Information**

For PLC-controlled equipment, backup and version tracking.

| Field | Description |
|-------|-------------|
| PLC Backup Status | Has backup / No backup / Unknown |
| Backup Version | Program version identifier |
| Last Backup Date | When the last backup was taken |
| Backup Location | Where backup file is stored |
| Program Version History | Chronological list of program changes |
| Software Notes | Special configuration or known issues |

**Firebase path:** `maintenance/equipment/{id}/plcInfo` (new sub-path; not in current schema)

---

**Section 7: Visual Reference**

Links or references to photos, videos, and manuals for this equipment.

| Field | Description |
|-------|-------------|
| Equipment Photo | Photo(s) of the equipment |
| Critical Point Photo | Photo highlighting weak points or inspection areas |
| Fault Reference Video | Video showing typical failure symptoms |
| Operating Manual | Link to manufacturer manual |
| Wiring Diagram | Link to electrical drawing |

**Firebase path:** `maintenance/equipment/{id}/documents` (new sub-path; not in current schema)

---

## 4. Dashboard Critical Equipment Widget

### 4.1 Widget Purpose

The Dashboard shows a summary of Critical and Important equipment that requires attention today.

### 4.2 Widget Content

```
┌──────────────────────────────────────────────────────┐
│  Critical Equipment                    [2 alerts]    │
├──────────────────────────────────────────────────────┤
│  ⚠ EQ-002  Line B Hydraulic Pump  [C]               │
│    Active fault WO │ MTTR: 3.2h avg                 │
│                                                      │
│  ● EQ-007  Cooling Tower Unit 2   [C]               │
│    PM due in 2 days │ Last fault: 45 days ago        │
│                                                      │
│  [View All Critical Equipment]                       │
└──────────────────────────────────────────────────────┘
```

**Alert conditions (equipment is shown in widget if):**
- Criticality == C AND has open fault WO
- Criticality == C AND PM due within 7 days
- Criticality == I AND has open HIGH priority WO
- Any equipment with criticality == C AND MTTR trending up month-over-month

---

## 5. Critical Equipment List View

Accessible from: Dashboard widget → "View All Critical Equipment" or Equipment section → filter by Critical.

```
┌──────────────────────────────────────────────────────┐
│  Critical Equipment                    [Sort: Risk ▾] │
├──────────────────────────────────────────────────────┤
│  Level C Equipment:                                  │
│                                                      │
│  ⚠ EQ-002  Line B Hydraulic Pump                    │
│    MTTR: 3.2h │ MTBF: 42 days │ Faults (12m): 8    │
│    Open WOs: 2 │ PM Status: Due                     │
│                                                      │
│  ● EQ-001  Line A Conveyor Motor                    │
│    MTTR: 1.8h │ MTBF: 68 days │ Faults (12m): 5    │
│    Open WOs: 0 │ PM Status: OK                      │
│  ─────────────────────────────────────────────────   │
│  Level I Equipment:                                  │
│  ● EQ-003  Cooling Tower Fan Unit 1                 │
│  ● EQ-004  Air Compressor Unit 2                    │
└──────────────────────────────────────────────────────┘
```

---

## 6. Implementation Notes

### 6.1 New Firebase Fields Required

The following fields do not exist in the current Firebase schema and must be added to implement Critical Equipment features:

| Firebase Path | New Field | Type | Required For |
|-------------|-----------|------|-------------|
| `equipment/{id}` | `criticality` | string (C/I/N) | Criticality level; criticality widget |
| `equipment/{id}` | `criticalityReviewDate` | timestamp | Classification section |
| `equipment/{id}` | `criticalityReviewedBy` | string | Classification section |
| `equipment/{id}` | `criticalityNote` | string | Classification section |
| `equipment/{id}/weakPoints` | sub-collection | object | Section 3 |
| `equipment/{id}/recoverySOP` | sub-collection | object | Section 5 |
| `equipment/{id}/plcInfo` | sub-collection | object | Section 6 |
| `equipment/{id}/documents` | sub-collection | object | Section 7 |
| `material/{id}` | `minStock` | number | Spare Parts section |
| `material/{id}` | `leadTimeDays` | number | Spare Parts section |
| `material/{id}` | `moq` | number | Spare Parts section |
| `material/{id}` | `altParts` | string[] | Spare Parts section |

### 6.2 v1 Implementation Scope

For the first implementation sprint, the minimum viable Critical Equipment feature is:

1. Add `criticality` field to equipment master (C/I/N)
2. Display criticality badge on Equipment Overview and Equipment detail header
3. Dashboard Critical Equipment widget (criticality == C, shows open WOs and PM status)
4. Critical Points tab shows MTTR, MTBF, fault frequency (calculated from existing data)

Sections 3 (Weak Points), 4 (Spare Parts min stock), 5 (Recovery SOP), 6 (PLC Info), 7 (Visual Reference) require new Firebase sub-paths and should be implemented in subsequent sprints.

### 6.3 No Company-Specific Data in Spec

This specification uses placeholder equipment IDs (EQ-001, EQ-002, etc.) and generic descriptions. No real BFY equipment names, line configurations, or operational data appears in this document.
