# Requirement Rewrite — 2026-06-30 (security-hardening)

Source: Code review of `tdv-room-booking-backend/src/main.js` (no developer `raw.md`; requirement derived from review findings)
Target repo(s): backend

---

## English

### Overview
Two contained, backend-only security weaknesses surfaced in the backend review:

1. **HTML email injection.** The two `htmlBody` emails (`sendPendingEmail`,
   `sendApprovalEmail`) interpolate user-controlled booking fields (`bookedBy`,
   `companyName`, `roomLabel`, `promotion`, `building`, `floor`) directly into HTML, so a
   crafted value renders as live markup in the recipient's inbox.
2. **Unvalidated public write.** `clientUpdateDocs` (no admin PIN) accepts any
   `docDriveLink` string, stores it, and forwards it to the admin notification email — so a
   caller with a booking ID can deliver an arbitrary off-domain (phishing) URL to the admin.

This task hardens both **without changing any API contract**: escape user values in the two
HTML emails, and validate `docDriveLink` so only well-formed Google Drive/Docs URLs are
accepted.

### Functional Requirements
1. Add an `escapeHtml(value)` helper (escaping `& < > " '`) and apply it to every
   user-controlled value interpolated into `htmlBody` in `sendPendingEmail` and
   `sendApprovalEmail`: `bookedBy`, `companyName`, `roomLabel`, `promotion`, `building`,
   `floor`.
2. In `clientUpdateDocs`, validate `docDriveLink`: it must be an `https://` URL whose host
   is `drive.google.com` or `docs.google.com`; otherwise reject with a clear bilingual error
   and make no changes to the row.
3. Enforce a sane length cap on `docDriveLink` (e.g. 2048 characters).
4. The validated link is what gets stored and forwarded to the admin notification email.

### Non-Functional Requirements
- No new dependencies (Google Apps Script).
- No API contract change — `clientUpdateDocs` still takes `bookingId` + `docDriveLink`; it
  only adds rejection of invalid links.
- The customer happy path is unchanged: legitimate Drive/Docs links pass exactly as before.
- Escaping must only neutralize HTML metacharacters; legitimate Thai/plain text must render
  unchanged.

### Out of Scope
- Ownership / access control on `clientUpdateDocs` (requiring a `bookerEmail` match) — needs
  a paired frontend change; deferred to a coordinated FE+BE follow-up.
- Validating the admin-supplied `docLink` in `editBooking` / `updateApplicationStatus` (those
  paths are PIN-gated; left unchanged to avoid breaking legitimate admin input).
- `LockService` on non-`addBooking` write handlers.
- Splitting `main.js` into multiple files.
- Rate limiting.

### Acceptance Criteria
- [ ] A `bookedBy` such as `<script>alert(1)</script>` or `"><b>x` appears **escaped** (as
      visible text) in the pending and approval HTML emails, not as live markup.
- [ ] `clientUpdateDocs` rejects a non-Drive URL (e.g. `https://evil.example/x`) and a
      non-URL string with a clear error; the booking row is not modified and no admin email
      is sent.
- [ ] A valid `https://drive.google.com/...` or `https://docs.google.com/...` link is
      accepted and behaves exactly as before.
- [ ] An over-length `docDriveLink` is rejected.
- [ ] Plain-text emails and all other endpoints are unchanged.

---

## ไทย (Thai)

### ภาพรวม
การรีวิว backend พบช่องโหว่ความปลอดภัยที่จำกัดขอบเขตและแก้ได้ที่ฝั่ง backend อย่างเดียว 2 จุด:

1. **HTML email injection** อีเมลที่ใช้ `htmlBody` สองฉบับ (`sendPendingEmail`,
   `sendApprovalEmail`) นำค่าที่ผู้ใช้ควบคุมได้ (`bookedBy`, `companyName`, `roomLabel`,
   `promotion`, `building`, `floor`) ไปแทรกใน HTML โดยตรง ค่าที่ถูกประดิษฐ์ขึ้นจึงแสดงเป็น
   markup ที่ทำงานได้จริงในกล่องจดหมายของผู้รับ
2. **การเขียนสาธารณะที่ไม่ตรวจสอบ** `clientUpdateDocs` (ไม่ต้องใช้ PIN แอดมิน) รับค่า
   `docDriveLink` ใด ๆ เก็บลงชีต และส่งต่อไปยังอีเมลแจ้งเตือนแอดมิน ดังนั้นผู้ที่มีรหัสการจอง
   สามารถส่งลิงก์นอกโดเมน (ฟิชชิ่ง) ไปยังแอดมินได้

งานนี้แก้ทั้งสองจุด **โดยไม่เปลี่ยนสัญญาของ API**: escape ค่าผู้ใช้ในอีเมล HTML สองฉบับ และ
ตรวจสอบ `docDriveLink` ให้รับเฉพาะ URL ของ Google Drive/Docs ที่ถูกต้อง

### ความต้องการเชิงฟังก์ชัน
1. เพิ่ม helper `escapeHtml(value)` (escape `& < > " '`) และใช้กับทุกค่าที่ผู้ใช้ควบคุมได้ซึ่ง
   ถูกแทรกใน `htmlBody` ของ `sendPendingEmail` และ `sendApprovalEmail`: `bookedBy`,
   `companyName`, `roomLabel`, `promotion`, `building`, `floor`
2. ใน `clientUpdateDocs` ตรวจสอบ `docDriveLink`: ต้องเป็น URL `https://` ที่ host เป็น
   `drive.google.com` หรือ `docs.google.com` มิฉะนั้นปฏิเสธพร้อมข้อความสองภาษาที่ชัดเจน และ
   ไม่แก้ไขแถวใด ๆ
3. จำกัดความยาวของ `docDriveLink` อย่างเหมาะสม (เช่น 2048 ตัวอักษร)
4. ลิงก์ที่ผ่านการตรวจสอบแล้วคือสิ่งที่ถูกเก็บและส่งต่อไปยังอีเมลแจ้งเตือนแอดมิน

### ความต้องการที่ไม่ใช่เชิงฟังก์ชัน
- ไม่เพิ่ม dependency ใหม่ (Google Apps Script)
- ไม่เปลี่ยนสัญญาของ API — `clientUpdateDocs` ยังรับ `bookingId` + `docDriveLink` เพียงเพิ่ม
  การปฏิเสธลิงก์ที่ไม่ถูกต้อง
- เส้นทางปกติของลูกค้าไม่เปลี่ยน: ลิงก์ Drive/Docs ที่ถูกต้องผ่านได้เหมือนเดิม
- การ escape ต้องทำให้เป็นกลางเฉพาะอักขระพิเศษของ HTML เท่านั้น ข้อความไทย/ข้อความธรรมดา
  ที่ถูกต้องต้องแสดงผลเหมือนเดิม

### นอกขอบเขต
- การควบคุมการเข้าถึง/ความเป็นเจ้าของของ `clientUpdateDocs` (บังคับให้ `bookerEmail` ตรงกัน) —
  ต้องแก้ฝั่ง frontend ด้วย เลื่อนไปเป็นงาน FE+BE ที่ประสานกัน
- การตรวจสอบ `docLink` ที่แอดมินกรอกใน `editBooking` / `updateApplicationStatus` (เส้นทางเหล่านี้
  มี PIN กำกับ ความเสี่ยงต่ำ จึงคงไว้เพื่อไม่ให้กระทบการกรอกของแอดมิน)
- `LockService` สำหรับ handler การเขียนที่ไม่ใช่ `addBooking`
- การแยก `main.js` เป็นหลายไฟล์
- การจำกัดอัตราการเรียก (rate limiting)

### เกณฑ์การยอมรับ
- [ ] ค่า `bookedBy` เช่น `<script>alert(1)</script>` หรือ `"><b>x` จะปรากฏแบบ **escape แล้ว**
      (เป็นข้อความที่มองเห็น) ในอีเมล HTML ของ pending และ approval ไม่ใช่ markup ที่ทำงานได้
- [ ] `clientUpdateDocs` ปฏิเสธ URL ที่ไม่ใช่ Drive (เช่น `https://evil.example/x`) และสตริงที่
      ไม่ใช่ URL พร้อมข้อความที่ชัดเจน โดยไม่แก้ไขแถวการจองและไม่ส่งอีเมลแอดมิน
- [ ] ลิงก์ `https://drive.google.com/...` หรือ `https://docs.google.com/...` ที่ถูกต้องผ่านได้
      และทำงานเหมือนเดิมทุกประการ
- [ ] `docDriveLink` ที่ยาวเกินกำหนดถูกปฏิเสธ
- [ ] อีเมลแบบข้อความธรรมดาและ endpoint อื่น ๆ ทั้งหมดไม่เปลี่ยนแปลง
