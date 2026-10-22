## Use Cases

## Scope

This document groups the StudyMatch user stories into use cases per module and shows them on UML use case diagrams.


## Modules

The system consists of a Student Application, an Admin Panel and a Backend Server. The backend is divided into four modules that communicate through services.

- Identity & Access: Accounts, login, sessions and account status.
- Profile & Matching: Study profiles, weekly availability and partner search.
- Group & Session: Study groups, memberships and study sessions.
- Moderation: User reports, report review and account suspension.

The Student Application and the Admin Panel are client modules. Students use the Student Application and administrators use the Admin Panel.


## Services

- UC-05 Validate session: Provided by Identity & Access. Used by Profile & Matching, Group & Session and Moderation.
- UC-06 Look up account: Provided by Identity & Access. Used by Profile & Matching and Moderation.
- UC-07 Change account status: Provided by Identity & Access. Used by Moderation.
- UC-13 Provide public profile: Provided by Profile & Matching. Used by Group & Session and Moderation.

## Actors

### Human Actors

- Visitor: A person without an account who can register.
- Registered User: A student or administrator who can log in and log out.
- Student: A user with an active student account.
- Group Organiser: A student who organises a particular study group.
- Administrator: A user who reviews reports and manages student account status.

### Non-Human Actors

- Identity & Access module: Validates sessions and provides account information.
- Profile & Matching module: Provides public study profiles.
- Group & Session module: Uses session validation and public profiles.
- Moderation module: Uses session validation, account lookup, account status changes and public profiles.

## Notation

- Stick figure: Actor.
- «module»: Non-human actor representing another module.
- Oval: Use case.
- «service»: Use case provided to other modules.
- Box: Module boundary.
- Solid line: The actor participates in the use case.
- Hollow triangle: Generalisation between actors.
- «include»: The base use case always performs the included use case.
- «extend»: The extending use case optionally adds behaviour to the base use case.
