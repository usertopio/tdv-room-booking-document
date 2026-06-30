# Requirement Rewrite — 2026-06-30 (split-main-into-modules)

Source: Backend code review (deferred structural item; no developer `raw.md`)
Target repo(s): backend

---

## English

### Overview
`src/main.js` is a single 1,810-line file holding 45 functions and the `CONFIG` object. Split
it into focused modules grouped by responsibility, with **zero behavior change**. Google
Apps Script concatenates all `.js`/`.gs` files into one global scope and evaluates every file
before any entry point runs, so cross-file calls work unchanged and load order is safe (the
only top-level statement, `const CONFIG`, is a pure literal referenced only at runtime).

### Functional Requirements
1. Partition `src/main.js` into the following modules (every function moved exactly once,
   byte-for-byte; no logic edits):
   - `config.js` — file banner + `CONFIG`
   - `routing.js` — `doGet`, `doPost`, `route`, `jsonResponse`
   - `sheets.js` — `ss`, `getOrCreateSheet`, `getBookingsSheet`, `getAuditSheet`,
     `readSheetAsObjects`, `clearBookingsCache`, `audit`, `logError`
   - `security.js` — `checkAdminPin`, `hashSHA256`, `escapeHtml`, `isValidDriveLink`
   - `bookings.js` — `getData`, `getStats`, `lookupBookingHandler`, `addBookingHandler`,
     `generateBookingId`, `cancelBookingHandler`, `editBookingHandler`
   - `workflow.js` — `applyStatusTransition`, `updateApplicationStatusHandler`, `receiptExt`,
     `uploadReceiptHandler`, `clientUpdateDocsHandler`
   - `email.js` — `notifyAdmin`, `sendPendingEmail`, `sendApprovalEmail`,
     `sendRejectionEmail`, `sendCompletedEmail`
   - `triggers.js` — `setupTriggers`, `dailyBackup`, `dailySummary`, `handleSheetEdit`
   - `admin.js` — `setup`, `diagnose`, `applyApplicationStatusValidation`, `testGetData`,
     `testGetDataAll`, `testGetStats`, `testAdd`, `testHash`
2. Delete `src/main.js` once its content is fully distributed.
3. Each new file gets a short bilingual header comment describing its role.

### Non-Functional Requirements
- **No behavior change** — function bodies are moved verbatim; no renames, no signature
  changes, no logic edits.
- No new dependencies; `appsscript.json` manifest unchanged.
- All 45 functions + `CONFIG` present exactly once across the new files (none lost or
  duplicated).
- `.clasp.json` continues to push everything under `src/`.

### Out of Scope
- Any logic change, rename, or signature change.
- `LockService` on non-add writes; the `findBooking`/`isActiveBooking` helper dedup; the
  `clientUpdateDocs` ownership check (separate items).
- Converting to ES modules / `import`-`export` (not supported by GAS).

### Acceptance Criteria
- [ ] `src/main.js` no longer exists; the 9 modules above exist under `src/`.
- [ ] The sorted line-multiset of all modules (excluding the added per-file headers) equals
      the original `main.js` — proving no content was lost, added, or altered.
- [ ] Every one of the 45 functions appears exactly once across the modules; `CONFIG` once.
- [ ] `node --check` passes for each file and for their concatenation; a `vm` load of the
      concatenation succeeds (no duplicate/redeclared identifiers).

---

## ไทย (Thai)

### ภาพรวม
`src/main.js` เป็นไฟล์เดียวขนาด 1,810 บรรทัด มี 45 ฟังก์ชันและอ็อบเจกต์ `CONFIG` งานนี้แยกไฟล์
ออกเป็นโมดูลตามหน้าที่ โดย **ไม่เปลี่ยนพฤติกรรมใด ๆ** Google Apps Script รวมไฟล์ `.js`/`.gs`
ทั้งหมดไว้ใน global scope เดียวและ evaluate ทุกไฟล์ก่อน entry point ทำงาน การเรียกข้ามไฟล์จึง
ทำงานเหมือนเดิมและลำดับการโหลดปลอดภัย (statement ระดับบนสุดเดียวคือ `const CONFIG` ซึ่งเป็น
literal ที่ถูกอ้างอิงเฉพาะตอน runtime)

### ความต้องการเชิงฟังก์ชัน
1. แบ่ง `src/main.js` เป็นโมดูลต่อไปนี้ (ย้ายทุกฟังก์ชันครั้งเดียวแบบ byte-for-byte ไม่แก้ลอจิก):
   - `config.js` — banner + `CONFIG`
   - `routing.js` — `doGet`, `doPost`, `route`, `jsonResponse`
   - `sheets.js` — `ss`, `getOrCreateSheet`, `getBookingsSheet`, `getAuditSheet`,
     `readSheetAsObjects`, `clearBookingsCache`, `audit`, `logError`
   - `security.js` — `checkAdminPin`, `hashSHA256`, `escapeHtml`, `isValidDriveLink`
   - `bookings.js` — `getData`, `getStats`, `lookupBookingHandler`, `addBookingHandler`,
     `generateBookingId`, `cancelBookingHandler`, `editBookingHandler`
   - `workflow.js` — `applyStatusTransition`, `updateApplicationStatusHandler`, `receiptExt`,
     `uploadReceiptHandler`, `clientUpdateDocsHandler`
   - `email.js` — `notifyAdmin`, `sendPendingEmail`, `sendApprovalEmail`,
     `sendRejectionEmail`, `sendCompletedEmail`
   - `triggers.js` — `setupTriggers`, `dailyBackup`, `dailySummary`, `handleSheetEdit`
   - `admin.js` — `setup`, `diagnose`, `applyApplicationStatusValidation`, `testGetData`,
     `testGetDataAll`, `testGetStats`, `testAdd`, `testHash`
2. ลบ `src/main.js` เมื่อกระจายเนื้อหาครบแล้ว
3. ทุกไฟล์ใหม่มีคอมเมนต์หัวไฟล์สองภาษาสั้น ๆ อธิบายบทบาท

### ความต้องการที่ไม่ใช่เชิงฟังก์ชัน
- **ไม่เปลี่ยนพฤติกรรม** — ย้ายเนื้อฟังก์ชันตามเดิมทุกตัวอักษร ไม่เปลี่ยนชื่อ/ลายเซ็น/ลอจิก
- ไม่เพิ่ม dependency; ไฟล์ `appsscript.json` ไม่เปลี่ยน
- ครบทั้ง 45 ฟังก์ชัน + `CONFIG` ปรากฏครั้งเดียวในไฟล์ใหม่ (ไม่หาย ไม่ซ้ำ)
- `.clasp.json` ยัง push ทุกไฟล์ใต้ `src/`

### นอกขอบเขต
- การเปลี่ยนลอจิก เปลี่ยนชื่อ หรือเปลี่ยนลายเซ็น
- `LockService` สำหรับการเขียนที่ไม่ใช่ add; การ dedup `findBooking`/`isActiveBooking`; การตรวจ
  ความเป็นเจ้าของของ `clientUpdateDocs` (เป็นรายการแยก)
- การแปลงเป็น ES modules / `import`-`export` (GAS ไม่รองรับ)

### เกณฑ์การยอมรับ
- [ ] ไม่มี `src/main.js` อีกต่อไป มีโมดูลทั้ง 9 ไฟล์ใต้ `src/`
- [ ] line-multiset ที่เรียงแล้วของทุกโมดูล (ไม่นับหัวไฟล์ที่เพิ่ม) เท่ากับ `main.js` เดิม พิสูจน์ว่า
      ไม่มีเนื้อหาหาย เพิ่ม หรือเปลี่ยน
- [ ] ทั้ง 45 ฟังก์ชันปรากฏครั้งเดียวในโมดูล; `CONFIG` ครั้งเดียว
- [ ] `node --check` ผ่านทุกไฟล์และผ่านการต่อรวม; การโหลดด้วย `vm` สำเร็จ (ไม่มี identifier ซ้ำ)
