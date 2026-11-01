# StudyMatch activity diagrams (Checkpoint 2, Section 3)

Eray's three activity flows cover finding a study partner, joining a group, and creating or editing a study session. Each includes successful and rejected paths.

[Presentation PDF](StudyMatch-Activity-Diagrams.pdf): three brief slides matching the shared deck. The PNGs and editable draw.io source below provide the detailed flows behind them.

| Activity | Diagram | User stories |
| --- | --- | --- |
| Find compatible study partners | [PNG](diagrams/01_partner_matching.png) | US-06 |
| Find and join a study group | [PNG](diagrams/02_join_group.png) | US-08 |
| Schedule or edit a study session | [PNG](diagrams/03_plan_session.png) | US-12, US-13 |

[Editable draw.io source](sources/StudyMatch-Activity-Diagrams.drawio) contains all three diagrams (AD-01, AD-02 and AD-04).

## Key rules

- Matching requires a shared subject and overlapping availability; missing profile data and empty results are handled.
- Joining requires an active group, a free place and no existing membership. Capacity checking and insertion are atomic.
- Only the organiser can create or edit sessions. Start must be in the future, end must be later, and a location or meeting link is required. Failed edits preserve saved data.

Solid dots mark the start, diamonds mark decisions, and bullseyes mark alternative end points. These are design diagrams, not executed tests. See the existing [user stories](../user-stories.md) and [class diagrams](../class-diagrams/README.md) for context.
