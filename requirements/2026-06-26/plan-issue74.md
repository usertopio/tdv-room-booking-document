# Implementation Plan: Promotion Meta in Admin

This plan details the steps to add the promotion duration under the package ID in the admin table, addressing Issue #74.

## Proposed Changes

### `src/components/admin/BookingsManager.jsx`
- **[MODIFY]** Import `PROMOTIONS` from `../../data/floorPlansData`.
- **[MODIFY]** In the table rendering logic (around line 588), update the "แพ็คเกจ" cell:
  - Continue to display the `r.promotion || '—'`.
  - Add logic to find the matched promotion: `const promoData = PROMOTIONS.find(p => p.id === r.promotion);`.
  - If `promoData` exists, render `<div className="cell-meta">{promoData.years} ปี</div>` beneath the package name.

## Verification Plan
### Automated Tests
- Run `npm run lint` and `npm run build` to ensure no imports are broken or syntax errors are introduced.

### Manual Verification
- Open the Admin Dashboard > รายการจอง.
- Verify that a booking with Package C displays "C" with a smaller text "3 ปี" directly below it.
- Ensure that bookings without a promotion still just show "—" without breaking.
