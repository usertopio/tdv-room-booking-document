# Implementation Plan — 2026-06-30-split-main-into-modules

Requirements: `requirements/2026-06-30/` (`rewrite-split-main-into-modules.md`)
Target repo(s): backend

## Draft Issue

```
Repo: depa-platform/tdv-room-booking-backend
Title: refactor(structure): split main.js into focused modules | แยก main.js เป็นโมดูลตามหน้าที่
Description:
  Requirements: requirements/2026-06-30/
  Implementation: issues/2026-06-30-split-main-into-modules/

  Split the single 1,810-line src/main.js (45 functions + CONFIG) into 9
  responsibility-based modules under src/ — config, routing, sheets, security, bookings,
  workflow, email, triggers, admin. Pure move, zero behavior change: function bodies are
  relocated verbatim. GAS shares one global scope across .js files and CONFIG is the only
  top-level statement (a pure literal used only at runtime), so cross-file calls and load
  order are safe. Verified by line-multiset equivalence to the original + node --check / vm
  load. main.js is deleted.

  แยกไฟล์เดียว src/main.js (1,810 บรรทัด, 45 ฟังก์ชัน + CONFIG) เป็น 9 โมดูลตามหน้าที่ใต้
  src/ — config, routing, sheets, security, bookings, workflow, email, triggers, admin
  เป็นการย้ายล้วน ไม่เปลี่ยนพฤติกรรม ย้ายเนื้อฟังก์ชันตามเดิมทุกตัวอักษร GAS ใช้ global scope
  เดียวและ CONFIG เป็น statement ระดับบนสุดเดียว (literal ล้วน ใช้ตอน runtime) การเรียกข้าม
  ไฟล์และลำดับโหลดจึงปลอดภัย ตรวจสอบด้วยความเท่ากันของ line-multiset + node --check / vm และ
  ลบ main.js
```

---

## English

### Approach
A pure file reorganization: relocate each function (and `CONFIG`) verbatim from `src/main.js`
into a responsibility module, add a short bilingual header to each, then delete `main.js`. No
function body is edited. Because GAS evaluates every project file into one global scope before
any entry point runs — and `CONFIG` (the only load-time statement) is a pure literal read only
at runtime — function order across files is irrelevant to behavior.

### Module layout (every function moved exactly once)
| Module | Functions |
|--------|-----------|
| `config.js` | file banner + `CONFIG` |
| `routing.js` | `doGet`, `doPost`, `route`, `jsonResponse` |
| `sheets.js` | `ss`, `getOrCreateSheet`, `getBookingsSheet`, `getAuditSheet`, `readSheetAsObjects`, `clearBookingsCache`, `audit`, `logError` |
| `security.js` | `checkAdminPin`, `hashSHA256`, `escapeHtml`, `isValidDriveLink` |
| `bookings.js` | `getData`, `getStats`, `lookupBookingHandler`, `addBookingHandler`, `generateBookingId`, `cancelBookingHandler`, `editBookingHandler` |
| `workflow.js` | `applyStatusTransition`, `updateApplicationStatusHandler`, `receiptExt`, `uploadReceiptHandler`, `clientUpdateDocsHandler` |
| `email.js` | `notifyAdmin`, `sendPendingEmail`, `sendApprovalEmail`, `sendRejectionEmail`, `sendCompletedEmail` |
| `triggers.js` | `setupTriggers`, `dailyBackup`, `dailySummary`, `handleSheetEdit` |
| `admin.js` | `setup`, `diagnose`, `applyApplicationStatusValidation`, `testGetData`, `testGetDataAll`, `testGetStats`, `testAdd`, `testHash` |

### Steps
1. Create each module by copying the exact source text of its functions (plus their existing
   section comments) out of `main.js` — no edits to bodies.
2. Prepend a short bilingual header comment to each module describing its role.
3. Delete `src/main.js`.
4. Leave `appsscript.json` unchanged. `.clasp.json` already pushes all of `src/`; load order
   is not behavior-relevant, so `filePushOrder` is left as-is (noted, not changed).

### Affected Areas
- `src/` — `main.js` removed; 9 new module files added. No manifest, dependency, or logic
  change; no frontend impact.

### Risks / Side Effects
- **Lost/duplicated code during the move** — the main risk. Mitigated by the line-multiset
  equivalence check below (a function silently dropped or duplicated fails it).
- **Duplicate `CONFIG` / redeclared function** — would throw at GAS load; caught by the `vm`
  load of the concatenation.
- **Load order** — not behavior-relevant here (only `CONFIG` runs at load, and it's a literal
  referenced only inside functions invoked at request time). No numeric filename prefixes
  needed.
- **GAS-only confirmation** — static checks give high confidence; final confirmation is a
  maintainer `clasp push` + redeploy (out of scope for the AI, as with all deploys).

### Validation
- `node --check` on every new module and on their concatenation.
- `vm` load of the concatenation — confirms single `CONFIG`, no redeclared identifiers, all
  45 functions defined.
- **Content equivalence:** the sorted multiset of lines across all modules, minus the added
  per-file header lines, equals the sorted lines of the original `main.js` — proving nothing
  was lost, added, or altered.
- Function inventory: `grep "^function "` across modules yields exactly the original 45 names,
  each once; `CONFIG` defined once.
- GAS has no `npm` lint/build; no runtime check beyond the above without a deploy.

---

## ไทย (Thai)

### แนวทาง
เป็นการจัดระเบียบไฟล์ล้วน: ย้ายแต่ละฟังก์ชัน (และ `CONFIG`) จาก `src/main.js` ไปยังโมดูลตาม
หน้าที่แบบคงเนื้อหาเดิมทุกตัวอักษร เพิ่มหัวไฟล์สองภาษาสั้น ๆ แล้วลบ `main.js` ไม่แก้เนื้อฟังก์ชันใด
เพราะ GAS evaluate ทุกไฟล์เข้าสู่ global scope เดียวก่อน entry point ทำงาน และ `CONFIG`
(statement ตอนโหลดเพียงตัวเดียว) เป็น literal ที่อ่านเฉพาะตอน runtime ลำดับฟังก์ชันข้ามไฟล์จึง
ไม่มีผลต่อพฤติกรรม

### การจัดวางโมดูล (ย้ายทุกฟังก์ชันครั้งเดียว)
ดูตารางในส่วนภาษาอังกฤษด้านบน (config / routing / sheets / security / bookings / workflow /
email / triggers / admin)

### ขั้นตอน
1. สร้างแต่ละโมดูลโดยคัดลอกข้อความต้นฉบับของฟังก์ชัน (พร้อมคอมเมนต์ section เดิม) ออกจาก
   `main.js` โดยไม่แก้เนื้อหา
2. เพิ่มคอมเมนต์หัวไฟล์สองภาษาสั้น ๆ อธิบายบทบาท
3. ลบ `src/main.js`
4. ไม่แก้ `appsscript.json`; `.clasp.json` push ทุกไฟล์ใน `src/` อยู่แล้ว ลำดับโหลดไม่มีผลต่อ
   พฤติกรรม จึงไม่แก้ `filePushOrder` (ระบุไว้ ไม่ได้เปลี่ยน)

### ส่วนที่ได้รับผลกระทบ
- `src/` — ลบ `main.js` เพิ่มไฟล์โมดูลใหม่ 9 ไฟล์ ไม่แก้ manifest/dependency/ลอจิก ไม่กระทบ frontend

### ความเสี่ยง / ผลข้างเคียง
- **โค้ดหาย/ซ้ำระหว่างย้าย** — ความเสี่ยงหลัก ลดด้วยการตรวจความเท่ากันของ line-multiset ด้านล่าง
- **`CONFIG` ซ้ำ / ฟังก์ชันถูกประกาศซ้ำ** — จะ error ตอน GAS โหลด ตรวจจับได้ด้วยการโหลด `vm`
- **ลำดับโหลด** — ไม่มีผลต่อพฤติกรรม (มีเพียง `CONFIG` ที่ทำงานตอนโหลด และเป็น literal) ไม่ต้องใช้
  คำนำหน้าตัวเลขในชื่อไฟล์
- **ยืนยันเฉพาะบน GAS** — การตรวจแบบ static ให้ความมั่นใจสูง การยืนยันสุดท้ายคือผู้ดูแล `clasp
  push` + redeploy (อยู่นอกขอบเขตของ AI เช่นเดียวกับการ deploy ทั้งหมด)

### การตรวจสอบ
- `node --check` ทุกโมดูลและการต่อรวม
- โหลด `vm` ของการต่อรวม — ยืนยัน `CONFIG` ตัวเดียว ไม่มี identifier ซ้ำ และมีครบ 45 ฟังก์ชัน
- **ความเท่ากันของเนื้อหา:** multiset ของบรรทัด (เรียงแล้ว) ของทุกโมดูล ลบบรรทัดหัวไฟล์ที่เพิ่ม
  เท่ากับบรรทัด (เรียงแล้ว) ของ `main.js` เดิม พิสูจน์ว่าไม่มีเนื้อหาหาย เพิ่ม หรือเปลี่ยน
- บัญชีฟังก์ชัน: `grep "^function "` ในโมดูลได้ครบ 45 ชื่อเดิม ชื่อละครั้ง; `CONFIG` ครั้งเดียว
- GAS ไม่มี lint/build ผ่าน `npm`; ไม่มีการตรวจ runtime นอกเหนือจากข้างต้นหากไม่ deploy
