# Implementation Plan — 2026-06-30-ci-manual-backend-select

Requirements: `requirements/2026-06-30/`
Target repo(s): frontend

## Draft Issue

```
Repo: depa-platform/tdv-room-booking-frontend
Title: chore(ci): add prod/demo backend selection to manual Docker builds | เพิ่มการเลือก backend prod/demo สำหรับการ build Docker แบบแมนนวล
Description:
  Requirements: requirements/2026-06-30/
  Implementation: issues/2026-06-30-ci-manual-backend-select/

  Add a `workflow_dispatch` choice input `backend` (prod | demo, default prod) to
  docker-publish.yml. When demo is selected, build with the demo secrets
  (VITE_APPS_SCRIPT_URL_DEMO, VITE_ADMIN_PIN_DEMO) and publish a `demo-<commit_hash>`
  tag only, so prod images (:dev, :<commit_hash>, :latest) are never overwritten.
  Push/tag builds and manual prod builds keep their current behavior.

  เพิ่ม choice input `backend` (prod | demo, ค่าเริ่มต้น prod) ใน workflow_dispatch ของ
  docker-publish.yml เมื่อเลือก demo จะ build ด้วย secret ของ demo
  (VITE_APPS_SCRIPT_URL_DEMO, VITE_ADMIN_PIN_DEMO) และสร้างเฉพาะแท็ก `demo-<commit_hash>`
  เพื่อไม่ให้ทับอิมเมจ prod (:dev, :<commit_hash>, :latest) ส่วนการ build จาก push/tag และ
  การ build แบบแมนนวลที่เลือก prod จะคงพฤติกรรมเดิม
```

---

## English

### Approach
Single-file YAML change to `.github/workflows/docker-publish.yml`. Introduce a
`workflow_dispatch` choice input `backend`, then make both the **secret selection**
(build-args) and the **tag set** conditional on it using GitHub Actions expressions.
The conditions are written so that non-manual events (where the `inputs` context is empty)
fall through to the existing prod behavior unchanged.

### Steps
1. Declare the input under `workflow_dispatch`:
   ```yaml
   workflow_dispatch:
     inputs:
       backend:
         description: 'Backend environment to build against'
         type: choice
         options: [prod, demo]
         default: prod
   ```
2. Make `build-args` select secrets by `inputs.backend` (ternary idiom):
   ```yaml
   build-args: |
     VITE_APPS_SCRIPT_URL=${{ inputs.backend == 'demo' && secrets.VITE_APPS_SCRIPT_URL_DEMO || secrets.VITE_APPS_SCRIPT_URL }}
     VITE_ADMIN_PIN=${{ inputs.backend == 'demo' && secrets.VITE_ADMIN_PIN_DEMO || secrets.VITE_ADMIN_PIN }}
   ```
   - Note: if `VITE_ADMIN_PIN_DEMO` is unset, the empty middle operand makes the idiom
     fall back to the prod PIN — so only `VITE_APPS_SCRIPT_URL_DEMO` is strictly required.
3. Gate the existing tags off for demo, and add a demo-only tag:
   ```yaml
   tags: |
     type=ref,event=branch,enable=${{ github.ref == 'refs/heads/dev' && inputs.backend != 'demo' }}
     type=ref,event=tag
     type=raw,value=latest,enable=${{ startsWith(github.ref, 'refs/tags/v') }}
     type=sha,prefix=,format=short,enable=${{ startsWith(github.ref, 'refs/heads/feat/') && inputs.backend != 'demo' }}
     type=sha,prefix=demo-,format=short,enable=${{ inputs.backend == 'demo' }}
   ```
4. Update the explanatory comment block above `tags:` to document the demo case.

### Affected Areas
- `.github/workflows/docker-publish.yml` (only). No application code, no dependencies.

### Risks / Side Effects
- **Empty-input safety:** for `push`/`tag` events the `inputs` context is empty, so
  `inputs.backend == 'demo'` is false and `inputs.backend != 'demo'` is true → existing
  prod behavior is preserved. Verified by reasoning about each event type.
- **Missing demo secret:** if `VITE_APPS_SCRIPT_URL_DEMO` is not created, a demo build
  falls back to the prod URL (build still succeeds, but is not actually "demo"). Documented
  as a developer prerequisite; not a breakage.
- **Tag collision:** mitigated by suppressing `:dev`/`:<hash>` when `backend == 'demo'` and
  emitting only `demo-<hash>`.
- **Ternary idiom caveat:** safe here because secret values are non-empty strings; the
  only intentional empty case (`VITE_ADMIN_PIN_DEMO`) yields the documented prod fallback.

### Validation
- `npm run lint` and `npm run build` (sanity; the change is YAML-only and does not touch
  app code, so these should be unaffected).
- YAML reviewed for correct expression syntax.
- Post-merge manual checks (developer, after adding the demo secret):
  - `gh workflow run docker-publish.yml --ref dev -f backend=demo` → only `demo-<hash>` tag.
  - `gh workflow run docker-publish.yml --ref dev` → `:dev`, prod secrets (unchanged).

---

## ไทย (Thai)

### แนวทาง
แก้ไฟล์ YAML ไฟล์เดียวคือ `.github/workflows/docker-publish.yml` โดยเพิ่ม choice input
`backend` ใน `workflow_dispatch` จากนั้นทำให้ทั้งการ **เลือก secret** (build-args) และ
**ชุดแท็ก** ขึ้นกับค่า input นี้ผ่าน expression ของ GitHub Actions เงื่อนไขถูกเขียนให้
เหตุการณ์ที่ไม่ใช่แมนนวล (ซึ่ง context `inputs` ว่าง) ไหลลงไปใช้พฤติกรรม prod เดิมไม่เปลี่ยน

### ขั้นตอน
1. ประกาศ input ใต้ `workflow_dispatch` (choice: prod/demo, ค่าเริ่มต้น prod)
2. ทำให้ `build-args` เลือก secret ตาม `inputs.backend` ด้วยสำนวน ternary
   - หมายเหตุ: หากไม่ได้ตั้ง `VITE_ADMIN_PIN_DEMO` สำนวนนี้จะ fallback ไปใช้ PIN ของ prod
     ดังนั้น secret ใหม่ที่จำเป็นจริง ๆ มีเพียง `VITE_APPS_SCRIPT_URL_DEMO`
3. ปิดแท็กเดิมเมื่อเป็น demo และเพิ่มแท็กเฉพาะ demo (`type=sha,prefix=demo-`)
4. อัปเดตคอมเมนต์อธิบายเหนือ `tags:` ให้ครอบคลุมกรณี demo

### ส่วนที่ได้รับผลกระทบ
- `.github/workflows/docker-publish.yml` เท่านั้น ไม่มีโค้ดแอป ไม่มี dependency

### ความเสี่ยง / ผลข้างเคียง
- **ความปลอดภัยเมื่อ input ว่าง:** เหตุการณ์ `push`/`tag` มี context `inputs` ว่าง ทำให้
  `inputs.backend == 'demo'` เป็นเท็จ และ `inputs.backend != 'demo'` เป็นจริง → คงพฤติกรรม
  prod เดิมไว้
- **ไม่มี secret demo:** หากยังไม่สร้าง `VITE_APPS_SCRIPT_URL_DEMO` การ build demo จะ
  fallback ไปใช้ URL ของ prod (build สำเร็จแต่ไม่ใช่ demo จริง) ถือเป็นสิ่งที่ผู้พัฒนาต้อง
  เตรียมก่อน ไม่ใช่ข้อผิดพลาด
- **แท็กชนกัน:** ลดความเสี่ยงด้วยการปิด `:dev`/`:<hash>` เมื่อ `backend == 'demo'` และสร้าง
  เฉพาะ `demo-<hash>`
- **ข้อควรระวังของสำนวน ternary:** ปลอดภัยเพราะค่า secret ไม่ว่าง ส่วนกรณีว่างโดยตั้งใจ
  (`VITE_ADMIN_PIN_DEMO`) จะให้ผลเป็น fallback ไปใช้ prod ตามที่ระบุไว้

### การตรวจสอบ
- `npm run lint` และ `npm run build` (ตรวจความเรียบร้อย แม้การเปลี่ยนแปลงเป็น YAML ล้วน
  และไม่แตะโค้ดแอป จึงไม่ควรได้รับผลกระทบ)
- ตรวจ syntax ของ expression ใน YAML
- ตรวจแบบแมนนวลหลัง merge (ผู้พัฒนา หลังเพิ่ม secret ของ demo แล้ว):
  - `gh workflow run docker-publish.yml --ref dev -f backend=demo` → ได้เฉพาะแท็ก `demo-<hash>`
  - `gh workflow run docker-publish.yml --ref dev` → ได้ `:dev` และใช้ secret prod (เหมือนเดิม)
