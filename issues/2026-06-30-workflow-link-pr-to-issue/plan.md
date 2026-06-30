# Implementation Plan — 2026-06-30-workflow-link-pr-to-issue

Requirements: `requirements/2026-06-30/` (`rewrite-workflow-link-pr-to-issue.md`)
Target repo(s): frontend, backend

## Draft Issue (frontend)

```
Repo: depa-platform/tdv-room-booking-frontend
Title: docs(workflow): require linking the PR to its issue with a closing keyword | กำหนดให้ลิงก์ PR กับ issue ด้วย closing keyword
Description:
  Requirements: requirements/2026-06-30/
  Implementation: issues/2026-06-30-workflow-link-pr-to-issue/

  Amend Workflow Rules step 12 ("Create Pull Request") in CLAUDE.md to require linking the
  PR to its issue via a closing keyword (Closes #<n>) in the PR body, so the PR is linked in
  GitHub and the issue auto-closes on merge. Documentation-only, English-only.

  แก้ Workflow Rules ข้อ 12 ("Create Pull Request") ใน CLAUDE.md ให้กำหนดการลิงก์ PR กับ
  issue ด้วย closing keyword (Closes #<n>) ใน PR body เพื่อให้ PR ถูกลิงก์ใน GitHub และ
  issue ปิดอัตโนมัติเมื่อ merge เป็นการแก้เอกสารเท่านั้น ภาษาอังกฤษอย่างเดียว
```

## Draft Issue (backend)

```
Repo: depa-platform/tdv-room-booking-backend
Title: docs(workflow): require linking the PR to its issue with a closing keyword | กำหนดให้ลิงก์ PR กับ issue ด้วย closing keyword
Description:
  Requirements: requirements/2026-06-30/
  Implementation: issues/2026-06-30-workflow-link-pr-to-issue/

  Amend the "Create Pull Request" workflow step in CLAUDE.md to require linking the PR to its
  issue via a closing keyword (Closes #<n>) in the PR body. Also fix the duplicate "10."
  numbering (Wait for developer review and merge → 11). Documentation-only, English-only.

  แก้ขั้นตอน "Create Pull Request" ใน CLAUDE.md ให้กำหนดการลิงก์ PR กับ issue ด้วย closing
  keyword (Closes #<n>) ใน PR body และแก้เลขข้อที่ซ้ำ ("10." → "Wait..." ควรเป็น 11) เป็นการ
  แก้เอกสารเท่านั้น ภาษาอังกฤษอย่างเดียว
```

---

## English

### Approach
Two independent, documentation-only PRs — one per code repo — applying the same one-clause
amendment to the "Create Pull Request" workflow step, plus a numbering fix in the backend.
No dependency between them; both branch from their repo's `dev`.

### Steps
**Frontend** (`tdv-room-booking-frontend/CLAUDE.md`, line 136):
- Change:
  `12. Create Pull Request (\`gh pr create --base dev\`)`
  → `12. Create Pull Request (\`gh pr create --base dev\`) and link it to the issue by including a closing keyword (\`Closes #<issue-number>\`) in the PR body, so the PR is linked to the issue and the issue auto-closes on merge`

**Backend** (`tdv-room-booking-backend/CLAUDE.md`, lines 75-76):
- Change line 75:
  `10. Create Pull Request (\`gh pr create --base dev\`)`
  → `10. Create Pull Request (\`gh pr create --base dev\`) and link it to the issue by including a closing keyword (\`Closes #<issue-number>\`) in the PR body, so the PR is linked to the issue and the issue auto-closes on merge`
- Fix line 76 numbering: `10. Wait for developer review and merge` → `11. Wait for developer review and merge`

### Affected Areas
- `tdv-room-booking-frontend/CLAUDE.md` (one line).
- `tdv-room-booking-backend/CLAUDE.md` (two lines).
- No code, no dependencies, no behavior change.

### Risks / Side Effects
- Documentation-only; the only risk is wording drift between the two repos — mitigated by
  using identical phrasing.
- Reflects existing practice (PRs #25, #27 already used `Closes #<n>`), so no process change
  for anyone, just codification.

### Validation
- Re-read the edited Workflow sections in both files; confirm wording matches and backend
  numbering is now sequential.
- Frontend: `npm run lint && npm run build` is unaffected (Markdown-only change) but can be
  run as a sanity check.
- Backend: no `npm` lint/build; doc change needs no runtime check.

---

## ไทย (Thai)

### แนวทาง
PR แบบแก้เอกสารอย่างเดียวสองอันที่เป็นอิสระต่อกัน — repo ละหนึ่งอัน — ใส่ข้อความเพิ่มประโยคเดียว
เดียวกันในขั้นตอน "Create Pull Request" พร้อมแก้เลขข้อในฝั่ง backend ไม่มีการพึ่งพากัน ทั้งคู่
แตกสาขาจาก `dev` ของ repo ตนเอง

### ขั้นตอน
**Frontend** (`tdv-room-booking-frontend/CLAUDE.md` บรรทัด 136):
- แก้ข้อ 12 ให้เพิ่มข้อความว่าต้องลิงก์ PR กับ issue ด้วย closing keyword (`Closes #<issue-number>`)
  ใน PR body เพื่อให้ issue ปิดอัตโนมัติเมื่อ merge

**Backend** (`tdv-room-booking-backend/CLAUDE.md` บรรทัด 75-76):
- แก้บรรทัด 75 (ข้อ 10 "Create Pull Request") เพิ่มข้อความลิงก์แบบเดียวกัน
- แก้เลขข้อบรรทัด 76: `10. Wait for developer review and merge` → `11. Wait for developer review and merge`

### ส่วนที่ได้รับผลกระทบ
- `tdv-room-booking-frontend/CLAUDE.md` (หนึ่งบรรทัด)
- `tdv-room-booking-backend/CLAUDE.md` (สองบรรทัด)
- ไม่มีโค้ด ไม่มี dependency ไม่เปลี่ยนพฤติกรรม

### ความเสี่ยง / ผลข้างเคียง
- แก้เอกสารเท่านั้น ความเสี่ยงเดียวคือถ้อยคำต่างกันระหว่างสอง repo — ลดด้วยการใช้ข้อความเหมือนกัน
- สะท้อนแนวปฏิบัติที่ทำอยู่แล้ว (PR #25, #27 ใช้ `Closes #<n>` แล้ว) จึงไม่เปลี่ยนกระบวนการ เพียง
  บันทึกให้เป็นลายลักษณ์อักษร

### การตรวจสอบ
- อ่านส่วน Workflow ที่แก้ในทั้งสองไฟล์ ยืนยันถ้อยคำตรงกันและเลขข้อของ backend เรียงต่อเนื่อง
- Frontend: `npm run lint && npm run build` ไม่ได้รับผลกระทบ (แก้ Markdown เท่านั้น) แต่รันเพื่อ
  ตรวจความเรียบร้อยได้
- Backend: ไม่มี lint/build ผ่าน `npm` การแก้เอกสารไม่ต้องตรวจขณะรัน
