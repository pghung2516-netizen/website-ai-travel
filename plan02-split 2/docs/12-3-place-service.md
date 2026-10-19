### 12.3. File `server/src/services/placeService.js` (Google Places API New + Cache TTL + Ẩn API Key)

> Nội dung giữ nguyên từ v1; chỉ sửa lại các URL bị lỗi định dạng markdown (`PLACES_SEARCH_URL`, `mediaUrl`).

```javascript
import pool from '../db/connection.js';

const PLACES_SEARCH_URL = 'https://places.googleapis.com/v1/places:searchText';
const FIELD_MASK = [
  'places.id',
  'places.displayName',
  'places.formattedAddress',
  'places.location',
  'places.rating',
  'places.types',
  'places.regularOpeningHours.weekdayDescriptions',
  'places.photos'
].join(',');

/**
 * Resolve photo reference sang link CDN lh3.googleusercontent.com để không lộ GOOGLE_PLACES_API_KEY
 */
async function resolveSafePhotoUrl(photoName) {
  if (!photoName || !process.env.GOOGLE_PLACES_API_KEY) return null;
  try {
    const mediaUrl = `https://places.googleapis.com/v1/${photoName}/media?maxWidthPx=600&key=${process.env.GOOGLE_PLACES_API_KEY}`;
    const res = await fetch(mediaUrl, { redirect: 'manual' });
    const redirectLocation = res.headers.get('location');
    return redirectLocation || null;
  } catch {
    return null;
  }
}

/**
 * Tìm địa điểm kết hợp PostgreSQL Cache (TTL 7 ngày) và Google Places API (New)
 */
export async function searchSingleQuery({ query, type, maxResults = 3 }) {
  // 1. Kiểm tra trong places_cache còn hạn trước
  const cachedRes = await pool.query(
    `SELECT place_id_google AS "placeId", name, rating, address, types,
            opening_hours AS "openingHours", latitude AS lat, longitude AS lng, photo_url AS "photoUrl"
     FROM places_cache
     WHERE expires_at > NOW()
       AND (name ILIKE $1 OR address ILIKE $1)
     LIMIT $2`,
    [`%${query}%`, maxResults]
  );

  if (cachedRes.rows.length >= 2) {
    return cachedRes.rows;
  }

  // 2. Gọi Google Places API (New) Text Search nếu chưa đủ cache
  const response = await fetch(PLACES_SEARCH_URL, {
    method: 'POST',
    headers: {
      'Content-Type': 'application/json',
      'X-Goog-Api-Key': process.env.GOOGLE_PLACES_API_KEY,
      'X-Goog-FieldMask': FIELD_MASK
    },
    body: JSON.stringify({
      textQuery: `${query} in Ho Chi Minh City, Vietnam`,
      includedType: type || undefined,
      maxResultCount: Math.min(maxResults, 5)
    })
  });

  if (!response.ok) {
    // Fallback về dữ liệu trong cache kể cả khi hết hạn nếu API lỗi
    const fallbackRes = await pool.query(
      `SELECT place_id_google AS "placeId", name, rating, address,
              opening_hours AS "openingHours", latitude AS lat, longitude AS lng, photo_url AS "photoUrl"
       FROM places_cache
       WHERE name ILIKE $1 OR address ILIKE $1
       LIMIT $2`,
      [`%${query}%`, maxResults]
    );
    return fallbackRes.rows;
  }

  const data = await response.json();
  const places = data.places || [];
  const results = [];

  for (const p of places) {
    const placeId = p.id;
    const name = p.displayName?.text || 'Unknown Venue';
    const rating = p.rating || 4.2;
    const address = p.formattedAddress || 'Ho Chi Minh City';
    const types = p.types || [];
    const openingHours = p.regularOpeningHours?.weekdayDescriptions || [];
    const lat = p.location?.latitude;
    const lng = p.location?.longitude;
    const firstPhotoName = p.photos?.[0]?.name;
    const photoUrl = await resolveSafePhotoUrl(firstPhotoName);

    if (lat && lng) {
      await pool.query(
        `INSERT INTO places_cache
          (place_id_google, name, rating, address, types, opening_hours, latitude, longitude, photo_url, cached_at, expires_at)
         VALUES ($1, $2, $3, $4, $5::jsonb, $6::jsonb, $7, $8, $9, NOW(), NOW() + INTERVAL '7 days')
         ON CONFLICT (place_id_google) DO UPDATE SET
          rating = EXCLUDED.rating,
          opening_hours = EXCLUDED.opening_hours,
          photo_url = COALESCE(EXCLUDED.photo_url, places_cache.photo_url),
          cached_at = NOW(),
          expires_at = NOW() + INTERVAL '7 days'`,
        [
          placeId,
          name,
          rating,
          address,
          JSON.stringify(types),
          JSON.stringify(openingHours),
          lat,
          lng,
          photoUrl
        ]
      );

      results.push({ placeId, name, rating, address, openingHours, photoUrl, lat, lng });
    }
  }

  return results;
}

export async function batchSearchPlaces(queries = []) {
  const allResults = await Promise.all(queries.map((q) => searchSingleQuery(q)));
  return allResults.flat();
}
```
