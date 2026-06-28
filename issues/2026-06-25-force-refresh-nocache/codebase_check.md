# Codebase Check — Force Refresh (Bypass Backend Cache)

Requirements: material/requirement/2026-06-25/

## English

### Requirement 1: Manual refresh must bypass backend cache

**Status: Not Found ❌**

Currently, the refresh button calls `apiGetData()` with no cache-bypass option:

- **Frontend `handleRefresh`** (Dashboard.jsx:82–98, BookingsManager.jsx:115–131): Calls `apiGetData({ silent: false, includeAll: true, ... })` — no `nocache` parameter.
- **Frontend `apiGetData`** (bookingService.js:23–48): Builds URL as `?action=getData&includeAll=1` — no support for `nocache` query parameter.
- **Backend `route("getData")`** (main.js:141–147): Passes only `includeAll` to `getData()` — no `nocache` parameter forwarded.
- **Backend `getData()`** (main.js:275–313): Always checks `CacheService.getScriptCache().get(cacheKey)` first — no bypass logic.

### Requirement 2: Auto-refresh must continue using cache

**Status: Implemented ✅**

- **`useAutoRefresh` hook** (useAutoRefresh.js:7–31): Calls the provided callback every `AUTO_REFRESH_INTERVAL` (10,000ms). The callback is `apiGetData()` without any special parameters, which hits the backend cache normally.
- **Backend `getData()`** (main.js:280–284): Reads from `CacheService` by default.

No changes needed for this requirement.

### Requirement 3: Cache should be refreshed after a force read

**Status: Partially Implemented ⚠️**

- **Backend `getData()`** (main.js:305–310): After reading from the Sheet, it writes the result to cache via `cache.put(cacheKey, JSON.stringify(result), 120)`.
- This already works — when the cache is bypassed and a fresh Sheet read occurs, the result will be stored in cache. The only addition needed is ensuring the cache is cleared before the fresh read (to avoid stale entries in other keys like `stats_cache`).

---

## Thai / ภาษาไทย

### ข้อกำหนดที่ 1: การรีเฟรชด้วยตนเองต้องข้าม backend cache

**สถานะ: ไม่พบ ❌**

ปัจจุบัน ปุ่มรีเฟรชเรียก `apiGetData()` โดยไม่มีตัวเลือกข้าม cache:

- **Frontend `handleRefresh`** (Dashboard.jsx:82–98, BookingsManager.jsx:115–131): เรียก `apiGetData()` โดยไม่มี parameter `nocache`
- **Frontend `apiGetData`** (bookingService.js:23–48): สร้าง URL เป็น `?action=getData&includeAll=1` — ไม่มี `nocache`
- **Backend `route("getData")`** (main.js:141–147): ส่งแค่ `includeAll` ไปยัง `getData()` — ไม่มี `nocache`
- **Backend `getData()`** (main.js:275–313): ตรวจ cache ก่อนเสมอ — ไม่มีทางข้าม

### ข้อกำหนดที่ 2: Auto-refresh ต้องยังใช้ cache ตามปกติ

**สถานะ: ใช้งานแล้ว ✅**

ไม่ต้องแก้ไข

### ข้อกำหนดที่ 3: Cache ต้องถูกอัปเดตหลังอ่านข้อมูลสด

**สถานะ: ใช้งานบางส่วน ⚠️**

`getData()` เขียน cache หลังอ่าน Sheet อยู่แล้ว แต่ต้องเพิ่มการ clear cache ก่อนอ่านเพื่อป้องกัน stale data ใน key อื่น
