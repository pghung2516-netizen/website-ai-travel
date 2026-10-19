### 12.4. File `server/src/services/tripService.js` (Transaction Lưu Lịch Trình, Ngân Sách, My Trips)

> `saveTripItinerary` và `getFullTripWithBudget` giữ nguyên từ v1. **Mới:** `createTrip` (gắn `user_id` nếu là thành viên) và `listTrips` (My Trips, P1). Quyền truy cập được kiểm tra ở route bằng `ownerClause()` (mục 12.6) trước khi gọi các hàm này.

```javascript
import pool from '../db/connection.js';
import { calculateTransit } from '../utils/transitCalculator.js';
import { ownerClause } from '../middleware/identity.js';

export async function saveTripItinerary(tripId, daysData) {
  const client = await pool.connect();
  try {
    await client.query('BEGIN');

    for (const dayObj of daysData) {
      const { day_number, items } = dayObj;

      // Xóa các mục cũ của ngày được cập nhật để tránh lệch order_index
      await client.query(
        `DELETE FROM itinerary_items WHERE trip_id = $1 AND day_number = $2`,
        [tripId, day_number]
      );

      const sortedItems = [...items].sort((a, b) => a.order_index - b.order_index);

      for (let i = 0; i < sortedItems.length; i++) {
        const item = sortedItems[i];
        const prevItem = i > 0 ? sortedItems[i - 1] : null;

        const transitInfo = prevItem
          ? calculateTransit(prevItem.latitude, prevItem.longitude, item.latitude, item.longitude)
          : null;

        await client.query(
          `INSERT INTO itinerary_items
            (trip_id, day_number, order_index, time_slot, place_name, description,
             category, place_id_google, latitude, longitude, estimated_cost, transit_info)
           VALUES ($1, $2, $3, $4, $5, $6, $7, $8, $9, $10, $11, $12::jsonb)`,
          [
            tripId,
            day_number,
            item.order_index,
            item.time_slot,
            item.place_name,
            item.description,
            item.category,
            item.place_id_google || null,
            item.latitude,
            item.longitude,
            item.estimated_cost || 0,
            transitInfo ? JSON.stringify(transitInfo) : null
          ]
        );
      }
    }

    await client.query(`UPDATE trips SET status = 'completed', updated_at = NOW() WHERE id = $1`, [
      tripId
    ]);

    await client.query('COMMIT');
    return await getFullTripWithBudget(tripId);
  } catch (err) {
    await client.query('ROLLBACK');
    throw err;
  } finally {
    client.release();
  }
}

export async function getFullTripWithBudget(tripId) {
  const tripRes = await pool.query(`SELECT * FROM trips WHERE id = $1`, [tripId]);
  const trip = tripRes.rows[0];
  if (!trip) return null;

  const itemsRes = await pool.query(
    `SELECT i.*, p.rating, p.address, p.opening_hours, p.photo_url
     FROM itinerary_items i
     LEFT JOIN places_cache p ON i.place_id_google = p.place_id_google
     WHERE i.trip_id = $1
     ORDER BY i.day_number ASC, i.order_index ASC`,
    [tripId]
  );

  const items = itemsRes.rows;
  const activitiesCost = items.reduce((sum, it) => sum + Number(it.estimated_cost || 0), 0);
  const transitCost = items.reduce(
    (sum, it) => sum + Number(it.transit_info?.estimated_fare_vnd || 0),
    0
  );
  const totalEstimated = activitiesCost + transitCost;
  const percentage = Math.round((totalEstimated / trip.budget) * 100);

  let health = 'under_budget';
  if (percentage > 100) health = 'over_budget';
  else if (percentage >= 80) health = 'near_budget';

  return {
    trip,
    itinerary_items: items,
    budget_summary: {
      total_budget: trip.budget,
      activities_cost: activitiesCost,
      transit_cost: transitCost,
      total_estimated: totalEstimated,
      percentage,
      health
    }
  };
}

/**
 * [v2] Tạo chuyến đi. Thành viên: gắn user_id ngay từ đầu. Khách: user_id = NULL.
 * Nếu JWT còn hạn nhưng tài khoản đã bị xóa, FK sẽ chặn INSERT -> route nên trả 401.
 */
export async function createTrip(principal, { start_location, budget, days, preferences = [], source = 'form' }) {
  // Tên mặc định: "2-day HCMC · street food, cafe" (người dùng đổi tên được trong My Trips)
  const prefText = preferences.slice(0, 2).map((p) => p.replace(/_/g, ' ')).join(', ');
  const title = `${days}-day HCMC · ${prefText || 'custom trip'}`.slice(0, 120);

  const res = await pool.query(
    `INSERT INTO trips (session_id, user_id, title, start_location, budget, days, preferences, source, status)
     VALUES ($1, $2, $3, $4, $5, $6, $7::jsonb, $8, 'draft')
     RETURNING *`,
    [principal.sessionId, principal.userId, title, start_location, budget, days, JSON.stringify(preferences), source]
  );
  return res.rows[0];
}

/**
 * [v2, P1] My Trips: danh sách chuyến của người dùng hiện tại (khách hoặc thành viên).
 * Tính sẵn số điểm đến và tổng chi phí ước tính để thẻ trong My Trips không cần gọi thêm API.
 */
export async function listTrips(principal, limit = 20) {
  const { clause, params } = ownerClause(principal, 1, 't'); // alias 't' vì có JOIN
  const res = await pool.query(
    `SELECT t.id, t.title, t.start_location, t.days, t.budget, t.status, t.created_at,
            (t.share_token IS NOT NULL) AS is_shared,
            COUNT(i.id)::int AS place_count,
            (COALESCE(SUM(i.estimated_cost), 0)
             + COALESCE(SUM((i.transit_info->>'estimated_fare_vnd')::int), 0))::int AS total_estimated
     FROM trips t
     LEFT JOIN itinerary_items i ON i.trip_id = t.id
     WHERE ${clause}
     GROUP BY t.id
     ORDER BY t.created_at DESC
     LIMIT $${params.length + 1}`,
    [...params, Math.min(limit, 50)]
  );
  return res.rows;
}
```
