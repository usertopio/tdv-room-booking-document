# Task Summary: Issue #78

Requirements: material/requirement/2026-06-26/raw-issue78.md
Issue: #78 (PR #79)

## English
**What was done:**
- Created a new `DatePickerButton.jsx` component to wrap the native `<input type="date">`.
- Updated `index.css` to add the `.date-picker-btn-wrapper`, `.date-picker-display-btn`, and `.date-picker-native-input` classes, styling the custom button consistently with the existing UI.
- Replaced the date filter in `BookingsManager.jsx` with the new component.
- Replaced the `fpDate` input in `FloorPlan.jsx` with the new component.
- Validated via `npm run lint` and `npm run build`.
- Created a feature branch, committed, and opened PR #79.

**Outcome:**
The date filter buttons now display the selected date in `DD-MM-YYYY` format while preserving full native calendar functionality when clicked.

## Thai
**สิ่งที่ทำไป:**
- สร้างคอมโพเนนต์ใหม่ `DatePickerButton.jsx` เพื่อใช้ครอบ `<input type="date">` ของระบบ
- อัปเดต `index.css` โดยเพิ่มคลาส `.date-picker-btn-wrapper`, `.date-picker-display-btn` และ `.date-picker-native-input` เพื่อให้ได้ปุ่มที่มีสไตล์เข้ากับหน้าตาปัจจุบัน
- นำคอมโพเนนต์ใหม่ไปใช้แทนช่องเลือกวันที่ในหน้า `BookingsManager.jsx` (ส่วนกรองข้อมูล)
- นำคอมโพเนนต์ใหม่ไปใช้แทนช่องเลือกวันที่ในหน้า `FloorPlan.jsx` (ปฏิทินเลือกวัน)
- ตรวจสอบความถูกต้องด้วย `npm run lint` และ `npm run build`
- สร้าง branch ใหม่ commit และเปิด Pull Request #79

**ผลลัพธ์:**
ปุ่มเลือกวันที่สามารถแสดงผลวันในรูปแบบ `DD-MM-YYYY` (วัน-เดือน-ปี) ได้แล้ว โดยยังคงสามารถกดเพื่อเปิดปฏิทินของระบบได้ตามปกติ
