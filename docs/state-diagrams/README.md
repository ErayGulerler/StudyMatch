# StudyMatch — State Diagrams (Checkpoint 2)

Four key business object lifecycles are modelled: Account, StudyGroup, StudySession, and Report. Sources are Graphviz DOT; PNG and SVG renders are in `diagrams/`.

## Consistency and exceptional paths

- **Account:** `AccountStatus` = ACTIVE / SUSPENDED. `IdentityService.registerStudent(data)` creates an active account; `ModerationService.changeStudentStatus` leads to `Account.changeStatus(status)`. Unauthorized transitions are rejected. Suspension invalidates sessions (US-18).
- **StudyGroup:** `GroupStatus` = ACTIVE / DELETED. `GroupSessionService.createGroup`, `transferOwnership`, `deleteGroup` correspond to `StudyGroup.transferOwnership` and `markDeleted`. Deleting a group cancels its future sessions (US-11).
- **StudySession:** `SessionStatus` = SCHEDULED / CANCELLED. `GroupSessionService.scheduleSession`, `editSession`, `cancelSession` correspond to `StudySession.edit(data, now)` and `cancel(now)`. Invalid/past/cancelled edits are rejected (US-13/14). Group deletion also cancels future sessions (US-11). The existing class model has no COMPLETED status.
- **Report:** `ReportStatus` = PENDING / RESOLVED. `ModerationService.submitReport` creates a pending report; `resolveReport` invokes `Report.resolve(adminId, outcome, now)`. Invalid reports and unauthorized resolutions are rejected. Resolving a report does not automatically change account status (US-16/17).

**Notation:** Dotted connectors lead to explanatory notes; they are not state transitions. Invalid actions preserve the current state. Guards are business rules checked by the relevant services. Cross-object side effects are described in notes rather than falsely shown as transitions of the same object.

**Follow-up:** At the later checkpoint, confirm any implementation-level status-change audit mechanism and whether past scheduled sessions need an additional state.

## Checkpoint 2 review (2026-11-03)

Rejected requests are shown as dashed self-transitions: they leave the business object in the same state. DELETED, CANCELLED, and RESOLVED have no outgoing state-changing transitions in the current model; no UML final pseudostate is used because the object record may continue to exist. Group deletion triggers cancellation of future sessions as specified by US-11.
