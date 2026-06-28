# Codebase Check: Issue #76 Dropdown Colors

1. **Target Files**:
   - `src/components/admin/BookingsManager.jsx`: Renders the `<select className="admin-status-dropdown status-badge {status}">`.
   - `src/index.css`: Contains CSS definitions for `.admin-status-dropdown` (Line ~1260) and `.status-badge` (Line ~196).

2. **Current Implementation**:
   - The dropdown has both `.admin-status-dropdown` and `.status-badge.[status]` classes.
   - However, `.admin-status-dropdown` is defined *after* `.status-badge` in `index.css`. Due to CSS cascading rules, the `background: var(--surface2)` and `color: var(--text)` from `.admin-status-dropdown` override the `.status-badge` colors.
   - This results in the dropdown always appearing gray.

3. **Proposed Changes**:
   - Add specific overriding rules directly beneath `.admin-status-dropdown` in `index.css` for each state (e.g., `.admin-status-dropdown.pending`, `.admin-status-dropdown.approved`).
   - Include custom border colors for each state to enhance the visual effect.
