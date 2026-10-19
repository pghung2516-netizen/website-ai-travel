### 12.5. Seed dữ liệu cho buổi thuyết trình

#### 12.5.1. File `server/src/db/seedDemoCache.js` (Pre-warmed Cache)

```javascript
import pool from './connection.js';

const DEMO_PLACES = [
  {
    place_id_google: 'ChIJ7aVxn8ItdTERF5Y65239Q2c',
    name: 'War Remnants Museum',
    rating: 4.6,
    address: '28 Võ Văn Tần, Phường 6, Quận 3, TP.HCM',
    types: ['museum', 'tourist_attraction'],
    opening_hours: ['Daily: 07:30 AM – 05:30 PM'],
    latitude: 10.77951,
    longitude: 106.69209,
    photo_url: 'https://images.unsplash.com/photo-1583417319070-4a69db38a482?auto=format&fit=crop&w=800&q=80'
  },
  {
    place_id_google: 'ChIJL5_p1sItdTERq1X6J8_p1sI',
    name: 'Phở Hòa Pasteur',
    rating: 4.3,
    address: '260C Pasteur, Phường Võ Thị Sáu, Quận 3, TP.HCM',
    types: ['restaurant', 'food'],
    opening_hours: ['Daily: 06:00 AM – 10:30 PM'],
    latitude: 10.78634,
    longitude: 106.68812,
    photo_url: 'https://images.unsplash.com/photo-1582878826629-29b7ad1cdc43?auto=format&fit=crop&w=800&q=80'
  },
  {
    place_id_google: 'ChIJW8_p1sItdTER98X6J8_p1sB',
    name: 'The Workshop Coffee',
    rating: 4.5,
    address: '27 Ngô Đức Kế, Bến Nghé, Quận 1, TP.HCM',
    types: ['cafe', 'food'],
    opening_hours: ['Daily: 08:00 AM – 09:00 PM'],
    latitude: 10.77452,
    longitude: 106.70391,
    photo_url: 'https://images.unsplash.com/photo-1501339847302-ac426a4a7cbb?auto=format&fit=crop&w=800&q=80'
  },
  {
    place_id_google: 'ChIJB_b1sItdTER12X6J8_p1sC',
    name: 'Bến Thành Street Food Market',
    rating: 4.2,
    address: '26-28-30 Thủ Khoa Huân, Bến Thành, Quận 1, TP.HCM',
    types: ['restaurant', 'food'],
    opening_hours: ['Daily: 09:00 AM – 11:30 PM'],
    latitude: 10.77341,
    longitude: 106.69745,
    photo_url: 'https://images.unsplash.com/photo-1555939594-58d7cb561ad1?auto=format&fit=crop&w=800&q=80'
  },
  {
    place_id_google: 'ChIJH_c1sItdTER44X6J8_p1sD',
    name: 'Heart of Darkness Craft Brewery',
    rating: 4.6,
    address: '31D Lý Tự Trọng, Bến Nghé, Quận 1, TP.HCM',
    types: ['bar', 'restaurant'],
    opening_hours: ['Daily: 11:00 AM – 12:00 AM'],
    latitude: 10.77815,
    longitude: 106.70162,
    photo_url: 'https://images.unsplash.com/photo-1514933651103-005eec06c04b?auto=format&fit=crop&w=800&q=80'
  },
  {
    place_id_google: 'ChIJK_d1sItdTER77X6J8_p1sE',
    name: 'Independence Palace (Reunification Palace)',
    rating: 4.5,
    address: '135 Nam Kỳ Khởi Nghĩa, Bến Thành, Quận 1, TP.HCM',
    types: ['tourist_attraction', 'museum'],
    opening_hours: ['Daily: 08:00 AM – 04:30 PM'],
    latitude: 10.77699,
    longitude: 106.69536,
    photo_url: 'https://images.unsplash.com/photo-1559592413-7cec4d0cae2b?auto=format&fit=crop&w=800&q=80'
  }
];

async function seed() {
  for (const p of DEMO_PLACES) {
    await pool.query(
      `INSERT INTO places_cache
        (place_id_google, name, rating, address, types, opening_hours, latitude, longitude, photo_url, cached_at, expires_at)
       VALUES ($1, $2, $3, $4, $5::jsonb, $6::jsonb, $7, $8, $9, NOW(), NOW() + INTERVAL '14 days')
       ON CONFLICT (place_id_google) DO UPDATE SET
        expires_at = NOW() + INTERVAL '14 days'`,
      [
        p.place_id_google,
        p.name,
        p.rating,
        p.address,
        JSON.stringify(p.types),
        JSON.stringify(p.opening_hours),
        p.latitude,
        p.longitude,
        p.photo_url
      ]
    );
  }
  console.log('✅ Demo cache pre-warmed successfully!');
  process.exit(0);
}

seed();
```

#### 12.5.2. File `server/src/db/seedDemoUser.js` (Tài khoản demo cho Type 2) — mới

Chạy **sau** `seedDemoCache.js`. Tạo tài khoản `demo_traveler` với 3 chuyến `completed` và một Travel Profile, để trên sân khấu chỉ cần đăng nhập là thấy ngay lời chào "Welcome back" của Type 2, không phải tạo 3 chuyến thật trước mặt giảng viên.

```javascript
import bcrypt from 'bcryptjs';
import pool from './connection.js';
import { calculateTransit } from '../utils/transitCalculator.js';

const DEMO_USERNAME = 'demo_traveler';

// 3 chuyến hoàn tất => vượt ngưỡng TIER2_MIN_COMPLETED_TRIPS (3) => Type 2
const DEMO_TRIPS = [
  { title: '2-day HCMC · street food, culture', days: 2, budget: 3000000, prefs: ['street_food', 'culture'] },
  { title: '2-day HCMC · street food, cafe', days: 2, budget: 3000000, prefs: ['street_food', 'cafe'] },
  { title: '3-day HCMC · street food, nightlife', days: 3, budget: 4500000, prefs: ['street_food', 'nightlife'] }
];

// Phải dùng giá trị trong TỪ VỰNG CỐ ĐỊNH (xem sanitizeProfile ở mục 12.8)
const DEMO_PROFILE = {
  food_styles: ['street food', 'specialty coffee'],
  favorite_districts: ['District 1', 'District 3'],
  dislikes: ['crowded places'],
  pace: 'relaxed',
  crowd_tolerance: 'low',
  typical_budget_per_day: 1500000
};

async function seedDemoUser() {
  const password = process.env.DEMO_USER_PASSWORD;
  if (!password) throw new Error('Set DEMO_USER_PASSWORD before seeding the demo user');

  const client = await pool.connect();
  try {
    await client.query('BEGIN');

    // Chạy lại nhiều lần được (idempotent): xóa tài khoản cũ nếu có (cascade xóa luôn trips)
    const old = await client.query(`SELECT id FROM users WHERE LOWER(username) = $1`, [DEMO_USERNAME]);
    for (const row of old.rows) {
      await client.query(`DELETE FROM travel_profiles WHERE owner_key = $1`, [`u:${row.id}`]);
      await client.query(`DELETE FROM users WHERE id = $1`, [row.id]);
    }

    const hash = await bcrypt.hash(password, 12);
    const u = await client.query(
      `INSERT INTO users (username, password_hash, display_name)
       VALUES ($1, $2, 'Demo Traveler') RETURNING id`,
      [DEMO_USERNAME, hash]
    );
    const userId = u.rows[0].id;

    // Lấy vài địa điểm đã nạp bởi seedDemoCache.js để My Trips có "place_count" thật
    const places = (await client.query(`SELECT * FROM places_cache ORDER BY name LIMIT 3`)).rows;
    const slots = ['09:00 - 11:00', '12:00 - 13:30', '15:00 - 17:00'];

    for (const t of DEMO_TRIPS) {
      const trip = await client.query(
        `INSERT INTO trips (session_id, user_id, title, start_location, budget, days, preferences, source, status)
         VALUES (gen_random_uuid(), $1, $2, 'District 1, Ho Chi Minh City', $3, $4, $5::jsonb, 'form', 'completed')
         RETURNING id`,
        [userId, t.title, t.budget, t.days, JSON.stringify(t.prefs)]
      );

      for (let i = 0; i < places.length; i++) {
        const p = places[i];
        const prev = i > 0 ? places[i - 1] : null;
        const transit = prev ? calculateTransit(prev.latitude, prev.longitude, p.latitude, p.longitude) : null;
        await client.query(
          `INSERT INTO itinerary_items
            (trip_id, day_number, order_index, time_slot, place_name, description, category,
             place_id_google, latitude, longitude, estimated_cost, transit_info)
           VALUES ($1, 1, $2, $3, $4, 'Seeded demo stop', $5, $6, $7, $8, 60000, $9::jsonb)`,
          [
            trip.rows[0].id, i + 1, slots[i], p.name,
            p.types?.includes('cafe') ? 'cafe' : p.types?.includes('museum') ? 'sightseeing' : 'food',
            p.place_id_google, p.latitude, p.longitude,
            transit ? JSON.stringify(transit) : null
          ]
        );
      }
    }

    await client.query(
      `INSERT INTO travel_profiles (owner_key, profile) VALUES ($1, $2::jsonb)`,
      [`u:${userId}`, JSON.stringify(DEMO_PROFILE)]
    );

    await client.query('COMMIT');
    console.log(`✅ Demo user "${DEMO_USERNAME}" created with ${DEMO_TRIPS.length} completed trips (Type 2).`);
  } catch (err) {
    await client.query('ROLLBACK');
    throw err;
  } finally {
    client.release();
  }
  process.exit(0);
}

seedDemoUser().catch((e) => {
  console.error(e);
  process.exit(1);
});
```
