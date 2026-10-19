## 3. Kiến trúc & Tech Stack

### 3.1. Tech stack

| Lớp | Công nghệ | Ghi chú |
| --- | --- | --- |
| Frontend | React 18 + Vite | SPA, gọi backend qua REST API |
| Routing *(mới)* | React Router v6 | Cần nhiều trang: `/`, `/trip/:id`, `/trips`, `/s/:token`, `/settings`, `/profile`, `/reset-password` |
| State management | React Context (`TripContext`, `AuthContext`, `SettingsContext`) | Đủ dùng cho project này, không cần Redux/Zustand |
| Styling | Tailwind CSS | Responsive sẵn, tiện cho mobile-first |
| Backend | Node.js (v20 LTS) + Express | REST API server |
| Database | **PostgreSQL 16** (Docker local → Supabase cloud) | Đồng nhất 1 database engine cả dev lẫn prod, tận dụng `JSONB` |
| Containerization | **Docker & Docker Compose** | Chạy PostgreSQL 16 local nhanh chóng + dự phòng chạy fullstack offline lúc demo |
| AI | Google Gemini Flash API (`@google/genai` SDK, `gemini-2.5-flash`) | Free tier ~15 req/phút; tối ưu Batch Function Calling; dùng thêm **JSON mode** cho Smart Start và tóm tắt profile |
| Dữ liệu địa điểm | Google Places API (New - Text Search) | Ảnh, rating, giờ mở cửa, tọa độ lấy trong 1 lần gọi qua `X-Goog-FieldMask` |
| Xác thực *(mới)* | JWT (`jsonwebtoken`) + `bcryptjs` | Token gửi qua header `Authorization: Bearer`. `bcryptjs` là bản JS thuần, không cần build native trên Render |
| Google Sign-In *(mới)* | `@react-oauth/google` (FE) + `google-auth-library` (BE) | FE lấy ID token, **BE xác minh** rồi cấp JWT của riêng mình |
| Email *(mới, P2)* | `nodemailer` + Gmail SMTP (App Password) | Chỉ để gửi link đặt lại mật khẩu. Miễn phí, gửi được cho mọi người nhận |
| Validation *(mới)* | `zod` | Dùng chung một schema cho `POST /api/trips`, auth và Smart Start |
| Deploy frontend | Vercel | Auto deploy từ GitHub `main` branch |
| Deploy backend | Render (free tier) | Web Service kết nối Supabase Postgres |

> **Vì sao JWT trong header mà không dùng cookie?** Frontend (Vercel) và backend (Render) khác domain, cookie phải bật `SameSite=None; Secure` và dễ bị trình duyệt chặn cookie bên thứ ba (đặc biệt Safari). Header `Authorization` đơn giản hơn nhiều cho đồ án và **không cần chống CSRF**. Đánh đổi: token để trong `localStorage` nên phải chống XSS cẩn thận (không dùng `dangerouslySetInnerHTML`, CSP cơ bản). Đồ án này chấp nhận đánh đổi đó; sản phẩm thật nên chuyển sang cookie `HttpOnly` cùng domain.

### 3.2. Sơ đồ kiến trúc tổng thể

```
┌──────────────────┐     REST API (JSON)    ┌────────────────────────────────────────┐
│                  │ ─────────────────────▶ │                                        │
│   React Frontend │  Headers:              │   Express Backend (Render)             │
│   (Vercel)       │  • x-session-id        │                                        │
│                  │    (luôn có, UUIDv4)   │   identity middleware                  │
│                  │  • Authorization:      │     → req.principal {sessionId,userId}│
│                  │    Bearer <JWT>        │   ┌────────────────────────────┐       │
│                  │    (nếu đã đăng nhập)  │   │ tripService   authService  │       │
│                  │ ◀───────────────────── │   │ aiService     profileSvc   │──────▶ Google Gemini API
└──────────────────┘     JSON Response      │   │ placeService  shareService │──────▶ Google Places API (New)
                                            │   │ tierService   emailService │──────▶ SMTP (quên mật khẩu)
                                            │   │ transitCalculator (Toán)   │──────▶ Google Identity (xác minh ID token)
                                            │   └────────────────────────────┘       │
                                            │                  │                     │
                                            │                  ▼                     │
                                            │   ┌────────────────────────────┐       │
                                            │   │ PostgreSQL 16 (Docker/     │       │
                                            │   │ Supabase Cloud)            │       │
                                            │   └────────────────────────────┘       │
                                            └────────────────────────────────────────┘
```

Frontend gọi backend qua REST API; backend tách riêng các service, kết nối database và các API ngoài (AI, dữ liệu địa điểm, Google Identity, SMTP). Frontend **không bao giờ** gọi trực tiếp API ngoài (ngoại lệ duy nhất: nút Google Sign-In của Google để lấy ID token, sau đó ID token vẫn được **backend** xác minh).

### 3.3. Cấu trúc thư mục chi tiết

```
travel-ai-hcmc/
├── docker-compose.yml              # Quản lý PostgreSQL local (+ fullstack demo backup)
├── client/                        # Giao diện (React + Vite)
│   ├── Dockerfile                 # Container hóa frontend (cho offline demo)
│   ├── index.html
│   ├── tailwind.config.js
│   └── src/
│       ├── components/
│       │   ├── Header/            # Logo, Dark Mode toggle, My Trips, Sign in / avatar menu
│       │   ├── Auth/                                   # (P2) Tầng FE 1
│       │   │   ├── AuthModal.jsx                       # Tab Sign in / Sign up, nút "Continue as guest"
│       │   │   ├── LoginForm.jsx
│       │   │   ├── RegisterForm.jsx
│       │   │   ├── GoogleButton.jsx
│       │   │   ├── ForgotPasswordForm.jsx
│       │   │   └── ResetPasswordPage.jsx
│       │   ├── Assistant/                              # (P3) Tầng FE 2
│       │   │   ├── SmartStart.jsx                      # Type 1: ô nhập tự do + nút Skip
│       │   │   ├── WelcomeBack.jsx                     # Type 2: lời chào theo mẫu + chips + "Not now"
│       │   │   └── SaveTripPrompt.jsx                  # Thẻ nhắc đăng ký sau khi có lịch trình (không chặn)
│       │   ├── TripForm/          # Form nhập điểm xuất phát, ngân sách, số ngày, sở thích
│       │   │   ├── StandardForm.jsx
│       │   │   └── NaturalSentenceMode.jsx  # Giao diện điền khuyết Mad-Libs
│       │   ├── ChatPanel/         # Khung chat trao đổi với AI
│       │   │   ├── MessageList.jsx
│       │   │   ├── QuickReplyChips.jsx
│       │   │   ├── ThinkingSteps.jsx        # 4-stage Agent Thinking progress
│       │   │   └── ChatInput.jsx
│       │   ├── ItineraryView/     # Hiển thị lịch trình dạng timeline + bản đồ
│       │   │   ├── TripSummaryCard.jsx      # Boarding pass summary
│       │   │   ├── BudgetBar.jsx            # Thanh theo dõi ngân sách xanh/vàng/đỏ
│       │   │   ├── DayTabs.jsx              # Chuyển ngày Day 1, Day 2...
│       │   │   ├── TimelineNode.jsx         # Card địa điểm + nút Expand/Swap
│       │   │   ├── TransitBadge.jsx         # Hiển thị khoảng cách đi bộ / giá Grab
│       │   │   └── MapPanel.jsx             # Google Maps Embed đồng bộ 2 chiều
│       │   ├── Trips/                                  # (P1)
│       │   │   ├── MyTripsPage.jsx                     # Danh sách chuyến đi
│       │   │   ├── TripCard.jsx                        # Đổi tên / xóa / nhân bản / chia sẻ
│       │   │   ├── ShareDialog.jsx                     # Tạo / sao chép / thu hồi link
│       │   │   └── SharedTripPage.jsx                  # Trang /s/:token chỉ xem + "Copy to my trips"
│       │   └── Settings/                               # (P2 cơ bản, P3 đầy đủ)
│       │       ├── SettingsPage.jsx
│       │       ├── PersonaPicker.jsx                   # Avatar + tên + giọng điệu (preset)
│       │       ├── AiInteractionToggle.jsx             # Guided / Minimal
│       │       └── TravelProfileEditor.jsx             # Xem / sửa / xóa trí nhớ của AI
│       ├── context/               # React Context cho state management
│       │   ├── TripContext.jsx    # State chuyến đi hiện tại + lịch trình + chat
│       │   ├── AuthContext.jsx    # user, token, login/logout, claim kết quả
│       │   └── SettingsContext.jsx# persona, ai_interaction, theme (khách: localStorage; member: đồng bộ server)
│       ├── hooks/
│       │   └── useUserTier.js     # Gọi /api/me/tier → { tier, identity }
│       ├── services/              # Gọi API sang backend (axios/fetch wrapper)
│       │   └── api.js             # Gắn x-session-id (luôn) + Authorization (nếu có token)
│       ├── utils/
│       │   └── session.js         # Khởi tạo & đọc UUIDv4 session_id; xoay session_id khi logout
│       └── App.jsx                # Khai báo routes
├── server/                        # Xử lý logic (Node.js + Express)
│   ├── Dockerfile                 # Container hóa backend (cho offline demo)
│   ├── package.json
│   └── src/
│       ├── routes/
│       │   ├── tripRoutes.js      # /api/trips/* (gồm list, duplicate, share, feedback)
│       │   ├── chatRoutes.js      # /api/trips/:id/chat
│       │   ├── placeRoutes.js     # /api/places/* (bao gồm photo proxy)
│       │   ├── authRoutes.js      # /api/auth/*                      (P2)
│       │   ├── meRoutes.js        # /api/me/* (tier, settings, profile)  (P2/P3)
│       │   ├── sharedRoutes.js    # /api/shared/:token (công khai, chỉ đọc)  (P1)
│       │   └── onboardingRoutes.js# /api/onboarding/parse, /api/assistant/welcome (P3)
│       ├── services/
│       │   ├── tripService.js     # Tạo/lưu/sửa lịch trình (PostgreSQL Transaction)
│       │   ├── aiService.js       # Điều phối hội thoại AI + vòng lặp function calling
│       │   ├── placeService.js    # Lấy dữ liệu địa điểm từ Google Places + Cache TTL
│       │   ├── authService.js     # Đăng ký / đăng nhập / Google / claim chuyến đi của khách (P2)
│       │   ├── emailService.js    # Gửi email đặt lại mật khẩu (P2)
│       │   ├── shareService.js    # Tạo / thu hồi share_token, đọc chuyến đi công khai (P1)
│       │   ├── tierService.js     # Tính Type 1 / Type 2 theo hành vi (P3)
│       │   ├── profileService.js  # Đọc / sửa / xóa Travel Profile + tóm tắt bằng Gemini (P3)
│       │   └── promptBuilder.js   # Lắp system prompt từ BASE + PERSONA + TIER + PROFILE (P3)
│       ├── personas.js            # Whitelist avatar / tên / giọng điệu → text cố định phía server (P3)
│       ├── utils/
│       │   └── transitCalculator.js # Công thức Haversine tính khoảng cách & giá Grab
│       ├── tools/                 # Function calling tools mà Gemini được phép gọi
│       │   ├── definitions.js     # Khai báo JSON Schema cho Gemini Tools
│       │   ├── searchPlaces.js    # Tìm địa điểm theo mảng queries[] + loại
│       │   ├── askPreference.js   # Gửi câu hỏi về sở thích cho user qua chat
│       │   └── saveItinerary.js   # Lưu lịch trình đã chuẩn hóa vào DB
│       ├── middleware/
│       │   ├── rateLimiter.js     # express-rate-limit config (chat / chung / auth / share)
│       │   ├── identity.js        # Thay cho sessionGuard: dựng req.principal + kiểm tra quyền sở hữu
│       │   ├── geminiBudget.js    # Giữ trần ~12 lượt gọi Gemini/phút toàn server (ưu tiên chat)
│       │   └── validator.js       # Input validation middleware (zod)
│       ├── db/                    # Kết nối PostgreSQL + schema migration
│       │   ├── connection.js      # pg.Pool connection
│       │   ├── schema.sql         # Khởi tạo bảng PostgreSQL 16 (bản v2, đã gồm users...)
│       │   ├── seedDemoCache.js   # Nạp sẵn dữ liệu cache cho kịch bản thuyết trình
│       │   └── seedDemoUser.js    # Tạo tài khoản demo + 3 chuyến hoàn tất + profile (demo Type 2)
│       └── app.js                 # Điểm khởi chạy server
├── .env.example
├── .gitignore
└── README.md
```

### 3.4. API Endpoints & Đặc tả Payload (API Contracts)

#### 3.4.1. Mô hình danh tính (Identity Model)

Mọi request gửi **luôn** header `x-session-id` (UUIDv4 do trình duyệt tạo), và gửi thêm `Authorization: Bearer <JWT>` nếu đã đăng nhập. Middleware `identity.js` biến hai header đó thành một `req.principal`:

| Trường hợp | `req.principal` | Thấy được chuyến đi nào |
| --- | --- | --- |
| Khách | `{ sessionId, userId: null }` | Chuyến có `user_id IS NULL` **và** `session_id` khớp |
| Thành viên | `{ sessionId, userId }` | Chuyến có `user_id` khớp |
| Token hết hạn / sai | Coi như khách, trả header `X-Auth-Status: expired` | Như khách. Frontend hiện "Please sign in again" |

**Claim:** khi đăng ký hoặc đăng nhập thành công, backend chạy trong **cùng một transaction**: `UPDATE trips SET user_id = $user WHERE session_id = $session AND user_id IS NULL`. Nhờ vậy khách dùng thử, thấy ổn, rồi mới đăng ký thì **không mất chuyến đi nào**. Khi đăng xuất, frontend **xoay `session_id` mới** để máy dùng chung không lộ dữ liệu của người trước.

> 💡 **Session Isolation (v1) vẫn còn hiệu lực** cho khách: `session_id` giúp các người dùng demo độc lập với nhau mà không cần đăng ký. Tài khoản chỉ thêm một lớp danh tính bền vững bên trên.

#### 3.4.2. Trips, Chat, Places

| Method | Endpoint | Mô tả | Request Body / Params | Response |
| --- | --- | --- | --- | --- |
| `POST` | `/api/trips` | Tạo chuyến đi mới | `{ start_location, budget, days, preferences, source? }` (danh tính lấy từ header) | `{ id, ...tripData }` |
| `GET` | `/api/trips` *(P1)* | **My Trips**: danh sách chuyến của người dùng hiện tại | `?limit=20` | `{ trips: [{ id, title, start_location, days, budget, status, created_at, place_count, total_estimated, is_shared }] }` |
| `GET` | `/api/trips/:id` | Lấy chi tiết chuyến đi + lịch trình | Header danh tính | `{ trip, itinerary_items[], budget_summary }` |
| `PUT` | `/api/trips/:id` | Cập nhật thông tin chuyến đi | `{ title?, budget?, days?, preferences? }` | `{ updated trip }` |
| `DELETE` | `/api/trips/:id` | Xóa chuyến đi | Header danh tính | `{ success: true }` |
| `POST` | `/api/trips/:id/duplicate` *(P1)* | Nhân bản chuyến đi (kèm lịch trình) | — | `{ id }` |
| `POST` | `/api/trips/:id/chat` | Gửi tin nhắn → nhận phản hồi AI | `{ message, client_settings? }` | `{ reply, quick_replies?, updated_itinerary?, budget_summary?, status }` |
| `GET` | `/api/trips/:id/chat` | Lấy lịch sử chat của chuyến đi | `?limit=50` (Header danh tính) | `{ messages[] }` |
| `POST` | `/api/trips/:id/feedback` *(P4)* | Đánh giá chuyến đi sau khi đi | `{ rating (1-5), tags[], comment? (≤300 ký tự) }` | `{ success: true }` |
| `GET` | `/api/places/search` | Tìm kiếm địa điểm (proxy Google Places) | `?query=...&type=...` | `{ places[] }` |
| `GET` | `/api/places/:placeId` | Chi tiết địa điểm (từ cache hoặc API) | — | `{ place }` |
| `GET` | `/api/places/photo/:ref` | Proxy/Redirect lấy ảnh địa điểm ẩn API key | `:ref` (Photo reference) | `302 Redirect` sang `lh3.googleusercontent.com` |

> `client_settings` (tùy chọn) là `{ persona_id, tone, ai_interaction }` do **khách** gửi kèm vì khách không có hồ sơ trên server. Backend chỉ chấp nhận giá trị nằm trong **whitelist**; với thành viên, nguồn đúng là `users.settings` trên server và `client_settings` bị bỏ qua.

#### 3.4.3. Chia sẻ (P1)

| Method | Endpoint | Mô tả | Request Body / Params | Response |
| --- | --- | --- | --- | --- |
| `POST` | `/api/trips/:id/share` | Tạo link chia sẻ (hoặc xoay token mới) | — | `{ share_url, token }` |
| `DELETE` | `/api/trips/:id/share` | Thu hồi link (link cũ trả 404) | — | `{ success: true }` |
| `GET` | `/api/shared/:token` | **Công khai, chỉ đọc**: xem chuyến đi được chia sẻ | `:token` | `{ trip (đã lược bớt), itinerary_items[], budget_summary }` |
| `POST` | `/api/shared/:token/clone` | "Copy to my trips": nhân bản vào tài khoản/khách hiện tại | Header danh tính | `{ id }` |

> Payload công khai **không** chứa `session_id`, `user_id`, lịch sử chat hay email. Token sinh bằng `crypto.randomBytes(16).toString('base64url')` (22 ký tự, không đoán được). Response có header `X-Robots-Tag: noindex`.

#### 3.4.4. Tài khoản (P2)

| Method | Endpoint | Mô tả | Request Body / Params | Response |
| --- | --- | --- | --- | --- |
| `POST` | `/api/auth/register` | Đăng ký username + password | `{ username, password, email? }` + Header `x-session-id` | `201 { token, user, claimed_trips }` |
| `POST` | `/api/auth/login` | Đăng nhập | `{ username, password }` + Header `x-session-id` | `{ token, user, claimed_trips }` |
| `POST` | `/api/auth/google` | Đăng nhập / đăng ký bằng Google | `{ id_token }` + Header `x-session-id` | `{ token, user, claimed_trips, is_new }` |
| `GET` | `/api/auth/me` | Thông tin tài khoản hiện tại | Header `Authorization` | `{ user }` |
| `POST` | `/api/auth/forgot-password` | Gửi email đặt lại mật khẩu | `{ email }` | **Luôn** `200 { ok: true }` (không tiết lộ email có tồn tại hay không) |
| `POST` | `/api/auth/reset-password` | Đặt mật khẩu mới bằng token trong email | `{ token, new_password }` | `{ ok: true }` |
| `PUT` | `/api/auth/password` | Đổi mật khẩu khi đang đăng nhập | `{ current_password, new_password }` | `{ ok: true }` |
| `DELETE` | `/api/auth/account` | Xóa tài khoản (kèm chuyến đi, hồ sơ) | `{ password }` (tài khoản Google: `{ id_token }`) | `{ success: true }` |

> Đăng xuất **không cần endpoint**: JWT là stateless, frontend xóa token và xoay `session_id`. Quy tắc mật khẩu: 8 đến 72 ký tự (bcrypt chỉ đọc 72 byte đầu). Username: 3 đến 30 ký tự gồm chữ, số, dấu gạch dưới, không phân biệt hoa thường.

#### 3.4.5. Thích ứng, Settings, Travel Profile (P2/P3)

| Method | Endpoint | Mô tả | Request Body / Params | Response |
| --- | --- | --- | --- | --- |
| `GET` | `/api/me/tier` | Type 1 hay Type 2 (tính theo hành vi) | — | `{ tier: 1 \| 2, completed_trips, identity: "guest" \| "member" }` |
| `GET` | `/api/assistant/welcome` | Lời chào theo **mẫu** (không gọi Gemini) | — | `{ tier, show, message, chips[], suggested_trip? }` |
| `POST` | `/api/onboarding/parse` | **Smart Start**: tách thông tin chuyến đi từ một câu tự do (1 lần gọi Gemini JSON mode) | `{ text }` (≤ 500 ký tự) | `{ status: "ready" \| "asking" \| "failed", draft, missing[], question?, quick_replies? }` |
| `GET` / `PUT` | `/api/me/settings` | Cài đặt của thành viên | `{ persona_id?, tone?, ai_interaction?, show_greeting?, memory_enabled?, theme? }` | `{ settings }` |
| `GET` | `/api/me/profile` | Xem Travel Profile (cả khách lẫn thành viên) | — | `{ profile, enabled }` |
| `PUT` | `/api/me/profile` | Sửa / xóa từng mục | `{ remove?: [{ field, value? }], set?: { pace?, crowd_tolerance? } }` | `{ profile }` |
| `DELETE` | `/api/me/profile` | Xóa toàn bộ trí nhớ của AI | — | `{ success: true }` |

> 💡 **Chi tiết về trường `status` trong response của chat**:
> - `"draft"`: Mới tạo chuyến đi, chưa sinh lịch trình.
> - `"asking"`: AI đang hỏi thêm câu hỏi sở thích (kèm mảng `quick_replies` để hiển thị nút bấm).
> - `"generating"`: AI đang xử lý gọi tool lấy dữ liệu.
> - `"completed"`: Lịch trình đã được cập nhật thành công (kèm object `updated_itinerary`).

#### Ví dụ Payload thực tế cho `POST /api/trips/:id/chat`

**Request:**
```http
POST /api/trips/9b1deb4d-3b7d-4bad-9bdd-2b0d7b3dcb6d/chat HTTP/1.1
Content-Type: application/json
x-session-id: 1b9d6bcd-bbfd-4b2d-9b5d-ab8dfbbd4bed
Authorization: Bearer eyJhbGciOiJIUzI1NiIs...   (chỉ có khi đã đăng nhập)

{
  "message": "Mix of both ✨"
}
```

**Response (Khi AI hỏi thêm sở thích - `status: "asking"`):**
```json
{
  "status": "asking",
  "reply": "I noticed you love street food! Do you prefer a walking street tour around District 1 or exploring hidden alleys in District 3 & 4?",
  "quick_replies": [
    "District 1 walking tour 🚶",
    "District 3 & 4 food alleys 🍜",
    "Mix of both ✨"
  ],
  "updated_itinerary": null
}
```

**Response (Khi AI hoàn tất sinh/cập nhật lịch trình - `status: "completed"`):**
```json
{
  "status": "completed",
  "reply": "Awesome! 🎉 I've crafted a 2-day HCMC itinerary blending iconic District 1 sights with authentic District 3 & 4 street food alleys. Total estimated cost is ₫1,850,000 (62% of your ₫3,000,000 budget).",
  "quick_replies": [
    "🔄 Swap Day 1 dinner to craft beer",
    "☕ Add a specialty café on Day 2",
    "💸 Make it more budget-friendly"
  ],
  "budget_summary": {
    "total_budget": 3000000,
    "activities_cost": 1735000,
    "transit_cost": 115000,
    "total_estimated": 1850000,
    "percentage": 62,
    "health": "under_budget"
  },
  "updated_itinerary": [
    {
      "day_number": 1,
      "order_index": 1,
      "time_slot": "08:30 - 11:00",
      "place_name": "War Remnants Museum",
      "description": "Explore deeply moving historical exhibits and photography from the Vietnam War.",
      "category": "sightseeing",
      "place_id_google": "ChIJ7aVxn8ItdTERF5Y65239Q2c",
      "latitude": 10.77951,
      "longitude": 106.69209,
      "estimated_cost": 40000,
      "rating": 4.6,
      "address": "28 Võ Văn Tần, Phường 6, Quận 3, TP.HCM",
      "opening_hours": ["Monday: 7:30 AM – 5:30 PM"],
      "photo_url": "https://lh3.googleusercontent.com/places/...",
      "transit_info": {
        "mode": "walk",
        "distance_km": 0.55,
        "duration_min": 7,
        "estimated_fare_vnd": 0,
        "label": "🚶 7 min walk (550m)"
      }
    }
  ]
}
```

#### Ví dụ Payload cho `POST /api/onboarding/parse` (Smart Start, Type 1)

**Request:** `{ "text": "2 days, around 3 million, I love street food and coffee" }`

**Response (thiếu điểm xuất phát → hỏi tối đa 1 câu, câu hỏi + nút do **mẫu** tạo ra, không tốn thêm lượt Gemini):**
```json
{
  "status": "asking",
  "draft": { "start_location": null, "budget": 3000000, "days": 2, "preferences": ["street_food", "cafe"] },
  "missing": ["start_location"],
  "question": "Where will you start each day?",
  "quick_replies": ["District 1", "District 3", "District 7", "Somewhere else…"]
}
```

#### Ví dụ Payload cho `POST /api/trips/:id/share`

```json
{ "share_url": "https://your-app.vercel.app/s/Xk3vT9mQ2rLpA8dW5nYhZg", "token": "Xk3vT9mQ2rLpA8dW5nYhZg" }
```

### 3.5. Database schema (Chuẩn hóa 100% PostgreSQL 16)

| Bảng | Trường chính | Mục đích |
| --- | --- | --- |
| `users` *(mới, P2)* | `id`, **username** (nullable, unique không phân biệt hoa thường), **password\_hash** (bcrypt, nullable), `email` (nullable, unique), **google\_sub** (unique, nullable), `display_name`, **settings** (`JSONB`), `created_at`, `last_login_at` | Tài khoản. Ràng buộc: có Google **hoặc** có cả username + password |
| `password_reset_tokens` *(mới, P2)* | `id`, `user_id` (FK), **token\_hash** (SHA-256), `expires_at`, `used_at` | Token đặt lại mật khẩu: chỉ lưu **hash**, dùng một lần, hết hạn sau 30 phút |
| `trips` | `id`, **session\_id** (UUID), **user\_id** (FK, nullable), `title`, `start_location`, `budget`, `days`, **preferences** (`JSONB`), **status** (`draft`/`asking`/`generating`/`completed`), `source`, **share\_token** (unique, nullable), `shared_at`, `feedback` (`JSONB`), `created_at` | Thông tin chuyến đi. Khách: `user_id` rỗng, phân tách theo `session_id`. Thành viên: theo `user_id` |
| `itinerary_items` | `id`, `trip_id` (FK), `day_number`, **order\_index**, `time_slot`, `place_name`, **description**, `category`, `place_id_google` (Indexed), **latitude**, **longitude**, `estimated_cost`, **transit_info** (`JSONB`) | Từng mục trong lịch trình kèm tọa độ và ước tính di chuyển |
| `conversations` | `id`, `trip_id` (FK), `role` (`user`/`model`), `content`, **metadata** (`JSONB`), `created_at` | Lưu lịch sử hội thoại để AI nhớ ngữ cảnh qua nhiều lượt. `metadata` ghi cả sự kiện swap (địa điểm bị đổi) làm nguyên liệu cho Travel Profile |
| `places_cache` | `place_id_google` (PK), `name`, `rating`, `address`, **types** (`JSONB`), **opening\_hours** (`JSONB`), **latitude**, **longitude**, `photo_url`, `cached_at`, **expires\_at** | Cache dữ liệu từ Google Places (TTL 7 ngày), tránh gọi API lặp lại |
| `travel_profiles` *(mới, P3)* | **owner\_key** (PK: `u:<user_id>` hoặc `s:<session_id>`), **profile** (`JSONB`), `updated_at` | Trí nhớ của AI về sở thích. Khóa theo `owner_key` để cả khách lẫn thành viên đều dùng được và chuyển được khi claim |

> **Trường in đậm** là các trường bổ sung giúp tăng độ tin cậy và phục vụ UI/UX bản đồ & timeline (v1) hoặc phục vụ danh tính, chia sẻ và thích ứng (v2).
>
> **Lưu ý thiết kế khóa (v1, giữ nguyên)**: Cột `place_id_google` trong `itinerary_items` không gắn cứng Foreign Key constraint vào `places_cache` mà dùng Index + `LEFT JOIN` khi truy vấn. Nhờ vậy, nếu người dùng yêu cầu thêm một địa điểm cá nhân ngoài Google Places (ví dụ: *"Nhà bạn ở Quận 7"*), hệ thống vẫn lưu lịch trình mượt mà không bị lỗi ràng buộc khóa ngoại.
>
> **Lưu ý thiết kế mới**:
> - `trips.session_id` vẫn `NOT NULL` kể cả với thành viên: trình duyệt nào cũng có `session_id`, và giữ lại giúp truy vết/claim.
> - `trips.user_id` dùng `ON DELETE CASCADE`: xóa tài khoản là xóa luôn chuyến đi, lịch trình và hội thoại. `travel_profiles` không có FK (do khóa có thể là `session_id`), nên `DELETE /api/auth/account` xóa nó **bằng code** trong cùng transaction.
> - Hai cột `username` và `email` đều có **unique index trên `LOWER(...)`** để "Linh" và "linh" không thể cùng tồn tại.
> - Docker chỉ chạy `schema.sql` **lần đầu** khi volume rỗng. Khi sửa schema trong lúc dev, chạy `docker compose down -v` rồi `up` lại (mất dữ liệu dev, chấp nhận được), còn trên Supabase thì chạy tay phần thay đổi.

---
