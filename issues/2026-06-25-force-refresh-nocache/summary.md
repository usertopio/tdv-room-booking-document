Requirements: material/requirement/2026-06-25/
Issue: #70
Backend PR: #18
Frontend PR: #69

## English

### Summary of Implementation

Successfully implemented the "Force Refresh" feature to bypass the backend cache, allowing administrators to fetch guaranteed-fresh data from the Google Sheet when clicking the refresh button.

1. **Backend Changes (tdv-room-booking-backend)**
   - Modified `route` in `main.js` to parse the `nocache` parameter.
   - Updated `getData()` to conditionally skip the cache and invoke `clearBookingsCache()` when `opts.nocache` is `true`.
   - The response is still written to the cache after the fresh read, ensuring background polling stays efficient.

2. **Frontend Changes (tdv-room-booking-frontend)**
   - Updated `apiGetData()` in `bookingService.js` to accept a `nocache` option and append `&nocache=1` to the API request URL.
   - Updated `handleRefresh()` in `Dashboard.jsx` to pass `nocache: true`.
   - Updated `handleRefresh()` in `BookingsManager.jsx` to pass `nocache: true`.

### Results
- Background polling (`useAutoRefresh`, every 10s) continues to use the cache normally.
- Manual refresh bypasses the cache, clears it, and provides real-time data.
- The build and linting checks passed successfully. Pull requests have been created for both frontend and backend repositories.

---

## Thai / ภาษาไทย

### สรุปการดำเนินการ

ดำเนินการเพิ่มฟีเจอร์ "Force Refresh" เพื่อข้าม backend cache สำเร็จ ทำให้แอดมินสามารถดึงข้อมูลล่าสุดจาก Google Sheet ได้โดยตรงเมื่อกดปุ่มรีเฟรช

1. **การแก้ไข Backend (tdv-room-booking-backend)**
   - แก้ไขฟังก์ชัน `route` ใน `main.js` เพื่ออ่านค่าพารามิเตอร์ `nocache`
   - ปรับปรุงฟังก์ชัน `getData()` ให้ข้ามการอ่าน cache และเรียกใช้ `clearBookingsCache()` เมื่อ `opts.nocache` เป็น `true`
   - ระบบจะยังคงบันทึกผลลัพธ์กลับลง cache หลังจากดึงข้อมูลสดแล้ว เพื่อให้ auto-refresh ทำงานได้อย่างมีประสิทธิภาพต่อไป

2. **การแก้ไข Frontend (tdv-room-booking-frontend)**
   - อัปเดต `apiGetData()` ใน `bookingService.js` ให้รองรับตัวเลือก `nocache` และส่งต่อ `&nocache=1` ใน URL ของ API
   - อัปเดตฟังก์ชัน `handleRefresh()` ใน `Dashboard.jsx` ให้ส่ง `nocache: true`
   - อัปเดตฟังก์ชัน `handleRefresh()` ใน `BookingsManager.jsx` ให้ส่ง `nocache: true`

### ผลลัพธ์
- การทำ Auto-refresh เบื้องหลัง (ทุก 10 วินาที) ยังคงอ่านจาก cache ตามปกติ
- การรีเฟรชด้วยตนเองจะข้าม cache, เคลียร์ cache, และดึงข้อมูลสดทันที
- การ build และ lint ผ่านเรียบร้อย และได้สร้าง Pull Request สำหรับทั้งฝั่ง frontend และ backend แล้ว
