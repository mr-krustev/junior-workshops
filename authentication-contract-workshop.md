# Junior Authentication Contract Workshop

## Goal

- Practice establishing a contract between frontend and backend
- Build a simple TypeScript authentication flow
- Roles: 1 backend junior, 1 frontend junior
- Timebox: ~2 hours

## Stack

- Everything in TypeScript
- Frontend: simple UI (React, Vue, or vanilla TS — pick one)
- Backend: simple HTTP API (Express, Fastify, or similar — pick one)
- Persistence: in-memory is enough (no real DB required)
- Auth: simple token or session (no JWT refresh / OAuth implementation)

## Workshop Phases

### Phase 1 — Separate planning (15 min)

Each junior works alone. Write a short plan of what **you** need.

**Frontend junior lists:**

- Pages / screens
- Form fields (required vs optional)
- Client-side validation
- Loading / error / success states
- DTOs you expect to send and receive
- What you need after login (token? user object? profile?)

**Backend junior lists:**

- Entities / storage shape
- Endpoints (path, method, auth required?)
- Request / response DTOs
- Validation and error codes
- How passwords are stored
- How a session / token is issued and checked

### Phase 2 — Contract alignment (20 min)

Get together. Compare plans and agree on one contract.

- Endpoint paths, HTTP methods, status codes
- Shared DTO names and field types
- Required vs optional vs nullable
- Error response shape
- What is never returned (password, hash, internal ids?)
- User vs Profile: 1 entity or 2?
- Adjust both plans so the two apps can collaborate

Write the agreed types down before coding. Treat them as the source of truth.

### Phase 3 — Execution (75 min)

- Build the two minimal apps
- Share / copy the agreed types
- Wire them up and make the flow work end to end
- If the contract is wrong, update it together — do not silently diverge

### Demo & reflection (10 min)

- Register → login → open profile → update profile → logout
- Show invalid login and unauthorized access
- Discuss: where did the contract change? which helper types helped?

---

## Frontend Requirements

### Login Page

- Fields: username, password
- Actions: **Login** / **Register** (button or link to register)
- On success: store auth token / session, go to Profile
- On failure: show a clear error (wrong credentials, server error)

### Register Page

- Required: username, password
- Optional fields (pick 2–3 only):
  - display name
  - bio
  - avatar URL
- On success: either auto-login or redirect to Login
- Client validation: required fields, password min length

**Think about (do not implement):**

- Account verification
  - email / phone confirm after register?
  - `unverified` vs `active` status?
  - extra endpoints (`POST /verify`, resend code)?
  - UI: blocked profile until verified?
- OAuth (Google / Facebook / etc.)
  - extra identity fields on User?
  - different login entry points?
  - what happens if the same email exists as a password account?

### Profile Page

- Available only when authenticated
- Show current user / profile data
- Edit selected profile fields
- Logout

**What else could a registered user save?** (pick a few to discuss, implement at most 2–3)

- display name
- bio
- avatar URL
- location
- preferences (theme, language, notifications)

### Frontend states to cover

- loading
- validation errors
- success
- unauthorized (401) → redirect to Login
- server / unexpected error

---

## Backend Requirements

### Endpoints

| Method | Path | Auth | Purpose |
| --- | --- | --- | --- |
| `POST` | `/register` | no | create account |
| `POST` | `/login` | no | authenticate, return token + public user |
| `GET` | `/me` | yes | current profile |
| `PATCH` | `/me` | yes | update profile |
| `POST` | `/logout` | yes | optional; invalidate token if using sessions |

Suggested status codes:

- `200` / `201` success
- `400` validation
- `401` missing / invalid / expired auth
- `409` username already taken
- `500` unexpected

### Entities for storing data

Minimum fields to think about:

- User
  - `id`
  - `username`
  - `passwordHash` (never return this)
  - `createdAt`
- Profile
  - `userId`
  - optional: `displayName`, `bio`, `avatarUrl`

Passwords:

- hash on register
- compare hash on login
- never store or return plain passwords

### User vs Profile — discuss before coding

- 1 entity or 2?
  - **One entity:** simpler for a 2-hour workshop
  - **Two entities:** User = credentials / auth; Profile = public / editable data
- 1:1 now, or could a user have multiple profiles later?
- How do you load a profile given a user id?

**Monolith vs microservice (discuss only):**

- Monolith: User + Profile in one service / one storage — easier contract, one deploy
- Microservice: Auth service vs Profile service
  - who owns `/me`?
  - how does Profile trust the token?
  - what is the shared user id?

### Auth / authorization

- Issue a simple token (or session id) on login
- Protect `/me` (GET + PATCH)
- Reject missing / invalid tokens with `401`
- Safe responses: omit `password` / `passwordHash`

---

## Shared Contract

Agree on these types (names can change, fields should not silently diverge).

```ts
type User = {
  id: string;
  username: string;
  passwordHash: string;
  createdAt: string;
};

type Profile = {
  userId: string;
  displayName?: string;
  bio?: string;
  avatarUrl?: string;
};

type PublicUser = Omit<User, "passwordHash">;
type PublicProfile = Profile;
type AuthUser = PublicUser & { profile: PublicProfile };

type RegisterDto = Pick<User, "username"> & {
  password: string;
} & Partial<Pick<Profile, "displayName" | "bio" | "avatarUrl">>;

type LoginDto = Pick<User, "username"> & {
  password: string;
};

type AuthResponse = {
  token: string;
  user: AuthUser;
};

type UpdateProfileDto = Partial<Pick<Profile, "displayName" | "bio" | "avatarUrl">>;

type ApiError = {
  error: string;
  details?: string[];
};
```

### TypeScript helper-type exercise

Compose types from the base entities. Do not duplicate field lists if you can derive them.

- `Pick` — take only the fields a form / endpoint needs
- `Omit` — strip secrets from responses (`passwordHash`)
- `Partial` — update DTOs and optional register fields
- `Required` — e.g. `Required<Pick<Profile, "displayName">>` if a field becomes mandatory later
- Intersection (`&`) — combine User + Profile for the public auth payload

Try at least:

- derive `PublicUser` from `User`
- derive `RegisterDto` from `User` + `Profile`
- derive `UpdateProfileDto` with `Partial`

### Contract checklist (fill this in during Phase 2)

- [ ] Endpoint paths + methods
- [ ] Request body for register / login / update
- [ ] Response body for auth + profile
- [ ] Optional vs required vs nullable
- [ ] Error shape + status codes
- [ ] Token: where it lives (header? cookie?) and the header name
- [ ] Username uniqueness rules
- [ ] Password rules (min length, etc.)

---

## Scope Guardrails

**In scope**

- username / password register + login
- 2–3 optional profile fields
- authenticated profile read / update
- logout
- shared TS types using helper types

**Out of scope**

- real OAuth (Google / Facebook)
- sending verification emails / SMS
- password reset
- JWT refresh rotation
- production database / migrations
- deployment
- polished CSS / design system
- more than 3 optional profile fields

Keep it small enough to finish in ~2 hours.

---

## Acceptance Criteria

The pair is done when all of these work:

- [ ] Register a new user (required + optional fields)
- [ ] Duplicate username is rejected
- [ ] Login with valid credentials
- [ ] Login with invalid credentials shows an error
- [ ] Authenticated user can read their profile
- [ ] Authenticated user can update their profile
- [ ] Unauthenticated request to `/me` is rejected (`401`) and UI sends the user to Login
- [ ] Logout clears client auth state
- [ ] Password / hash never appears in API responses
- [ ] Frontend and backend use the same DTO names / shapes

---

## Reflection

- Where did Phase 1 plans disagree?
- What did you change in Phase 2 to make the parts fit?
- Did the contract break during implementation? What did you update?
- Which helper types (`Pick`, `Omit`, `Partial`, `Required`) actually reduced duplication?
- If you added verification or OAuth next, what would change in the contract first?
