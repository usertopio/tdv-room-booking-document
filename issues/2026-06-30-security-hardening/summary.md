# Task Summary — 2026-06-30-security-hardening

Requirements: `requirements/2026-06-30/` (`rewrite-security-hardening.md`)
Issue: depa-platform/tdv-room-booking-backend#26
Pull Request: depa-platform/tdv-room-booking-backend#27

---

## English

### What Was Done
Two backend-only security hardening fixes in `src/main.js`:

1. Added `escapeHtml()` and applied it to user-controlled fields (`bookedBy`, `companyName`,
   `roomLabel`, `promotion`, `building`, `floor`) in the two `htmlBody` emails
   (`sendPendingEmail`, `sendApprovalEmail`). Plain-text emails left unchanged (not HTML
   vectors).
2. Added `isValidDriveLink()` and gated the public `clientUpdateDocs` endpoint on it: accept
   only `https://drive.google.com/…` / `https://docs.google.com/…` (≤2048 chars), reject
   otherwise before any write or admin email — closing the phishing-link delivery vector.

### Files Changed
- `src/main.js` (+63 / −7)
  - New `escapeHtml(value)` and `isValidDriveLink(link)` helpers (new "SECURITY /
    SANITISATION HELPERS" section, bilingual docs).
  - `clientUpdateDocs` → rejects invalid links right after the presence check.
  - `sendPendingEmail` / `sendApprovalEmail` → compute escaped `safe.*` locals and use them
    in the `htmlBody` templates.

### Validation
- `node --check src/main.js` — syntax valid.
- Helper behavior verified by loading the real functions from `main.js` via `vm`:
  - `escapeHtml('<script>"&x\'')` → `&lt;script&gt;&quot;&amp;x&#39;`.
  - `isValidDriveLink` truth table **10/10**: valid Drive folder/file + Docs links pass;
    `http://`, other hosts, the `drive.google.com.evil.com` host-confusion case, empty, and a
    3000-char string all rejected; whitespace trimmed.
- Confirmed remaining raw `${booking.*}` interpolations are only in plain-text `body:` strings.
- GAS has no `npm` lint/build; runtime checks (editor / `clasp` on a non-prod copy) remain a
  manual pre-deploy step.

### Notes / Follow-ups
- PR #27 is open against `dev` and **not merged**; awaiting developer review/merge.
- **Deferred (follow-ups):** ownership/access check on `clientUpdateDocs` (require
  `bookerEmail` match) — needs a paired frontend change, so it's a coordinated FE+BE task.
  Also still open from the review: `LockService` on non-add write handlers, and splitting
  `main.js` into modules.

---

## ไทย (Thai)

### สิ่งที่ทำ
การแก้ไขเพื่อเพิ่มความปลอดภัยที่ฝั่ง backend อย่างเดียว 2 จุดใน `src/main.js`:

1. เพิ่ม `escapeHtml()` และใช้กับฟิลด์ที่ผู้ใช้ควบคุมได้ (`bookedBy`, `companyName`,
   `roomLabel`, `promotion`, `building`, `floor`) ในอีเมล `htmlBody` สองฉบับ
   (`sendPendingEmail`, `sendApprovalEmail`) ส่วนอีเมลข้อความธรรมดาคงไว้ (ไม่ใช่ช่องทาง HTML)
2. เพิ่ม `isValidDriveLink()` และใช้ตรวจ endpoint สาธารณะ `clientUpdateDocs`: รับเฉพาะ
   `https://drive.google.com/…` / `https://docs.google.com/…` (≤2048 ตัวอักษร) นอกนั้นปฏิเสธ
   ก่อนการเขียนหรือส่งอีเมลแอดมิน — ปิดช่องส่งลิงก์ฟิชชิ่ง

### ไฟล์ที่เปลี่ยน
- `src/main.js` (+63 / −7)
  - helper ใหม่ `escapeHtml(value)` และ `isValidDriveLink(link)` (ส่วนใหม่ "SECURITY /
    SANITISATION HELPERS" พร้อมคอมเมนต์สองภาษา)
  - `clientUpdateDocs` → ปฏิเสธลิงก์ที่ไม่ถูกต้องทันทีหลังการตรวจว่ามีค่า
  - `sendPendingEmail` / `sendApprovalEmail` → คำนวณค่า `safe.*` ที่ escape แล้วและใช้ใน
    เทมเพลต `htmlBody`

### การตรวจสอบ
- `node --check src/main.js` — syntax ถูกต้อง
- ตรวจพฤติกรรม helper โดยโหลดฟังก์ชันจริงจาก `main.js` ผ่าน `vm`:
  - `escapeHtml('<script>"&x\'')` → `&lt;script&gt;&quot;&amp;x&#39;`
  - ตาราง truth ของ `isValidDriveLink` **10/10**: ลิงก์ Drive โฟลเดอร์/ไฟล์ + Docs ที่ถูกต้องผ่าน;
    `http://`, โฮสต์อื่น, กรณีสับสนโฮสต์ `drive.google.com.evil.com`, ค่าว่าง และสตริง 3000
    ตัวอักษร ถูกปฏิเสธ; ตัดช่องว่างรอบข้าง
- ยืนยันว่าการแทรก `${booking.*}` แบบดิบที่เหลืออยู่ในสตริง `body:` แบบข้อความธรรมดาเท่านั้น
- GAS ไม่มี lint/build ผ่าน `npm` การตรวจสอบขณะรัน (editor / `clasp` บนสำเนาที่ไม่ใช่ prod) ยังเป็น
  ขั้นตอนแมนนวลก่อน deploy

### หมายเหตุ / งานต่อเนื่อง
- PR #27 เปิดอยู่กับ `dev` และ **ยังไม่ merge** กำลังรอผู้พัฒนา review/merge
- **เลื่อนออกไป (งานต่อเนื่อง):** การตรวจความเป็นเจ้าของของ `clientUpdateDocs` (บังคับ
  `bookerEmail` ให้ตรงกัน) ต้องแก้ฝั่ง frontend ด้วย จึงเป็นงาน FE+BE ที่ประสานกัน และยังเหลือ
  จากการรีวิว: `LockService` สำหรับ handler การเขียนที่ไม่ใช่ add และการแยก `main.js` เป็นหลายไฟล์
