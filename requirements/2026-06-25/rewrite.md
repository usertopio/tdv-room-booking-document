# Force Refresh — Bypass Backend Cache on Manual Refresh

## English

### Background

The admin dashboard provides a "รีเฟรช" (Refresh) button on both the **Dashboard** and **BookingsManager** pages. Currently, this button calls `apiGetData()` which sends a standard `GET ?action=getData` request to the Apps Script backend. The backend's `getData()` function checks `CacheService` first (TTL: 120 seconds) and returns cached data if available.

This means the refresh button often returns the same data as the automatic background polling (via `useAutoRefresh`, every 10 seconds), making it functionally redundant when the cache has not expired.

### Requirement

1. **Manual refresh must bypass backend cache**: When the admin clicks the "รีเฟรช" button, the request must skip `CacheService` and read directly from the Google Sheet, ensuring fresh data is returned.
2. **Auto-refresh must continue using cache**: The background polling (every 10 seconds via `useAutoRefresh`) must continue reading from the cache as normal to minimize Google Sheet reads and quota usage.
3. **Cache should be refreshed after a force read**: After reading fresh data from the Sheet, the backend should update the cache so that subsequent auto-refresh polls benefit from the fresh data.

### Scope

- **Backend (Apps Script)**: Add support for a `nocache` parameter in the `getData` endpoint.
- **Frontend (React)**: Add a `nocache` option to `apiGetData()` and use it in `handleRefresh()` on both Dashboard and BookingsManager.

---

## Thai / ภาษาไทย

### บริบท

แดชบอร์ดแอดมินมีปุ่ม "รีเฟรช" ทั้งในหน้า **Dashboard** และ **BookingsManager** ปัจจุบันปุ่มนี้เรียก `apiGetData()` ซึ่งส่ง request `GET ?action=getData` ไปยัง Apps Script backend ฟังก์ชัน `getData()` ใน backend จะตรวจสอบ `CacheService` ก่อน (TTL: 120 วินาที) และส่งข้อมูลจาก cache กลับมาถ้ายังไม่หมดอายุ

ทำให้ปุ่มรีเฟรชมักจะได้ข้อมูลเดิมเหมือนกับที่ auto-refresh (ทุก 10 วินาที ผ่าน `useAutoRefresh`) ดึงมา ปุ่มจึงแทบไม่มีประโยชน์เมื่อ cache ยังไม่หมดอายุ

### ข้อกำหนด

1. **การรีเฟรชด้วยตนเองต้องข้าม backend cache**: เมื่อแอดมินกดปุ่ม "รีเฟรช" request ต้องข้าม `CacheService` และอ่านข้อมูลจาก Google Sheet โดยตรง เพื่อให้ได้ข้อมูลล่าสุดเสมอ
2. **Auto-refresh ต้องยังใช้ cache ตามปกติ**: การ poll อัตโนมัติ (ทุก 10 วินาที ผ่าน `useAutoRefresh`) ต้องยังอ่านจาก cache เพื่อลดการอ่าน Sheet และ quota
3. **Cache ต้องถูกอัปเดตหลังอ่านข้อมูลสด**: หลังจากอ่านข้อมูลสดจาก Sheet แล้ว backend ต้องเก็บผลลัพธ์ลง cache ใหม่ เพื่อให้ auto-refresh ครั้งถัดไปได้ข้อมูลที่สดกว่า
