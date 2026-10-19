# StudyMatch — User Stories

## Scope

StudyMatch helps students find compatible study partners, join study groups and organise study sessions.

The system consists of a student application, an admin panel and a backend server that communicate through services.

Messaging, video calls, file sharing and automatic reminders are outside the scope of this version.

## Priorities

- Must have: Essential for the first working version.
- Should have: Important, but may be delivered after the core features.
- Could have: Optional if time allows.
- Won't have this time: Outside the scope of this version.

## Size Estimates

Size represents estimated development effort, not priority.

- S: Small
- M: Medium
- L: Large

All estimates and proposed business rules are subject to team review.

## Shared Rules

- Protected actions require an authenticated, active account.
- Students can modify only their own profiles and availability.
- Only administrators can access moderation features.
- Each study group has one organiser.
- All displayed study times use the Europe/Warsaw time zone.

## US-01 — Create an Account

**User story:** As a visitor, I want to create a student account so that I can find study partners and join study groups.

**Priority:** Must have
**Size:** M

### Acceptance Criteria

1. Given a name, an unused valid email address and a password of at least 8 characters, when a visitor submits the registration form, then the system creates an active student account.
2. Given an email address already linked to an account, when a visitor attempts to register with it, then the system rejects the reques and does not create a duplicate account.
3. Given missing or invalid required information, when the visitor submits the form, then the system identifies the invalid fields and does not create an account.

### Notes

- The proposed minimum password length is 8 characters.
- Visitors cannot register as administrators.
- Email verification is outside this version's scope.

## US-02 — Log In

**User story:** As a registered user, I want to log in so that I can access the features available to my role.

**Priority:** Must have
**Size:** M

### Acceptance Criteria

1. Given an active account and correct credentials, when the user submits the login form, then the system authenticates them and opens the student dashboard or admin panel according to their role.

2. Given incorrect credentials, when the user submits the form, then the system displays "Invalid email or password" and denies access.

3. Given missing credentials, when the user submits the form, then the system identifies the missing fields and denies access.

4. Given a suspended account and correct credentials, when the user attempts to log in, then the system denies access and displays an account suspension message.

### Notes

- Registered users include students and administrators.
- Invalid credentials must not reveal whether an email is registered.
- Administrator accounts are provisioned separately.

## US-03 — Log Out

**User story:** As a logged-in user, I want to log out so that I can securely end access to my account on the current device.

**Priority:** Must have
**Size:** S

### Acceptance Criteria

1. Given a logged-in user, when they select "Log out", then the system ends their current session and opens the login page.

2. Given an ended session, when a request uses that session to access a protected feature, then the system denies access and requires authentication again.

### Notes

- Logging out does not delete the account or its data.
- This action ends the current session.

## US-04 — Manage a Study Profile

**User story:** As a student, I want to manage my study profile so that other students can understand my academic interests and preferences.

**Priority:** Must have
**Size:** M

### Acceptance Criteria

1. Given a logged-in student, when they save a university, at least one subject, a study level and a preferred study style, then the system stores and displays their profile.

2. Given an existing profile, when its owner saves valid changes, then the system displays the updated information and uses it in subsequent partner searches.

3. Given missing required information, when the student saves the profile, then the system identifies the missing fields and preserves the previously saved profile.

### Notes

- Study level means the student's programme stage, such as year of undergraduate study.
- Proposed study styles are quiet individual work together and discussion-based study.
- Email addresses are not displayed in public study profiles.
- Availability is covered in US-05.

## US-05 — Set Availability

**User story:** As a student, I want to manage my available study times so that I can find partners with compatible schedules.

**Priority:** Must have
**Size:** M

### Acceptance Criteria

1. Given a day of the week and an end time later than the start time, when the student saves an availability slot, then the system adds it to their weekly schedule.

2. Given an existing slot, when its owner edits or deletes it, then the system updates the schedule used in subsequent searches.

3. Given an end time not later than the start time, when the student saves the slot, then the system rejects it and displays an error.

### Notes

- Availability is a recurring weekly schedule.
- A student may have multiple availability slots.
- Overlapping slots are treated as a combined available period, not counted twice.

## US-06 — Find Compatible Study Partners

**User story:** As a student, I want to find compatible study partners so that I can study with people who share my subjects and availability.

**Priority:** Must have
**Size:** M

### Acceptance Criteria

1. Given a complete profile and saved availability, when the student searches, then the system returns other active students sharing at least one subject and a positive period of overlapping availability.

2. Given eligible partners, when results appear, then the system orders them by the matching score defined below and displays each partner's profile, shared subjects and overlapping availability.

3. Given selected university or subject filters, when the student searches, then every result satisfies all selected filters.

4. Given no eligible partners, when the search finishes, then the system displays "No matching study partners found."

5. Given missing profile information or availability, when the student searches, then the system identifies what they need to complete.

### Notes

Proposed matching score:

- 1 point for each distinct shared subject.
- 1 point for the same university.
- 1 point for the same study level.
- 1 point for the same preferred study style.
- Overlapping availability is an eligibility condition.
- Equal scores are ordered by a stable student identifier.

Example: Two shared subjects, the same university and study style,but different study levels, give a score of 4.

The student's own account and suspended accounts are excluded. Searches do not automatically create groups or memberships.

## US-07 — Create a Study Group

**User story:** As a student, I want to create a study group so that other students can join me to study a particular subject.

**Priority:** Must have
**Size:** M

### Acceptance Criteria

1. Given a group name, subject and integer member limit of at least 2, when the student submits the form, then the system creates a group.

2. Given successful creation, when the group opens, then its creator is the organiser and first member, and the group appears in searches.

3. Given missing required information or an invalid limit, when the student submits the form, then the system identifies the errors and does not create the group.

### Notes

- The member limit includes the organiser.
- A group may have an optional description.
- The organiser is a student role within a particular group.

## US-08 — Find and Join a Study Group

**User story:** As a student, I want to find and join a study group so that I can study my chosen subject with others.

**Priority:** Must have
**Size:** M

### Acceptance Criteria

1. Given a selected subject, when the student searches for groups, then the system lists matching groups with their organiser,member count and member limit.

2. Given a group with an available place, when a non-member selects "Join", then the system adds them and updates the member count.

3. Given a full group or an existing membership, when a join request is processed, then the system rejects it without changing memberships and explains the reason.

4. Given no matching groups, when the search finishes, then the system displays "No matching study groups found."

### Notes

- Membership does not require organiser approval.
- Students may belong to multiple groups.
- Concurrent requests must not exceed the group's member limit.

## US-09 — Leave a Study Group

**User story:** As a group member, I want to leave a study group so that I can stop participating when it no longer meets my needs.

**Priority:** Must have
**Size:** S

### Acceptance Criteria

1. Given a member who is not the organiser, when they confirm leaving, then the system removes their membership and reduces the member count.

2. Given a former member, when they view their upcoming sessions, then sessions belonging to that group no longer appear.

3. Given the group organiser, when they attempt to leave, then the system explains that they must first transfer ownership or delete the group.

### Notes

- Leaving does not delete the student's account.
- A former member may rejoin if a place is available.
- Ownership transfer and group deletion are covered separately.

---

## US-10 — Transfer Group Ownership

**User story:** As a group organiser, I want to transfer ownership to another member so that the group can continue without me as organiser.

**Priority:** Should have
**Size:** S

### Acceptance Criteria

1. Given another active member of the group, when the organiser selects that member and confirms transfer, then the selected member becomes the sole organiser.

2. Given successful transfer, when the previous organiser views the
   group, then they remain a regular member and can leave normally.

3. Given a selected user who is not an active group member, when transfer is attempted, then the system rejects it and preserves the existing organiser.

### Notes

- Only the current organiser can transfer ownership.
- The group always has exactly one organiser.
- If this feature is deferred, organisers can delete their groups
  but cannot leave a continuing group.

## US-11 — Delete a Study Group

**User story:** As a group organiser, I want to delete my study group so that it no longer appears as available when it is no longer active.

**Priority:** Must have
**Size:** S

### Acceptance Criteria

1. Given a group organiser, when they confirm group deletion, then the system removes the group from search results and members' group lists.

2. Given future sessions belonging to the deleted group, when deletion completes, then the system cancels those sessions and no longer displays them as upcoming active sessions.

3. Given a regular member, when they attempt to delete the group, then the system denies the action and preserves the group.

### Notes

- Deletion requires explicit confirmation.
- Deleting a group does not delete its members' accounts.
- Data retention details belong to the technical design.

## US-12 — Schedule a Study Session

**User story:** As a group organiser, I want to schedule a study session so that members know when and where we will study.
**Priority:** Must have
**Size:** M

### Acceptance Criteria

1. Given a title, future start date and time, later end time and a location or meeting link, when the organiser submits the form,then the system creates a session belonging to their group.

2. Given successful creation, when a current group member opens their upcoming sessions, then the new session appears with its details.

3. Given missing required information or invalid times, when the organiser submits the form, then the system identifies the errors and does not create a session.

### Notes

- Only the group's organiser can schedule its sessions.
- Availability assists matching but does not restrict session scheduling.
- Booking a room or generating a video meeting link is outside scope.

## US-13 — Edit a Study Session

**User story:** As a group organiser, I want to edit an upcoming session so that members can see changes to our study plan.

**Priority:** Should have
**Size:** S

### Acceptance Criteria

1. Given an upcoming active session, when its group organiser saves valid changes, then the system updates its details.

2. Given an updated session, when group members reload their session list, then they see the new details.

3. Given invalid changes, when the organiser saves them, then the system identifies the errors and preserves the previous details.

### Notes

- The same field and time rules as session creation apply.
- Past and cancelled sessions cannot be edited.
- Automated change notifications are outside scope.

## US-14 — Cancel a Study Session

**User story:** As a group organiser, I want to cancel an upcoming session so that members know it will not take place.

**Priority:** Must have
**Size:** S

### Acceptance Criteria

1. Given an upcoming active session, when its organiser confirms cancellation, then the system marks the session as cancelled.

2. Given a cancelled session, when members view upcoming sessions,then the system clearly labels it "Cancelled".

3. Given a regular member, when they attempt to cancel the session,then the system denies the action and preserves the session.

### Notes

- Cancellation does not delete the study group.
- Cancelled sessions cannot be reactivated in this version.

## US-15 — View Upcoming Study Sessions

**User story:** As a student, I want to view my upcoming study sessions so that I can plan my time.

**Priority:** Must have
**Size:** S

### Acceptance Criteria

1. Given membership in groups with future sessions, when the student opens their session list, then the system displays those sessions ordered by start date and time.

2. Given a displayed session, when the student opens its details,then the system shows its title, group, start and end times,location or meeting link, and cancellation status.

3. Given no future sessions in the student's groups, when the list opens, then the system displays "No upcoming study sessions."

### Notes

- Session details are available only to current group members.
- Past sessions are excluded.
- Cancelled future sessions remain visible with a cancellation label.

## US-16 — Report a User

**User story:** As a student, I want to report inappropriate behaviour so that an administrator can review it.

**Priority:** Should have
**Size:** S

### Acceptance Criteria

1. Given another student's profile and a non-empty report reason,when the student submits a report, then the system saves it as pending and confirms submission.

2. Given an empty reason, when the student submits the report, then the system requests a reason and does not save it.

3. Given the student's own profile, when they view available actions, then reporting their own account is not available.

### Notes

- Reports record the reporter, reported student, reason and time.
- Report content is accessible only to administrators after submission.
- Reporting does not automatically suspend an account.

## US-17 — Review Reports

**User story:** As an administrator, I want to review and resolve reports so that I can address reported behaviour.

**Priority:** Should have
**Size:** M

### Acceptance Criteria

1. Given an authenticated administrator, when they open reports,then the system lists reports and supports filtering by pending or resolved status.

2. Given a report, when the administrator opens it, then the system displays its reporter, reported student, reason and submission time.

3. Given a pending report and a written review outcome, when the administrator resolves it, then the system records the outcome,administrator and resolution time, and marks it resolved.

### Notes

- Reviewing a report does not automatically change account status.
- Administrators may resolve a report without taking account action.
- Account status management is covered in US-18.

## US-18 — Manage Student Account Status

**User story:** As an administrator, I want to suspend and reactivate student accounts so that I can enforce rules and restore access when appropriate.

**Priority:** Should have
**Size:** M

### Acceptance Criteria

1. Given an active student account and an action reason, when the administrator confirms suspension, then the system marks it suspended, invalidates its sessions and rejects further protected actions.

2. Given a suspended student account, when partner searches run,then the account does not appear in results.

3. Given a suspended account and an action reason, when the administrator confirms reactivation, then the system marks it active and permits login with valid credentials.

4. Given a status change, when it completes, then the system records the administrator, reason, action and time.

### Notes

- Administrators can find accounts by email or student identifier.
- Account status changes do not require an existing report.
- Suspension preserves profiles and group memberships.
- A suspended organiser cannot manage groups until reactivation.
- Managing administrator accounts is outside scope.