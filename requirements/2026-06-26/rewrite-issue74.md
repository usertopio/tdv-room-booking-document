# Issue #74: Promotion Description in Admin Table

**Goal:**
Display the duration (e.g., "3 ปี") under the selected promotion package in the admin Bookings Manager table.

**Requirements:**
1. In the `BookingsManager.jsx` table, locate the column that displays the selected promotion package.
2. Cross-reference the booking's `promotion` ID (e.g., "A", "B", "C", "D") with the `PROMOTIONS` array from `src/data/floorPlansData.js`.
3. Render a `<div className="cell-meta">X ปี</div>` beneath the promotion ID, where X is the `years` property of the matched promotion.
