## 5. Timeline (8 tuần — Deadline: 09/12/2026)

Áp theo tỷ lệ chuẩn: phân tích/thiết kế ~20%, code core ~40%, tích hợp & testing ~20%, tài liệu & luyện thuyết trình ~10%, buffer ~10%. Dựng khung giao diện tĩnh từ Tuần 2 để giảm tải cho Tuần 5.

**Nguyên tắc xếp lịch v2:** các tính năng mới (P1 đến P3) **không có tuần riêng**. Chúng được xếp vào Tuần 5 đến 7, thay chỗ cho các mục P4 (Dark mode, Gamification) vốn là "Nên có" ở v1, và **chỉ bắt đầu khi qua Gate** (mục 5.2).

| Tuần | Thời gian | Phase | Công việc chi tiết | Deliverable |
| --- | --- | --- | --- | --- |
| 1 | 15/10 – 21/10 | Phân tích & Thiết kế | Phân tích yêu cầu chi tiết, vẽ wireframe 3 màn hình chính (Form → Chat → Itinerary) **và phác nhanh My Trips / Settings / Welcome**, thiết kế DB schema PostgreSQL (`JSONB`) **bản v2 (đủ mọi bảng, chốt mô hình danh tính `principal`)**, setup repo GitHub + project structure | Wireframe, repo GitHub với boilerplate chạy được (`npm run dev` cả client + server), `schema.sql` v2 |
| 2 | 22/10 – 28/10 | Thiết kế, Setup & Static UI | Khởi động PostgreSQL 16 local qua Docker Compose (`docker compose up -d postgres`), tạo DB schema, viết CRUD API cho trips (danh tính ở giai đoạn này chỉ có **khách** qua `identity.js`), kết nối + test Google Places API (kèm cache & photo proxy). Dựng khung UI tĩnh (`TripForm`, `ChatPanel`, `ItineraryView`) với dữ liệu mock | PostgreSQL chạy ổn định trên Docker, API `/api/trips` CRUD hoạt động, `placeService.js` trả dữ liệu thật, UI tĩnh hiển thị được timeline mẫu |
| 3 | 29/10 – 04/11 | Code Core AI | Tích hợp Gemini API (`@google/genai`) + function calling (`searchPlaces`, `askPreference`, `saveItinerary`), viết `transitCalculator.js`, viết system prompt (khối BASE), xây dựng conversation flow cơ bản | AI nhận input → gọi tool → lưu lịch trình vào DB kèm `transit_info` (test qua Postman) |
| 4 | 05/11 – 11/11 | Code Core & Nối UI | Hoàn thiện chat tương tác (AI hỏi lại, chỉnh sửa lịch trình, nút "Swap this place"), lưu conversation history, xử lý multi-turn dialogue, kết nối Frontend ↔ Backend. **Cuối tuần: deploy thử (smoke deploy) lên Vercel + Render** | Chat flow hoàn chỉnh trên trình duyệt: nhập info → AI hỏi → trả lời → nhận lịch trình → chỉnh sửa qua chat. **🚦 Gate A (11/11)** |
| 5 | 12/11 – 18/11 | Polish Frontend, Responsive & **P1 (My Trips)** | Hoàn thiện Natural Sentence Mode, Framer Motion animations, Loading 4 bước, Budget Bar đổi màu, responsive design (Desktop Split-view + Mobile Bottom Tab). **P1: `GET /api/trips` + trang My Trips (đổi tên, xóa, nhân bản) + nguyên tắc Skip + React Router** | Giao diện hoàn chỉnh, mượt mà, dùng được end-to-end trên browser + mobile. Quay lại web vẫn thấy chuyến cũ |
| 6 | 19/11 – 25/11 | Tích hợp, Security, Map & **P1 (Share)** | Security hardening (rate limiting 10 req/phút, input validation, CORS, parameterized queries `$1, $2`), viết 10 unit tests lõi (TC-01 đến TC-10), tích hợp Google Maps Embed, **P1: chia sẻ link chỉ xem + thu hồi + "Copy to my trips"**, sửa bug | Tất cả security checklist của P0/P1 hoàn thành, 10 unit tests lõi pass, bản đồ hiển thị pins, link chia sẻ chạy được. **🚦 Gate B (25/11)** |
| 7 | 26/11 – 02/12 | Deploy, **P2 (Tài khoản)** & Feature Freeze | Deploy chính thức lên Vercel + Render (kết nối Supabase Postgres - cùng engine với Docker dev), hoàn thiện Docker Compose fullstack backup cho demo. **P2: username + password, JWT, claim chuyến của khách, `AuthModal`, Settings cơ bản** → Google Sign-In → (nếu còn) quên mật khẩu. **P3 chỉ khi đã qua Gate B và P2 xong sớm: Smart Start (Type 1)**. **Feature freeze cuối tuần (02/12)** | Bản deploy online hoạt động ổn định + Docker local sẵn sàng cho offline demo backup + đăng ký/đăng nhập không mất chuyến đã tạo |
| 8 | 03/12 – 09/12 | Tài liệu & Thuyết trình | Chạy `seedDemoCache.js` + `seedDemoUser.js` làm nóng dữ liệu, viết slide tiếng Anh (gồm slide Roadmap, mục 15), viết kịch bản demo, quay video backup, luyện thuyết trình, viết README. **Không thêm tính năng mới** | Sẵn sàng nộp bài + thuyết trình |

> **Lưu ý**: Render free tier sleep sau 15 phút không hoạt động — nhớ mở link web trước ít nhất 2-3 phút trước khi vào thuyết trình.

> **Buffer**: v1 để buffer ở Tuần 7. Ở v2, buffer đó **được dùng làm vùng đệm cho P2/P3**, nên nếu Gate B không đạt thì Tuần 7 quay lại là buffer sửa lỗi như cũ.

### 5.1. Ước lượng công sức cho các tính năng mới (1 người, ngày làm việc)

Con số thô để quyết định cắt gì, không phải cam kết:

| Nhóm | Hạng mục | Ước lượng |
| --- | --- | --- |
| P1 | My Trips (list, đổi tên, xóa, nhân bản) | 1.5 |
| P1 | Chia sẻ link (trang chỉ xem, thu hồi, clone) | 1.5 |
| P1 | Nguyên tắc Skip + thẻ nhắc đăng ký không chặn | 0.5 |
| P2 | Username + password + JWT + `AuthModal` + claim | 3 |
| P2 | Google Sign-In | 1 |
| P2 | Quên / đặt lại mật khẩu (email) | 1.5 |
| P2 | Settings cơ bản (đổi mật khẩu, xóa tài khoản, giao diện, mức chủ động của AI) | 1 |
| P3 | Type 1: Smart Start | 2 |
| P3 | Type 2: tier + lời chào mẫu + tóm tắt profile + prompt blocks + trang Travel Profile | 4 |
| P3 | Persona preset | 1 |
| | **Tổng P1** | **3.5** |
| | **Tổng P2** | **7** |
| | **Tổng P3** | **7** |

Tuần 5 đến 7 chỉ có khoảng 15 ngày làm việc và **đã gần kín** với polish, security, test, map, deploy của P0. Vì vậy **không thể làm hết P1 đến P3** mà vẫn giữ chất lượng P0. Kỳ vọng thực tế là Cut line 2 (bên dưới); đừng lên kế hoạch cho đủ.

### 5.2. Gate và Cut line

| Mốc | Điều kiện qua | Nếu không đạt |
| --- | --- | --- |
| 🚦 **Gate A** (11/11) | Luồng form → AI hỏi → lịch trình → chỉnh sửa qua chat chạy end-to-end trên trình duyệt với dữ liệu Google Places thật; smoke deploy đã chạy được | **Đóng băng phạm vi**: Tuần 5 dùng để hoàn thiện P0, không bắt đầu P1 |
| 🚦 **Gate B** (25/11) | P0 polish xong, security checklist P0 xong, 10 test lõi pass, P1 xong | **Bỏ P2/P3**: Tuần 7 dành cho deploy, buffer và tài liệu |

| Cut line | Gồm | Ghi chú |
| --- | --- | --- |
| **Cut line 1** (an toàn) | P0 + P1 (My Trips, chia sẻ link, Skip) | Đủ để nói "không mất dữ liệu, gửi được cho bạn bè" mà không cần tài khoản |
| **Cut line 2** (kỳ vọng hợp lý) | + username/password + claim + Settings cơ bản | Hoạt động cả khi demo offline |
| **Cut line 3** (nếu thuận lợi) | + Google Sign-In + Type 1 Smart Start | |
| **Cut line 4** (nhiều khả năng chỉ nêu trong slide) | + quên mật khẩu, Type 2, Travel Profile, persona | Kể trong slide như Roadmap có thiết kế sẵn |

> ⚠️ Nếu **chưa** làm quên mật khẩu mà đã có đăng ký bằng username, màn đăng ký phải ghi rõ "Password cannot be recovered yet" và **ẩn ô email**. Đừng để người dùng tưởng có khôi phục.

---
