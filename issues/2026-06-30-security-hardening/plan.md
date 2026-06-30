# Implementation Plan — 2026-06-30-security-hardening

Requirements: `requirements/2026-06-30/` (`rewrite-security-hardening.md`)
Target repo(s): backend

## Draft Issue

```
Repo: depa-platform/tdv-room-booking-backend
Title: fix(security): escape HTML in emails and validate clientUpdateDocs link | escape HTML ในอีเมลและตรวจสอบลิงก์ของ clientUpdateDocs
Description:
  Requirements: requirements/2026-06-30/
  Implementation: issues/2026-06-30-security-hardening/

  Two backend-only hardening fixes from the code review:
  1. HTML email injection — sendPendingEmail and sendApprovalEmail interpolate
     user-controlled booking fields (bookedBy, companyName, roomLabel, promotion,
     building, floor) raw into htmlBody. Add an escapeHtml() helper and apply it.
  2. Unvalidated public write — clientUpdateDocs accepts any docDriveLink, stores it,
     and forwards it to the admin email (phishing-delivery vector). Validate it as an
     https Google Drive/Docs URL with a length cap; reject otherwise.
  No API contract change. Ownership/access control on clientUpdateDocs (bookerEmail
  match) is deferred to a coordinated FE+BE follow-up.

  การแก้ไขเพื่อเพิ่มความปลอดภัยที่ฝั่ง backend อย่างเดียว 2 จุดจากการรีวิวโค้ด:
  1. HTML email injection — sendPendingEmail และ sendApprovalEmail แทรกฟิลด์ที่ผู้ใช้
     ควบคุมได้ (bookedBy, companyName, roomLabel, promotion, building, floor) ลงใน
     htmlBody แบบดิบ เพิ่ม helper escapeHtml() และนำไปใช้
  2. การเขียนสาธารณะที่ไม่ตรวจสอบ — clientUpdateDocs รับ docDriveLink ใด ๆ เก็บลงชีต และ
     ส่งต่อไปยังอีเมลแอดมิน (ช่องส่งฟิชชิ่ง) ตรวจสอบให้เป็น URL https ของ Google Drive/Docs
     พร้อมจำกัดความยาว มิฉะนั้นปฏิเสธ
  ไม่เปลี่ยนสัญญาของ API ส่วนการควบคุมการเข้าถึงของ clientUpdateDocs (ตรวจ bookerEmail
  ให้ตรงกัน) เลื่อนไปเป็นงาน FE+BE ที่ประสานกัน
```

---

## English

### Approach
Single-file change to `src/main.js`. Add two small pure helpers near the other utilities —
`escapeHtml(value)` and `isValidDriveLink(link)` — then (a) route the user-controlled
interpolations in the two `htmlBody` emails through `escapeHtml`, and (b) gate
`clientUpdateDocs` on `isValidDriveLink` before any write or email. Plain-text emails and
the admin (PIN-gated) `docLink` paths are intentionally left unchanged.

### Steps
1. Add helpers (placed near `hashSHA256` / the sheet helpers):
   ```js
   // EN: Escape HTML metacharacters so user-supplied values can't inject markup
   //     into an htmlBody email. TH: escape อักขระพิเศษของ HTML เพื่อกันการแทรก markup
   function escapeHtml(value) {
     return String(value == null ? "" : value)
       .replace(/&/g, "&amp;")
       .replace(/</g, "&lt;")
       .replace(/>/g, "&gt;")
       .replace(/"/g, "&quot;")
       .replace(/'/g, "&#39;");
   }

   // EN: Accept only https Google Drive/Docs URLs (length-capped). TH: รับเฉพาะ URL
   //     https ของ Google Drive/Docs (จำกัดความยาว)
   function isValidDriveLink(link) {
     const s = String(link == null ? "" : link).trim();
     if (!s || s.length > 2048) return false;
     return /^https:\/\/(drive|docs)\.google\.com\//i.test(s);
   }
   ```
2. `sendPendingEmail` — compute escaped locals once and use them in the `htmlBody` template
   (the plain-text `body` fallback stays raw; it is not an injection vector):
   ```js
   const safe = {
     bookedBy: escapeHtml(booking.bookedBy),
     building: escapeHtml(booking.building),
     floor: escapeHtml(booking.floor),
     roomLabel: escapeHtml(booking.roomLabel),
     promotion: escapeHtml(booking.promotion || "-"),
     companyName: escapeHtml(booking.companyName || booking.bookedBy),
   };
   ```
   Replace `${booking.bookedBy}` → `${safe.bookedBy}`, etc., inside the HTML string only.
3. `sendApprovalEmail` — same pattern for `bookedBy`, `building`, `floor`, `roomLabel`,
   `promotion` in its `htmlBody`. (`bookingId` is server-generated; CONFIG/contract URL are
   trusted — left as-is.)
4. `clientUpdateDocs` — after the existing presence check, add validation before any write:
   ```js
   if (!isValidDriveLink(docDriveLink))
     return { ok: false, error: "ลิงก์เอกสารไม่ถูกต้อง — ต้องเป็นลิงก์ Google Drive/Docs (https)" };
   ```
   The rest of the handler is unchanged, so only validated links are stored (`main.js:1112`)
   and forwarded to the admin email (`main.js:1138`).

### Affected Areas
- `src/main.js` only — two new helpers, `sendPendingEmail`, `sendApprovalEmail`,
  `clientUpdateDocs`. No manifest, no dependencies, no frontend change.

### Risks / Side Effects
- **Over-escaping:** `escapeHtml` only touches the HTML email bodies; the stored Sheet
  values and plain-text emails are untouched, so data and non-HTML output are unaffected.
- **Legitimate-link rejection:** the regex must accept the real Drive/Docs link shapes the
  app produces (folder links `drive.google.com/drive/folders/...`, file links
  `drive.google.com/file/d/...`, docs `docs.google.com/...`). All are `https://(drive|docs).google.com/…`,
  so the pattern covers them. `http://` and shortened/other-host links are intentionally
  rejected.
- **Contract stability:** request/response shapes unchanged; only an added rejection branch.
- **Single-file blast radius:** changes are localized to three functions + two pure helpers.

### Validation
- `node --check src/main.js` — syntax.
- Unit-style reasoning in the editor: call `escapeHtml('<script>"&x')` → expect
  `&lt;script&gt;&quot;&amp;x`; `isValidDriveLink` truth table for: valid drive folder/file,
  valid docs, `http://drive.google.com/…` (reject), `https://evil.example/…` (reject),
  empty (reject), 3000-char string (reject).
- Manual (non-prod copy / demo): submit `clientUpdateDocs` with a bad link → `{ok:false}`,
  row unchanged, no admin email; with a good link → behaves as before. Trigger
  `sendPendingEmail` with a crafted `bookedBy` → markup appears escaped.
- GAS has no `npm` lint/build; production deploy stays a manual maintainer step.

---

## ไทย (Thai)

### แนวทาง
แก้ไฟล์เดียวคือ `src/main.js` เพิ่ม helper บริสุทธิ์ขนาดเล็กสองตัวใกล้ utility อื่น ๆ —
`escapeHtml(value)` และ `isValidDriveLink(link)` — แล้ว (ก) ส่งค่าที่ผู้ใช้ควบคุมได้ในอีเมล
`htmlBody` สองฉบับผ่าน `escapeHtml` และ (ข) ตรวจสอบ `clientUpdateDocs` ด้วย
`isValidDriveLink` ก่อนการเขียนหรือส่งอีเมลใด ๆ ส่วนอีเมลข้อความธรรมดาและเส้นทาง `docLink`
ของแอดมิน (ที่มี PIN กำกับ) คงไว้ตามเดิมโดยตั้งใจ

### ขั้นตอน
1. เพิ่ม helper `escapeHtml` (escape `& < > " '`) และ `isValidDriveLink` (รับเฉพาะ URL https
   ของ `drive.google.com`/`docs.google.com` จำกัดความยาว 2048) ตามโค้ดในฉบับภาษาอังกฤษ
2. `sendPendingEmail` — คำนวณค่าที่ escape แล้วครั้งเดียวและใช้ใน `htmlBody` (ส่วน `body`
   แบบข้อความธรรมดาคงค่าดิบ เพราะไม่ใช่ช่องทาง injection): `bookedBy`, `building`, `floor`,
   `roomLabel`, `promotion`, `companyName`
3. `sendApprovalEmail` — ใช้รูปแบบเดียวกันกับ `bookedBy`, `building`, `floor`, `roomLabel`,
   `promotion` ใน `htmlBody` (`bookingId` สร้างจากเซิร์ฟเวอร์ และค่า CONFIG/ลิงก์สัญญาเชื่อถือได้
   จึงคงไว้)
4. `clientUpdateDocs` — หลังการตรวจว่ามีค่าเดิม เพิ่มการตรวจสอบก่อนการเขียนใด ๆ:
   `if (!isValidDriveLink(docDriveLink)) return { ok:false, error:"ลิงก์เอกสารไม่ถูกต้อง — ต้องเป็นลิงก์ Google Drive/Docs (https)" };`
   ส่วนที่เหลือของ handler ไม่เปลี่ยน จึงเก็บเฉพาะลิงก์ที่ผ่านการตรวจสอบ (`main.js:1112`) และ
   ส่งต่อไปยังอีเมลแอดมิน (`main.js:1138`)

### ส่วนที่ได้รับผลกระทบ
- `src/main.js` เท่านั้น — helper ใหม่สองตัว, `sendPendingEmail`, `sendApprovalEmail`,
  `clientUpdateDocs` ไม่มี manifest, dependency หรือการแก้ frontend

### ความเสี่ยง / ผลข้างเคียง
- **escape เกินจำเป็น:** `escapeHtml` แตะเฉพาะเนื้อหาอีเมล HTML ค่าที่เก็บในชีตและอีเมลข้อความ
  ธรรมดาไม่ถูกแตะ ข้อมูลและเอาต์พุตที่ไม่ใช่ HTML จึงไม่ได้รับผลกระทบ
- **ปฏิเสธลิงก์ที่ถูกต้อง:** regex ต้องรับรูปแบบลิงก์จริงที่แอปสร้าง (โฟลเดอร์
  `drive.google.com/drive/folders/...`, ไฟล์ `drive.google.com/file/d/...`, docs
  `docs.google.com/...`) ซึ่งล้วนเป็น `https://(drive|docs).google.com/…` จึงครอบคลุม ส่วน
  `http://` และลิงก์ย่อ/โฮสต์อื่นถูกปฏิเสธโดยตั้งใจ
- **ความเสถียรของสัญญา:** รูปแบบ request/response ไม่เปลี่ยน เพียงเพิ่ม branch การปฏิเสธ
- **ผลกระทบของไฟล์เดียว:** การเปลี่ยนแปลงจำกัดอยู่ที่สามฟังก์ชัน + helper บริสุทธิ์สองตัว

### การตรวจสอบ
- `node --check src/main.js` — syntax
- ไล่ตรรกะแบบ unit ใน editor: `escapeHtml('<script>"&x')` → คาดได้ `&lt;script&gt;&quot;&amp;x`;
  ตาราง truth ของ `isValidDriveLink` สำหรับ: โฟลเดอร์/ไฟล์ Drive ที่ถูกต้อง, docs ที่ถูกต้อง,
  `http://drive.google.com/…` (ปฏิเสธ), `https://evil.example/…` (ปฏิเสธ), ว่าง (ปฏิเสธ),
  สตริง 3000 ตัวอักษร (ปฏิเสธ)
- ทดสอบด้วยมือ (สำเนาที่ไม่ใช่ prod / demo): เรียก `clientUpdateDocs` ด้วยลิงก์ไม่ถูกต้อง →
  `{ok:false}`, แถวไม่เปลี่ยน, ไม่ส่งอีเมลแอดมิน; ด้วยลิงก์ถูกต้อง → ทำงานเหมือนเดิม กระตุ้น
  `sendPendingEmail` ด้วย `bookedBy` ที่ประดิษฐ์ → markup ปรากฏแบบ escape แล้ว
- GAS ไม่มี lint/build ผ่าน `npm` การ deploy production ยังเป็นขั้นตอนของผู้ดูแลด้วยมือ
