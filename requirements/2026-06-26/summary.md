# Summary: Issue #72 Date Format

**Objective:**
Change the date display format in the Admin dashboard (`BookingsManager.jsx`) from `YYYY-MM-DD` to `DD-MM-YYYY`.

**Changes Implemented:**
1. **Utility Extension (`src/utils/date.js`)**:
   - Added `fmtDateDDMMYYYY(dateStr)` which splits standard `YYYY-MM-DD` strings by `-` and reassembles them in `DD-MM-YYYY` format.
2. **UI Update (`src/components/admin/BookingsManager.jsx`)**:
   - Replaced raw output of `{r.bookingDate}` and `{r.bookingEndDate || r.bookingDate}` in the `<span className="cell-date">` elements with the new `fmtDateDDMMYYYY()` formatter.

**Verification:**
- **Code Stability**: Ran `npm run lint` and `npm run build` successfully with no new errors.
- **Data Integrity**: Filtering and logic states continue to use standard `YYYY-MM-DD`, ensuring that filtering functionality is not broken while only the display is updated.

**Outcome:**
- Committed to branch `feat/admin-date-format`.
- Created Pull Request resolving Issue #72.
