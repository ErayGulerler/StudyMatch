# StudyMatch — Reverse Side of User Stories

## Scope

This document is the reverse side of the user story cards in [user-stories.md](user-stories.md). The front side states who needs a feature and why. The reverse side adds the detail the front does not show.

Each card contains:

- Acceptance Criteria: yes/no questions a tester can answer to decide when the story is done.
- Edge Cases: atypical situations, boundary values and rejected actions.
- Non-Functional Requirements: quality conditions specific to the story.
- Dependencies: related stories and the state or activity diagrams the story relies on.
- Open Questions: points raised since Checkpoint 1 that the team still needs to decide.
- Estimation Notes: why the story has its size.

## Diagram References

- Account SD, Study Group SD, Study Session SD, Report SD: [state diagrams](state-diagrams/README.md).
- AD-01 Find partners, AD-02 Join group, AD-04 Plan session: [activity diagrams](activity-diagrams/README.md).

## Shared Non-Functional Requirements

These apply to every card and are not repeated below. All values are proposals subject to team review.

- Security: Every protected action validates an active session (UC-05). Passwords and session tokens are stored only as hashes.
- Privacy: Email addresses never appear in public profiles or search results.
- Reliability: A rejected action leaves all data unchanged, as shown by the dashed self-transitions in the state diagrams.
- Usability: Every rejection displays its reason next to the related field or action.
- Performance: Typical requests respond within 1 second.
- Time: All dates and times use the Europe/Warsaw time zone.

## US-01 — Create an Account

**Front:** As a visitor, I want to create a student account so that I can find study partners and join study groups.
**Priority:** Must have · **Size:** M

### Acceptance Criteria

- [ ] Is an active student account created from a name, an unused valid email address and a password of at least 8 characters?
- [ ] Is registration with an email address that already has an account rejected, without creating a second account?
- [ ] Are missing or invalid fields identified, with no account created?

### Edge Cases

- [ ] Is a password of exactly 8 characters accepted, and one of 7 characters rejected?
- [ ] Is an email without a domain, such as `filip@`, identified as invalid?
- [ ] Is the new account a student account even if the request asks for the administrator role?

### Dependencies

- Starts the Account lifecycle: initial state → ACTIVE (Account SD).
- Every other story requires an account.

### Open Questions

- Are email addresses compared case-insensitively, so that `Anna@pw.edu.pl` counts as already used by `anna@pw.edu.pl`?

### Estimation Notes

- M: form validation and a uniqueness check. Email verification is outside this version, which keeps the story from growing to L.

## US-02 — Log In

**Front:** As a registered user, I want to log in so that I can access the features available to my role.
**Priority:** Must have · **Size:** M

### Acceptance Criteria

- [ ] Does a student with correct credentials reach the student dashboard, and an administrator the admin panel?
- [ ] Does a wrong password display "Invalid email or password" and deny access?
- [ ] Are missing credentials identified, with access denied?
- [ ] Is a SUSPENDED account denied access with a suspension message, even with correct credentials?

### Edge Cases

- [ ] Does an unregistered email address produce exactly the same message as a wrong password?
- [ ] Can a student who was suspended and then reactivated log in again with the same password?

### Non-Functional Requirements

- Security: The response must not reveal whether an email address is registered.

### Dependencies

- US-01 creates the account. US-18 sets the SUSPENDED state checked here (Account SD).

### Open Questions

- Should repeated failed attempts be limited, for example a temporary lock after 5 failures? Checkpoint 1 does not cover this.

### Estimation Notes

- M: credential check, session creation and role-based routing to two different clients.

## US-03 — Log Out

**Front:** As a logged-in user, I want to log out so that I can securely end access to my account on the current device.
**Priority:** Must have · **Size:** S

### Acceptance Criteria

- [ ] Does "Log out" end the current session and open the login page?
- [ ] Is a later request using the ended session denied and asked to authenticate again?

### Edge Cases

- [ ] Do the user's sessions on other devices stay active after logging out on one device?
- [ ] Are the account, profile and memberships unchanged after logging out?

### Dependencies

- US-02 creates the session. Ending sessions (UC-04) is shared with suspension in US-18.

### Open Questions

- Should inactive sessions expire automatically? The class diagram mentions expired sessions, but no duration is agreed.

### Estimation Notes

- S: one session is revoked. No data changes.

## US-04 — Manage a Study Profile

**Front:** As a student, I want to manage my study profile so that other students can understand my academic interests and preferences.
**Priority:** Must have · **Size:** M

### Acceptance Criteria

- [ ] Is a profile with a university, at least one subject, a study level and a study style stored and displayed?
- [ ] Are saved changes displayed and used in the next partner search?
- [ ] Is a profile with missing required information rejected, with the previous profile preserved?

### Edge Cases

- [ ] Is a profile whose last subject was removed rejected?
- [ ] Does another student viewing the profile see no email address?
- [ ] Is a student prevented from editing someone else's profile?

### Dependencies

- Required by US-06: AD-01 checks the profile before searching.

### Open Questions

- Who maintains the list of universities and subjects? The class diagram uses `SubjectId` values, but managing the catalogue is not covered by any story.

### Estimation Notes

- M: form with several fields, validation and a public view without private data.

## US-05 — Set Availability

**Front:** As a student, I want to manage my available study times so that I can find partners with compatible schedules.
**Priority:** Must have · **Size:** M

### Acceptance Criteria

- [ ] Is a slot with a weekday and an end time later than its start time added to the weekly schedule?
- [ ] Do edited or deleted slots change the schedule used in the next search?
- [ ] Is a slot whose end time is not later than its start time rejected with an error?

### Edge Cases

- [ ] Is a slot of 14:00–14:00 (zero length) rejected?
- [ ] Are overlapping slots, Thu 10:00–12:00 and Thu 11:00–13:00, treated as one period, Thu 10:00–13:00?

### Dependencies

- Required by US-06: overlapping availability is an eligibility condition.

### Open Questions

- Should a slot be able to cross midnight, for example Fri 22:00–01:00? The current rule makes this impossible.

### Estimation Notes

- M: simple form, plus merging overlapping slots.

## US-06 — Find Compatible Study Partners

**Front:** As a student, I want to find compatible study partners so that I can study with people who share my subjects and availability.
**Priority:** Must have · **Size:** M

### Acceptance Criteria

- [ ] Are only active students with at least one shared subject and a positive overlap of availability returned?
- [ ] Are results ordered by matching score, with each result showing the profile, shared subjects and overlapping time?
- [ ] Does every result satisfy all selected university and subject filters?
- [ ] Is "No matching study partners found." displayed when nobody qualifies?
- [ ] Is a student with a missing profile or availability told what to complete, instead of seeing results?

### Edge Cases

- [ ] Do two shared subjects, the same university and study style, but a different study level give a score of 4?
- [ ] Are equal scores ordered by ascending student ID?
- [ ] Are the searching student and suspended students never returned?
- [ ] Is a student whose availability only touches at one moment (Mon 16:00–19:00 and Mon 19:00–21:00) excluded?
- [ ] Does a search create no group or membership?

### Non-Functional Requirements

- Performance: Results appear within 2 seconds for up to 10,000 students.

### Dependencies

- US-04, US-05 and US-18. Follows AD-01. Excludes SUSPENDED accounts (Account SD).

### Open Questions

- Should results be split into pages when there are many matches?

### Estimation Notes

- M: the score is a simple sum, but availability overlap must be calculated for every candidate. Adding pages would make it L.

## US-07 — Create a Study Group

**Front:** As a student, I want to create a study group so that other students can join me to study a particular subject.
**Priority:** Must have · **Size:** M

### Acceptance Criteria

- [ ] Is a group created from a name, a subject and an integer member limit of at least 2?
- [ ] Is the creator the organiser and first member, and does the group appear in searches?
- [ ] Are missing information and an invalid limit identified, with no group created?

### Edge Cases

- [ ] Is a limit of 2 accepted, and limits of 1 and 2.5 rejected?
- [ ] Is a SUSPENDED student unable to create a group?

### Non-Functional Requirements

- Reliability: The group and the organiser's membership are created together, so a group never exists without an organiser.

### Dependencies

- Starts the Study Group lifecycle: initial state → ACTIVE (Study Group SD). Required by US-08 to US-14.

### Open Questions

- Is there a maximum member limit?

### Estimation Notes

- M: creates two records (group and membership) that must stay consistent.

## US-08 — Find and Join a Study Group

**Front:** As a student, I want to find and join a study group so that I can study my chosen subject with others.
**Priority:** Must have · **Size:** M

### Acceptance Criteria

- [ ] Does a subject search list matching groups with their organiser, member count and member limit?
- [ ] Is a non-member who joins a group with a free place added, with the member count updated?
- [ ] Is a join request to a full group, or from an existing member, rejected with a reason and no change?
- [ ] Is "No matching study groups found." displayed when no group matches?

### Edge Cases

- [ ] When two students press "Join" for the last free place at the same moment, does exactly one succeed?
- [ ] Is a DELETED group hidden from search and impossible to join?
- [ ] Is a student added without organiser approval?

### Non-Functional Requirements

- Reliability: The capacity check and the new membership are saved in one atomic operation.

### Dependencies

- US-07. Follows AD-02. Joining a deleted group is rejected (Study Group SD).

### Open Questions

- Should full groups still appear in search results, marked as full?

### Estimation Notes

- M: search plus concurrency handling for the last place.

## US-09 — Leave a Study Group

**Front:** As a group member, I want to leave a study group so that I can stop participating when it no longer meets my needs.
**Priority:** Must have · **Size:** S

### Acceptance Criteria

- [ ] Is a member who is not the organiser removed after confirming, with the member count reduced?
- [ ] Do the group's sessions disappear from the former member's upcoming sessions?
- [ ] Is the organiser told to transfer ownership or delete the group first?

### Edge Cases

- [ ] Can a former member rejoin while a place is available?
- [ ] Does the group stay ACTIVE when the organiser is its only remaining member?

### Dependencies

- US-08, US-10 and US-11. Leaving changes a membership, not the group's state, so it is not a transition in the Study Group SD.

### Open Questions

- None.

### Estimation Notes

- S: removes one membership. The organiser rule is a single check.

## US-10 — Transfer Group Ownership

**Front:** As a group organiser, I want to transfer ownership to another member so that the group can continue without me as organiser.
**Priority:** Should have · **Size:** S

### Acceptance Criteria

- [ ] Does the selected active member become the sole organiser after the organiser confirms?
- [ ] Does the previous organiser remain a regular member who can leave normally?
- [ ] Is a transfer to a user who is not an active member rejected, with the organiser preserved?

### Edge Cases

- [ ] Is a transfer to a SUSPENDED member rejected?
- [ ] Is a regular member unable to make themselves organiser?
- [ ] Is a transfer in a DELETED group rejected?

### Dependencies

- ACTIVE → ACTIVE self-transition `transferOwnership` (Study Group SD). Allows the organiser to use US-09.

### Open Questions

- Should the new organiser have to accept the role before the transfer completes?

### Estimation Notes

- S: changes one field after membership and permission checks.

## US-11 — Delete a Study Group

**Front:** As a group organiser, I want to delete my study group so that it no longer appears as available when it is no longer active.
**Priority:** Must have · **Size:** S

### Acceptance Criteria

- [ ] Is the group removed from search results and members' group lists after the organiser confirms?
- [ ] Are the group's future sessions cancelled and no longer shown as upcoming active sessions?
- [ ] Is a regular member denied, with the group preserved?

### Edge Cases

- [ ] Does closing the confirmation dialog without confirming change nothing?
- [ ] Are the group's past sessions left unchanged?
- [ ] Do all former members keep their accounts?

### Non-Functional Requirements

- Reliability: Marking the group deleted, removing memberships and cancelling future sessions happen in one operation.

### Dependencies

- ACTIVE → DELETED (Study Group SD), which triggers SCHEDULED → CANCELLED for each future session (Study Session SD). Affects US-14 and US-15.

### Open Questions

- How long are deleted groups kept? Retention belongs to the technical design.

### Estimation Notes

- S for the user, but the cascade to sessions and memberships needs careful testing.

## US-12 — Schedule a Study Session

**Front:** As a group organiser, I want to schedule a study session so that members know when and where we will study.
**Priority:** Must have · **Size:** M

### Acceptance Criteria

- [ ] Is a session created from a title, a future start, a later end and a location or meeting link?
- [ ] Does the new session appear in every current member's upcoming sessions?
- [ ] Are missing information and invalid times identified, with no session created?

### Edge Cases

- [ ] Is a session with a meeting link and no location accepted?
- [ ] Are a start in the past and an end before the start both rejected?
- [ ] Is a regular member unable to schedule a session?
- [ ] Is a session accepted at a time when no member is available?

### Dependencies

- US-07. Follows AD-04. Starts the Study Session lifecycle: initial state → SCHEDULED (Study Session SD).

### Open Questions

- May two sessions of the same group overlap in time?

### Estimation Notes

- M: several date and time rules, plus visibility for all members.

## US-13 — Edit a Study Session

**Front:** As a group organiser, I want to edit an upcoming session so that members can see changes to our study plan.
**Priority:** Should have · **Size:** S

### Acceptance Criteria

- [ ] Are valid changes to an upcoming active session saved?
- [ ] Do members see the new details after reloading their session list?
- [ ] Are invalid changes identified, with the previous details preserved?

### Edge Cases

- [ ] Are past sessions impossible to edit?
- [ ] Are CANCELLED sessions impossible to edit?
- [ ] Is moving the start time into the past rejected?
- [ ] Is a regular member unable to edit?

### Dependencies

- Reuses the rules of US-12. Follows AD-04. SCHEDULED → SCHEDULED, with rejected edits shown as a dashed self-transition (Study Session SD).

### Open Questions

- Can a session be edited after it has started but before it ends?

### Estimation Notes

- S: reuses the US-12 form and validation.

## US-14 — Cancel a Study Session

**Front:** As a group organiser, I want to cancel an upcoming session so that members know it will not take place.
**Priority:** Must have · **Size:** S

### Acceptance Criteria

- [ ] Is an upcoming session marked cancelled after the organiser confirms?
- [ ] Do members see the session labelled "Cancelled"?
- [ ] Is a regular member denied, with the session preserved?

### Edge Cases

- [ ] Is a CANCELLED session impossible to reactivate or cancel again?
- [ ] Is a past session impossible to cancel?
- [ ] Does the group stay ACTIVE after the cancellation?

### Dependencies

- SCHEDULED → CANCELLED (Study Session SD). Visible through US-15.

### Open Questions

- Should the organiser give a cancellation reason that members can see?

### Estimation Notes

- S: one status change.

## US-15 — View Upcoming Study Sessions

**Front:** As a student, I want to view my upcoming study sessions so that I can plan my time.
**Priority:** Must have · **Size:** S

### Acceptance Criteria

- [ ] Are future sessions from the student's groups listed by start date and time?
- [ ] Do session details show the title, group, start and end times, location or link, and cancellation status?
- [ ] Is "No upcoming study sessions." displayed when there are none?

### Edge Cases

- [ ] Are past sessions hidden?
- [ ] Are cancelled future sessions shown with a "Cancelled" label?
- [ ] Are sessions of deleted groups hidden?
- [ ] Is a non-member who opens a session link denied?

### Dependencies

- US-08, US-09, US-11 and US-14. "Upcoming" is the query `isUpcoming(now)`, not a separate state (Study Session SD).

### Open Questions

- Should a session that is currently in progress still appear in the list?

### Estimation Notes

- S: one filtered and sorted list.

## US-16 — Report a User

**Front:** As a student, I want to report inappropriate behaviour so that an administrator can review it.
**Priority:** Should have · **Size:** S

### Acceptance Criteria

- [ ] Is a report about another student with a non-empty reason saved as pending and confirmed?
- [ ] Is a report with an empty reason refused, with a request for a reason?
- [ ] Is the report action unavailable on the student's own profile?

### Edge Cases

- [ ] Does the backend reject a self-report sent directly, bypassing the interface?
- [ ] Is the reporter unable to read the report after submitting it?
- [ ] Does the reported account stay ACTIVE?

### Dependencies

- Starts the Report lifecycle: initial state → PENDING (Report SD). Reviewed in US-17.

### Open Questions

- Can the same student report the same person repeatedly?

### Estimation Notes

- S: one form and one record.

## US-17 — Review Reports

**Front:** As an administrator, I want to review and resolve reports so that I can address reported behaviour.
**Priority:** Should have · **Size:** M

### Acceptance Criteria

- [ ] Are reports listed with a filter for pending or resolved status?
- [ ] Does an opened report show its reporter, reported student, reason and submission time?
- [ ] Does resolving a pending report with a written outcome record the outcome, administrator and time, and mark it resolved?

### Edge Cases

- [ ] Is resolving with an empty outcome rejected, leaving the report PENDING?
- [ ] Is an already RESOLVED report impossible to resolve again?
- [ ] Is a student unable to open the report list?
- [ ] Is the reported student's account status unchanged after resolution?

### Dependencies

- US-16 and US-18. PENDING → RESOLVED, with rejected resolutions shown as a dashed self-transition (Report SD).

### Open Questions

- Should a resolved report link to the account status change it led to? `AccountStatusChange` currently has no report reference.

### Estimation Notes

- M: list, filter, detail view and resolution in the admin panel.

## US-18 — Manage Student Account Status

**Front:** As an administrator, I want to suspend and reactivate student accounts so that I can enforce rules and restore access when appropriate.
**Priority:** Should have · **Size:** M

### Acceptance Criteria

- [ ] Does suspension with a reason mark the account suspended, end its sessions and reject further protected actions?
- [ ] Are suspended accounts excluded from partner search results?
- [ ] Does reactivation with a reason mark the account active and permit login with valid credentials?
- [ ] Is every status change recorded with its administrator, reason, action and time?

### Edge Cases

- [ ] Is suspending an already SUSPENDED account rejected as an invalid change?
- [ ] Is a status change by a student denied?
- [ ] Can an account be suspended without any report?
- [ ] Are the profile and group memberships preserved during suspension?
- [ ] Is a suspended organiser unable to manage their group until reactivated?

### Non-Functional Requirements

- Security: The status change and its audit entry are saved together.

### Dependencies

- ACTIVE ⇄ SUSPENDED (Account SD). Affects US-02, US-06 and the organiser rights in US-10 to US-14.

### Open Questions

- While an organiser is suspended, nobody can manage the group. Should an administrator be able to transfer ownership in that case?

### Estimation Notes

- M: account search, two transitions, session revocation and an audit record.

## Changes Since Checkpoint 1

Writing the reverse side exposed rules that the state diagrams already use but the front side of the cards does not state. These are proposed updates to [user-stories.md](user-stories.md):

- US-08: Add a note that deleted groups cannot be found or joined (Study Group SD).
- US-10: Add an edge case that ownership cannot be transferred to a suspended member.
- US-14: Add a note that cancelled and past sessions cannot be cancelled again.
- US-15: Define "upcoming" as a start time later than now, and decide how in-progress sessions are shown.
- US-18: Add a note that a change to the current status, such as suspending a suspended account, is rejected (Account SD).

The open questions above are recorded for later checkpoints. None of them blocks the current state or activity diagrams.
