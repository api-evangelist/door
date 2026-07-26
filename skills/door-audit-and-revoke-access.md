---
name: Audit and revoke resident access
description: List a building's residents and a key's accesses, then update or revoke a user's access to a specific door.
api: openapi/door-partner-openapi.json
operations: [authenticatePartner, listBuildings, listBuildingResidents, listKeyAccesses, getUser, updateUserAccess, revokeUserAccess]
---

# Audit and revoke resident access (OpenDOOR)

Use this to review who has access and to change or remove it.

## Safety
`updateUserAccess` and `revokeUserAccess` change physical access rights. Treat as
**safety-critical**: confirm the user, door, and intent with a human first.

## Auth
Obtain a partner-scoped JWT via `authenticatePartner` and send it as `Authorization: Bearer <token>`
to the Partner API at `https://rest.latchaccess.com/access/sdk`.

## Steps
1. `listBuildings` - pick the building `uuid`.
2. `listBuildingResidents` - GET `/v1/buildings/{buildingUuid}/residents` to see residents with
   active accesses; or `listKeyAccesses` (`/v1/keys/{keyUuid}/accesses`) to see all accesses on a key.
3. `getUser` - GET `/v1/users/{userUuid}` to inspect a single user's accesses.
4. To change a grant: `updateUserAccess` - PATCH `/v1/users/{userUuid}/doors/{doorUuid}`.
5. To remove a grant: `revokeUserAccess` - DELETE `/v1/users/{userUuid}/doors/{doorUuid}`.

## Rules
- Paginate list calls with `pageSize` + `pageToken`.
- Some passcode types cannot be revoked (400 `PASSCODE_TYPE_CANT_BE_REVOKED`).
- Revoke/update return 200/204 on success; 404 `NOT_FOUND` if the user or door does not exist.
- Errors are `{ "message": "<CODE>" }` (see `errors/door-error-codes.yml`).
