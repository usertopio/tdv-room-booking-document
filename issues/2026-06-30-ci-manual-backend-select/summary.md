# Task Summary — 2026-06-30-ci-manual-backend-select

Requirements: `requirements/2026-06-30/`
Issue: depa-platform/tdv-room-booking-frontend#92
Pull Request: depa-platform/tdv-room-booking-frontend#93

---

## English

### What Was Done
Added a `workflow_dispatch` choice input `backend` (`prod` | `demo`, default `prod`) to
the frontend's `docker-publish.yml`. A manual run with `backend=demo` builds the image
against the demo secrets and publishes a demo-only tag, leaving prod images untouched.
Automatic builds (push to `dev`/`feat/*`, `v*` tags) and manual `prod` builds are
unchanged because `inputs.backend` is empty on non-dispatch events and resolves to prod.

### Files Changed
- `.github/workflows/docker-publish.yml`
  - Added the `backend` choice input under `workflow_dispatch`.
  - `build-args` select secrets via `inputs.backend == 'demo' && secrets.*_DEMO || secrets.*`.
  - Tags: gated `:dev` / `:<commit_hash>` off when `backend == 'demo'`, added
    `type=sha,prefix=demo-` so demo builds publish only `demo-<commit_hash>`.
  - Updated the explanatory comments.

### Validation
- `npm run lint` — pass (no errors).
- `npm run build` — pass (pre-existing >500 kB chunk warning only).
- YAML parse check — OK. Tag/secret resolution traced for push, tag, manual-prod, and
  manual-demo — only `demo-<hash>` is emitted for demo; all other paths unchanged.

### Notes / Follow-ups
- **Developer prerequisite:** add the `VITE_APPS_SCRIPT_URL_DEMO` repo secret for demo
  builds to be meaningful. `VITE_ADMIN_PIN_DEMO` is optional (demo falls back to the prod
  PIN when unset).
- The GitHub Actions VS Code extension warns "Context access might be invalid" for the
  `*_DEMO` secrets until they exist — expected, clears once the secrets are created.
- Usage: `gh workflow run docker-publish.yml --ref dev -f backend=demo`.
- PR #93 is open and **not merged**; awaiting developer review/merge.

---

## ไทย (Thai)

### สิ่งที่ทำ
เพิ่ม choice input `backend` (`prod` | `demo`, ค่าเริ่มต้น `prod`) ใน `workflow_dispatch`
ของไฟล์ `docker-publish.yml` ฝั่ง frontend เมื่อ run แบบแมนนวลด้วย `backend=demo` จะ build
อิมเมจโดยใช้ secret ของ demo และสร้างเฉพาะแท็กของ demo โดยไม่กระทบอิมเมจ prod ส่วนการ build
อัตโนมัติ (push `dev`/`feat/*`, tag `v*`) และการ build แบบแมนนวลที่เลือก `prod` ไม่เปลี่ยนแปลง
เพราะ `inputs.backend` ว่างในเหตุการณ์ที่ไม่ใช่ dispatch จึง resolve เป็น prod

### ไฟล์ที่เปลี่ยน
- `.github/workflows/docker-publish.yml`
  - เพิ่ม choice input `backend` ใต้ `workflow_dispatch`
  - `build-args` เลือก secret ผ่าน `inputs.backend == 'demo' && secrets.*_DEMO || secrets.*`
  - แท็ก: ปิด `:dev` / `:<commit_hash>` เมื่อ `backend == 'demo'` และเพิ่ม
    `type=sha,prefix=demo-` เพื่อให้ demo สร้างเฉพาะ `demo-<commit_hash>`
  - ปรับคอมเมนต์อธิบาย

### การตรวจสอบ
- `npm run lint` — ผ่าน (ไม่มี error)
- `npm run build` — ผ่าน (มีเพียงคำเตือน chunk > 500 kB ที่มีอยู่เดิม)
- ตรวจ YAML parse — ผ่าน ตรวจสอบการ resolve แท็ก/secret ครบทุกกรณี push, tag, manual-prod
  และ manual-demo — กรณี demo สร้างเฉพาะ `demo-<hash>` ส่วนเส้นทางอื่นไม่เปลี่ยน

### หมายเหตุ / งานต่อเนื่อง
- **สิ่งที่ผู้พัฒนาต้องเตรียม:** เพิ่ม repo secret `VITE_APPS_SCRIPT_URL_DEMO` เพื่อให้การ build
  demo มีความหมาย ส่วน `VITE_ADMIN_PIN_DEMO` ไม่บังคับ (ถ้าไม่มี demo จะ fallback ไปใช้ PIN prod)
- ส่วนขยาย GitHub Actions ของ VS Code จะเตือน "Context access might be invalid" สำหรับ
  secret `*_DEMO` จนกว่าจะสร้าง — เป็นเรื่องปกติ จะหายเมื่อสร้าง secret แล้ว
- วิธีใช้: `gh workflow run docker-publish.yml --ref dev -f backend=demo`
- PR #93 เปิดอยู่และ **ยังไม่ merge** กำลังรอผู้พัฒนา review/merge
