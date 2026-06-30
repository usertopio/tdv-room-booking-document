# Codebase Check — 2026-06-30-workflow-link-pr-to-issue

Requirements: `requirements/2026-06-30/` (`rewrite-workflow-link-pr-to-issue.md`)
Target repo(s): frontend, backend

---

## English

For each requirement, mark **Implemented**, **Partially Implemented**, or **Not Found**,
with file paths and line numbers as evidence.

| # | Requirement (target state) | Status | Evidence (file:line) | Notes |
|---|----------------------------|--------|----------------------|-------|
| 1 | Frontend CLAUDE.md requires PR↔issue link (closing keyword) | Partially Implemented | `tdv-room-booking-frontend/CLAUDE.md:130`, `:136` | Step 6 creates the issue and step 12 creates the PR, but step 12 says only "Create Pull Request (`gh pr create --base dev`)" — no linking instruction. |
| 2 | Backend CLAUDE.md requires PR↔issue link (closing keyword) | Partially Implemented | `tdv-room-booking-backend/CLAUDE.md:70`, `:75` | Step 5 creates the issue and step 10 creates the PR; step 10 has no linking instruction. |
| 3 | Backend workflow list numbered correctly | Not Found | `tdv-room-booking-backend/CLAUDE.md:75-76` | Two consecutive `10.` items ("Create Pull Request" and "Wait for developer review and merge"); the second should be `11.`. |

### Summary
Both repos already document creating an issue and creating a PR, but neither ties them
together — the link (a closing keyword in the PR body) is undocumented. The backend list also
has a duplicate `10.` numbering. Work needed: one clause added to each repo's "Create Pull
Request" step, plus the backend numbering fix. Documentation-only, English-only (per
CLAUDE.md's own rule).

---

## ไทย (Thai)

สำหรับแต่ละความต้องการ ให้ระบุสถานะ **Implemented**, **Partially Implemented** หรือ
**Not Found** พร้อมอ้างอิงไฟล์และหมายเลขบรรทัดเป็นหลักฐาน

| # | ความต้องการ (สถานะเป้าหมาย) | สถานะ | หลักฐาน (file:line) | หมายเหตุ |
|---|------------------------------|-------|---------------------|----------|
| 1 | CLAUDE.md ของ frontend กำหนดการลิงก์ PR↔issue (closing keyword) | Partially Implemented | `tdv-room-booking-frontend/CLAUDE.md:130`, `:136` | ข้อ 6 สร้าง issue และข้อ 12 สร้าง PR แต่ข้อ 12 ระบุเพียง "Create Pull Request (`gh pr create --base dev`)" ไม่มีคำสั่งให้ลิงก์ |
| 2 | CLAUDE.md ของ backend กำหนดการลิงก์ PR↔issue (closing keyword) | Partially Implemented | `tdv-room-booking-backend/CLAUDE.md:70`, `:75` | ข้อ 5 สร้าง issue และข้อ 10 สร้าง PR แต่ข้อ 10 ไม่มีคำสั่งให้ลิงก์ |
| 3 | เลขข้อในรายการ workflow ของ backend ถูกต้อง | Not Found | `tdv-room-booking-backend/CLAUDE.md:75-76` | มี `10.` ติดกันสองข้อ ("Create Pull Request" และ "Wait for developer review and merge") ข้อหลังควรเป็น `11.` |

### สรุป
ทั้งสอง repo มีการบันทึกขั้นตอนสร้าง issue และสร้าง PR อยู่แล้ว แต่ยังไม่ผูกสองสิ่งเข้าด้วยกัน —
การลิงก์ (closing keyword ใน PR body) ยังไม่ถูกบันทึก อีกทั้งรายการของ backend ยังมีเลข `10.`
ซ้ำ งานที่ต้องทำ: เพิ่มข้อความหนึ่งประโยคในขั้นตอน "Create Pull Request" ของแต่ละ repo พร้อม
แก้เลขข้อของ backend เป็นการแก้เอกสารเท่านั้น และเป็นภาษาอังกฤษอย่างเดียว (ตามกฎของ CLAUDE.md เอง)
