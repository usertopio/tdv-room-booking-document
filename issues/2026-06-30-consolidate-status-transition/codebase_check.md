# Codebase Check — 2026-06-30-consolidate-status-transition

Requirements: `requirements/2026-06-30/` (`rewrite-consolidate-status-transition.md`)
Target repo(s): backend

---

## English

For each requirement, mark **Implemented**, **Partially Implemented**, or **Not Found**,
with file paths and line numbers as evidence. All references are to
`tdv-room-booking-backend/src/main.js` on `feat/config-script-properties`.

| # | Requirement (target state) | Status | Evidence (file:line) | Notes |
|---|----------------------------|--------|----------------------|-------|
| 1 | Single shared `applyStatusTransition` function; 3 call sites delegate to it | Not Found | `main.js:946-992`, `main.js:751-813`, `main.js:1778-1814` | Transition block is hand-rolled 3× — `updateApplicationStatusHandler`, `editBookingHandler`, `handleSheetEdit`. |
| 2 | `completed` writes `receiptUrl` on all 3 paths | Partially Implemented | `main.js:983` (API only) | Edit path (`main.js:799-812`) and Sheet path (`main.js:1803-1811`) omit `receiptUrl` entirely. |
| 3 | `rejected` fills both `cancelReason` and `rejectReason` everywhere | Partially Implemented | `main.js:970` | API path writes `rejectReason` as `reason \|\| ""` → blank when no reason. Edit (`main.js:783`) and Sheet (`main.js:1795`) do fill it. |
| 4 | `pending` / `reviewing` reset `status = "booked"` uniformly | Partially Implemented | `main.js:1812-1813` (Sheet only) | `reviewing` is handled in all 3; `pending` is handled **only** in the Sheet path. API/edit have no `pending` branch. |
| 5 | `editBookingHandler` keeps recording changes in `changes{}` | Implemented | `main.js:751-757` | Must be preserved — the shared function must accept a change-recording `writeField`. |
| 6 | Per-path email error-log contexts distinguishable | Partially Implemented | `main.js:963`, `main.js:773`, `main.js:1787` | Already distinct but ad-hoc (`approvalEmail`, `approvalEmailEdit`, `sheet_approvalEmail`); should be a consistent `:source` suffix. |
| 7 | Valid-status list defined once | Not Found | `main.js:923-930`, `main.js:1737-1744`, `main.js:1686-1693` | Identical 6-status array hardcoded in `updateApplicationStatusHandler`, `handleSheetEdit`, and `applyApplicationStatusValidation`. |

### Summary
The transition logic **works**, but it is triplicated and the copies have already drifted:
the receipt is dropped on two of three paths, the API path can leave `rejectReason` blank,
and `pending` is handled inconsistently. Nothing here is "missing functionality" so much as
**duplication that has caused behavioral divergence**. The work is a consolidation: extract
one `applyStatusTransition(writeField, booking, newStatus, opts)`, route all three call
sites through it (reconciling the drift), and lift the status list into `CONFIG`. The
`changes{}` audit detail in `editBookingHandler` must be preserved by injecting its
change-recording `writeField`. No endpoint contract changes.

---

## ไทย (Thai)

สำหรับแต่ละความต้องการ ให้ระบุสถานะ **Implemented**, **Partially Implemented** หรือ
**Not Found** พร้อมอ้างอิงไฟล์และหมายเลขบรรทัดเป็นหลักฐาน ทุกการอ้างอิงคือไฟล์
`tdv-room-booking-backend/src/main.js` บนสาขา `feat/config-script-properties`

| # | ความต้องการ (สถานะเป้าหมาย) | สถานะ | หลักฐาน (file:line) | หมายเหตุ |
|---|------------------------------|-------|---------------------|----------|
| 1 | ฟังก์ชันกลาง `applyStatusTransition` เดียว และ 3 จุดเรียกมอบหมายงานให้ | Not Found | `main.js:946-992`, `main.js:751-813`, `main.js:1778-1814` | บล็อก transition ถูกเขียนเองซ้ำ 3 ครั้ง — `updateApplicationStatusHandler`, `editBookingHandler`, `handleSheetEdit` |
| 2 | `completed` เขียน `receiptUrl` ในทั้ง 3 เส้นทาง | Partially Implemented | `main.js:983` (เฉพาะ API) | เส้นทางแก้ไข (`main.js:799-812`) และเส้นทางชีต (`main.js:1803-1811`) ไม่เขียน `receiptUrl` เลย |
| 3 | `rejected` เติมทั้ง `cancelReason` และ `rejectReason` ทุกเส้นทาง | Partially Implemented | `main.js:970` | เส้นทาง API เขียน `rejectReason` เป็น `reason \|\| ""` → ว่างเมื่อไม่มีเหตุผล ส่วนเส้นทางแก้ไข (`main.js:783`) และชีต (`main.js:1795`) เติมให้ |
| 4 | `pending` / `reviewing` รีเซ็ต `status = "booked"` ให้เหมือนกัน | Partially Implemented | `main.js:1812-1813` (เฉพาะชีต) | `reviewing` จัดการครบทั้ง 3 แต่ `pending` จัดการ **เฉพาะ** เส้นทางชีต ส่วน API/แก้ไขไม่มี branch `pending` |
| 5 | `editBookingHandler` ยังบันทึกการเปลี่ยนแปลงใน `changes{}` | Implemented | `main.js:751-757` | ต้องคงไว้ — ฟังก์ชันกลางต้องรับ `writeField` ที่บันทึก changes ได้ |
| 6 | context ของ log อีเมลผิดพลาดแยกแยะตามเส้นทางได้ | Partially Implemented | `main.js:963`, `main.js:773`, `main.js:1787` | แยกได้อยู่แล้วแต่ตั้งชื่อไม่เป็นระบบ (`approvalEmail`, `approvalEmailEdit`, `sheet_approvalEmail`) ควรใช้ suffix `:source` ให้สม่ำเสมอ |
| 7 | รายการสถานะที่ถูกต้องนิยามครั้งเดียว | Not Found | `main.js:923-930`, `main.js:1737-1744`, `main.js:1686-1693` | อาเรย์ 6 สถานะเหมือนกันถูก hardcode ใน `updateApplicationStatusHandler`, `handleSheetEdit`, และ `applyApplicationStatusValidation` |

### สรุป
ตรรกะ transition **ทำงานได้** แต่ถูกเขียนซ้ำ 3 ชุด และชุดเหล่านั้นเคลื่อนออกจากกันแล้ว: ใบเสร็จ
ถูกทิ้งใน 2 จาก 3 เส้นทาง, เส้นทาง API อาจปล่อย `rejectReason` ว่าง, และ `pending` ถูกจัดการ
ไม่สม่ำเสมอ ไม่มีสิ่งใดที่เป็น "ฟังก์ชันที่หายไป" แต่เป็น **ความซ้ำซ้อนที่ทำให้พฤติกรรมต่างกัน**
งานนี้คือการรวมโค้ด: แยกฟังก์ชัน `applyStatusTransition(writeField, booking, newStatus, opts)`
เดียว ให้ทั้งสามจุดเรียกไหลผ่านฟังก์ชันนี้ (พร้อมปรับ drift ให้ตรงกัน) และย้ายรายการสถานะไปไว้
ใน `CONFIG` ส่วนรายละเอียด audit `changes{}` ใน `editBookingHandler` ต้องคงไว้โดยส่ง
`writeField` ที่บันทึก changes เข้าไป ไม่มีการเปลี่ยนสัญญาของ endpoint ใด ๆ
