# EMS_AUDIT_MODEL.md

## BFY EMS — Audit Trail Model

**Work Order:** WO-EMS-004A
**Status:** FROZEN — Decision Resolved
**Date:** 2026-06-30
**Resolves:** WO-EMS-004 P1-01 (Verified Stage Audit Trail)

---

## 1. Decision

**Decision ID:** DEC-005
**Resolved:** Yes

**Audit Trail:** Every state-changing action in the EMS writes a structured audit record to `maintenance/auditLog`. This includes all WO lifecycle transitions, report submissions, inventory changes, and configuration changes. The audit log is append-only. Records are never deleted.

**Verification audit trail (P1-01):** Stage 9 (Verified) writes `verifiedBy` and `verifiedAt` directly to the WO record (see [EMS_DATA_MODEL.md](EMS_DATA_MODEL.md) § 4.1). It also writes a structured `AUDIT_WO_VERIFIED` event to the audit log. Both are required.

---

## 2. Audit Record Structure

**Path:** `maintenance/auditLog/{autoId}`

```json
{
  "timestamp":  1751299200000,
  "actor":      "string — name of person who performed the action",
  "actorRole":  "string — role ID (ADM/MGR/ENG/...) at time of action",
  "action":     "string — event type (see § 3)",
  "entityType": "string — WO / REPORT / EQUIPMENT / MATERIAL / REQUISITION / CONFIG / ROSTER",
  "entityId":   "string — ID of the affected record",
  "before":     "string — previous value (for status changes; null for creates)",
  "after":      "string — new value (for status changes; full record summary for creates)",
  "details":    "string — human-readable summary",
  "sessionId":  "string — browser session identifier (for grouped activity; optional in MVP)"
}
```

| Field | Type | Required | Notes |
|-------|------|----------|-------|
| `timestamp` | number (ms) | Yes | Server timestamp preferred; client timestamp acceptable in MVP |
| `actor` | string | Yes | Name from identity selection or "system" for automated events |
| `actorRole` | string | Yes | Role at time of action; resolved from roster or "ENG" default |
| `action` | string | Yes | See Section 3 — Event Catalogue |
| `entityType` | string | Yes | Category of affected data |
| `entityId` | string | Yes | The specific record ID |
| `before` | string | No | Previous value for status transitions; null for creates |
| `after` | string | No | New value; for creates, use a short summary |
| `details` | string | No | Human-readable plain-text description |
| `sessionId` | string | No | Optional in MVP; useful for grouping a session's actions |

---

## 3. Event Catalogue

### 3.1 Work Order Events

| Event Code | Trigger | `before` | `after` | Example `details` |
|-----------|---------|---------|--------|-------------------|
| `WO_CREATED` | New WO submitted | null | WO ID + type | "WO-20260630-1234 created (故障叫修) for EQ-001" |
| `WO_ASSIGNED` | Engineer assigned to WO | previous `assignedTo` or null | engineer name | "Assigned to [Name]" |
| `WO_ACCEPTED` | Engineer accepts WO (Stage 4) | 已派工 | 進行中 | "[Name] accepted WO-..." |
| `WO_STATUS_CHANGED` | Any status field change | old status | new status | "Status: 進行中 → 待料" |
| `WO_TESTING_MARKED` | Stage 7 triggered | null | timestamp | "[Name] marked as Testing" |
| `WO_COMPLETED` | Stage 8 — report submitted | 進行中 | 已完成 | "[Name] submitted work report; hours: 3.5h" |
| `WO_VERIFIED` | Stage 9 — admin verifies | null | verifiedBy name | "[MGR Name] verified WO-..." |
| `WO_CLOSED` | Stage 10 — WO closed | 已完成/verified | closed | "[ADM Name] closed WO-..." |
| `WO_REJECTED` | WO rejected at Stage 2 | 待派工 | rejected | "[MGR Name] rejected: duplicate" |
| `WO_DELETED` | ADM deletes WO | full WO summary | null | "WO deleted by [ADM Name]" |
| `WO_REOPENED` | ADM re-opens closed WO | closed | 進行中 | "WO re-opened by [ADM Name]" |
| `WO_PRIORITY_CHANGED` | Priority edited | old priority | new priority | "Priority: 中 → 高" |

### 3.2 Work Report Events

| Event Code | Trigger | Notes |
|-----------|---------|-------|
| `REPORT_SUBMITTED` | Report written at Stage 8 | Records report ID, hours, engineer |
| `REPORT_EDITED` | Report modified within 24h window | Records field changed, old/new value |

### 3.3 Parts / Inventory Events

| Event Code | Trigger | Notes |
|-----------|---------|-------|
| `REQUISITION_CREATED` | Parts requested (Stage 6) | Records WO key, materials list |
| `REQUISITION_CONFIRMED` | WHS confirms requisition | Records confirmer name, materials |
| `STOCK_ADJUSTED` | WHS adjusts stock quantity | Records material ID, old qty, new qty |

### 3.4 Equipment Events

| Event Code | Trigger | Notes |
|-----------|---------|-------|
| `EQUIPMENT_CREATED` | New equipment added | Records equipment ID and name |
| `EQUIPMENT_EDITED` | Equipment field changed | Records field, old/new value |
| `EQUIPMENT_CRITICALITY_SET` | Criticality classification assigned | Records level (C/I/N), reviewer |

### 3.5 Configuration Events

| Event Code | Trigger | Notes |
|-----------|---------|-------|
| `CONFIG_PIN_CHANGED` | Admin PIN changed | Records only that it changed; does NOT record old or new PIN value |
| `CONFIG_LABOR_RATE_CHANGED` | Labor rate changed | Records old and new rate |
| `ROSTER_ADDED` | New roster record created | Records name and role |
| `ROSTER_ROLE_CHANGED` | Role assignment changed | Records name, old role, new role |
| `ROSTER_DEACTIVATED` | Engineer deactivated | Records name |

---

## 4. Audit Log Access

| Role | Can Read | Can Delete |
|------|----------|-----------|
| ADM | Full audit log | Never (append-only) |
| MGR | Full audit log | Never |
| ENG | Own actions only | Never |
| All others | None | Never |

The audit log is displayed in Administration → Audit Log. The current `admAuditLog` tab becomes this screen.

---

## 5. Retention Policy

| Policy | Rule |
|--------|------|
| Retention period | Indefinite — records are never deleted |
| Archiving | Future sprint: records older than 3 years may be exported and removed from live Firebase |
| Minimum retention | All WO lifecycle events must be retained for the life of the WO plus 2 years |

---

## 6. Existing Audit Log Migration

The current app writes unstructured string entries to the audit log. The new schema adds structure but does not delete old entries. During the refactor:

- Old unstructured entries remain in `maintenance/auditLog` as-is.
- New structured entries use the schema defined in § 2.
- The Audit Log UI should render both: structured records use the defined schema; unstructured records display as raw text.
- No migration of old records is required.

---

## 7. Privacy Rules

| Rule | Rationale |
|------|-----------|
| Admin PIN is never logged | Security — the PIN value must not appear in audit records |
| Financial costs are not included in audit record `details` | Sensitivity — cost data is in Analytics, not in the audit trail |
| Personal identifiers beyond name are not logged | The app uses display names only; no email, phone, or employee ID in scope |
