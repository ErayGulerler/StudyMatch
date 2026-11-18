# Profile & Matching: Checkpoint 3

Eray Gulerler. US-04 profile, US-05 weekly availability, US-06 partner matching.

## Agreed endpoints
Confirmed in Bade's endpoint message supplied on 19 November 2026.

| Method | Path | Success |
|---|---|---|
| GET | `/api/v1/profiles/me` | 200 Profile |
| PATCH | `/api/v1/profiles/me` | 200 Profile; proposed 201 on first complete save |
| GET | `/api/v1/profiles/{userId}` | 200 PublicProfile |
| GET | `/api/v1/availability/me` | 200 Availability, possibly empty |
| PUT | `/api/v1/availability/me` | 200 Availability, whole schedule replaced |
| GET | `/api/v1/matches` | 200 Results, possibly empty |

Example: `GET /api/v1/matches?subject=Mathematics&mode=ONLINE`.

`subject` is a catalogue name resolved to UUID(s), using trimmed case-insensitive exact matching. Profile `subjects` remain UUID arrays. `mode` filters supported meeting modes; optional `university` retains the US-06 filter. All supplied filters combine with AND. Unknown subject or invalid mode returns 422.

## Files
- `../api/matching.yaml`: standalone OpenAPI 3.0.3 module.
- `sources/matching_*.puml` and `diagrams/matching_*.svg`: seven editable/rendered sequences.
- `StudyMatch_CP3_Profile_Matching_Eray_Gulerler - Needs Integration Review.pdf`: submission copy.

## Successful and abnormal flows
- Shared authentication guard: verified JWT, ACTIVE account and STUDENT role; 401 invalid/expired/revoked session, 403 wrong role or suspended account. Every business sequence references this guard. PostgreSQL failure returns 500.
- Read own profile: 200; 404 if it has never been saved.
- PATCH profile: merge supplied fields, preserve omitted values, replace supplied arrays, validate then atomically save; 422 on empty/invalid patch. Proposed first save requires all original mandatory profile fields and returns 201 with Location; later changes return 200.
- Public profile: authenticated active students can view an ACTIVE STUDENT complete profile. 404 for absent/ineligible targets. Only public fields are returned; no email, credentials, tokens or private availability. Path userId selects a target and never grants permissions.
- Read availability: 200 with weekly slots; an unsaved schedule is an empty list.
- PUT availability: complete body with timeZone and items. Empty items clears the schedule. Validate all slots, then replace in one transaction. 422 preserves old data; 500 rolls back. Identical retries do not append slots. Last committed valid replacement wins.
- Matching: require shared subjects and positive overlapping availability; union overlaps, exclude self/inactive/non-STUDENT accounts, apply filters. 409 if own profile/availability is missing. 200 with items: [] when no candidates match.

All request bodies are JSON, fields camelCase, exposed account IDs UUIDs. Backend derives current owner from JWT sub; frontend owner IDs and roles are rejected. Error format: `{"error":{"code":"FORBIDDEN","message":"Permission denied.","timestamp":"2026-11-19T00:00:00Z"}}`. Malformed format/unknown fields: 400; invalid field values: 422. All timestamp instants use UTC ending Z. Weekly weekday/HH:mm recurrence values retain Europe/Warsaw and are not timestamp instants.

## Prior-model alignment
`userId` in the wire contract maps to the existing class model's `studentId` and Identity's JWT sub. Subject names in the query resolve to the existing UUID subject IDs. Slot IDs remain internal UUIDs; no per-slot REST operations are exposed. Adding, editing and removing slots in US-05 are represented by replacing the complete schedule. Matching score remains distinct shared subjects + one each for same university, studyLevel and preferredStyle. Sort by score descending then canonical userId ascending. Mode is a filter, not a score bonus.

## Integration decisions to confirm
Endpoint names are now confirmed. Two choices were not specified in the endpoint message:
1. First profile: this module proposes creating it through a complete initial PATCH (201). Confirm this or agree that Identity provisions it; no extra profile endpoint has been invented.
2. Modes: ONLINE is confirmed by the example; IN_PERSON and optional profile/public-profile studyModes are proposed. Confirm the complete enum and update the earlier class model/user story at integration. Missing studyModes means unspecified and does not satisfy a mode filter; it does not prevent unfiltered matching.

Before the final shared specification, align these fields with the other modules and replace the non-production HTTPS host. Merge the four module YAML files into openapi.yaml together. This branch edits only matching deliverables, not teammates' modules or shared openapi.yaml.

Validation: OpenAPI 3.0.3; six exact method/path pairs; schema/example validation; partial profile updates; rejected owner/role injection; empty schedule acceptance; invalid input and private public-profile fields rejected; seven PlantUML sources rendered to SVG. All PDF pages visually checked separately before submission.
