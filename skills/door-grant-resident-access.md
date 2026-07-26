---
name: Grant a resident access to doors
description: Provision a DOOR/OpenDOOR user and grant them access to one or more doors or keys in a building, using a partner-scoped token.
api: openapi/door-partner-openapi.json
operations: [authenticatePartner, listBuildings, listDoors, listKeys, grantAccessToUserV2]
---

# Grant a resident access to doors (OpenDOOR)

Use this to onboard a resident/end-user and give them physical access to doors.

## Safety
Granting access actuates physical door locks. Treat this as a **safety-critical** action:
confirm the target building, doors, and user with a human before calling, and use a
short-lived partner-scoped token. See `agentic-access/door-agentic-access.yml`.

## Auth
1. Obtain a partner-scoped (M2M) JWT via `authenticatePartner` against
   `https://auth.prod.latch.com/oauth/token` using the partner `client_id`/`client_secret`.
2. Send it as `Authorization: Bearer <token>` on every Partner API call
   (base URL `https://rest.latchaccess.com/access/sdk`).

## Steps
1. `listBuildings` - find the target building and note its `uuid`.
2. `listDoors` (and/or `listKeys`) - resolve the `doorUuid`/`keyUuid` values to grant.
3. `grantAccessToUserV2` - POST `/v2/users` with the user's `email` (or `phone`),
   `firstName`/`lastName`, the `doorUuids`/`keyUuids`, `startTime`/`endTime`, `passcodeType`,
   `role`, and `shareable`. V2 returns a richer response than `grantAccessToUser` (V1).

## Rules
- Provide exactly one of `email` or `phone` (400 `EMAIL_AND_PHONE_PROVIDED` / `EMAIL_OR_PHONE_REQUIRED`).
- Permanent access requires an email (400 `EMAIL_REQUIRED_FOR_PERMANENT`).
- Validate `startTime`/`endTime` (400 `INVALID_START_TIME` / `INVALID_END_TIME` / `MISSING_END_TIME`).
- On 401 `UNAUTHORIZED`, refresh the token and retry. Errors return `{ "message": "<CODE>" }`
  (see `errors/door-error-codes.yml`); the API is not RFC 9457.
- No idempotency key exists; do not blindly retry a POST that may have succeeded.
