## 9. Deploy

| Thành phần | Nền tảng | Chi tiết |
| --- | --- | --- |
| Frontend | **Vercel** (free) | Auto deploy từ GitHub `main` branch. Custom domain nếu có. **v2:** thêm file `vercel.json` rewrite mọi đường dẫn về `index.html` để `/trips`, `/s/:token` không bị 404 khi refresh |
| Backend | **Render** (free tier) | Web Service kết nối Supabase PostgreSQL. **Lưu ý**: sleep sau 15 phút không hoạt động |
| Database (dev) | **PostgreSQL (Docker)** | Chạy container `postgres:16-alpine` local qua `docker-compose.yml`, dữ liệu mount volume |
| Database (prod) | **Supabase** (free tier) | PostgreSQL, không giới hạn thời gian (cùng engine với bản Docker dev) |
| Đăng nhập Google *(P2)* | **Google Cloud Console** | Tạo OAuth Client ID loại Web; thêm Vercel URL và `http://localhost:5173` vào **Authorized JavaScript origins** |
| Email *(P2)* | **Gmail SMTP** | Dùng App Password (cần bật xác minh 2 bước). Chỉ dùng cho email đặt lại mật khẩu |

### 9.1. Environment Variables cần set

**Render (Backend)**:
```env
PORT=5000
NODE_ENV=production
GEMINI_API_KEY=your_gemini_api_key_here
GOOGLE_PLACES_API_KEY=your_google_places_api_key_here
DATABASE_URL=postgresql://postgres.[project-ref]:[password]@aws-0-ap-southeast-1.pooler.supabase.com:6543/postgres
FRONTEND_URL=https://your-app.vercel.app
DEMO_FALLBACK_MODE=false

# --- v2: tài khoản & thích ứng ---
JWT_SECRET=run_openssl_rand_-hex_32_and_paste_here
JWT_EXPIRES_IN=7d
GOOGLE_CLIENT_ID=your_oauth_client_id.apps.googleusercontent.com
SMTP_HOST=smtp.gmail.com
SMTP_PORT=465
SMTP_USER=your_gmail_address
SMTP_PASS=your_gmail_app_password
APP_BASE_URL=https://your-app.vercel.app
GEMINI_GLOBAL_RPM=12
TIER2_MIN_COMPLETED_TRIPS=3
DEMO_USER_PASSWORD=choose_a_strong_demo_password
```

**Vercel (Frontend)**:
```env
VITE_API_URL=https://your-api.onrender.com
VITE_GOOGLE_MAPS_API_KEY=your_restricted_maps_js_key_here
VITE_GOOGLE_CLIENT_ID=your_oauth_client_id.apps.googleusercontent.com
```

> `GOOGLE_CLIENT_ID` là giá trị **công khai** (được phép nằm ở frontend). Ngược lại `JWT_SECRET`, `SMTP_PASS`, `DEMO_USER_PASSWORD` tuyệt đối không commit và không đưa lên Vercel.

### 9.2. Quy trình deploy

```
1. Push code lên GitHub (main branch)
2. Vercel tự động build + deploy frontend
3. Render tự động build + deploy backend
4. Set environment variables trên cả 2 platform
5. Chạy schema.sql, seedDemoCache.js và seedDemoUser.js trên Supabase Postgres
6. (P2) Thêm Vercel URL vào Authorized JavaScript origins của OAuth Client
7. Test link public end-to-end (guest flow, rồi đăng ký và kiểm tra claim)
```

### 9.3. Cấu hình Docker & Docker Compose (`docker-compose.yml`)

File `docker-compose.yml` phục vụ cả 2 mục đích: chạy PostgreSQL 16 khi code hàng ngày (`docker compose up -d postgres`) và chạy nguyên cụm Fullstack Offline khi thuyết trình nếu hội trường mất mạng (`docker compose --profile demo up --build`):

```yaml
version: '3.9'

services:
  postgres:
    image: postgres:16-alpine
    container_name: hcmc_travel_postgres
    restart: unless-stopped
    environment:
      POSTGRES_USER: postgres
      POSTGRES_PASSWORD: postgrespassword
      POSTGRES_DB: travel_ai_hcmc
    ports:
      - "5432:5432"
    volumes:
      - pgdata:/var/lib/postgresql/data
      - ./server/src/db/schema.sql:/docker-entrypoint-initdb.d/01-schema.sql:ro
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U postgres -d travel_ai_hcmc"]
      interval: 5s
      timeout: 5s
      retries: 5

  server:
    build:
      context: ./server
      dockerfile: Dockerfile
    container_name: hcmc_travel_server
    profiles: ["demo"]
    restart: unless-stopped
    depends_on:
      postgres:
        condition: service_healthy
    environment:
      PORT: 5000
      NODE_ENV: development
      DATABASE_URL: postgresql://postgres:postgrespassword@postgres:5432/travel_ai_hcmc
      GEMINI_API_KEY: ${GEMINI_API_KEY}
      GOOGLE_PLACES_API_KEY: ${GOOGLE_PLACES_API_KEY}
      FRONTEND_URL: http://localhost:5173
      # --- v2 ---
      JWT_SECRET: ${JWT_SECRET}
      JWT_EXPIRES_IN: 7d
      GOOGLE_CLIENT_ID: ${GOOGLE_CLIENT_ID:-}
      APP_BASE_URL: http://localhost:5173
      GEMINI_GLOBAL_RPM: 12
      TIER2_MIN_COMPLETED_TRIPS: 3
      DEMO_USER_PASSWORD: ${DEMO_USER_PASSWORD}
    ports:
      - "5000:5000"

  client:
    build:
      context: ./client
      dockerfile: Dockerfile
    container_name: hcmc_travel_client
    profiles: ["demo"]
    depends_on:
      - server
    environment:
      VITE_API_URL: http://localhost:5000
      VITE_GOOGLE_CLIENT_ID: ${GOOGLE_CLIENT_ID:-}
    ports:
      - "5173:5173"

volumes:
  pgdata:
```

> **Về demo offline:** đăng nhập bằng **username + password** và tài khoản demo từ `seedDemoUser.js` chạy được hoàn toàn trên Docker local, không phụ thuộc dịch vụ ngoài. **Google Sign-In** và **quên mật khẩu** cần Internet nên không dùng trong kịch bản offline.

### 9.4. Chi phí & Quota Safeguards (Quan trọng)

| Dịch vụ | Quota miễn phí | Rủi ro vượt mức | Giải pháp kiểm soát |
| --- | --- | --- | --- |
| **Google Places API** | $200 tín dụng/tháng (~5,000 requests Place Details) | Bị trừ tiền thẻ tín dụng nếu loop API | 1. Bật **Budget Alert $5** trên Google Cloud Console.<br>2. Cài đặt Quota Cap tối đa 150 requests/ngày.<br>3. Dùng `X-Goog-FieldMask` lấy trọn thông tin ngay trong `places:searchText`.<br>4. Tận dụng bảng `places_cache` với TTL 7 ngày (giảm >75% request lặp). |
| **Gemini Flash API** | ~15 requests/phút (Free Tier) | HTTP 429 Too Many Requests khi demo nhanh; **v2:** thêm Smart Start và tóm tắt profile cùng dùng chung quota | 1. Gộp mảng `queries[]` trong `searchPlaces` để chỉ tốn tối đa 2 Gemini calls/lượt chat.<br>2. Backend rate limit đồng nhất: **10 req/phút/IP** cho route `/chat` và Smart Start.<br>3. Lưu `conversations` vào database, không gửi lại toàn bộ prompt dài không cần thiết (chỉ lấy 10 lượt gần nhất).<br>4. Fallback retry với exponential backoff.<br>5. **v2:** `geminiBudget` giữ trần 12/phút toàn server, ưu tiên chat > Smart Start > tóm tắt profile; lời chào Type 2 dùng mẫu (0 lượt). |
| **Google Sign-In (OAuth)** *(P2)* | Miễn phí | Không đáng kể | Chỉ xác minh ID token ở backend, không gọi thêm API trả phí |
| **Gmail SMTP** *(P2)* | Miễn phí, giới hạn ~500 email/ngày | Bị khóa gửi nếu spam; cần App Password | Chỉ gửi email đặt lại mật khẩu; rate limit 3 lần/giờ; lỗi gửi không chặn luồng |

---
