# Identity API contract (Checkpoint 3)

Base path: `/api/v1`. All IDs: UUID. JSON camelCase. UTC ISO 8601 timestamps. JWT bearer on protected endpoints. Roles: STUDENT / ADMIN.

**Important:** The illustrative server URL is not deployed. Passwords are never returned; hashes use Argon2id. JWT `sub` contains the user UUID, role is verified server-side, and protected operations also check account status and ownership/ADMIN privileges. Tokens should be short-lived (example 3600 seconds); production revocation/session management is a future design decision. Account suspension must prevent subsequent authenticated requests; immediate revocation of already-issued tokens requires a database status check or token revocation strategy.

`POST /auth/register`, `POST /auth/login`, `GET/PATCH /users/me`, `PATCH /admin/users/{userId}/status`. Logout is **not** specified until a token revocation/refresh-session model is agreed.

Normal and abnormal flows are in the diagrams. HTTP 409 for duplicate email or invalid status transition; 422 for semantically invalid data; 401 for invalid credentials or JWT; 403 for inactive account or insufficient permissions. Audit every admin status change. Admin cannot suspend their own account in this design.

OpenAPI is modular: this file is a complete standalone OpenAPI document. When merging, combine paths and component schemas carefully, retaining unique names.
