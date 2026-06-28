# Codebase Check: Issue #78

## English
**Requirement:** Change the date format of the date filter buttons to `DD-MM-YYYY`.
**Finding:** **Not Found** (or rather, currently using native `<input type="date">` which does not format as `DD-MM-YYYY` by default).
**Evidence:**
- `src/components/admin/BookingsManager.jsx` (Lines 436-441): Uses `<input type="date">` for the date filter.
- `src/components/user/FloorPlan.jsx` (Lines 125-131): Uses `<input type="date">` for the date picker.
- The standard HTML `<input type="date">` depends on the browser's locale and cannot be strictly formatted to `DD-MM-YYYY` using just HTML attributes. A custom component wrapper is required.

## Thai
**ข้อกำหนด:** เปลี่ยนรูปแบบวันที่บนปุ่มตัวกรองวันที่ให้เป็น `DD-MM-YYYY`
**ผลการตรวจสอบ:** **ยังไม่ได้อิมพลีเมนต์** (ปัจจุบันใช้ `<input type="date">` ซึ่งไม่สามารถกำหนดฟอร์แมตให้เป็น `DD-MM-YYYY` ได้โดยตรง)
**หลักฐาน:**
- `src/components/admin/BookingsManager.jsx` (บรรทัด 436-441): ใช้ `<input type="date">` สำหรับตัวกรองวันที่
- `src/components/user/FloorPlan.jsx` (บรรทัด 125-131): ใช้ `<input type="date">` สำหรับการเลือกวันที่
- HTML มาตรฐาน `<input type="date">` จะแสดงผลตาม locale ของเบราว์เซอร์ และไม่สามารถบังคับฟอร์แมต `DD-MM-YYYY` ได้ด้วย HTML/CSS ธรรมดา จำเป็นต้องสร้าง Component ครอบเพื่อจัดการการแสดงผล
