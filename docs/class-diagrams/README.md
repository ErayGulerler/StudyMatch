# StudyMatch class diagrams

These diagrams describe the classes behind the existing [user stories](../user-stories.md) and [use cases](../use-cases/modules-actors-notation.md). They follow the same four backend modules: Identity & Access, Profile & Matching, Group & Session, and Moderation.

The Student Application and Admin Panel are clients of the Backend Server. The overall services view shows their communication; the other four diagrams show the main data, operations and relationships inside the backend. This is a design specification, not an implementation.

## Diagrams

| View | Preview | Editable source |
| --- | --- | --- |
| System services and clients | [PNG](diagrams/00_system_services.png) / [SVG](diagrams/00_system_services.svg) | [DOT](sources/00_system_services.dot) |
| Identity & Access | [PNG](diagrams/01_identity_access.png) / [SVG](diagrams/01_identity_access.svg) | [DOT](sources/01_identity_access.dot) |
| Profile & Matching | [PNG](diagrams/02_profile_matching.png) / [SVG](diagrams/02_profile_matching.svg) | [DOT](sources/02_profile_matching.dot) |
| Group & Session | [PNG](diagrams/03_groups_sessions.png) / [SVG](diagrams/03_groups_sessions.svg) | [DOT](sources/03_groups_sessions.dot) |
| Moderation | [PNG](diagrams/04_moderation.png) / [SVG](diagrams/04_moderation.svg) | [DOT](sources/04_moderation.dot) |

[Class diagrams PDF](StudyMatch-Class-Diagrams.pdf) contains all five views and their main rules. It is the class-diagram section only; the complete Checkpoint 1 submission must also contain the team's user stories and use-case material.

## How to read the diagrams

- Classes list important attributes and operations. `-` means private and `+` means public.
- A solid line is an association; numbers at its ends are multiplicities. `1` means exactly one, `0..1` optional, and `0..*` zero or more.
- A filled diamond denotes composition: an authentication session belongs to its account, and weekly availability belongs to its profile.
- A dashed arrow denotes a dependency, pointing towards the class or service used.
- `entity` has a persistent identity; `value` represents a value or returned data; `service` coordinates operations. `reference` repeats a type from another view rather than declaring a new entity.
- Attribute types also express references. For example, each `Report.reporterId` refers to one student account; many reports may refer to the same account. IDs avoid drawing every cross-module association repeatedly.
- Basic types such as `UUID`, `DateTime`, `LocalTime`, `SubjectId`, `SessionToken`, request data and collections are deliberately not expanded into utility classes.

## Main design decisions and rules

### System services

The three communicating applications are Student Application, Admin Panel and Backend Server. Internal services are logical modules of one backend, not additional deployed applications. ProfileService and MatchingService belong to the same Profile & Matching module. The overview shows selected operations and main client flows; the detailed diagrams show the remaining operations. The Student Application also uses IdentityService, ProfileService and ModerationService; the Admin Panel also uses IdentityService.

Every protected entry point validates its session through IdentityService (UC-05), including ProfileService, MatchingService, GroupSessionService and ModerationService. A service derives the acting account and role from the validated token; clients cannot grant themselves rights by sending a different account ID. The service also checks ownership or membership where required. Internal account lookup (UC-06) and account status changes (UC-07) are trusted backend operations, not unrestricted public endpoints. Public profile lookup implements UC-13.

### Identity & Access

`Account` represents both students and administrators through `AccountRole`. Public registration always creates a student; administrator provisioning is outside scope. A student without a completed profile may still have an account. `AuthSession` is a login session, distinct from a `StudySession`.

Passwords and session tokens are stored as hashes. Login returns the token to the client; validation rejects expired or revoked sessions and checks that the account is active. Logout revokes the current session. Suspension revokes all of the student's sessions, while reactivation permits a new login without restoring revoked sessions. AccountSummary exposes identity, display name, role and status to other services without exposing credentials or email addresses. ProfileService obtains the display name through this internal lookup. Moderation's email search uses IdentityService internally.

### Profile & Matching

Each student has zero or one saved StudyProfile, identified by that student's Account ID. Saved profiles contain a university, at least one distinct subject, a programme study level and a study style. Study level means programme stage, as defined in US-04, rather than subject proficiency. The common subject catalogue is represented by SubjectId values rather than a separate catalogue-management feature.

Availability is a recurring weekly schedule in Europe/Warsaw. Every slot has an end time later than its start time; overlapping slots on the same day are combined before calculating shared availability. Returned overlap uses WeeklyTimeRange values (weekday, start time, end time), not the identities of stored availability slots. Account activity is checked through IdentityService, so suspended students and the searching student never appear as matches.

Both a shared subject and positive availability overlap are required. Optional filters are combined with AND. The score is the number of distinct shared subjects, plus one each for the same university, study level and study style. Equal scores use ascending student ID as a stable tie-breaker. MatchResult is calculated on demand and does not create a group or membership. PublicProfile deliberately excludes email and credentials.

### Group & Session

Each active StudyGroup has exactly one organiser, who must also have a Membership in that group. Organiser is a role within a group, not a separate account type: the same student can organise one group and be a regular member of another. The creator becomes the first member, and memberLimit includes the organiser. Membership is unique for each (groupId, studentId) pair.

Joining checks capacity and creates membership atomically, so simultaneous requests cannot overfill a group. An organiser cannot leave without transferring ownership to another active member or deleting the group. Transfer keeps the previous organiser as a regular member.

The design represents deletion with GroupStatus.DELETED. Deletion hides the group from searches and member lists, removes its memberships and cancels future sessions in one operation. The diagram permits zero memberships for deleted groups; an active group must have at least one member, its organiser. Accounts are retained. Keeping a deleted group record allows its session records to keep a valid group reference; a retention policy can be specified later.

Only the current organiser can create, edit or cancel sessions. Starts must be in the future, ends must be later, and a location or meeting link is required. Past or cancelled sessions cannot be edited. Cancellation cannot be reversed. All current members can view the group's session details; upcoming cancelled sessions remain visible with a cancellation label, except when their group has been deleted. Leaving a group removes its sessions from the former member's list. Availability does not restrict scheduling, and there is no separate session enrolment entity because the agreed stories use group membership.

### Moderation

A report has one reporter and one reported student; these must be different accounts. Pending reports have no reviewer, outcome or resolution time. Resolution records all three fields and changes status to RESOLVED. Only administrators can read report content after submission.

Report resolution and account suspension are separate actions. AccountStatusChange records the student, administrator, old/new status, reason and time even when no report exists. ModerationService validates the administrator and target student, then changes status through IdentityService and records the audit entry atomically. Suspension preserves profiles and memberships. A suspended organiser cannot manage a group until reactivated. Status changes apply only to student accounts.

## Traceability

| User stories | Julia's use cases | Class-diagram responsibility |
| --- | --- | --- |
| US-01 to US-03 | UC-01 to UC-04 | Account, AuthSession, IdentityService |
| Shared authenticated access | UC-05, UC-06, UC-07 | Principal, AccountSummary, IdentityService |
| US-04, US-05 | UC-08, UC-09, UC-13 | StudyProfile, AvailabilitySlot, PublicProfile, ProfileService |
| US-06 | UC-10, UC-11, UC-12 | MatchingService, MatchFilters, MatchResult |
| US-07 | UC-14 | StudyGroup, Membership, GroupSessionService |
| US-08, US-09 | UC-15, UC-16, UC-17 | Membership, StudyGroup, GroupSessionService |
| US-10, US-11 | UC-19, UC-20 | StudyGroup, Membership, StudySession, GroupSessionService |
| US-12 to US-15 | UC-18, UC-21, UC-22, UC-23 | StudySession, GroupSessionService |
| US-16, US-17 | UC-24, UC-25, UC-26 | Report, ModerationService |
| US-18 | UC-27, UC-28, UC-06, UC-07 | AccountStatusChange, ModerationService, IdentityService |

## Suggested speaking notes

1. Our two clients communicate with a backend containing the same four modules shown in the use cases. Services validate identity and coordinate operations.
2. Account and AuthSession handle access. StudyProfile and AvailabilitySlot support the rule-based matching score, while PublicProfile keeps email and credentials private.
3. Membership connects students to groups. The organiser is one of those members. StudySession belongs to a group, so there is no separate session enrolment.
4. Reports support review; AccountStatusChange records suspensions and reactivations independently. These models cover all 18 user stories.

For a short presentation, show the overall services diagram and the group/session diagram, then keep the other three as supporting slides. Allow about two minutes for this section.

## Editing and rendering

Edit the files in `sources/` with any text editor and render them using Graphviz, for example:

```sh
dot -Tsvg sources/03_groups_sessions.dot -o diagrams/03_groups_sessions.svg
dot -Tpng -Gdpi=200 sources/03_groups_sessions.dot -o diagrams/03_groups_sessions.png
```

This section follows repository revision `243412a` (22 October 2026). It preserves the agreed scope: no messaging, video calls, file sharing, automatic reminders, room booking or automatic meeting-link creation.
