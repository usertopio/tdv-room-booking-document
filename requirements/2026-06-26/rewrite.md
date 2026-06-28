# Issue #72: Update Admin Date Format

**Goal:** Change the date display format in the Admin dashboard from `YYYY-MM-DD` (year-month-date) to `DD-MM-YYYY` (date-month-year) to improve readability for local users.

**Requirements:**
1. Identify all places in the Admin section where dates are displayed as `YYYY-MM-DD` (specifically `bookingDate` and `bookingEndDate`).
2. Convert these strings to `DD-MM-YYYY` before rendering.
3. Ensure sorting and filtering (which rely on the original ISO format) are not broken.
