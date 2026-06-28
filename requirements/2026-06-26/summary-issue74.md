# Summary: Issue #74 Promotion Description

**Objective:**
Display the duration of the selected package (e.g., "3 ปี") under the package ID in the Admin dashboard's Bookings table.

**Changes Implemented:**
1. **Import `PROMOTIONS`**: Updated `src/components/admin/BookingsManager.jsx` to import the `PROMOTIONS` array from `src/data/floorPlansData.js`.
2. **Render Meta Tag**: Located the table cell rendering the `r.promotion` ID (e.g., "C"). Below it, implemented logic to find the matched promotion object and display its `years` property wrapped in `<div className="cell-meta">X ปี</div>`.

**Verification:**
- **Build Checks**: Passed `npm run lint` and `npm run build` without errors.
- **Rendering**: The `cell-meta` UI component perfectly matches the styling used in other columns (like the phone number beneath the company name) to maintain a cohesive UI.

**Outcome:**
- Committed to branch `feat/admin-promotion-meta`.
- Created Pull Request resolving Issue #74.
