# EMS_DATA_MODEL.md

## BFY EMS — Official Data Model (Frozen)

**Work Order:** WO-EMS-004A
**Status:** FROZEN — Decision Resolved
**Date:** 2026-06-30
**Resolves:** WO-EMS-004 P0-03 (Material-Equipment Association), P1-02 (`finishAt`), P1-01 (Verified audit trail), DEC-003 (Testing stage persistence)

---

## 1. Overview

This document is the authoritative source for the EMS Firebase data model as of the WO-EMS-004A decision freeze. It documents:

1. The **current schema** (as-is from `index.html` analysis)
2. **New fields** added by this decision set
3. **New paths** introduced for Role Model and Equipment-Parts Association
4. The **equipment-parts association** design decision

All additions are backward-compatible. No existing fields are renamed or removed.

---

## 2. Firebase Root

```
ROOT = db.ref("maintenance")
```

All paths below are relative to this root.

---

## 3. Current Schema (Preserved As-Is)

### 3.1 `workorders/{id}`

| Field | Type | Description | Status |
|-------|------|-------------|--------|
| `id` | string | WO identifier (WO-YYYYMMDD-NNN or FR-YYYYMMDD-NNN) | Existing |
| `source` | string | Work type: 故障叫修 / 保養 / 維修 / 優化 | Existing |
| `equip` | string | Equipment ID | Existing |
| `equipName` | string | Equipment display name | Existing |
| `item` | string | Work description | Existing |
| `priority` | string | 高 / 中 / 低 | Existing |
| `assignedTo` | string | Engineer name (single; see note) | Existing |
| `status` | string | 待派工 / 已派工 / 進行中 / 待料 / 已完成 | Existing |
| `createdAt` | number | Unix ms timestamp | Existing |
| `due` | string | Due date (YYYY-MM-DD) | Existing |
| `from` | string | Origin: engineer / fault / admin / plan | Existing |
| `symptom` | string | Fault symptom description | Existing |
| `urgency` | string | 緊急 / 普通 | Existing |
| `reporter` | string | Person who reported the issue | Existing |
| `area` | string | Area/zone code | Existing |
| `dept` | string | Department | Existing |

### 3.2 `reports/{id}`

| Field | Type | Description | Status |
|-------|------|-------------|--------|
| `woKey` | string | Key of parent work order | Existing |
| `equip` | string | Equipment ID | Existing |
| `equipName` | string | Equipment name | Existing |
| `reporter` | string | Primary engineer | Existing |
| `reporters` | string[] | All engineers on job | Existing |
| `progress` | string | 進行中 / 已完成 / 待料 | Existing |
| `hours` | number | Work hours for this report | Existing |
| `totalHours` | number | Cumulative hours across all reports | Existing |
| `materials` | object[] | `[{id, name, qty}]` — materials used | Existing |
| `note` | string | Root cause / work notes | Existing |
| `source` | string | Work type (mirrors WO source) | Existing |
| `createdAt` | number | Unix ms timestamp | Existing |

### 3.3 `equipment/{id}`

| Field | Type | Description | Status |
|-------|------|-------------|--------|
| `id` | string | Equipment identifier | Existing |
| `name` | string | Display name | Existing |
| `line` | string | Production line / area | Existing |
| `status` | string | Active / inactive | Existing |

### 3.4 `material/{id}`

| Field | Type | Description | Status |
|-------|------|-------------|--------|
| `id` | string | Material identifier | Existing |
| `name` | string | Display name | Existing |
| `cat` | string | Category | Existing |
| `brand` | string | Brand | Existing |
| `model` | string | Model number | Existing |
| `unit` | string | Unit of measure | Existing |
| `price` | number | Unit cost | Existing |

### 3.5 `plans/{planId}`

| Field | Type | Description | Status |
|-------|------|-------------|--------|
| `planId` | string | Plan identifier | Existing |
| `months` | number[] | Interval months (e.g., [3] = every 3 months) | Existing |
| `lastDone` | string | Date of last PM (YYYY-MM-DD) | Existing |
| `nextDue` | string | Date of next PM (YYYY-MM-DD) | Existing |

### 3.6 `requisitions/{id}`

| Field | Type | Description | Status |
|-------|------|-------------|--------|
| `woKey` | string | Parent WO key | Existing |
| `reporter` | string | Engineer who requested | Existing |
| `reporters` | string[] | All engineers on job | Existing |
| `materials` | object[] | `[{id, name, qty}]` | Existing |
| `confirmedAt` | number | Confirmation timestamp (null if pending) | Existing |
| `confirmedBy` | string | Name of confirmer (null if pending) | Existing |

### 3.7 `areas/{id}`

| Field | Type | Description | Status |
|-------|------|-------------|--------|
| `code` | string | Area code | Existing |
| `name` | string | Area name | Existing |
| `dept` | string | Department | Existing |

### 3.8 `config`

| Field | Type | Description | Status |
|-------|------|-------------|--------|
| `laborRate` | number | Labor cost per hour | Existing |
| `pin` | string | Admin PIN | Existing |

---

## 4. New Fields — Existing Paths

These fields are **added to existing records**. No existing fields are modified.

### 4.1 `workorders/{id}` — New Fields

| Field | Type | Description | Written At | Resolves |
|-------|------|-------------|-----------|---------|
| `finishAt` | number | Unix ms timestamp when WO status set to 已完成 | Stage 8 — report submission | P1-02 |
| `testingAt` | number | Unix ms timestamp when engineer marks Testing | Stage 7 — "Mark as Testing" button | DEC-003 |
| `verifiedAt` | number | Unix ms timestamp when WO verified | Stage 9 — "Verify" button | P1-01 |
| `verifiedBy` | string | Name of person who verified | Stage 9 — "Verify" button | P1-01 |
| `closedAt` | number | Unix ms timestamp when WO closed | Stage 10 — "Close" button | New |
| `closedBy` | string | Name of person who closed | Stage 10 — "Close" button | New |

**Combined updated `workorders/{id}` record:**

```
id, source, equip, equipName, item, priority, assignedTo, status,
createdAt, due, from, symptom, urgency, reporter, area, dept,
+ finishAt, testingAt, verifiedAt, verifiedBy, closedAt, closedBy
```

**Null handling:** `testingAt`, `verifiedAt`, `verifiedBy`, `closedAt`, `closedBy` are `null` until the corresponding stage is reached. `finishAt` is null until Stage 8.

### 4.2 `equipment/{id}` — New Fields

| Field | Type | Description | Resolves |
|-------|------|-------------|---------|
| `criticality` | string | C / I / N — criticality classification | EMS_CRITICAL_EQUIPMENT.md § 2.3 |
| `criticalityReviewDate` | string | YYYY-MM-DD of last review | EMS_CRITICAL_EQUIPMENT.md § 3.1 |
| `criticalityReviewedBy` | string | Name of reviewer | EMS_CRITICAL_EQUIPMENT.md § 3.1 |
| `criticalityNote` | string | Justification text | EMS_CRITICAL_EQUIPMENT.md § 3.1 |

**Default value:** Equipment without a `criticality` field is treated as `N` (Normal) by the UI.

---

## 5. New Firebase Paths

### 5.1 `maintenance/roster/{autoId}` — Engineer Roster with Roles

**Purpose:** Stores structured engineer profiles with role assignments. Resolves P0-02 (Role Model).

**Record structure:**

```json
{
  "name":    "string — display name, matches REPORTERS constant",
  "role":    "string — ADM / MGR / ENG / PLR / WHS / EQO / VND / RDO",
  "active":  "boolean",
  "addedAt": "number — Unix ms",
  "addedBy": "string — admin name"
}
```

**Query patterns:**
- Get all active engineers: `roster.orderByChild("active").equalTo(true)`
- Get role for a name: `roster.orderByChild("name").equalTo(name)` → read `.role`

---

### 5.2 `maintenance/equipmentParts/{autoId}` — Equipment-Parts Association

**Purpose:** Defines the official many-to-many relationship between Equipment and Material. Resolves P0-03.

**Record structure:**

```json
{
  "equipId":      "string — equipment ID (matches equipment/{id})",
  "matId":        "string — material ID (matches material/{id})",
  "minStock":     "number — minimum required quantity",
  "leadTimeDays": "number — days to procure if stock reaches zero",
  "moq":          "number — minimum order quantity",
  "altParts":     ["string", "..."] ,
  "notes":        "string — optional usage notes",
  "addedBy":      "string — who created this association",
  "addedAt":      "number — Unix ms"
}
```

**Query patterns:**
- All parts for equipment: `equipmentParts.orderByChild("equipId").equalTo(equipId)`
- All equipment using a part: `equipmentParts.orderByChild("matId").equalTo(matId)`

---

### 5.3 `maintenance/auditLog/{autoId}` — Structured Audit Log

**Purpose:** Stores structured event records. The current app already has an audit tab (`admAuditLog`) — this formalizes its schema. See [EMS_AUDIT_MODEL.md](EMS_AUDIT_MODEL.md) for full specification.

---

## 6. Equipment-Parts Association Design Decision

**Decision ID:** DEC-003 (Material-Equipment Association)
**Resolved:** Yes

### 6.1 Relationship Type

| Relationship | Description | Rationale |
|-------------|-------------|-----------|
| Equipment → Parts | **One-to-Many** | One piece of equipment uses multiple spare parts |
| Part → Equipment | **One-to-Many** | One part type can be used in multiple equipment units |
| Combined | **Many-to-Many** | Requires an association/junction record |

**Official model: Many-to-Many via `maintenance/equipmentParts`**

### 6.2 Why Not a Sub-Collection on Equipment?

Option considered: `equipment/{id}/parts/{autoId}` — embedding parts list inside equipment record.

**Rejected because:**
- Firebase Realtime Database reads an entire node when you read any child. Embedding all parts data inside equipment means every equipment fetch includes parts data, even when parts are not needed.
- The reverse query (which equipment uses this part?) requires a full scan of all equipment records.
- The junction path `equipmentParts` allows both directions to be queried efficiently with `orderByChild`.

### 6.3 Backward Compatibility

The current `maintenance/material` path stores all materials with no equipment link. Adding `maintenance/equipmentParts` does not alter or delete any existing material records. The association is additive.

Until `equipmentParts` records are created for a piece of equipment, the Equipment → Spare Parts tab shows a "No parts associated yet — tap to add" placeholder.

---

## 7. Stock Quantity Decision

**Decision:** Add `stock` field to `maintenance/material/{id}`.

**Field:**

| Path | Field | Type | Description |
|------|-------|------|-------------|
| `material/{id}` | `stock` | number | Current quantity on hand |
| `material/{id}` | `stockUpdatedAt` | number | Unix ms of last stock update |
| `material/{id}` | `stockUpdatedBy` | string | Name of last person to update stock |

**Why not infer from requisitions:** Requisition history can only tell you what was requested and confirmed, not what was received, consumed outside a WO, or physically counted. A direct `stock` field edited by Warehouse (WHS role) is the correct approach.

**Stock update rules:**
- WHS role can edit stock directly (Administration → Inventory Management)
- Stock is decremented automatically when a requisition is confirmed (`confirmedAt` set)
- Stock is not automatically incremented on receipt — WHS manually updates on physical receipt

---

## 8. Full Schema Summary (Post-Decision)

```
maintenance/
├── workorders/{id}
│   ├── [all existing fields]
│   ├── finishAt          ← NEW
│   ├── testingAt         ← NEW
│   ├── verifiedAt        ← NEW
│   ├── verifiedBy        ← NEW
│   ├── closedAt          ← NEW
│   └── closedBy          ← NEW
│
├── reports/{id}          [unchanged]
│
├── equipment/{id}
│   ├── [all existing fields]
│   ├── criticality       ← NEW (C/I/N)
│   ├── criticalityReviewDate   ← NEW
│   ├── criticalityReviewedBy   ← NEW
│   └── criticalityNote   ← NEW
│
├── material/{id}
│   ├── [all existing fields]
│   ├── stock             ← NEW
│   ├── stockUpdatedAt    ← NEW
│   └── stockUpdatedBy    ← NEW
│
├── plans/{planId}        [unchanged]
├── requisitions/{id}     [unchanged]
├── areas/{id}            [unchanged]
│
├── config
│   ├── laborRate         [unchanged]
│   └── pin               [unchanged]
│
├── roster/{id}           ← NEW PATH
│   ├── name
│   ├── role
│   ├── active
│   ├── addedAt
│   └── addedBy
│
├── equipmentParts/{id}   ← NEW PATH
│   ├── equipId
│   ├── matId
│   ├── minStock
│   ├── leadTimeDays
│   ├── moq
│   ├── altParts[]
│   ├── notes
│   ├── addedBy
│   └── addedAt
│
└── auditLog/{id}         ← NEW STRUCTURED PATH
    └── [see EMS_AUDIT_MODEL.md]
```
