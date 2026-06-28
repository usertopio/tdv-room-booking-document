# Implementation Plan: Admin Date Format (DD-MM-YYYY)

This plan outlines the steps to change the date format in the admin dashboard from `YYYY-MM-DD` to `DD-MM-YYYY` as requested in Issue #72.

## Proposed Changes

### `src/utils/date.js`
- **[MODIFY]** Add a new exported function `fmtDateDDMMYYYY(dateStr)`.
  - It will split a `YYYY-MM-DD` string by `-` and return `DD-MM-YYYY`.
  - Fallback to the original string if the format is unexpected.

### `src/components/admin/BookingsManager.jsx`
- **[MODIFY]** Import `fmtDateDDMMYYYY` from `../../utils/date`.
- **[MODIFY]** Wrap the rendering of `r.bookingDate` and `r.bookingEndDate` in the `cell-date` spans with `fmtDateDDMMYYYY()`.
  - Example: `<span className="cell-date">{fmtDateDDMMYYYY(r.bookingDate)}</span>`

## Verification Plan

### Automated Tests
- Run `npm run lint` and `npm run build` to ensure no syntax errors.

### Manual Verification
- Render the `BookingsManager` component.
- Verify that the "วันที่จอง" column displays dates as `DD-MM-YYYY` (e.g. `25-06-2026`).
- Verify that date filtering and sorting still work as expected (since underlying data state is unmodified).
