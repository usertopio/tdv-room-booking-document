# Codebase Check — 2026-06-30-split-main-into-modules

Requirements: `requirements/2026-06-30/` (`rewrite-split-main-into-modules.md`)
Target repo(s): backend

---

## English

For each requirement, mark **Implemented**, **Partially Implemented**, or **Not Found**,
with file paths and line numbers as evidence. Reference: `tdv-room-booking-backend/src/main.js`
on `dev` (1,810 lines, 45 functions, 1 top-level `const`).

| # | Requirement (target state) | Status | Evidence (file:line) | Notes |
|---|----------------------------|--------|----------------------|-------|
| 1 | Code split into focused modules | Not Found | `src/main.js` (whole file) | Everything lives in one file; only `src/main.js` + `src/appsscript.json` exist under `src/`. |
| 2 | `CONFIG` isolated in a config module | Not Found | `src/main.js:43` | `const CONFIG` is the only top-level statement; pure literal (no `SCRIPT_PROPS` on `dev`). |
| 3 | Routing isolated | Not Found | `src/main.js:137,147,163,214` | `doGet`/`doPost`/`route`/`jsonResponse`. |
| 4 | Sheet/cache/audit helpers isolated | Not Found | `src/main.js:223-338,786,802` | `ss`…`readSheetAsObjects`, `clearBookingsCache`, `audit`, `logError`. |
| 5 | Security helpers isolated | Not Found | `src/main.js:826,833,1181,1190` | `checkAdminPin`, `hashSHA256`, `escapeHtml`, `isValidDriveLink`. |
| 6 | Booking + workflow handlers isolated | Not Found | `src/main.js:341-782,864-1167` | read/CRUD handlers + `applyStatusTransition`/status/receipt/clientUpdateDocs. |
| 7 | Email functions isolated | Not Found | `src/main.js:1199-1465` | `notifyAdmin` + 4 send* functions. |
| 8 | Triggers + editor utilities isolated | Not Found | `src/main.js:1475-1810` | `setupTriggers`/`dailyBackup`/`dailySummary`/`handleSheetEdit` + `setup`/`diagnose`/`applyApplicationStatusValidation`/`test*`. |

### Summary
The backend is a single 1,810-line `main.js`. Nothing is modularized. Because GAS shares one
global scope and only `CONFIG` runs at load time (a pure literal), the file can be partitioned
into 9 responsibility-based modules with no behavior change. Work needed: move each function
verbatim into its module, add bilingual file headers, delete `main.js`, and verify by
line-multiset equivalence + `node --check`/`vm` load.

---

## ไทย (Thai)

สำหรับแต่ละความต้องการ ให้ระบุสถานะ **Implemented**, **Partially Implemented** หรือ
**Not Found** พร้อมหลักฐาน อ้างอิง: `tdv-room-booking-backend/src/main.js` บน `dev`
(1,810 บรรทัด, 45 ฟังก์ชัน, top-level `const` 1 ตัว)

| # | ความต้องการ (สถานะเป้าหมาย) | สถานะ | หลักฐาน (file:line) | หมายเหตุ |
|---|------------------------------|-------|---------------------|----------|
| 1 | แยกโค้ดเป็นโมดูลตามหน้าที่ | Not Found | `src/main.js` (ทั้งไฟล์) | ทุกอย่างอยู่ในไฟล์เดียว มีเพียง `src/main.js` + `src/appsscript.json` |
| 2 | แยก `CONFIG` ไปไว้โมดูล config | Not Found | `src/main.js:43` | `const CONFIG` เป็น statement ระดับบนสุดเดียว เป็น literal ล้วน (ไม่มี `SCRIPT_PROPS` บน `dev`) |
| 3 | แยกส่วน routing | Not Found | `src/main.js:137,147,163,214` | `doGet`/`doPost`/`route`/`jsonResponse` |
| 4 | แยก helper ของ sheet/cache/audit | Not Found | `src/main.js:223-338,786,802` | `ss`…`readSheetAsObjects`, `clearBookingsCache`, `audit`, `logError` |
| 5 | แยก helper ด้านความปลอดภัย | Not Found | `src/main.js:826,833,1181,1190` | `checkAdminPin`, `hashSHA256`, `escapeHtml`, `isValidDriveLink` |
| 6 | แยก handler ของ booking + workflow | Not Found | `src/main.js:341-782,864-1167` | handler อ่าน/CRUD + `applyStatusTransition`/status/receipt/clientUpdateDocs |
| 7 | แยกฟังก์ชันอีเมล | Not Found | `src/main.js:1199-1465` | `notifyAdmin` + send* 4 ตัว |
| 8 | แยก trigger + utility สำหรับ editor | Not Found | `src/main.js:1475-1810` | `setupTriggers`/`dailyBackup`/`dailySummary`/`handleSheetEdit` + `setup`/`diagnose`/`applyApplicationStatusValidation`/`test*` |

### สรุป
backend เป็นไฟล์ `main.js` เดียวขนาด 1,810 บรรทัด ยังไม่ได้แยกโมดูล เนื่องจาก GAS ใช้ global
scope เดียวและมีเพียง `CONFIG` ที่ทำงานตอนโหลด (เป็น literal ล้วน) จึงแบ่งไฟล์เป็น 9 โมดูลตาม
หน้าที่ได้โดยไม่เปลี่ยนพฤติกรรม งานที่ต้องทำ: ย้ายแต่ละฟังก์ชันตามเดิมไปยังโมดูลของมัน เพิ่มหัวไฟล์
สองภาษา ลบ `main.js` และตรวจสอบด้วยความเท่ากันของ line-multiset + `node --check`/โหลดด้วย `vm`
