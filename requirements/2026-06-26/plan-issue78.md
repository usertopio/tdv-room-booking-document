# Implementation Plan: Issue #78

## English
**Objective:** Change date filter buttons to `DD-MM-YYYY` format.
**Proposed Changes:**
1. **Create `DatePickerButton.jsx`**: A new UI component in `src/components/ui/DatePickerButton.jsx` that wraps the native `<input type="date">`. It will use a standard `<button>` to display the `DD-MM-YYYY` text (via `fmtDateDDMMYYYY`), with the native input overlaid transparently to handle the system calendar popup.
2. **Update `index.css`**: Add styles for `.date-picker-btn-wrapper`, `.date-picker-display-btn`, and `.date-picker-native-input` to match the existing `.filter-group input[type="date"]`.
3. **Update Components**: Replace `<input type="date">` in `BookingsManager.jsx` (Date Filter) and `FloorPlan.jsx` with the new `<DatePickerButton>`.

## Thai
**วัตถุประสงค์:** เปลี่ยนรูปแบบปุ่มตัวกรองวันที่ให้เป็น `DD-MM-YYYY`
**การเปลี่ยนแปลงที่เสนอ:**
1. **สร้าง `DatePickerButton.jsx`**: คอมโพเนนต์ใหม่ใน `src/components/ui/DatePickerButton.jsx` เพื่อครอบ `<input type="date">` แบบ native โดยจะใช้ `<button>` แสดงผลข้อความวันที่ในรูปแบบ `DD-MM-YYYY` (ผ่าน `fmtDateDDMMYYYY`) และมี input ซ้อนทับแบบโปร่งใสเพื่อเรียกใช้ปฏิทินของระบบ
2. **อัปเดต `index.css`**: เพิ่มสไตล์สำหรับ `.date-picker-btn-wrapper`, `.date-picker-display-btn`, และ `.date-picker-native-input` ให้ตรงกับดีไซน์ปัจจุบันของ `.filter-group input[type="date"]`
3. **อัปเดต Components**: เปลี่ยนไปใช้ `<DatePickerButton>` แทน `<input type="date">` เดิมใน `BookingsManager.jsx` (ส่วนตัวกรอง) และ `FloorPlan.jsx`

## Draft Issue
Title: เปลี่ยน format ปุ่ม filter วันที่เป็น date-month-year
Description:
  Requirements: material/requirement/2026-06-26/
  Implementation: material/requirement/2026-06-26/
  - Create DatePickerButton component to wrap native date input
  - Format displayed date as DD-MM-YYYY
  - Update BookingsManager and FloorPlan date filters
