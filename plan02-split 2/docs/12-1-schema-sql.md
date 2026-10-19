## 12. Phụ lục Code Implementation Cốt Lõi (Production-Ready Boilerplate)

### 12.1. File `server/src/db/schema.sql` (PostgreSQL 16, bản v2)

```sql
CREATE EXTENSION IF NOT EXISTS "pgcrypto";

-- 0. [v2] Bảng người dùng. Tạo TRƯỚC trips vì trips tham chiếu tới users.
CREATE TABLE IF NOT EXISTS users (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    username VARCHAR(30),                         -- NULL nếu chỉ đăng nhập bằng Google
    password_hash TEXT,                           -- bcrypt hash, NULL nếu chỉ đăng nhập bằng Google
    email VARCHAR(254),                           -- tùy chọn; cần cho quên mật khẩu
    google_sub VARCHAR(64) UNIQUE,                -- ID cố định của tài khoản Google
    display_name VARCHAR(60),
    settings JSONB NOT NULL DEFAULT '{}'::jsonb,  -- persona_id, tone, ai_interaction, theme...
    created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    last_login_at TIMESTAMPTZ,
    -- Phải có ít nhất một cách đăng nhập: Google, hoặc cả username lẫn mật khẩu
    CONSTRAINT users_login_method CHECK (
        google_sub IS NOT NULL OR (username IS NOT NULL AND password_hash IS NOT NULL)
    ),
    CONSTRAINT users_username_format CHECK (username IS NULL OR username ~ '^[A-Za-z0-9_]{3,30}$')
);

-- Unique KHÔNG phân biệt hoa thường: "Linh" và "linh" không thể cùng tồn tại
CREATE UNIQUE INDEX IF NOT EXISTS idx_users_username_lower ON users (LOWER(username)) WHERE username IS NOT NULL;
CREATE UNIQUE INDEX IF NOT EXISTS idx_users_email_lower ON users (LOWER(email)) WHERE email IS NOT NULL;

-- 0b. [v2] Token đặt lại mật khẩu: chỉ lưu HASH, dùng một lần, hết hạn nhanh
CREATE TABLE IF NOT EXISTS password_reset_tokens (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    user_id UUID NOT NULL REFERENCES users(id) ON DELETE CASCADE,
    token_hash CHAR(64) NOT NULL UNIQUE,          -- SHA-256 dạng hex
    expires_at TIMESTAMPTZ NOT NULL,
    used_at TIMESTAMPTZ,
    created_at TIMESTAMPTZ NOT NULL DEFAULT NOW()
);

-- 1. Bảng lưu thông tin chuyến đi (khách: phân tách theo session_id; thành viên: theo user_id)
CREATE TABLE IF NOT EXISTS trips (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    session_id UUID NOT NULL,
    user_id UUID REFERENCES users(id) ON DELETE CASCADE,   -- [v2] NULL = chuyến của khách
    title VARCHAR(120),                                    -- [v2] tên hiển thị trong My Trips
    start_location TEXT NOT NULL,
    budget INTEGER NOT NULL CHECK (budget > 0),
    days SMALLINT NOT NULL CHECK (days BETWEEN 1 AND 14),
    preferences JSONB NOT NULL DEFAULT '[]'::jsonb,
    status VARCHAR(20) NOT NULL DEFAULT 'draft' CHECK (status IN ('draft', 'asking', 'generating', 'completed')),
    source VARCHAR(20) NOT NULL DEFAULT 'form' CHECK (source IN ('form', 'sentence', 'assistant', 'clone')),
    share_token VARCHAR(32) UNIQUE,                        -- [v2] NULL = chưa chia sẻ
    shared_at TIMESTAMPTZ,
    feedback JSONB,                                        -- [v2] đánh giá sau chuyến (P4)
    created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    updated_at TIMESTAMPTZ NOT NULL DEFAULT NOW()
);

CREATE INDEX IF NOT EXISTS idx_trips_session_id ON trips(session_id);
CREATE INDEX IF NOT EXISTS idx_trips_user_id ON trips(user_id, created_at DESC);

-- 2. Bảng Cache dữ liệu từ Google Places API (TTL 7 ngày)
CREATE TABLE IF NOT EXISTS places_cache (
    place_id_google VARCHAR(255) PRIMARY KEY,
    name TEXT NOT NULL,
    rating NUMERIC(2,1),
    address TEXT,
    types JSONB DEFAULT '[]'::jsonb,
    opening_hours JSONB DEFAULT '[]'::jsonb,
    latitude DOUBLE PRECISION NOT NULL,
    longitude DOUBLE PRECISION NOT NULL,
    photo_url TEXT,
    cached_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    expires_at TIMESTAMPTZ NOT NULL DEFAULT (NOW() + INTERVAL '7 days')
);

CREATE INDEX IF NOT EXISTS idx_places_cache_name ON places_cache USING GIN (to_tsvector('simple', name));

-- 3. Bảng từng mục hoạt động trong lịch trình
CREATE TABLE IF NOT EXISTS itinerary_items (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    trip_id UUID NOT NULL REFERENCES trips(id) ON DELETE CASCADE,
    day_number SMALLINT NOT NULL CHECK (day_number >= 1),
    order_index SMALLINT NOT NULL CHECK (order_index >= 1),
    time_slot VARCHAR(50) NOT NULL,
    place_name TEXT NOT NULL,
    description TEXT,
    category VARCHAR(30) NOT NULL CHECK (category IN ('food', 'sightseeing', 'cafe', 'shopping', 'nature', 'entertainment')),
    place_id_google VARCHAR(255),
    latitude DOUBLE PRECISION NOT NULL,
    longitude DOUBLE PRECISION NOT NULL,
    estimated_cost INTEGER NOT NULL DEFAULT 0 CHECK (estimated_cost >= 0),
    transit_info JSONB DEFAULT NULL,
    UNIQUE (trip_id, day_number, order_index)
);

CREATE INDEX IF NOT EXISTS idx_itinerary_trip_day ON itinerary_items(trip_id, day_number, order_index);
CREATE INDEX IF NOT EXISTS idx_itinerary_place_id ON itinerary_items(place_id_google);

-- 4. Bảng lưu lịch sử hội thoại nhiều lượt với Gemini
CREATE TABLE IF NOT EXISTS conversations (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    trip_id UUID NOT NULL REFERENCES trips(id) ON DELETE CASCADE,
    role VARCHAR(15) NOT NULL CHECK (role IN ('user', 'model')),
    content TEXT NOT NULL,
    metadata JSONB DEFAULT '{}'::jsonb,                    -- [v2] có thể ghi sự kiện swap
    created_at TIMESTAMPTZ NOT NULL DEFAULT NOW()
);

CREATE INDEX IF NOT EXISTS idx_conversations_trip_id ON conversations(trip_id, created_at);

-- 5. [v2] Trí nhớ của AI (Travel Profile). Khóa: 'u:<user_id>' hoặc 's:<session_id>'.
--    Không có FK vì khóa có thể là session của khách -> xóa tài khoản phải xóa bằng code.
CREATE TABLE IF NOT EXISTS travel_profiles (
    owner_key VARCHAR(50) PRIMARY KEY,
    profile JSONB NOT NULL DEFAULT '{}'::jsonb,
    updated_at TIMESTAMPTZ NOT NULL DEFAULT NOW()
);
```

### 12.1.b. Lưu ý driver `pg` cho cột NUMERIC (Hữu Lam cấu hình trong `server/src/db/connection.js`)

Cột `rating NUMERIC(2,1)` trong PostgreSQL có type OID là 1700. Mặc định driver `pg` trả về kiểu chuỗi (ví dụ `"4.3"`) để bảo toàn độ chính xác số học. Để contract test và frontend nhận kiểu `number` (`4.3`), Lam cần cấu hình type parser ngay khi khởi tạo connection pool:

```javascript
import pg from 'pg';

// Ép kiểu OID 1700 (NUMERIC) từ chuỗi về số thực (float)
pg.types.setTypeParser(1700, (v) => (v === null ? null : parseFloat(v)));
```
