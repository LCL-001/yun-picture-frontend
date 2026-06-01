# User Date Time Display Log - 2026-05-16

## Context

- Fixed date display for user management and personal profile views.
- Backend user date fields may come from `java.util.Date`, which can be serialized as a string, timestamp, array, or object depending on JSON configuration.

## Changes

- Added `formatDateTime` in `src/utils/index.ts`.
- Updated personal profile modal in `src/components/GlobalHeader.vue` to format `createTime` and `vipExpireTime`.
- Updated user management table in `src/pages/admin/UserManagePage.vue` to format `createTime`.

## Files Modified

- `src/utils/index.ts`
- `src/components/GlobalHeader.vue`
- `src/pages/admin/UserManagePage.vue`
