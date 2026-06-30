# Codebase Check — 2026-06-30-ci-manual-backend-select

Requirements: `requirements/2026-06-30/`
Target repo(s): frontend

---

## English

For each requirement, mark **Implemented**, **Partially Implemented**, or **Not Found**,
with file paths and line numbers as evidence. All references are to
`.github/workflows/docker-publish.yml` on `dev` (after PR #91 merged).

| # | Requirement | Status | Evidence (file:line) | Notes |
|---|-------------|--------|----------------------|-------|
| 1 | `workflow_dispatch` choice input `backend` (prod/demo, default prod) | Not Found | `docker-publish.yml:11` | `workflow_dispatch:` exists but declares no `inputs`. |
| 2 | Demo build uses demo secrets | Not Found | `docker-publish.yml:52-54` | `build-args` hardcode `secrets.VITE_APPS_SCRIPT_URL` / `secrets.VITE_ADMIN_PIN` only. |
| 3 | Prod/push build uses prod secrets (preserve) | Implemented | `docker-publish.yml:52-54` | Current behavior; must be kept as the default branch of the new logic. |
| 4 | Distinct demo image tag; no `:dev` / `:<hash>` collision | Not Found | `docker-publish.yml:39-43` | `type=ref,event=branch` (`:dev`, line 40) and `type=sha` (`:<hash>`, line 43) would still fire on a manual demo run and overwrite prod images. |
| 5 | Existing tag behavior preserved for non-demo builds | Implemented | `docker-publish.yml:39-43` | `:dev`, `:<hash>`, `:vX.Y.Z`+`:latest` rules already present; must remain for non-demo. |

### Summary
The manual trigger exists (`workflow_dispatch:`) but has **no inputs**, the build always
uses the **prod** secrets, and the tag rules have **no demo branch** — so a manual demo
run today would both use prod secrets and overwrite prod images. Work needed: add the
`backend` choice input, make secret selection conditional on it, and add a `demo-`prefixed
tag while suppressing the normal tags when `backend = demo`. Prod/push paths are already
correct and only need to be preserved.

---

## ไทย (Thai)

สำหรับแต่ละความต้องการ ให้ระบุสถานะ **Implemented**, **Partially Implemented** หรือ
**Not Found** พร้อมอ้างอิงไฟล์และหมายเลขบรรทัดเป็นหลักฐาน ทุกการอ้างอิงคือไฟล์
`.github/workflows/docker-publish.yml` บน `dev` (หลังจาก merge PR #91)

| # | ความต้องการ | สถานะ | หลักฐาน (file:line) | หมายเหตุ |
|---|-------------|-------|---------------------|----------|
| 1 | choice input `backend` ใน `workflow_dispatch` (prod/demo, ค่าเริ่มต้น prod) | Not Found | `docker-publish.yml:11` | มี `workflow_dispatch:` แต่ยังไม่ประกาศ `inputs` |
| 2 | การ build แบบ demo ใช้ secret ของ demo | Not Found | `docker-publish.yml:52-54` | `build-args` กำหนดตายตัวเป็น `secrets.VITE_APPS_SCRIPT_URL` / `secrets.VITE_ADMIN_PIN` เท่านั้น |
| 3 | การ build แบบ prod/push ใช้ secret ของ prod (คงไว้) | Implemented | `docker-publish.yml:52-54` | เป็นพฤติกรรมปัจจุบัน ต้องคงไว้เป็นค่าเริ่มต้นของลอจิกใหม่ |
| 4 | แท็ก demo แยกต่างหาก ไม่ชนกับ `:dev` / `:<hash>` | Not Found | `docker-publish.yml:39-43` | `type=ref,event=branch` (`:dev`, บรรทัด 40) และ `type=sha` (`:<hash>`, บรรทัด 43) จะยังทำงานตอน run demo แบบแมนนวลและไปทับอิมเมจ prod |
| 5 | คงพฤติกรรมแท็กเดิมสำหรับการ build ที่ไม่ใช่ demo | Implemented | `docker-publish.yml:39-43` | กฎ `:dev`, `:<hash>`, `:vX.Y.Z`+`:latest` มีอยู่แล้ว ต้องคงไว้สำหรับกรณีที่ไม่ใช่ demo |

### สรุป
ปุ่มสั่งแบบแมนนวลมีอยู่แล้ว (`workflow_dispatch:`) แต่ **ไม่มี inputs**, การ build ใช้ secret
ของ **prod** เสมอ และกฎการติดแท็ก **ไม่มีสาขาสำหรับ demo** ดังนั้นการ run demo แบบแมนนวล
ในตอนนี้จะทั้งใช้ secret ของ prod และทับอิมเมจของ prod งานที่ต้องทำ: เพิ่ม choice input
`backend`, ทำให้การเลือก secret ขึ้นกับค่านี้ และเพิ่มแท็กที่ขึ้นต้นด้วย `demo-` พร้อมระงับ
แท็กปกติเมื่อ `backend = demo` ส่วนเส้นทาง prod/push ถูกต้องอยู่แล้ว เพียงแค่ต้องคงไว้
