# Implementation Plan — Force Refresh (Bypass Backend Cache)

Requirements: material/requirement/2026-06-25/

## Draft Issue

Title: feat: add force refresh to bypass backend cache / เพิ่มรีเฟรชข้าม cache ดึงข้อมูลสดจาก Sheet

Description:
  Requirements: material/requirement/2026-06-25/
  Implementation: material/implementation/implementation_01/

  Add a `nocache` parameter to the `getData` API endpoint so that the admin refresh button
  fetches data directly from Google Sheet, bypassing the 120-second CacheService TTL.
  Auto-refresh (background polling) continues using the cache as normal.

  เพิ่ม parameter `nocache` ให้กับ endpoint `getData` เพื่อให้ปุ่มรีเฟรชในหน้าแอดมิน
  ดึงข้อมูลจาก Google Sheet โดยตรง ข้าม cache 120 วินาที
  auto-refresh (polling อัตโนมัติ) ยังใช้ cache ตามปกติ

---

## English

### Changes Required

#### Backend (tdv-room-booking-backend)

##### [MODIFY] main.js — `route("getData")`
- Parse `nocache` parameter from request params (same pattern as `includeAll`)
- Pass `nocache` option to `getData()`

**Lines 141–147:**
```javascript
case "getData": {
  const includeAll =
    params.includeAll === "1" || params.includeAll === 1 || params.includeAll === true;
  const nocache =
    params.nocache === "1" || params.nocache === 1 || params.nocache === true;
  return jsonResponse({ ok: true, ...getData({ includeAll, nocache }) });
}
```

##### [MODIFY] main.js — `getData()`
- If `opts.nocache` is true, call `clearBookingsCache()` to clear all cache keys, then skip the cache read
- The fresh Sheet read and subsequent `cache.put()` already handle re-caching

**Lines 275–287:**
```javascript
function getData(opts) {
  opts = opts || {};
  const cache = CacheService.getScriptCache();
  const cacheKey = opts.includeAll ? "bookings_all" : "bookings_active";

  if (opts.nocache) {
    clearBookingsCache();
  } else {
    try {
      const cached = cache.get(cacheKey);
      if (cached) {
        return JSON.parse(cached);
      }
    } catch (e) {
      Logger.log("Cache read error: " + e.message);
    }
  }
  // ... rest unchanged — reads Sheet and writes to cache
```

---

#### Frontend (tdv-room-booking-frontend)

##### [MODIFY] bookingService.js — `apiGetData()`
- Add `nocache` option to function signature
- Append `&nocache=1` to URL when `nocache` is true

**Line 23 and 31:**
```javascript
export async function apiGetData({ silent = false, includeAll = false, nocache = false, onStatus, onSpinner } = {}) {
  // ...
  const url = `${CONFIG.APPS_SCRIPT_URL}?action=getData${includeAll ? '&includeAll=1' : ''}${nocache ? '&nocache=1' : ''}&_=${Date.now()}`;
```

##### [MODIFY] Dashboard.jsx — `handleRefresh()`
- Add `nocache: true` to the `apiGetData()` call

**Line 85:**
```javascript
const data = await apiGetData({
  silent: false,
  includeAll: true,
  nocache: true,
  onStatus: setConnection,
  onSpinner: setLoading,
});
```

##### [MODIFY] BookingsManager.jsx — `handleRefresh()`
- Add `nocache: true` to the `apiGetData()` call

**Line 118:**
```javascript
const data = await apiGetData({
  silent: false,
  includeAll: true,
  nocache: true,
  onStatus: setConnection,
  onSpinner: setLoading,
});
```

### Files Unchanged
- `useAutoRefresh.js` — no changes; continues calling callback without `nocache`
- All other `apiGetData()` calls (in `updateStatus`, `handleEditSubmit`, `handleCancelSubmit`, etc.) — these are post-mutation calls that already benefit from `clearBookingsCache()` being called in the backend mutation handlers

---

## Thai / ภาษาไทย

### สรุปการแก้ไข

| Repo | ไฟล์ | สิ่งที่แก้ |
|---|---|---|
| Backend | main.js `route` | เพิ่ม parse `nocache` param แล้วส่งเข้า `getData()` |
| Backend | main.js `getData` | ถ้า `nocache` → เรียก `clearBookingsCache()` แล้วข้ามการอ่าน cache |
| Frontend | bookingService.js | เพิ่ม option `nocache` ใน `apiGetData()` |
| Frontend | Dashboard.jsx | ส่ง `nocache: true` ตอนกดปุ่มรีเฟรช |
| Frontend | BookingsManager.jsx | ส่ง `nocache: true` ตอนกดปุ่มรีเฟรช |

### Validation

- **Backend**: Deploy with `clasp push`, test via browser `?action=getData&nocache=1` and verify fresh Sheet data
- **Frontend**: `npm run lint && npm run build`
