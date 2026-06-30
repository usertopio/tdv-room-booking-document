# Requirement Rewrite — 2026-06-30 (consolidate-status-transition)

Source: Code review of `tdv-room-booking-backend/src/main.js` (no developer `raw.md`; requirement derived from review findings)
Target repo(s): backend

---

## English

### Overview
The backend's `applicationStatus` transition logic — the block that writes the
dependent row fields (`status`, `approvedAt`, `cancelReason`, `depositPaid`, …) and
fires the matching customer email — is **duplicated across three functions**:
`updateApplicationStatusHandler` (API), `editBookingHandler` (admin edit), and
`handleSheetEdit` (live Sheet-edit trigger). The three copies have **drifted apart**, so
the same logical action produces different data depending on which path the admin uses.
This task consolidates the logic into a single function and, in doing so, reconciles the
divergences so all three paths behave identically.

### Functional Requirements
1. A single function (`applyStatusTransition`) owns the per-status field writes and email
   dispatch. The three call sites delegate to it via an injected `writeField(name, value)`
   closure bound to the target row.
2. `completed` must write `receiptUrl` (when available) on **all three** paths — today only
   the API path does; the edit and Sheet paths silently drop the receipt.
3. `rejected` must store the resolved reason in **both** `cancelReason` and `rejectReason`
   on all paths — today the API path can leave `rejectReason` blank.
4. `pending` and `reviewing` must reset `status = "booked"` uniformly — today only the Sheet
   path handles `pending`.
5. `editBookingHandler` must continue to record field changes in its `changes{}` audit
   object (no loss of audit detail).
6. Per-path email error-log contexts must stay distinguishable (e.g. an `:api` / `:edit` /
   `:sheet` suffix), so failures can still be traced to their origin.
7. The list of valid statuses should have a **single definition** (in `CONFIG`) rather than
   being hardcoded in three places.

### Non-Functional Requirements
- No new dependencies (Google Apps Script does not support npm packages).
- No change to the public API contract — request/response shapes for every endpoint stay
  identical.
- Net reduction in duplicated code (≈120 transition lines → ≈50 in one place).
- The production path (`updateApplicationStatusHandler`) keeps its current behavior, with
  the single exception of now filling `rejectReason` (a fix, not a regression).

### Out of Scope
- Adding `LockService` to non-`addBooking` write handlers.
- HTML-escaping interpolated values in email templates.
- Authentication / rate-limiting for the public `clientUpdateDocs` endpoint.
- Splitting `main.js` into multiple files.
- Sharing the `findBooking` boilerplate (open sheet → read → find → headers) across
  handlers — a related but separate cleanup.

  *(These were noted in the review but are deferred to their own issues to keep this change
  focused.)*

### Acceptance Criteria
- [ ] All three call sites (`updateApplicationStatusHandler`, `editBookingHandler`,
      `handleSheetEdit`) delegate to one shared `applyStatusTransition` function.
- [ ] Marking a booking `completed` via the admin edit form or by editing the Sheet writes
      `receiptUrl` when one is available.
- [ ] Rejecting a booking via the API fills `rejectReason` (not just `cancelReason`).
- [ ] Setting `pending` resets `status = "booked"` on every path.
- [ ] `editBookingHandler` still records every changed field in `changes{}`.
- [ ] No endpoint's request or response shape changes.
- [ ] The valid-status list is defined once and reused.

---

## ไทย (Thai)

### ภาพรวม
ตรรกะการเปลี่ยนสถานะ `applicationStatus` ของ backend — ส่วนที่เขียนฟิลด์ที่เกี่ยวข้องในแถว
(`status`, `approvedAt`, `cancelReason`, `depositPaid`, …) และส่งอีเมลถึงลูกค้าที่ตรงกับ
สถานะ — ถูก **เขียนซ้ำใน 3 ฟังก์ชัน** ได้แก่ `updateApplicationStatusHandler` (API),
`editBookingHandler` (แอดมินแก้ไข) และ `handleSheetEdit` (ทริกเกอร์แก้ไขในชีตโดยตรง) ทั้ง
สามชุดได้ **เคลื่อนออกจากกัน (drift)** แล้ว ทำให้การกระทำเชิงตรรกะเดียวกันให้ผลข้อมูลต่างกัน
ขึ้นกับว่าแอดมินทำผ่านเส้นทางใด งานนี้จะรวมตรรกะให้เหลือฟังก์ชันเดียว และปรับให้ทั้งสามเส้นทาง
ทำงานเหมือนกัน

### ความต้องการเชิงฟังก์ชัน
1. มีฟังก์ชันเดียว (`applyStatusTransition`) เป็นเจ้าของการเขียนฟิลด์ตามสถานะและการส่งอีเมล
   โดยผู้เรียกส่ง closure `writeField(name, value)` ที่ผูกกับแถวเป้าหมายเข้ามา
2. สถานะ `completed` ต้องเขียน `receiptUrl` (เมื่อมี) ใน **ทั้งสาม** เส้นทาง — ปัจจุบันมีเฉพาะ
   เส้นทาง API เท่านั้น ส่วนเส้นทางแก้ไขและชีตทิ้งค่าใบเสร็จไป
3. สถานะ `rejected` ต้องเก็บเหตุผลที่ resolve แล้วใน **ทั้ง** `cancelReason` และ `rejectReason`
   ทุกเส้นทาง — ปัจจุบันเส้นทาง API อาจปล่อย `rejectReason` ว่าง
4. สถานะ `pending` และ `reviewing` ต้องรีเซ็ต `status = "booked"` ให้เหมือนกัน — ปัจจุบันมีเพียง
   เส้นทางชีตที่จัดการ `pending`
5. `editBookingHandler` ต้องยังบันทึกการเปลี่ยนแปลงฟิลด์ใน `changes{}` เหมือนเดิม (ไม่สูญเสีย
   รายละเอียด audit)
6. context ของ log อีเมลที่ผิดพลาดในแต่ละเส้นทางต้องยังแยกแยะได้ (เช่นใส่ suffix `:api` /
   `:edit` / `:sheet`) เพื่อให้ติดตามต้นทางของความล้มเหลวได้
7. รายการสถานะที่ถูกต้องควรมี **คำนิยามเดียว** (ใน `CONFIG`) แทนการ hardcode ซ้ำสามที่

### ความต้องการที่ไม่ใช่เชิงฟังก์ชัน
- ไม่เพิ่ม dependency ใหม่ (Google Apps Script ไม่รองรับแพ็กเกจ npm)
- ไม่เปลี่ยนสัญญา (contract) ของ API สาธารณะ — รูปแบบ request/response ของทุก endpoint คงเดิม
- ลดโค้ดที่ซ้ำซ้อนลง (≈120 บรรทัดของตรรกะ transition → ≈50 บรรทัดในที่เดียว)
- เส้นทาง production (`updateApplicationStatusHandler`) คงพฤติกรรมเดิม ยกเว้นการเติม
  `rejectReason` (ซึ่งเป็นการแก้ไข ไม่ใช่ regression)

### นอกขอบเขต
- การเพิ่ม `LockService` ให้กับ handler การเขียนที่ไม่ใช่ `addBooking`
- การ HTML-escape ค่าที่นำไปแทรกในเทมเพลตอีเมล
- การยืนยันตัวตน / จำกัดอัตราการเรียกของ endpoint สาธารณะ `clientUpdateDocs`
- การแยก `main.js` ออกเป็นหลายไฟล์
- การแชร์โค้ดซ้ำ `findBooking` (เปิดชีต → อ่าน → ค้นหา → headers) ระหว่าง handler — เป็นการ
  ทำความสะอาดที่เกี่ยวข้องแต่แยกต่างหาก

  *(รายการเหล่านี้ถูกระบุไว้ในการรีวิวแล้ว แต่เลื่อนไปเป็น issue ของตัวเองเพื่อให้งานนี้โฟกัส)*

### เกณฑ์การยอมรับ
- [ ] ทั้งสามจุดเรียก (`updateApplicationStatusHandler`, `editBookingHandler`,
      `handleSheetEdit`) มอบหมายงานให้ฟังก์ชัน `applyStatusTransition` เดียวกัน
- [ ] การตั้งสถานะ `completed` ผ่านฟอร์มแก้ไขของแอดมินหรือการแก้ในชีต เขียน `receiptUrl`
      เมื่อมีค่า
- [ ] การปฏิเสธ (`rejected`) ผ่าน API เติม `rejectReason` (ไม่ใช่แค่ `cancelReason`)
- [ ] การตั้งสถานะ `pending` รีเซ็ต `status = "booked"` ในทุกเส้นทาง
- [ ] `editBookingHandler` ยังบันทึกทุกฟิลด์ที่เปลี่ยนใน `changes{}`
- [ ] ไม่มี endpoint ใดที่รูปแบบ request หรือ response เปลี่ยนไป
- [ ] รายการสถานะที่ถูกต้องถูกนิยามเพียงครั้งเดียวและนำกลับมาใช้ซ้ำ
