# AGENTS.md — Travel AI HCMC (plan v2)

> File vào cửa cho Agent. **Chỉ đọc file này trước.** Đừng nạp cả `docs/`: mỗi file là một mục của plan02, chỉ mở file đúng việc đang làm.

## Dự án
Website AI lên lịch trình du lịch TP.HCM. Đồ án cá nhân, 8 tuần, deadline 09/12/2026, demo tiếng Anh.

## Stack (chi tiết ở `docs/03`)
React 18 + Vite + Tailwind + React Router · Node 20 + Express · PostgreSQL 16 (Docker local, Supabase production) · Gemini 2.5 Flash (`@google/genai`, function calling) · Google Places API (New) · Vercel + Render.

## Quy tắc làm việc
1. Làm theo thứ tự ưu tiên: **P0 MVP → P1 (My Trips, share, Skip) → P2 (tài khoản) → P3 (Type 1/2, profile, persona) → P4 bonus**. Không làm tầng sau khi tầng trước chưa xong và qua gate (`docs/05`).
2. Mục **Roadmap / Out of Scope** (NPC tùy biến, gói trả phí…) **không làm**.
3. Số mục trong plan (ví dụ §3.4, §12.7) khớp tiền tố tên file: §3.x → `docs/03`, §10.2 → `docs/10b`, §12.7 → `docs/12-7`.
4. Code trong `docs/12-*` là **bản mẫu tham khảo**, phải chạy và test thật trước khi dùng.
5. API (§3.4), schema (`docs/12-1`), tool schema (§4.2) là hợp đồng giữa các phần. Muốn đổi thì nói với người dùng trước, rồi sửa tài liệu cùng lúc với code.
6. Có chỗ mâu thuẫn giữa các file thì dừng và hỏi, không tự đoán.
7. Đổi `schema.sql` thì cần `docker compose down -v` để tạo lại DB.

## Làm việc nhóm (7 người)
- Phân công, quyền sở hữu file, quy tắc nhánh/PR: `team/TEAM_EDITION.md`. Chỉ đọc khi cần biết ai phụ trách gì.
- **Phân chia Frontend (tránh xung đột git)**: Hạo phụ trách UI cốt lõi, design system, chat, map; Đăng hỗ trợ từ tuần 2 phụ trách TripForm, Header, trang tĩnh, MyTripsPage, SharedTripPage (chi tiết xem `team/TEAM_EDITION.md`).
- **Hợp đồng API** là `team/openapi.yaml`. Làm FE hoặc gọi API thì đọc file này thay vì đọc code backend. Đổi API phải sửa file này trong cùng PR (CI kiểm tra bằng contract test).
- Chỉ sửa file thuộc phần được giao. File của người khác (xem `team/CODEOWNERS`) thì đề xuất qua PR, không tự sửa.

## Khi làm việc gì thì đọc file nào

| Việc | Đọc |
|---|---|
| Hiểu sản phẩm, nguyên tắc | `01`, `02` |
| Dựng khung dự án, cấu trúc thư mục | `03` (§3.1–3.3), `09` |
| Viết endpoint / sửa DB | `03` (§3.4–3.5), `12-1`, `12-4` |
| Đăng ký, đăng nhập, Google, quên mật khẩu, claim chuyến khách | `03`, `07`, `12-6`, `12-7` |
| Chat AI, function calling, prompt | `04`, `12-8`, `12-3` |
| Type 1 / Type 2, Smart Start, Welcome back, persona | `04` (§4.7), `12-8`, `10c` (§10.11–10.12) |
| Màn hình UI | `10a`, `10b`, `10c` |
| Test | `08` |
| Lỗi, rủi ro | `06`, `11` |
| Deploy, Docker | `09`, `12-5` |
| Lịch tuần, gate, cut line | `05` |
| Slide, demo, Q&A | `13`, `14`, `15` |

## Danh sách file

| File | Nội dung | Cỡ | Dùng khi |
|---|---|---|---|
| `docs/01-tong-quan.md` | §1 Tổng quan, mục tiêu, nguyên tắc sản phẩm (kèm changelog v1→v2) | ~2k | Hiểu bức tranh chung, 6 nguyên tắc sản phẩm |
| `docs/02-scope.md` | §2 Phạm vi P0–P4, Out of Scope, ma trận Type 1/2 | ~2k | Quyết định cái gì làm / không làm, thứ tự ưu tiên |
| `docs/03-kien-truc-api-db.md` | §3 Tech stack, kiến trúc, cây thư mục, API, schema DB | ~9k | Viết endpoint, DB, cấu trúc thư mục (hợp đồng FE↔BE) |
| `docs/04-prompt-ai-flow.md` | §4 Prompt, tool schema, luồng AI, lớp thích ứng (Smart Start/Profile/Persona) | ~7k | Làm Gemini, function calling, Type 1/2 |
| `docs/05-timeline.md` | §5 Timeline 8 tuần, gate, cut line, ước lượng công | ~2k | Lập kế hoạch từng tuần, quyết định cắt gì |
| `docs/06-error-handling.md` | §6 Xử lý lỗi | ~1k | Viết error code, thông báo, fallback |
| `docs/07-security.md` | §7 Bảo mật + checklist | ~1k | Làm auth, rate limit, share link, review bảo mật |
| `docs/08-testing.md` | §8 Testing (TC-01..TC-25) | ~2k | Viết test |
| `docs/09-deploy.md` | §9 Deploy, env, docker-compose, quota | ~2k | Deploy, Docker, biến môi trường |
| `docs/10a-ui-design-system.md` | §10.1 Design system (màu, font, spacing) | ~1k | Style UI, Tailwind token |
| `docs/10b-ui-wireframes.md` | §10.2 Wireframe 9 màn hình | ~11k | Dựng từng màn hình / trạng thái |
| `docs/10c-ui-chat-motion-settings.md` | §10.3–10.12 Chat UX, timeline, animation, responsive, a11y, dark mode, Settings, nguyên tắc Skip | ~6k | Chat UI, animation, responsive, Settings/Profile |
| `docs/11-risks.md` | §11 Rủi ro & dự phòng | ~1k | Đánh giá rủi ro, kế hoạch B |
| `docs/12-1-schema-sql.md` | §12.1 schema.sql v2 (code) | ~2k | Tạo/sửa DB |
| `docs/12-2-transit-calculator.md` | §12.2 transitCalculator.js (code) | ~1k | Tính khoảng cách, giá Grab |
| `docs/12-3-place-service.md` | §12.3 placeService.js (code) | ~1k | Gọi Google Places + cache |
| `docs/12-4-trip-service.md` | §12.4 tripService.js (code) | ~1k | Lưu lịch trình, My Trips, ngân sách |
| `docs/12-5-seed-demo.md` | §12.5 seedDemoCache.js + seedDemoUser.js (code) | ~2k | Dữ liệu seed cho demo |
| `docs/12-6-identity.md` | §12.6 identity.js + ownerClause (code) | ~1k | Danh tính khách/thành viên |
| `docs/12-7-auth-service.md` | §12.7 authService.js (code) | ~2k | Đăng ký/đăng nhập/Google/claim |
| `docs/12-8-adaptive-layer.md` | §12.8 tierService, personas, promptBuilder, geminiBudget (code) | ~2k | Code lớp thích ứng AI |
| `docs/13-demo-qa.md` | §13 Kịch bản demo tiếng Anh + Q&A | ~2k | Chuẩn bị thuyết trình |
| `docs/14-checklist.md` | §14 Checklist nộp bài | ~1k | Kiểm tra trước ngày nộp |
| `docs/15-roadmap.md` | §15 Roadmap (NPC, gói trả phí…, chỉ nêu trong slide) | ~1k | Slide hướng phát triển |

## Trạng thái hiện tại (cập nhật tay)
- Tuần: **Tuần 1 (15–23/10)** — Đang tiến tới **Gate 0 (23/10)**
- Tầng đang làm: P0 (Khởi tạo repo, API Contract v2, Database schema v2, Design system, Prism mock server)
- Gate đã qua: Chuẩn bị chốt Gate 0 vào 23/10
- Các mục bổ sung đã chốt ngoài plan02: helmet, `/api/health`, `DEMO_MODE`, contract test (`jest-openapi`), Prism mock server, `server/src/app.js` central error-handler (`{ error: { code, message } }`).
