# StudyMatch — Detailed use cases

These descriptions expand the existing UML diagrams and follow the rules in [user-stories.md](../user-stories.md) and the [class design](../class-diagrams/README.md). They do not change the existing 28 use cases. Other crucial requirements already have acceptance criteria in the user-story cards.

## Shared rules

Protected actions require a valid session and an active account. Services derive the actor's identity and role from UC-05 and check ownership or membership where needed. Displayed study times use Europe/Warsaw. Invalid or unauthorised requests do not change stored data. Messaging, video calls, file sharing and automatic reminders remain outside scope.

## UC-10 — Find study partners

![Partner search detail](diagrams/05_partner_search_detail.svg)

| Field | Description |
| --- | --- |
| Scope / level | Profile & Matching module / user goal |
| Primary actor | Student |
| Supporting actor | Identity & Access module |
| Stakeholders | Searching student wants relevant matches; other students want their private account data protected. |
| User story / priority | US-06 / Must have |
| Preconditions | Student has an authenticated, active account, a complete study profile and saved weekly availability. |
| Trigger | Student submits a partner search, optionally selecting university or subject filters. |
| Success guarantee | System returns eligible partners ordered by descending score, then ascending student ID for ties. Each result contains a public profile, shared subjects and overlapping availability. |
| Minimal guarantee | Search changes no profiles or memberships and exposes no email addresses or credentials. |

### Main success scenario

1. Student submits the search.
2. MatchingService calls IdentityService.validateSession(token), UC-05, and verifies the student role.
3. System loads the student's profile and weekly availability. It combines overlapping slots on the same day before comparing schedules.
4. System finds other students with at least one distinct shared subject and a positive period of overlapping availability. Identity UC-06 supplies account information so the search excludes suspended accounts and the searching student.
5. If the student selected filters, UC-12 **Apply search filters** retains candidates satisfying every selected university and subject filter.
6. UC-11 **Calculate matching score** calculates the score for each remaining candidate: one point per distinct shared subject, plus one each for the same university, study level and study style.
7. System orders results by descending score and ascending student ID for ties, then displays each public profile, shared subjects and overlapping periods.

### Extensions

- **2a — Invalid, expired or revoked session; inactive account:** Deny access and require authentication with an active account.
- **3a — Incomplete profile or no availability:** Identify the missing information and stop the search until the student completes it.
- **4a/5a — No eligible candidates:** Display “No matching study partners found.” This is a valid empty result.
- **5b — No filters selected:** Skip UC-12 and continue with scoring.

### Relationships and contract

UC-10 **includes** UC-11. UC-12 **extends** UC-10 at candidate filtering, only when a filter is selected. Mandatory scoring means the search executes the scoring stage even if there are zero candidates to score.

`MatchingService.findPartners(token, filters) → MatchResult[]`

A MatchResult contains `candidate: PublicProfile`, `score`, `sharedSubjects` and `overlap: WeeklyTimeRange[]`. Matching does not automatically create a group or membership.

## UC-16 — Join study group

| Field | Description |
| --- | --- |
| Scope / level | Group & Session module / user goal |
| Primary actor | Student |
| Supporting actor | Identity & Access module |
| Stakeholders | Student wants to join; organiser and members need the member limit respected. |
| User story / priority | US-08 / Must have |
| Preconditions | Authenticated, active student account; student has selected an active group found through UC-15. |
| Trigger | Student selects Join. |
| Success guarantee | Exactly one membership connects the student to the group, the member count updates and capacity is not exceeded. |
| Minimal guarantee | Rejected requests leave memberships and member count unchanged. |

### Main success scenario

1. Student selects Join for a group shown by UC-15 **Search study groups**.
2. GroupSessionService validates the session through Identity UC-05 and verifies the student role.
3. System checks that the group is active, the student is not already a member and the group has an available place. The member limit includes the organiser.
4. In one atomic operation, system checks capacity and creates the membership with the joining time. Concurrent joins cannot exceed the limit.
5. System confirms membership and displays the updated member count. The group's future sessions now appear in the student's upcoming sessions.

### Extensions

- **2a — Invalid session or inactive account:** Deny the request without changing membership.
- **3a — Group is no longer available:** Reject the join without creating a membership.
- **3b — Existing membership:** Reject the request and explain that the student is already a member. Do not silently treat this as a new successful join.
- **3c/4a — Group is full, including a competing request taking the last place:** Reject the join and explain that the group is full.

### Relationships and contract

UC-16 **extends** UC-15 at the displayed search results, when the student selects Join. Organiser approval is not required. A student may belong to several groups.

`GroupSessionService.joinGroup(token, groupId)` creates the membership on success or reports the reason for rejection. UC-18 subsequently lists sessions using the new membership.

## UC-21 — Schedule study session

![Scheduling detail](diagrams/06_scheduling_detail.svg)

| Field | Description |
| --- | --- |
| Scope / level | Group & Session module / user goal |
| Primary actor | Group Organiser |
| Supporting actor | Identity & Access module |
| Stakeholders | Organiser wants to publish a study plan; current members need accurate time and location details. |
| User story / priority | US-12 / Must have |
| Preconditions | Authenticated, active account; actor is the current organiser of an active group. |
| Trigger | Organiser submits a new session form for the group. |
| Success guarantee | A SCHEDULED StudySession belongs to the group and appears with its details in current members' upcoming sessions. |
| Minimal guarantee | Rejected requests create no session and preserve existing sessions. |

### Main success scenario

1. Organiser enters a title, future start date/time, later end date/time and a location or meeting link.
2. GroupSessionService validates the session through Identity UC-05 and obtains the acting student identity.
3. System checks that the group is active and that the actor is its current organiser.
4. System validates all required fields, confirms that the start is in the future and that the end is later than the start.
5. System saves a StudySession linked to the group with status SCHEDULED.
6. System confirms creation. When current members open UC-18 **View upcoming sessions**, the new session appears with its details, displayed in Europe/Warsaw time.

### Extensions

- **2a — Invalid session or inactive account:** Deny access and create no session.
- **3a — Actor is a regular member or no longer the organiser:** Deny the action.
- **3b — Group is no longer active:** Reject the request and create no session.
- **4a — Missing title, time or location/link; start not in the future; end not later than start:** Identify invalid fields and let the organiser correct them. Save nothing until all checks pass.

### Relationships and contract

`GroupSessionService.scheduleSession(token, groupId, data)` validates permissions and creates the session. Its required service call is `IdentityService.validateSession(token) → Principal` (UC-05).

UC-18 is a later member action, not an included step that every schedule request must invoke. Availability assists matching but does not restrict scheduling. Room booking and automatic meeting-link generation remain outside scope.

## Inter-module services

Service calls are trusted backend operations. The calling service validates the user's access and permissions. These are contracts already present in the class design, not new public endpoints.

| ID | Provider | Calling module(s) | Input | Success result / effect |
| --- | --- | --- | --- | --- |
| UC-05 Validate session | Identity & Access | Profile & Matching; Group & Session; Moderation | token | Principal: accountId and role. Invalid, expired, revoked or suspended-account sessions are rejected. |
| UC-06 Look up account | Identity & Access | Profile & Matching; Moderation | idOrEmail | AccountSummary: accountId, displayName, role, status. No credentials or email in the result. |
| UC-07 Change account status | Identity & Access | Moderation | id, status | Changes the student account status. Suspension revokes all sessions. Reactivation permits a new login and never restores revoked sessions. |
| UC-13 Provide public profile | Profile & Matching | Group & Session; Moderation | student ID | PublicProfile: studentId, displayName, university, subjects, studyLevel, preferredStyle. No email or credentials. |

Account/profile lookup requires an existing target. An unresolved target supplies no account/profile data; exact API error codes remain an implementation detail.

## Traceability: existing IDs

| Module | Use case | User-story link |
| --- | --- | --- |
| Identity & Access | UC-01 Register student account | US-01 |
| Identity & Access | UC-02 Log in | US-02 |
| Identity & Access | UC-03 Log out | US-03 |
| Identity & Access | UC-04 End user session(s) | US-03, US-18 |
| Identity & Access | UC-05 Validate session | Shared protected-access rule; US-02, US-03, US-18 |
| Identity & Access | UC-06 Look up account | Supporting service for US-04, US-06, US-18 |
| Identity & Access | UC-07 Change account status | US-18 |
| Profile & Matching | UC-08 Manage study profile | US-04 |
| Profile & Matching | UC-09 Manage availability | US-05 |
| Profile & Matching | UC-10 Find study partners | US-06 |
| Profile & Matching | UC-11 Calculate matching score | US-06 |
| Profile & Matching | UC-12 Apply search filters | US-06 |
| Profile & Matching | UC-13 Provide public profile | Supports US-04, US-08, US-16, US-17 |
| Group & Session | UC-14 Create study group | US-07 |
| Group & Session | UC-15 Search study groups | US-08 |
| Group & Session | UC-16 Join study group | US-08 |
| Group & Session | UC-17 Leave study group | US-09 |
| Group & Session | UC-18 View upcoming sessions | US-15 |
| Group & Session | UC-19 Transfer group ownership | US-10 |
| Group & Session | UC-20 Delete study group | US-11 |
| Group & Session | UC-21 Schedule study session | US-12 |
| Group & Session | UC-22 Edit study session | US-13 |
| Group & Session | UC-23 Cancel study session | US-14; reused by US-11 |
| Moderation | UC-24 Report a user | US-16 |
| Moderation | UC-25 Review reports | US-17 |
| Moderation | UC-26 Resolve report | US-17 |
| Moderation | UC-27 Change student account status (suspend / reactivate) | US-18 |
| Moderation | UC-28 Find student account | US-18 |

The original Identity diagram draws UC-07 including UC-04. Read session invalidation in the context of the requested status: suspension revokes sessions, while reactivation does not restore them. The detailed descriptions preserve the explicit rule in US-18 and the class design.
