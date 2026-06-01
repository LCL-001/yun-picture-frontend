# Personal Info Feature Log - 2026-05-16

## Scope

- Add a user-profile entry to the top-right user dropdown.
- Fetch profile data from the existing `/api/user/get/vo` API wrapper.
- Show the information in the current page without adding a new route.

## Changed Files

- `src/components/GlobalHeader.vue`
  - Added a `个人信息` dropdown option.
  - Added a modal that fetches and displays current user information via `getUserVoByIdUsingGet`.
  - Styled the modal to match the existing clean blue/white UI direction.

## Verification

- `npm run pure-build` passed.
