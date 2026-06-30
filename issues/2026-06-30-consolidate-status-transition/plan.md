# Implementation Plan — 2026-06-30-consolidate-status-transition

Requirements: `requirements/2026-06-30/` (`rewrite-consolidate-status-transition.md`)
Target repo(s): backend

## Draft Issue

```
Repo: depa-platform/tdv-room-booking-backend
Title: refactor(status): consolidate applicationStatus transition logic into one function | รวมตรรกะการเปลี่ยน applicationStatus ให้เป็นฟังก์ชันเดียว
Description:
  Requirements: requirements/2026-06-30/
  Implementation: issues/2026-06-30-consolidate-status-transition/

  The applicationStatus transition logic (field writes + customer email) is duplicated
  across updateApplicationStatusHandler, editBookingHandler, and handleSheetEdit, and the
  three copies have drifted: `completed` writes receiptUrl on the API path only, the API
  path can leave rejectReason blank, and `pending` is handled only on the Sheet path.
  Extract one applyStatusTransition(writeField, booking, newStatus, opts) and route all
  three call sites through it, reconciling the drift so every path behaves identically.
  Lift the valid-status list into CONFIG. Preserve editBookingHandler's changes{} audit by
  injecting its change-recording writeField. No endpoint contract changes.

  ตรรกะการเปลี่ยน applicationStatus (การเขียนฟิลด์ + อีเมลถึงลูกค้า) ถูกเขียนซ้ำใน
  updateApplicationStatusHandler, editBookingHandler และ handleSheetEdit และทั้งสามชุด
  เคลื่อนออกจากกัน: `completed` เขียน receiptUrl เฉพาะเส้นทาง API, เส้นทาง API อาจปล่อย
  rejectReason ว่าง, และ `pending` ถูกจัดการเฉพาะเส้นทางชีต งานนี้จะแยกฟังก์ชัน
  applyStatusTransition(writeField, booking, newStatus, opts) เดียวและให้ทั้งสามจุดเรียก
  ไหลผ่านฟังก์ชันนี้ พร้อมปรับ drift ให้ทุกเส้นทางทำงานเหมือนกัน ย้ายรายการสถานะไปไว้ใน
  CONFIG และคง audit changes{} ของ editBookingHandler ไว้โดยส่ง writeField ที่บันทึก
  changes เข้าไป ไม่มีการเปลี่ยนสัญญาของ endpoint
```

---

## English

### Approach
Single-file change to `tdv-room-booking-backend/src/main.js`. Introduce one
`applyStatusTransition(writeField, booking, newStatus, opts)` that owns every per-status
field write and email dispatch. The function takes the `writeField(name, value)` closure as
its first argument, so each caller keeps targeting its own row (and `editBookingHandler` can
pass a change-recording closure to preserve `changes{}`). Replace the three duplicated
blocks with a single delegating call each, reconciling the known drift in the process. Add a
single `CONFIG.APPLICATION_STATUSES` array and reference it everywhere the status list is
currently hardcoded.

### Steps
1. Add the canonical status list to `CONFIG`:
   ```js
   APPLICATION_STATUSES: [
     "pending", "reviewing", "approved",
     "rejected", "payment_submitted", "completed",
   ],
   ```
   Reference it in `updateApplicationStatusHandler` (validation), `handleSheetEdit`
   (validation), and `applyApplicationStatusValidation` (dropdown).
2. Add the shared function (placed near the other workflow helpers):
   ```js
   function applyStatusTransition(writeField, booking, newStatus, opts) {
     opts = opts || {};
     const source = opts.source || "api";
     writeField("applicationStatus", newStatus);

     if (newStatus === "reviewing" || newStatus === "pending") {
       writeField("status", "booked");
     } else if (newStatus === "approved") {
       writeField("status", "booked");
       writeField("approvedAt", new Date());
       writeField("approvedBy", opts.approvedBy || "admin");
       if (opts.docLink) writeField("docDriveLink", opts.docLink);
       try {
         sendApprovalEmail({
           ...booking,
           applicationStatus: "approved",
           docDriveLink: opts.docLink || booking.docDriveLink,
         });
       } catch (e) { logError("approvalEmail:" + source, e); }
     } else if (newStatus === "rejected") {
       const reason = opts.reason || "ไม่ผ่านการพิจารณา";
       writeField("status", "cancelled");
       writeField("cancelledAt", new Date());
       writeField("cancelReason", reason);
       writeField("rejectedAt", new Date());
       writeField("rejectReason", reason);
       try {
         sendRejectionEmail({ ...booking, applicationStatus: "rejected" }, reason);
       } catch (e) { logError("rejectionEmail:" + source, e); }
     } else if (newStatus === "payment_submitted") {
       writeField("status", "booked");
       if (opts.docLink) writeField("docDriveLink", opts.docLink);
     } else if (newStatus === "completed") {
       writeField("status", "booked");
       writeField("depositPaid", "ใช่");
       writeField("depositAmount", CONFIG.DEPOSIT_AMOUNT);
       if (opts.receiptUrl) writeField("receiptUrl", opts.receiptUrl);
       try {
         sendCompletedEmail({
           ...booking,
           applicationStatus: "completed",
           receiptUrl: opts.receiptUrl || booking.receiptUrl || "",
         });
       } catch (e) { logError("completedEmail:" + source, e); }
     }
   }
   ```
3. `updateApplicationStatusHandler` (`main.js:946-992`) → replace the if/else-if chain with:
   ```js
   applyStatusTransition(writeField, target, newStatus, {
     reason, approvedBy, docLink, receiptUrl, source: "api",
   });
   ```
4. `editBookingHandler` (`main.js:748-814`) → after the editable-field loop, replace the
   status block with a single call using the **change-recording** `writeField`:
   ```js
   if (newAppStatus && newAppStatus !== oldAppStatus) {
     applyStatusTransition(writeFieldTracked, target, newAppStatus, {
       reason: updates.notes || "ไม่ผ่านการพิจารณา (แก้ไขโดยแอดมิน)",
       approvedBy: "admin",
       source: "edit",
     });
   }
   ```
   Remove `"applicationStatus"` from the `editable` array (`main.js:731`) so it is written
   once (by the transition), not twice.
5. `handleSheetEdit` (`main.js:1778-1814`) → replace the if/else-if chain with:
   ```js
   applyStatusTransition(writeField, booking, newValue, {
     reason: booking.rejectReason,   // default applied inside
     approvedBy: "Sheet Editor",
     source: "sheet",
   });
   ```
6. Update the header doc-comment at `main.js:883-902` to state the logic now lives in one
   place (remove the "mirrored in editBookingHandler and handleSheetEdit" wording).
7. Add bilingual (EN/TH) comments to the new function per project convention.

### Affected Areas
- `tdv-room-booking-backend/src/main.js` only — `CONFIG`, `updateApplicationStatusHandler`,
  `editBookingHandler`, `handleSheetEdit`, `applyApplicationStatusValidation`, plus the new
  `applyStatusTransition`. No manifest, no dependencies, no frontend change.

### Risks / Side Effects
- **Intentional behavior changes (the drift fixes):** the edit and Sheet paths will now
  write `receiptUrl` on `completed`, the API path will now fill `rejectReason`, and
  `pending` will reset `status="booked"` everywhere. These are corrections aligned with the
  code's stated intent, but they *are* changes — called out for explicit sign-off.
- **`editBookingHandler` audit:** `changes{}` must keep recording status-driven writes →
  mitigated by passing the change-recording `writeField`, not the plain one. Will be verified
  by inspecting the audit row after an edit-driven status change.
- **Double-write of `applicationStatus` in edit path:** avoided by removing it from
  `editable` (step 4) so only the transition writes it.
- **`approved`/`payment_submitted` `docLink`:** the edit path has no `docLink` concept today;
  passing `undefined` leaves `docDriveLink` untouched (handled by the `if (opts.docLink)`
  guard) — no behavior change there.
- **Single-file blast radius:** GAS is one global scope; the change touches the workflow
  core. Mitigated by keeping the public route/handler signatures and all endpoint
  request/response shapes identical.

### Validation
GAS has no `npm run lint` / `build`; validation is by reasoning + manual runs in the editor
or via `clasp`.
- Re-read each of the three call sites and confirm the produced field writes/emails match
  the pre-refactor behavior for every status (a transition-by-transition diff table).
- In the Apps Script editor against a **non-production** copy (or the demo project): run
  `addBookingHandler` test data, then exercise `updateApplicationStatusHandler` for each
  status and confirm sheet fields + emails.
- Edit the `applicationStatus` cell directly to fire `handleSheetEdit`; confirm identical
  results and that `AuditLog` records the change.
- `clasp push` to a test deployment only; production deploy remains a manual maintainer step.

---

## ไทย (Thai)

### แนวทาง
แก้ไฟล์เดียวคือ `tdv-room-booking-backend/src/main.js` โดยเพิ่มฟังก์ชัน
`applyStatusTransition(writeField, booking, newStatus, opts)` หนึ่งเดียวที่เป็นเจ้าของการเขียน
ฟิลด์ตามสถานะและการส่งอีเมลทั้งหมด ฟังก์ชันรับ closure `writeField(name, value)` เป็นอาร์กิวเมนต์
แรก เพื่อให้ผู้เรียกแต่ละจุดยังเขียนลงแถวของตนได้ (และ `editBookingHandler` ส่ง closure ที่บันทึก
การเปลี่ยนแปลงเข้าไปเพื่อรักษา `changes{}`) จากนั้นแทนที่บล็อกซ้ำทั้งสามด้วยการเรียกฟังก์ชันเดียว
พร้อมปรับ drift ให้ตรงกัน และเพิ่มอาเรย์ `CONFIG.APPLICATION_STATUSES` เดียวเพื่อใช้แทนรายการ
สถานะที่ hardcode อยู่

### ขั้นตอน
1. เพิ่มรายการสถานะมาตรฐานใน `CONFIG` (`APPLICATION_STATUSES`) และอ้างอิงใน
   `updateApplicationStatusHandler` (validation), `handleSheetEdit` (validation) และ
   `applyApplicationStatusValidation` (dropdown)
2. เพิ่มฟังก์ชันกลาง `applyStatusTransition` (วางใกล้ helper ของ workflow อื่น ๆ) ตามโค้ดในฉบับ
   ภาษาอังกฤษ
3. `updateApplicationStatusHandler` (`main.js:946-992`) → แทน if/else-if ด้วยการเรียก
   `applyStatusTransition(writeField, target, newStatus, { reason, approvedBy, docLink, receiptUrl, source: "api" })`
4. `editBookingHandler` (`main.js:748-814`) → หลังลูปแก้ไขฟิลด์ แทนบล็อกสถานะด้วยการเรียก
   ฟังก์ชันโดยใช้ `writeField` ที่บันทึก changes และลบ `"applicationStatus"` ออกจากอาเรย์
   `editable` (`main.js:731`) เพื่อไม่ให้เขียนซ้ำสองครั้ง
5. `handleSheetEdit` (`main.js:1778-1814`) → แทน if/else-if ด้วยการเรียก
   `applyStatusTransition(writeField, booking, newValue, { reason: booking.rejectReason, approvedBy: "Sheet Editor", source: "sheet" })`
6. อัปเดตคอมเมนต์หัวข้อที่ `main.js:883-902` ให้ระบุว่าตรรกะอยู่ในที่เดียวแล้ว (ลบข้อความ
   "mirrored in editBookingHandler and handleSheetEdit")
7. เพิ่มคอมเมนต์สองภาษา (EN/TH) ให้ฟังก์ชันใหม่ตามแนวทางของโปรเจกต์

### ส่วนที่ได้รับผลกระทบ
- `tdv-room-booking-backend/src/main.js` เท่านั้น — `CONFIG`, `updateApplicationStatusHandler`,
  `editBookingHandler`, `handleSheetEdit`, `applyApplicationStatusValidation` และฟังก์ชันใหม่
  `applyStatusTransition` ไม่แตะ manifest, dependency หรือ frontend

### ความเสี่ยง / ผลข้างเคียง
- **การเปลี่ยนพฤติกรรมโดยตั้งใจ (การแก้ drift):** เส้นทางแก้ไขและชีตจะเขียน `receiptUrl` เมื่อ
  `completed`, เส้นทาง API จะเติม `rejectReason`, และ `pending` จะรีเซ็ต `status="booked"` ทุก
  เส้นทาง เป็นการแก้ให้ตรงเจตนาของโค้ด แต่ก็ถือเป็นการเปลี่ยนแปลง จึงต้องขออนุมัติชัดเจน
- **audit ของ `editBookingHandler`:** `changes{}` ต้องยังบันทึกการเขียนที่เกิดจากสถานะ →
  ลดความเสี่ยงด้วยการส่ง `writeField` ที่บันทึก changes เข้าไป จะตรวจสอบจากแถว audit หลังการแก้ไข
- **การเขียน `applicationStatus` ซ้ำในเส้นทางแก้ไข:** เลี่ยงด้วยการลบออกจาก `editable` (ขั้นตอน 4)
- **`docLink` ของ `approved`/`payment_submitted`:** เส้นทางแก้ไขไม่มีแนวคิด `docLink` การส่ง
  `undefined` จะไม่แตะ `docDriveLink` (มี guard `if (opts.docLink)`) จึงไม่เปลี่ยนพฤติกรรม
- **ผลกระทบวงกว้างของไฟล์เดียว:** GAS เป็น global scope เดียว การแก้แตะแกนของ workflow ลดความ
  เสี่ยงด้วยการคงลายเซ็นของ route/handler และรูปแบบ request/response ของทุก endpoint ไว้เหมือนเดิม

### การตรวจสอบ
GAS ไม่มี `npm run lint` / `build` การตรวจสอบทำโดยการไล่ตรรกะ + การรันด้วยมือใน editor หรือผ่าน
`clasp`
- ไล่อ่านทั้งสามจุดเรียกและยืนยันว่าการเขียนฟิลด์/อีเมลที่ได้ตรงกับพฤติกรรมก่อน refactor ทุกสถานะ
  (ตารางเทียบสถานะต่อสถานะ)
- ใน Apps Script editor บนสำเนาที่ **ไม่ใช่ production** (หรือโปรเจกต์ demo): รันข้อมูลทดสอบด้วย
  `addBookingHandler` แล้วเรียก `updateApplicationStatusHandler` ทุกสถานะ ตรวจฟิลด์ในชีต + อีเมล
- แก้เซลล์ `applicationStatus` โดยตรงเพื่อกระตุ้น `handleSheetEdit` ยืนยันผลตรงกันและ `AuditLog`
  บันทึกการเปลี่ยนแปลง
- `clasp push` ไปยัง deployment ทดสอบเท่านั้น การ deploy production ยังเป็นขั้นตอนของผู้ดูแลด้วยมือ
