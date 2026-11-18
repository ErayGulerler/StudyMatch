# Profile & Matching: Checkpoint 3

Eray Gulerler. US-04 profile, US-05 weekly availability, US-06 partner matching.

## Files
- `../api/matching.yaml`: standalone OpenAPI 3.0.3 module, ready for contract review.
- `sources/matching_*.puml`: editable sequence diagrams.
- `diagrams/matching_*.svg`: actual PlantUML SVG exports of those sources.
- `StudyMatch_CP3_Profile_Matching_Eray_Gulerler - Needs Endpoint Confirmation.pdf`: readable submission copy.

## Standards and scope
Frontend -> REST Controller -> Service -> PostgreSQL. REST/JSON over HTTPS, `/api/v1` paths, camelCase fields and UUID identifiers. All protected requests use `Authorization: Bearer <access_token>`. Backend validation checks JWT signature, issuer, audience, expiry, account status and revocation; STUDENT role and ownership come from verified identity. ADMIN is defined by Identity but cannot use these student-only operations. Each diagram has Student, Frontend, REST Controller, Service and PostgreSQL participants, success responses and UML alt branches. The authentication diagram is referenced by every business diagram; it changes no Identity module file.

Every error uses `{ "error": { "code": "...", "message": "...", "timestamp": "...Z" } }`. 400 covers malformed request format; 401 invalid authentication; 403 denied role/status; 404 absent or non-owned resource; 409 profile not ready for matching; 422 invalid field values; 500 unexpected failure. Storage failures roll back writes and expose no internals. First profile PUT returns 201; replacing a profile returns 200. Slot creation returns 201, edit 200, deletion 204 with no body. Unexpected input fields are rejected; frontend owner/role fields never grant authorization.

## Successful and abnormal flows
| Source | Success | Key abnormal paths |
|---|---|---|
| matching_00_authentication | verified ACTIVE STUDENT principal | 401 invalid/expired/revoked token; 403 suspended/wrong role |
| matching_01_get_profile | 200 Profile | 404 no saved profile |
| matching_02_put_profile | 201 first profile / 200 replacement | 422 invalid required fields or unknown subjects; old data preserved |
| matching_03_get_availability | 200 Availability, including empty list | 500 storage failure |
| matching_04_post_availability | 201 Slot + Location | 422 weekday/range invalid |
| matching_05_patch_availability | 200 updated Slot | 404 absent/not owned; 422 invalid merged slot |
| matching_06_delete_availability | 204, no response body | 404 absent/not owned |
| matching_07_get_matches | 200 Results, including items: [] | 409 profile/availability missing; 400 malformed query; 422 unknown subject |

## Alignment with earlier models
The existing StudyProfile/PublicProfile field `studentId` is the same account UUID called `userId` by Identity and carried in JWT sub. Existing availability fields remain `weekday`, `startTime`, `endTime`; `id` is a slot UUID. Weekly HH:mm values are recurring wall-clock slots, not dates or timestamps, so they retain Europe/Warsaw from US-05 and the class model. All actual timestamp instants use ISO 8601 UTC ending Z. Confirm this distinction during integration rather than converting recurring times to fixed UTC and breaking daylight-saving behaviour.

Matching uses shared subject + positive overlap for eligibility, unions overlapping slots before comparison, excludes self and SUSPENDED/non-STUDENT accounts, and never creates memberships. Score = distinct shared subjects + one each for same university, studyLevel and preferredStyle. Sort score descending then canonical UUID ascending. University comparison trims and case-folds; studyLevel uses its stored value. Empty results remain 200. Public results exclude email, credentials and tokens. Suspension revokes sessions and prevents actions in line with US-18 and the Account state diagram.

## Pending integration confirmation
No shared endpoint list or `docs/api/openapi.yaml` was present in the inspected main branch. The seven operations below are consistent within this module but require confirmation against the team's agreed specification:
- GET / PUT `/api/v1/profiles/me`
- GET / POST `/api/v1/availability/me`
- PATCH / DELETE `/api/v1/availability/me/{slotId}`
- GET `/api/v1/matches`

HTTPS host `studymatch.example.invalid` is a non-production placeholder. Match the deployment host and shared security-scheme name during merge. Components are prefixed `Matching` to avoid collisions. Only matching module files are included in this branch; the shared `openapi.yaml` and teammates' modules are untouched. This PR should remain unmerged until endpoint names are confirmed.

Sources: Leon CP3 https://leon.pw.edu.pl/mod/assign/view.php?id=4369; existing docs/user-stories.md, docs/class-diagrams/sources/02_profile_matching.dot and docs/state-diagrams/README.md; team technical standards supplied 18/11/2026.
