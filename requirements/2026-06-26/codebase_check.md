# Codebase Check: Issue #72 Date Format

1. **Target Files**: 
   - `src/components/admin/BookingsManager.jsx`: This file contains the main admin data table where `r.bookingDate` and `r.bookingEndDate` are rendered inside `<span className="cell-date">`.
   - `src/utils/date.js`: Contains utility functions for formatting dates (`fmtDate`, `fmtDateTime`, `localDateISO`, etc.).
2. **Current Implementation**:
   - Dates from the backend are mostly ISO strings (`YYYY-MM-DD`).
   - `BookingsManager.jsx` renders `{r.bookingDate}` directly, outputting `YYYY-MM-DD`.
3. **Proposed Changes**:
   - Create a new utility function `fmtDateDDMMYYYY(dateStr)` in `src/utils/date.js`.
   - Update `BookingsManager.jsx` to use `fmtDateDDMMYYYY(r.bookingDate)` and `fmtDateDDMMYYYY(r.bookingEndDate || r.bookingDate)` instead of rendering the raw strings.
