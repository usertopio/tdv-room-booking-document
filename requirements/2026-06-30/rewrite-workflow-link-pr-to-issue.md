# Requirement Rewrite — 2026-06-30 (workflow-link-pr-to-issue)

Source: Developer instruction (2026-06-30) — "always create issue and link PR to issue"; on
review, issue+PR creation is already in CLAUDE.md, but the PR↔issue link is not.
Target repo(s): frontend, backend

---

## English

### Overview
The "create an issue" and "create a PR" steps already exist in both repos' CLAUDE.md
Workflow sections, but neither says to **link the PR back to its issue**. Make that explicit:
the "Create Pull Request" step must require a closing keyword (`Closes #<n>`) in the PR body,
so the PR is linked in GitHub's Development panel and the issue auto-closes on merge.

### Functional Requirements
1. Amend the "Create Pull Request" workflow step in **frontend** `CLAUDE.md` (step 12) to
   require linking the PR to its issue with `Closes #<issue-number>` in the PR body.
2. Amend the equivalent step in **backend** `CLAUDE.md` (step 10) the same way.
3. While editing the backend list, fix the duplicate `10.` numbering (the "Wait for
   developer review and merge" line should be `11.`).

### Non-Functional Requirements
- Documentation change only — no code, no behavior change.
- CLAUDE.md stays **English only** (per its own Documentation Rules); the bilingual
  requirement applies to issues and the artifact docs, not to CLAUDE.md itself.
- Wording consistent across both repos.

### Out of Scope
- Any change to the issue-creation step (linking is a PR-side action).
- Deployment/document repos (their workflows are not affected).
- Enforcing the rule via automation/hooks (documentation only for now).

### Acceptance Criteria
- [ ] Frontend `CLAUDE.md` step 12 states the PR must link its issue via a closing keyword.
- [ ] Backend `CLAUDE.md` "Create Pull Request" step states the same.
- [ ] Backend workflow list numbering is corrected (no duplicate `10.`).
- [ ] No other content changes.

---

## ไทย (Thai)

### ภาพรวม
ขั้นตอน "สร้าง issue" และ "สร้าง PR" มีอยู่แล้วในส่วน Workflow ของ CLAUDE.md ทั้งสอง repo แต่
ยังไม่มีข้อความให้ **ลิงก์ PR กลับไปยัง issue** จึงทำให้ชัดเจน: ขั้นตอน "Create Pull Request"
ต้องกำหนดให้ใส่ closing keyword (`Closes #<n>`) ใน PR body เพื่อให้ PR ถูกลิงก์ใน Development
panel ของ GitHub และ issue ปิดอัตโนมัติเมื่อ merge

### ความต้องการเชิงฟังก์ชัน
1. แก้ขั้นตอน "Create Pull Request" ใน `CLAUDE.md` ของ **frontend** (ข้อ 12) ให้กำหนดการลิงก์
   PR กับ issue ด้วย `Closes #<issue-number>` ใน PR body
2. แก้ขั้นตอนเทียบเท่าใน `CLAUDE.md` ของ **backend** (ข้อ 10) ในลักษณะเดียวกัน
3. ระหว่างแก้รายการของ backend ให้แก้เลขข้อที่ซ้ำ (`10.` ซ้ำ — บรรทัด "Wait for developer
   review and merge" ควรเป็น `11.`)

### ความต้องการที่ไม่ใช่เชิงฟังก์ชัน
- เป็นการแก้เอกสารเท่านั้น — ไม่มีโค้ด ไม่เปลี่ยนพฤติกรรม
- CLAUDE.md ยังคงเป็น **ภาษาอังกฤษอย่างเดียว** (ตาม Documentation Rules ของตัวเอง) ส่วน
  ข้อกำหนดสองภาษาใช้กับ issue และเอกสาร artifact ไม่ใช่ตัว CLAUDE.md
- ใช้ถ้อยคำให้สอดคล้องกันทั้งสอง repo

### นอกขอบเขต
- การเปลี่ยนขั้นตอนสร้าง issue (การลิงก์เป็นการกระทำฝั่ง PR)
- repo deployment/document (ไม่ได้รับผลกระทบ)
- การบังคับใช้กฎผ่าน automation/hooks (ตอนนี้เป็นเอกสารเท่านั้น)

### เกณฑ์การยอมรับ
- [ ] `CLAUDE.md` ของ frontend ข้อ 12 ระบุว่า PR ต้องลิงก์ issue ด้วย closing keyword
- [ ] ขั้นตอน "Create Pull Request" ของ backend ระบุเช่นเดียวกัน
- [ ] เลขข้อในรายการ workflow ของ backend ถูกแก้ (ไม่มี `10.` ซ้ำ)
- [ ] ไม่มีการเปลี่ยนเนื้อหาอื่น
