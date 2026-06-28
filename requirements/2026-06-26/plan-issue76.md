# Implementation Plan: Admin Dropdown Colors

This plan outlines the steps to add dynamic colors to the application status dropdown in the admin dashboard, resolving Issue #76.

## Proposed Changes

### `src/index.css`
- **[MODIFY]** Locate `.admin-status-dropdown` at line ~1260.
- **[MODIFY]** Append the following specific class overrides immediately after it to ensure they take precedence:
  - `.admin-status-dropdown.pending` (Yellow)
  - `.admin-status-dropdown.reviewing` (Blue)
  - `.admin-status-dropdown.approved` (Light Green)
  - `.admin-status-dropdown.payment-submitted` (Orange)
  - `.admin-status-dropdown.completed` (Green)
  - `.admin-status-dropdown.cancelled` and `.admin-status-dropdown.rejected` (Red)
- **[MODIFY]** Each rule will specify a `background`, `border-color`, and `color`.

## Verification Plan
### Automated Tests
- Run `npm run lint` and `npm run build` to verify CSS changes don't break the build.

### Manual Verification
- Open the Admin Dashboard > รายการจอง.
- Verify that the dropdown in the "สถานะใบสมัคร" column is colored according to the selected value instead of being plain gray.
