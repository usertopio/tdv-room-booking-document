# Task Summary — 2026-06-30-split-main-into-modules

Requirements: `requirements/2026-06-30/` (`rewrite-split-main-into-modules.md`)
Issue: depa-platform/tdv-room-booking-backend#30
Pull Request: depa-platform/tdv-room-booking-backend#31

---

## English

### What Was Done
Split the single 1,810-line `src/main.js` (45 functions + `CONFIG`) into 9
responsibility-based modules and deleted `main.js`. Pure move — function bodies relocated
verbatim, no renames/signatures/logic changes. Each module has a short bilingual header.

### Files Changed
- Removed: `src/main.js`.
- Added: `src/config.js`, `src/routing.js`, `src/sheets.js`, `src/security.js`,
  `src/bookings.js`, `src/workflow.js`, `src/email.js`, `src/triggers.js`, `src/admin.js`.
- `appsscript.json` and `.clasp.json` unchanged (`.clasp.json` already pushes all of `src/`;
  load order is not behavior-relevant since `CONFIG` is the only top-level statement and a
  pure literal).

### Validation
- **Content equivalence:** the 9 modules minus their headers are line-for-line identical to
  the original `main.js` (1810 = 1810; sorted line-multiset diff empty).
- `node --check` passes on each module and on the concatenation.
- A `vm` load of the concatenation defines all 45 functions with a single `CONFIG` (no
  redeclaration error); no duplicate function names; 45/45 inventory.
- GAS has no `npm` lint/build; final runtime confirmation is a maintainer `clasp push` +
  redeploy (out of scope for the AI).

### Notes / Follow-ups
- PR #31 is open against `dev` and **not merged**; awaiting developer review/merge.
- **Post-merge (maintainer):** `clasp push` then redeploy the Web App so the multi-file
  project goes live; run `setup()` is NOT required (no schema change).
- Still open from the review: `findBooking`/`isActiveBooking` helper dedup; `LockService`
  on non-add writes; `clientUpdateDocs` ownership check (FE+BE).

---

## ไทย (Thai)

### สิ่งที่ทำ
แยกไฟล์เดียว `src/main.js` (1,810 บรรทัด, 45 ฟังก์ชัน + `CONFIG`) เป็น 9 โมดูลตามหน้าที่ และ
ลบ `main.js` เป็นการย้ายล้วน — ย้ายเนื้อฟังก์ชันตามเดิม ไม่เปลี่ยนชื่อ/ลายเซ็น/ลอจิก แต่ละโมดูลมี
คอมเมนต์หัวไฟล์สองภาษาสั้น ๆ

### ไฟล์ที่เปลี่ยน
- ลบ: `src/main.js`
- เพิ่ม: `src/config.js`, `src/routing.js`, `src/sheets.js`, `src/security.js`,
  `src/bookings.js`, `src/workflow.js`, `src/email.js`, `src/triggers.js`, `src/admin.js`
- `appsscript.json` และ `.clasp.json` ไม่เปลี่ยน (`.clasp.json` push ทุกไฟล์ใน `src/` อยู่แล้ว;
  ลำดับโหลดไม่มีผลต่อพฤติกรรม เพราะ `CONFIG` เป็น statement ระดับบนสุดเดียวและเป็น literal ล้วน)

### การตรวจสอบ
- **ความเท่ากันของเนื้อหา:** โมดูลทั้ง 9 (ไม่นับหัวไฟล์) เหมือน `main.js` เดิมบรรทัดต่อบรรทัด
  (1810 = 1810; diff ของ line-multiset ที่เรียงแล้วว่าง)
- `node --check` ผ่านทุกโมดูลและการต่อรวม
- การโหลด `vm` ของการต่อรวมนิยามครบ 45 ฟังก์ชันโดยมี `CONFIG` ตัวเดียว (ไม่มี error ประกาศซ้ำ)
  ไม่มีชื่อฟังก์ชันซ้ำ ครบ 45/45
- GAS ไม่มี lint/build ผ่าน `npm`; การยืนยัน runtime สุดท้ายคือผู้ดูแล `clasp push` + redeploy
  (อยู่นอกขอบเขตของ AI)

### หมายเหตุ / งานต่อเนื่อง
- PR #31 เปิดอยู่กับ `dev` และ **ยังไม่ merge** กำลังรอผู้พัฒนา review/merge
- **หลัง merge (ผู้ดูแล):** `clasp push` แล้ว redeploy Web App เพื่อให้โปรเจกต์หลายไฟล์ใช้งานจริง
  ไม่จำเป็นต้องรัน `setup()` (ไม่มีการเปลี่ยน schema)
- ยังเหลือจากการรีวิว: การ dedup `findBooking`/`isActiveBooking`; `LockService` สำหรับการเขียน
  ที่ไม่ใช่ add; การตรวจความเป็นเจ้าของของ `clientUpdateDocs` (FE+BE)
