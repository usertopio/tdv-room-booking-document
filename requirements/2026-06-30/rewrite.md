# Requirement Rewrite — 2026-06-30

Source: `requirements/2026-06-30/raw.md`
Target repo(s): frontend

---

## English

### Overview
Allow a manual run of the frontend Docker CI (`docker-publish.yml`) to select which
backend environment the image is built against — **prod** (default) or **demo** — using
two separate sets of GitHub Actions secrets. Automatic builds (push to `dev` / `feat/*`,
tag pushes) and manual `prod` builds keep their current behavior. Demo-built images must
be tagged distinctly so they can never overwrite prod images on Docker Hub.

### Functional Requirements
1. Add a `workflow_dispatch` **choice input** `backend` with options `prod` and `demo`,
   defaulting to `prod`.
2. When `backend = demo`, build with the demo secrets (`VITE_APPS_SCRIPT_URL_DEMO`, and
   `VITE_ADMIN_PIN_DEMO` when present) instead of the prod secrets.
3. When `backend = prod`, or for any non-manual event (push/tag), build with the existing
   prod secrets (`VITE_APPS_SCRIPT_URL`, `VITE_ADMIN_PIN`).
4. A demo build must publish a **distinct image tag** (e.g. `demo-<commit_hash>`) and must
   **not** emit the normal `:dev` or plain `:<commit_hash>` tags, so prod images are never
   overwritten.
5. Existing tag behavior is preserved for non-demo builds: `dev` push → `:dev`,
   `feat/*` push → `:<commit_hash>`, `v*` tag → `:vX.Y.Z` + `:latest`.

### Non-Functional Requirements
- Secret values must stay **masked** — never typed on the CLI and never printed in run logs.
- YAML-only change to one workflow file; no new dependencies.
- The default selection must be `prod` so existing/manual runs are safe by default.

### Out of Scope
- Free-text / arbitrary backend URL override on the CLI (the rejected "Option A").
- Cross-repo (backend → frontend) triggering.
- Creating the secret **values** themselves (the developer adds the `*_DEMO` secrets).

### Acceptance Criteria
- [ ] `gh workflow run docker-publish.yml --ref dev -f backend=demo` builds against the
      demo secrets and publishes a `demo-`prefixed tag only.
- [ ] `gh workflow run docker-publish.yml --ref dev -f backend=prod` (or no input) builds
      against prod secrets and the existing tag scheme.
- [ ] A demo build does not overwrite `:dev` or a plain `:<commit_hash>` image.
- [ ] Push to `dev`/`feat/*` and `v*` tag pushes behave exactly as before.
- [ ] If `VITE_ADMIN_PIN_DEMO` is not set, a demo build falls back to the prod PIN
      (the demo URL secret is the only required new secret).

---

## ไทย (Thai)

### ภาพรวม
อนุญาตให้การสั่ง CI ของ frontend แบบแมนนวล (`docker-publish.yml`) เลือกได้ว่าจะ build
อิมเมจโดยชี้ไปที่ backend ตัวใด — **prod** (ค่าเริ่มต้น) หรือ **demo** — โดยใช้ชุด secret
ของ GitHub Actions แยกกันสองชุด ส่วนการ build อัตโนมัติ (push `dev` / `feat/*`, push tag)
และการ build แบบแมนนวลที่เลือก `prod` จะคงพฤติกรรมเดิม อิมเมจที่ build แบบ demo ต้องติด
แท็กแยกต่างหากเพื่อไม่ให้ไปทับอิมเมจของ prod บน Docker Hub

### ความต้องการเชิงฟังก์ชัน
1. เพิ่ม **choice input** ชื่อ `backend` ใน `workflow_dispatch` โดยมีตัวเลือก `prod` และ
   `demo` ค่าเริ่มต้นเป็น `prod`
2. เมื่อ `backend = demo` ให้ build ด้วย secret ของ demo (`VITE_APPS_SCRIPT_URL_DEMO`
   และ `VITE_ADMIN_PIN_DEMO` หากมี) แทน secret ของ prod
3. เมื่อ `backend = prod` หรือเหตุการณ์ที่ไม่ใช่แมนนวล (push/tag) ให้ build ด้วย secret ของ
   prod เดิม (`VITE_APPS_SCRIPT_URL`, `VITE_ADMIN_PIN`)
4. การ build แบบ demo ต้องสร้าง **แท็กอิมเมจแยกต่างหาก** (เช่น `demo-<commit_hash>`) และ
   ต้อง **ไม่** สร้างแท็ก `:dev` หรือ `:<commit_hash>` ปกติ เพื่อไม่ให้ทับอิมเมจของ prod
5. คงพฤติกรรมแท็กเดิมสำหรับการ build ที่ไม่ใช่ demo: push `dev` → `:dev`,
   push `feat/*` → `:<commit_hash>`, tag `v*` → `:vX.Y.Z` + `:latest`

### ความต้องการที่ไม่ใช่เชิงฟังก์ชัน
- ค่า secret ต้องถูก **ปกปิด (masked)** — ไม่พิมพ์บน CLI และไม่ปรากฏใน log ของ run
- แก้ไขเฉพาะไฟล์ YAML ไฟล์เดียว ไม่เพิ่ม dependency ใหม่
- ค่าเริ่มต้นต้องเป็น `prod` เพื่อให้การ run เดิม/แมนนวลปลอดภัยโดยปริยาย

### นอกขอบเขต
- การ override URL ของ backend แบบพิมพ์อิสระบน CLI (ซึ่งคือ "Option A" ที่ไม่เลือก)
- การ trigger ข้าม repo (backend → frontend)
- การสร้าง **ค่า** ของ secret เอง (ผู้พัฒนาเป็นผู้เพิ่ม secret `*_DEMO`)

### เกณฑ์การยอมรับ
- [ ] `gh workflow run docker-publish.yml --ref dev -f backend=demo` build ด้วย secret
      ของ demo และสร้างเฉพาะแท็กที่ขึ้นต้นด้วย `demo-`
- [ ] `gh workflow run docker-publish.yml --ref dev -f backend=prod` (หรือไม่ใส่ input)
      build ด้วย secret ของ prod และใช้รูปแบบแท็กเดิม
- [ ] การ build แบบ demo ไม่ทับอิมเมจ `:dev` หรือ `:<commit_hash>`
- [ ] การ push `dev`/`feat/*` และ tag `v*` ทำงานเหมือนเดิมทุกประการ
- [ ] หากไม่ได้ตั้งค่า `VITE_ADMIN_PIN_DEMO` การ build แบบ demo จะ fallback ไปใช้ PIN ของ
      prod (secret URL ของ demo จึงเป็น secret ใหม่ตัวเดียวที่จำเป็น)
