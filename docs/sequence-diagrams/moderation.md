# Moderation – Sequence Diagrams (Checkpoint 3, Member 4)

Scenarios: **Submit Report**, **Resolve Report**, **Block User**.
Related user stories: US-16 (report a student), US-17 (review and resolve reports), US-18 (suspend an account).

Participants in every diagram: Actor (Student / Admin) → Frontend → REST Controller → Service → PostgreSQL.
API contract: [`docs/api/moderation.yaml`](../api/moderation.yaml). Block User reuses the Identity endpoint from [`docs/api/identity.yaml`](../api/identity.yaml).

| # | Scenario | Endpoint | Source | Diagram |
|---|----------|----------|--------|---------|
| 01 | Submit Report | `POST /api/v1/reports` | [puml](sources/moderation_01_submit_report.puml) | [svg](diagrams/moderation_01_submit_report.svg) |
| 02 | Resolve Report | `GET /api/v1/admin/reports`, `PATCH /api/v1/admin/reports/{reportId}` | [puml](sources/moderation_02_resolve_report.puml) | [svg](diagrams/moderation_02_resolve_report.svg) |
| 03 | Block User | `PATCH /api/v1/admin/users/{userId}/status` (Identity) | [puml](sources/moderation_03_block_user.puml) | [svg](diagrams/moderation_03_block_user.svg) |

## Shared rules

- Every request carries `Authorization: Bearer <access_token>`. JWT claims: `sub` (userId, UUID), `role` (`STUDENT` | `ADMIN`), `exp`.
- `reporterId` and `reviewedById` are taken from `sub`, never from the request body.
- IDs are UUIDs, timestamps are ISO 8601 in UTC, statuses are uppercase enums.
- Errors use the shared format `{ "error": { "code", "message", "timestamp" } }`.
- The `alt` branches are ordered as in the lecture: identity (401) → permission (403) → existence (404) → data (422) → state (409) → server (500).

## 01 Submit Report (US-16)

**Standard path.** The student sends `reportedUserId` and `reason`. The service checks the token, takes the reporter from `sub`, confirms the reported student exists, stores the report as `PENDING` and returns `201 Created` with the `Report`. Reporting never changes the reported account's status.

**Abnormal paths.**
- `401` – missing, invalid or expired token.
- `403` – caller is suspended or is not a `STUDENT`.
- `404 USER_NOT_FOUND` – the reported student does not exist.
- `422 VALIDATION_ERROR` – empty reason or the student reports themself.
- `409 DUPLICATE_REPORT` – the same reporter already has a `PENDING` report on this student. This also makes a retried POST safe.
- `500` – database error; nothing is saved.

## 02 Resolve Report (US-17)

**Standard path.** The admin lists reports with `GET /api/v1/admin/reports?status=PENDING&page=0&size=20` (`200`, `ReportPage`), then sends `PATCH /api/v1/admin/reports/{reportId}` with `status: "RESOLVED"` and a written `outcome`. The service stores the outcome, `reviewedById` and `resolvedAt` and returns `200 OK` with the `Report`. Resolving a report does not suspend the account.

**Abnormal paths.**
- `401` – missing, invalid or expired token.
- `403` – caller is not an `ADMIN`.
- `404 REPORT_NOT_FOUND` – unknown `reportId`.
- `422 VALIDATION_ERROR` – empty outcome or a status other than `RESOLVED`.
- `409 STATUS_CONFLICT` – the report is already `RESOLVED`, which is a terminal state in the Report state diagram.
- `500` – database error; the update is rolled back.

## 03 Block User (US-17 + US-18)

Blocking a reported user means the admin suspends the account. The diagram uses the same endpoint and the same `IdentityService` as the Identity module's Account Suspension diagram, so the contract is defined once (`AccountStatusUpdate` request, `Account` response).

**Standard path.** From the report, the admin sends `PATCH /api/v1/admin/users/{userId}/status` with `status: "SUSPENDED"` and a `reason`. In one transaction the account becomes `SUSPENDED`, an `AccountStatusChange` audit record is saved and the student's sessions are revoked. The response is `200 OK` with the `Account`. The reported student's old token then gets `401`, login returns `403`, and suspended students are excluded from matches. The report itself is resolved separately (diagram 02).

**Abnormal paths.**
- `401` – missing, invalid or expired token.
- `403` – caller is not an `ADMIN`, or the admin tries to suspend themself.
- `404 USER_NOT_FOUND` – the account does not exist or is not a `STUDENT`.
- `422 VALIDATION_ERROR` – reason shorter than 5 characters or an unknown status.
- `409 STATUS_CONFLICT` – the account is already `SUSPENDED`.
- `500` – database error; the transaction is rolled back.
