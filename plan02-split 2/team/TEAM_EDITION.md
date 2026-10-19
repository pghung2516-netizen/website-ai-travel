# TEAM EDITION — Travel AI HCMC (7 người)

> Bổ sung cho plan02. plan02 viết cho 1 người; file này chuyển nó thành kế hoạch cho nhóm.
> Hợp đồng API nằm ở `team/openapi.yaml`. Nội dung từng mục kỹ thuật vẫn ở `docs/`.

**Hạn nộp: 09/12/2026. Feature freeze: 02/12.**

## 0. Giả định cần xác nhận

Các điểm dưới đây tôi đã tự chọn để có kế hoạch cụ thể. Nếu sai, đổi lại ở bảng phân công:

1. **Hạo (UI/UX) cũng code giao diện React**, không chỉ thiết kế. Nếu Hạo chỉ làm Figma thì cần thêm một người code FE.
2. **Danh (Leader) kiêm DevOps**: repo, CI, deploy Vercel + Render + Supabase, Docker Compose.
3. **Đăng (Presenter) hỗ trợ FE từ tuần 2** (trang tĩnh, TripForm, Header — chia component với Hạo) vì phần thuyết trình chủ yếu dồn vào tuần 7–8.
4. **Định dạng lỗi chuẩn toàn hệ thống là `{ error: { code, message } }`** (đã chốt chính thức). Mọi controller/service/middleware đều phải trả về định dạng lồng này.

## 1. Phân công và quyền sở hữu

| Thành viên | Role | Chịu trách nhiệm | Sở hữu thư mục / file |
|---|---|---|---|
| **Khang An** | Developer (Backend core) | Server runtime, central error handler, Trips, My Trips, share, auth (đăng ký/đăng nhập/Google/quên mật khẩu), identity, claim chuyến khách | `server/src/app.js`, `server/src/routes/*` (trừ chat/onboarding), `services/tripService`, `authService`, `emailService`, `shareService`, `middleware/identity.js` |
| **Gia Hưng** | AI-Integration | Gemini, function calling, prompt, Smart Start, tier, profile, persona, Places + cache, transit | `server/src/tools/*`, `services/aiService`, `placeService`, `tierService`, `profileService`, `promptBuilder`, `personas.js`, `middleware/geminiBudget.js`, `utils/transitCalculator.js`, `routes/chatRoutes`, `onboardingRoutes` |
| **Minh Hạo** | UI/UX + Frontend | Design system, wireframe, toàn bộ React UI, animation, responsive | `client/src/*`, `docs/10a`, `10b`, `10c` |
| **Hữu Lam** | Security & Data | Schema DB, seed, bảo mật (helmet, rate limit, validation khung), review PR auth | `server/src/db/*` (**chủ duy nhất của `schema.sql`**), `middleware/rateLimiter.js`, `validator.js`, `docs/07` |
| **Tấn Đạt** | QA/QC | Test case TC-01…TC-25, test API, contract test (jest-openapi), smoke E2E, kiểm tra gate, ghi bug | `**/*.test.js`, `tests/`, `docs/08` |
| **Bảo Danh** | Leader + DevOps | Kế hoạch, gate, cut line, merge, CI, deploy, Docker, cron ping | `.github/*`, `docker-compose.yml`, `Dockerfile*`, `.env.example`, `docs/05`, `09`, `11` |
| **Hải Đăng** | Presenter + hỗ trợ FE | Slide, kịch bản demo, Q&A, README, video backup; **tuần 2–6 hỗ trợ FE** (TripForm, Header, trang tĩnh, MyTripsPage, SharedTripPage) cùng Hạo | `docs/13`, `14`, `15`, `README.md`, thư mục slide/video |

**Quy tắc quyền sở hữu:** ai muốn sửa file của người khác thì mở PR và người sở hữu duyệt (xem `CODEOWNERS`). Không tự sửa trực tiếp.

**Làm rõ trách nhiệm một số file liên quan nhiều người:**
- `server/src/app.js`: An làm chủ (khởi chạy Express, gắn middleware theo thứ tự: `cors()` TRƯỚC `identity`, central error-handler chuẩn hóa mọi lỗi thành `{ error: { code, message } }`, gồm cả SyntaxError của JSON và Zod).
- `GET /api/trips/:id/chat` (Lịch sử chat): An phụ trách route và truy vấn DB; An thực hiện đổi tên trường `role`: `model` (DB) → `assistant` (API) khi trả về client.
- `meRoutes.js`: An phụ trách route/controller; Hưng phụ trách logic nghiệp vụ trong `tierService`, `profileService`.
- `validator.js` & DB connection: Lam quản lý middleware khung `validator.js` (schema Zod chi tiết do chủ route viết) và cấu hình `pg.types.setTypeParser(1700, ...)` trong `connection.js` để `rating` trả về kiểu `number`.
- `middleware/rateLimiter.js`: Lam cấu hình option `handler` để khi chạm ngưỡng 429 luôn trả về `{ error: { code: 'AI_RATE_LIMITED', message } }`.

**Phân chia component FE (tránh xung đột Hạo ↔ Đăng):**
- **Hạo**: `ChatPanel/*`, `ItineraryView/*`, `MapPanel/*`, `Settings/*`, `Assistant/*`, `context/`, `hooks/`, `services/`.
- **Đăng**: `TripForm/*`, `Header/*`, `Trips/MyTripsPage`, `Trips/SharedTripPage`, các trang tĩnh (Error, Empty).
- Hạo chốt design system (`docs/10a`) ở tuần 1 → Đăng dùng làm chuẩn. Hai người không sửa cùng một component trừ khi pair.

## 2. Các điểm nối giữa người với người (interface)

Đây là những chỗ dễ lệch nhất. Mỗi điểm nối có một "hợp đồng" và hai người chịu trách nhiệm.

| # | Giữa ai | Hợp đồng | Chốt khi nào |
|---|---|---|---|
| 1 | **Hạo ↔ An** | `team/openapi.yaml` (mọi endpoint trừ chat). Hạo làm UI dựa trên mock server từ file này | Hết tuần 1 |
| 2 | **Hạo ↔ Hưng** | Response của `POST /api/trips/:id/chat` (`status`, `reply`, `quick_replies`, `updated_itinerary`, `budget_summary`) và các trạng thái để vẽ `ThinkingSteps` | Hết tuần 2 |
| 3 | **Hưng ↔ An** | `saveItinerary` (Hưng) gọi `tripService` (An): định dạng `itinerary_items` ở §3.4 và `docs/12-4`; `req.principal` từ `identity.js` truyền xuống `aiService` | Hết tuần 2 |
| 4 | **Lam ↔ mọi người** | `schema.sql` v2 (`docs/12-1`). Đổi cột phải qua PR của Lam, báo ở nhóm chat. Chú ý constraint `source IN ('form', 'sentence', 'assistant', 'clone')` | Hết tuần 1 |
| 5 | **Đạt ↔ mọi người** | Điều kiện hoàn thành (mục 5) và danh sách test case `docs/08`; contract test trong CI kiểm tra response theo `openapi.yaml` | Hết tuần 1 |
| 6 | **Hạo ↔ Hưng** | **Khởi tạo AI**: Ngay sau `POST /api/trips`, FE tự động gọi `POST /api/trips/:id/chat` với tin nhắn mặc định `"Plan my trip"` để kích hoạt lượt tạo lịch trình đầu tiên | Hết tuần 2 |
| 7 | **Hạo ↔ Hưng** | **Nút "Swap this place"**: FE gửi tin nhắn chat theo cú pháp chuẩn `"Swap {place_name} on Day {n}"` (AI nhận diện và gọi tool thay đổi địa điểm tương ứng) | Hết tuần 4 |
| 8 | **An ↔ Hạo** | **Bypass danh tính**: `identity.js` bỏ qua `GET /api/health` và toàn bộ `/api/shared/:token` (ngoại trừ đuôi `/clone`). Request thử PUT/DELETE không có header vẫn lọt tới router trả 404/405 theo TC-12 | Hết tuần 1 |

**Mock server cho Hạo & Đăng** (không cần chờ An):
```bash
npx @stoplight/prism-cli mock team/openapi.yaml -p 3001
```
Prism trả dữ liệu mẫu theo schema. Frontend trỏ `VITE_API_URL=http://localhost:3001` vào đó. Khi backend thật sẵn sàng, chỉ đổi biến môi trường thành `http://localhost:5000`.
> 💡 **Mẹo Prism**: Mặc định Prism trả ví dụ đầu tiên (`askingPreference`). Để nhận dữ liệu lịch trình hoàn chỉnh (`updated_itinerary`), Frontend gửi kèm header: `Prefer: example=itineraryReady`.

## 3. Cách làm việc chung

**Nhánh và PR**
- `main` được bảo vệ: không push trực tiếp, merge qua PR, cần **1 người duyệt là chủ file** (CODEOWNERS) và CI xanh.
- Tên nhánh: `feat/<role>-<việc>` (ví dụ `feat/be-my-trips`), `fix/…`.
- PR nhỏ (dưới ~300 dòng), một việc một PR. Dùng mẫu `pull_request_template.md`.
- Merge bằng **squash**. Danh là người bấm merge cho `main`.

**Quy ước code**
- ESLint + Prettier cùng một cấu hình ở gốc repo (Danh thiết lập tuần 1).
- Tuyệt đối không commit `.env` hay API key. Chỉ commit `.env.example`.
- Đổi API: sửa `team/openapi.yaml` **trong cùng PR** với code; endpoint `/chat` và onboarding do Hưng và Hạo duyệt; các endpoint CRUD khác do An và Hạo duyệt.

**CI pipeline (Danh thiết lập, Đạt bổ sung contract test):**
1. Lint & format: `eslint` + `prettier --check`.
2. `spectral lint --fail-severity=error team/openapi.yaml` — kiểm tra file YAML đúng quy ước OpenAPI. Cờ `--fail-severity=error` giúp CI chỉ đỏ khi có lỗi cú pháp/schema nghiêm trọng, bỏ qua warning về operationId hay contact.
3. **Contract test (chính):** Đạt viết test Supertest gọi API thật rồi validate response theo schema trong `openapi.yaml` (dùng `jest-openapi` hoặc `express-openapi-validator` ở chế độ test). Backend trả response lệch file hoặc thiếu trường required → CI đỏ. Đây là cách duy nhất bắt được API drift.

**Chính sách API key khi phát triển:**
- **Gemini**: mỗi người tạo **project Google riêng** (tài khoản cá nhân) → key riêng. Hạn mức Gemini tính theo **project**, không theo key; nhiều key trong cùng project vẫn dùng chung quota.
- **Google Places**: chỉ **Hưng** và **An** cần key thật (kèm Budget Alert $5 và quota cap trên Cloud Console). Hạo dùng mock server (`prism`), Đạt dùng `DEMO_FALLBACK_MODE=true` + dữ liệu seed. Không bắt mọi người bật thanh toán.
- Nhiều key = nhiều chỗ lộ. Nhắc lại: **không commit `.env`**, review `.gitignore` ở tuần 1.

**Giao tiếp**
- Họp đồng bộ ngắn 2 lần/tuần (15 phút): ai làm gì, ai đang bị chặn.
- Mỗi gate (11/11, 25/11): Danh chủ trì, Đạt báo cáo kết quả kiểm tra, cả nhóm quyết định cut line.
- Bị chặn quá 1 ngày thì nói ngay, đừng đợi buổi họp.

**Với Agent coding:** mỗi người đưa Agent `AGENTS.md` và chỉ các file thuộc phần mình (xem `huong_dan_prompt.md`). Không đưa `plan02-full.md`.

## 4. Kế hoạch từng tuần theo role

Bám theo `docs/05`. "Gate" là điểm quyết định cắt hay giữ.

> ⏱️ **Cập nhật tiến độ & Điều chỉnh lịch (Hiện tại: 19/10):**
> Tuần 1 (15–21/10) đã trôi qua 4 ngày nhưng các deliverable cốt lõi đang chuẩn bị chốt.
> - **🚦 Gate 0 (23/10)**: Chốt hợp đồng `openapi.yaml` v2 (đủ `required` và ví dụ mock), merge `schema.sql` v2, setup repo chạy được `npm run dev` (client + server) + Prism mock server.
> - **Bù 2 ngày trượt**: Lấy từ Tuần 2 (Tuần 2 nhẹ hơn dự kiến nhờ FE đã có Prism mock server với ví dụ đầy đủ).
> - **Các mốc cứng giữ nguyên**: Gate A (11/11), Gate B (25/11), Feature Freeze (02/12), Nộp bài (09/12).
> - **Khuyến nghị theo dõi tiến độ**: Tạo **GitHub Project** (Kanban board) với 1 issue cho mỗi deliverable trong bảng dưới đây.

| Tuần | Danh (Leader/DevOps) | An (Backend) | Hưng (AI) | Hạo + Đăng (FE) | Lam (Sec/Data) | Đạt (QA) |
|---|---|---|---|---|---|---|
| **1** (15–23/10) | Repo, ESLint/Prettier, CI lint, Docker Compose Postgres, chốt `openapi.yaml` v2. **🚦 Gate 0 (23/10)** | Dựng Express, `app.js` + `identity.js` khung, xử lý bypass public routes | Thử Gemini + Places, nháp tool schema | Design system, wireframe 3 màn chính + phác My Trips/Settings; dựng Vite + Tailwind; chạy Prism mock | **`schema.sql` v2** (đủ bảng, chú ý CHECK source) | Đọc plan, viết danh sách test case, dựng khung test Jest + Supertest |
| **2** (24–28/10) | CI chạy test, quản lý PR | CRUD `/api/trips` (chỉ khách) | `placeService` + cache, `transitCalculator` | **Hạo**: `ChatPanel`, `ItineraryView` với mock. **Đăng**: `TripForm`, `Header` | Seed cache, `validator`, `rateLimiter` | TC cho CRUD + Places |
| **3** (29/10–4/11) | Hỗ trợ blocker | Hoàn thiện trips, history chat | **Gemini + function calling**, prompt BASE, luồng cơ bản | **Hạo**: nối API thật, `ThinkingSteps`. **Đăng**: Error/Empty states, trang tĩnh | helmet, review bảo mật khung | Test AI qua Postman/Supertest; **bắt đầu contract test** |
| **4** (5–11/11) | Smoke deploy Vercel + Render. **🚦 Gate A (11/11)** | Hỗ trợ nối FE↔BE | Multi-turn, Swap this place, sửa lịch trình qua chat | **Hạo**: Chat flow hoàn chỉnh. **Đăng**: Natural Sentence Mode, hỗ trợ nối | Review rate limit | **Kiểm tra Gate A**, báo cáo |
| **5** (12–18/11) | Theo dõi tiến độ, quyết định bắt đầu P1 | **P1: My Trips API** (list/đổi tên/xóa/nhân bản) | Hỗ trợ prompt, fallback | Animation, responsive; **Đăng: MyTripsPage, TripCard** | Review SQL của My Trips | TC My Trips |
| **6** (19–25/11) | **🚦 Gate B (25/11)** | **P1: Share API** (tạo/thu hồi/clone) | Bắt đầu Smart Start nếu qua Gate | Maps + marker, **Đăng: SharedTripPage, ShareDialog** | Hardening security P0/P1, 10 test lõi | **Kiểm tra Gate B**, 10 test lõi |
| **7** (26/11–2/12) | Deploy chính thức, Docker backup, cron ping. **Feature freeze 02/12** | **P2: username/password + JWT + claim**, rồi Google | Smart Start (Type 1) nếu còn | `AuthModal`, Settings cơ bản | Review auth, seed demo user | Test auth, claim |
| **8** (3–9/12) | Kiểm tra sau cùng, README | Sửa bug | Sửa bug, hỗ trợ demo | Sửa bug UI | Kiểm tra bảo mật lần cuối | Regression, checklist nộp bài |

**Đăng (Presenter + FE):** tuần 1 viết khung slide, tìm hiểu sản phẩm, học design system (`docs/10a`); **tuần 2–6 hỗ trợ FE** (TripForm, Header, MyTripsPage, SharedTripPage, trang tĩnh); tuần 7–8 hoàn thiện slide, kịch bản `docs/13`, video backup.

## 5. Điều kiện hoàn thành (Definition of Done)

Một việc chỉ gọi là xong khi:
1. Chạy được trên máy người khác (không phụ thuộc cấu hình riêng).
2. Có test hoặc ít nhất một kịch bản kiểm tra thủ công do Đạt xác nhận.
3. `openapi.yaml` và tài liệu `docs/` cập nhật nếu API/schema đổi.
4. CI xanh và PR được chủ file duyệt.
5. Không có lỗi trong luồng chính khi demo bằng dữ liệu seed.

## 6. Phạm vi với nhóm 7 người

Plan02 ước lượng P1 ≈ 3.5 ngày, P2 ≈ 7 ngày, P3 ≈ 7 ngày cho **1 người**. Nhóm 7 người có thể làm song song nên **Cut line 3** (thêm Google Sign-In và Type 1 Smart Start) khả thi hơn, và **Cut line 4** (Type 2, Travel Profile, persona) vẫn nên coi là **tùy tiến độ**. Lý do: chi phí phối hợp tăng, và nhiều việc trong P2/P3 đều chạm vào cùng các file (`aiService`, `identity`, `schema.sql`).

Đừng lập kế hoạch cho đủ P3. Hãy quyết định ở Gate B.

## 7. Rủi ro riêng của làm nhóm

| Rủi ro | Cách giảm |
|---|---|
| FE và BE lệch API | `openapi.yaml` + mock server + **contract test trong CI** (jest-openapi, lệch → đỏ) + duyệt chéo khi đổi |
| Xung đột khi sửa chung `schema.sql`, `tripService`, `identity.js` | Mỗi file một chủ, qua PR |
| Xung đột FE giữa Hạo và Đăng | Chia component cố định (mục 1); không sửa cùng component trừ khi pair |
| Hạo quá tải (một mình FE) | Đăng hỗ trợ FE **từ tuần 2** (không đợi tuần 5); chia component rõ |
| An quá tải ở tuần 5–7 (My Trips, Share, Auth) | Lam review thay vì viết; cắt Google Sign-In/quên mật khẩu trước |
| Một người nghỉ hoặc chậm | Mỗi phần có người dự phòng: An↔Hưng (BE), Hạo↔Đăng (FE), Lam↔Đạt (kiểm thử) |
| Hạn chế quota Gemini / Places khi phát triển | **Gemini**: mỗi người dùng **project Google riêng** (cùng project = chung quota dù khác key). **Places**: chỉ Hưng và An cần key thật (kèm Budget Alert); Hạo dùng mock, Đạt dùng seed/`DEMO_MODE`. `geminiBudget` giữ trần cho bản deploy |
