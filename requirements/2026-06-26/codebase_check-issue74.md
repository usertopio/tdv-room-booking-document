# Codebase Check: Issue #74 Promotion Meta

1. **Target Files**:
   - `src/components/admin/BookingsManager.jsx`: Renders the admin table. The promotion is currently rendered as `{r.promotion || '—'}` in a `<td>`.
   - `src/data/floorPlansData.js`: Contains the `PROMOTIONS` constant which holds objects like `{ id: 'C', label: 'แพ็คเกจ C', desc: 'เช่า 3 ปี', color: '#a855f7', years: 3 }`.

2. **Current Implementation**:
   - `BookingsManager.jsx` does not import `PROMOTIONS` currently.
   - The promotion cell only shows the string value of `r.promotion` without extra metadata.

3. **Proposed Changes**:
   - Import `PROMOTIONS` into `BookingsManager.jsx`.
   - In the table rendering loop, find the matching promotion object: `const promoData = PROMOTIONS.find(p => p.id === r.promotion);`.
   - Add `<div className="cell-meta">{promoData.years} ปี</div>` beneath the promotion ID string.
