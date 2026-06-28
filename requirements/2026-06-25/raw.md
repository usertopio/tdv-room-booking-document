# Force Refresh from Google Sheet

Currently, the refresh button in admin dashboard (Dashboard + BookingsManager) just calls `apiGetData` which hits the backend cache. If the cache hasn't expired (120s TTL), the button returns stale data — same data as auto-refresh.

I want the refresh button to bypass backend cache and fetch fresh data directly from Google Sheet. Auto-refresh (every 10s) should still use the cache as normal.
