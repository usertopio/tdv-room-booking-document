# Task Summary — 2026-06-30-workflow-link-pr-to-issue

Requirements: `requirements/2026-06-30/` (`rewrite-workflow-link-pr-to-issue.md`)
Issue: depa-platform/tdv-room-booking-frontend#94 · depa-platform/tdv-room-booking-backend#28
Pull Request: depa-platform/tdv-room-booking-frontend#95 · depa-platform/tdv-room-booking-backend#29

---

## English

### What Was Done
Documented the PR↔issue linking rule in both code repos' `CLAUDE.md` Workflow sections. The
"Create Pull Request" step now requires a closing keyword (`Closes #<issue-number>`) in the
PR body, so the PR is linked in GitHub's Development panel and the issue auto-closes on
merge. The backend list's duplicate `10.` numbering was also fixed.

### Files Changed
- `tdv-room-booking-frontend/CLAUDE.md` — step 12 ("Create Pull Request") amended (1 line).
- `tdv-room-booking-backend/CLAUDE.md` — step 10 ("Create Pull Request") amended + step 11
  renumbered (was a duplicate `10.`).

### Validation
- Markdown-only change in both repos; lint/build do not cover docs (and GAS has no
  npm lint/build). Verified by re-reading both edited Workflow sections; backend list now
  numbers sequentially.
- Both PRs themselves follow the new rule (`Closes #94`, `Closes #28`).

### Notes / Follow-ups
- Two independent PRs, both open against `dev` and **not merged**; awaiting developer
  review/merge: frontend #95, backend #29.
- After both merge, delete the now-redundant `feedback-issue-and-link-pr` auto-memory, since
  CLAUDE.md becomes the source of truth for this rule.
- Backend PR #27 (security hardening) remains open in parallel; this PR touches only
  `CLAUDE.md`, so there is no overlap.

---

## ไทย (Thai)

### สิ่งที่ทำ
บันทึกกฎการลิงก์ PR↔issue ลงในส่วน Workflow ของ `CLAUDE.md` ทั้งสอง code repo โดยขั้นตอน
"Create Pull Request" กำหนดให้ใส่ closing keyword (`Closes #<issue-number>`) ใน PR body เพื่อ
ให้ PR ถูกลิงก์ใน Development panel ของ GitHub และ issue ปิดอัตโนมัติเมื่อ merge และแก้เลขข้อ
`10.` ที่ซ้ำในรายการของ backend ด้วย

### ไฟล์ที่เปลี่ยน
- `tdv-room-booking-frontend/CLAUDE.md` — แก้ข้อ 12 ("Create Pull Request") (1 บรรทัด)
- `tdv-room-booking-backend/CLAUDE.md` — แก้ข้อ 10 ("Create Pull Request") + แก้เลขข้อ 11
  (เดิมเป็น `10.` ซ้ำ)

### การตรวจสอบ
- แก้ Markdown เท่านั้นทั้งสอง repo lint/build ไม่ครอบคลุมเอกสาร (และ GAS ไม่มี npm lint/build)
  ตรวจสอบโดยอ่านส่วน Workflow ที่แก้ทั้งสองไฟล์ซ้ำ รายการของ backend เรียงเลขต่อเนื่องแล้ว
- PR ทั้งสองเองทำตามกฎใหม่ (`Closes #94`, `Closes #28`)

### หมายเหตุ / งานต่อเนื่อง
- PR สองอันที่เป็นอิสระต่อกัน เปิดอยู่กับ `dev` และ **ยังไม่ merge** กำลังรอผู้พัฒนา review/merge:
  frontend #95, backend #29
- หลังจาก merge ทั้งคู่ ให้ลบ auto-memory `feedback-issue-and-link-pr` ที่ซ้ำซ้อน เพราะ CLAUDE.md
  จะเป็นแหล่งความจริงของกฎนี้
- PR #27 ของ backend (security hardening) ยังเปิดอยู่คู่ขนาน PR นี้แตะเฉพาะ `CLAUDE.md` จึงไม่ทับซ้อน
