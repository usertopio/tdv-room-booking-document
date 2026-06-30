# Codebase Check — 2026-06-30-security-hardening

Requirements: `requirements/2026-06-30/` (`rewrite-security-hardening.md`)
Target repo(s): backend

---

## English

For each requirement, mark **Implemented**, **Partially Implemented**, or **Not Found**,
with file paths and line numbers as evidence. All references are to
`tdv-room-booking-backend/src/main.js` on `dev` (after PR #25 merged).

| # | Requirement (target state) | Status | Evidence (file:line) | Notes |
|---|----------------------------|--------|----------------------|-------|
| 1 | `escapeHtml` helper exists | Not Found | — | `grep` for `escapeHtml`/`function escape` returns nothing. |
| 2 | User values escaped in `sendPendingEmail` HTML | Not Found | `main.js:1235`, `main.js:1240-1242` | `bookedBy`, `building`, `floor`, `roomLabel`, `promotion`, `companyName` interpolated raw into `htmlBody`. |
| 3 | User values escaped in `sendApprovalEmail` HTML | Not Found | `main.js:1247`, `main.js:1284-1290` | `bookedBy`, `building`, `floor`, `roomLabel`, `promotion` interpolated raw into `htmlBody`. |
| 4 | `docDriveLink` validated as Google Drive/Docs URL | Not Found | `main.js:1097-1099` | Only presence is checked (`if (!docDriveLink)`); any string is accepted. |
| 5 | Validated link stored / forwarded to admin email | Partially Implemented | `main.js:1112`, `main.js:1138` | The link IS stored and forwarded — but unvalidated, so an arbitrary URL reaches the admin's inbox. |
| 6 | Length cap on `docDriveLink` | Not Found | `main.js:1097-1099` | No length check. |

### Summary
Neither protection exists today. The two `htmlBody` emails (`sendPendingEmail`,
`sendApprovalEmail`) are the only HTML-injection vectors — the rejection/completed/admin
emails use plain-text `body`, so escaping is cleanly scoped to those two functions. The
public `clientUpdateDocs` endpoint checks only that `docDriveLink` is non-empty, then stores
it and emails it to the admin, so it is both a stored-arbitrary-URL and a phishing-delivery
vector. Work needed: add an `escapeHtml` helper and apply it to the user fields in the two
HTML emails; add Drive/Docs URL validation + a length cap in `clientUpdateDocs`. All
backend-only, no API contract change.

---

## ไทย (Thai)

สำหรับแต่ละความต้องการ ให้ระบุสถานะ **Implemented**, **Partially Implemented** หรือ
**Not Found** พร้อมอ้างอิงไฟล์และหมายเลขบรรทัดเป็นหลักฐาน ทุกการอ้างอิงคือไฟล์
`tdv-room-booking-backend/src/main.js` บน `dev` (หลัง merge PR #25)

| # | ความต้องการ (สถานะเป้าหมาย) | สถานะ | หลักฐาน (file:line) | หมายเหตุ |
|---|------------------------------|-------|---------------------|----------|
| 1 | มี helper `escapeHtml` | Not Found | — | `grep` หา `escapeHtml`/`function escape` ไม่พบ |
| 2 | escape ค่าผู้ใช้ใน HTML ของ `sendPendingEmail` | Not Found | `main.js:1235`, `main.js:1240-1242` | `bookedBy`, `building`, `floor`, `roomLabel`, `promotion`, `companyName` ถูกแทรกใน `htmlBody` แบบดิบ |
| 3 | escape ค่าผู้ใช้ใน HTML ของ `sendApprovalEmail` | Not Found | `main.js:1247`, `main.js:1284-1290` | `bookedBy`, `building`, `floor`, `roomLabel`, `promotion` ถูกแทรกใน `htmlBody` แบบดิบ |
| 4 | ตรวจสอบ `docDriveLink` ว่าเป็น URL ของ Google Drive/Docs | Not Found | `main.js:1097-1099` | ตรวจเพียงว่ามีค่า (`if (!docDriveLink)`) รับสตริงใด ๆ |
| 5 | เก็บ/ส่งต่อลิงก์ที่ผ่านการตรวจสอบไปยังอีเมลแอดมิน | Partially Implemented | `main.js:1112`, `main.js:1138` | ลิงก์ถูกเก็บและส่งต่อ — แต่ไม่ได้ตรวจสอบ จึงมี URL ใด ๆ ไปถึงกล่องจดหมายแอดมิน |
| 6 | จำกัดความยาวของ `docDriveLink` | Not Found | `main.js:1097-1099` | ไม่มีการตรวจความยาว |

### สรุป
ปัจจุบันยังไม่มีการป้องกันทั้งสองอย่าง อีเมลที่ใช้ `htmlBody` สองฉบับ (`sendPendingEmail`,
`sendApprovalEmail`) เป็นช่องทาง HTML injection เพียงจุดเดียว — อีเมล rejection/completed/admin
ใช้ `body` แบบข้อความธรรมดา การ escape จึงจำกัดอยู่ที่สองฟังก์ชันนี้ ส่วน endpoint สาธารณะ
`clientUpdateDocs` ตรวจเพียงว่า `docDriveLink` ไม่ว่าง แล้วเก็บและส่งต่อไปยังแอดมิน จึงเป็นทั้ง
ช่องเก็บ URL ใด ๆ และช่องส่งฟิชชิ่ง งานที่ต้องทำ: เพิ่ม helper `escapeHtml` และใช้กับฟิลด์ผู้ใช้
ในอีเมล HTML สองฉบับ; เพิ่มการตรวจสอบ URL ของ Drive/Docs + จำกัดความยาวใน `clientUpdateDocs`
ทั้งหมดทำที่ backend อย่างเดียว ไม่เปลี่ยนสัญญาของ API
