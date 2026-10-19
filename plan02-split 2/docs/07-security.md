## 7. Security

| Rủi ro | Giải thích | Cách xử lý |
| --- | --- | --- |
| SQL Injection | Raw SQL nối chuỗi trực tiếp từ input có thể bị chèn SQL độc hại | Dùng parameterized query của PostgreSQL (`pool.query("... WHERE id = $1 AND session_id = $2", [id, sessionId])`), không nối chuỗi. Điều kiện sở hữu do `ownerClause()` sinh ra chỉ ghép **số thứ tự placeholder**, không ghép dữ liệu người dùng |
| API key lộ ra frontend (bao gồm URL ảnh) | Gọi thẳng Gemini/Google Places từ React hoặc gắn `key=` vào URL ảnh Places sẽ lộ key trong DevTools | Mọi API key chỉ gọi từ backend, frontend không bao giờ gọi thẳng API ngoài. Ảnh Google Places được resolve sang URL `lh3.googleusercontent.com` hoặc đi qua `/api/places/photo/:ref` |
| Truy cập trái phép chuyến đi của người khác (IDOR) | Kẻ tấn công truyền `trip_id` bất kỳ để đọc/xóa chuyến đi của phiên hoặc tài khoản khác | Middleware `identity` + `ownerClause()`: mọi truy cập `/api/trips/:id/*` phải khớp `user_id` (thành viên) hoặc `session_id` với `user_id IS NULL` (khách). Sai quyền trả `404`, không phải `403`, để không lộ sự tồn tại |
| Không giới hạn request | Gemini free tier chỉ ~15 request/phút — dễ sập quota khi demo | Thêm rate limiting backend (`express-rate-limit`): **10 req/phút/IP** cho chat **và Smart Start (dùng chung bucket vì cùng tiêu thụ Gemini)**, **30 req/phút/IP** cho các route khác. Thêm trần **toàn server** `geminiBudget` |
| Input không kiểm tra | Số ngày âm, ngân sách sai kiểu có thể gây lỗi/crash | Validate input cả frontend lẫn backend bằng `zod` (xem mục 6) |
| CORS mở toàn bộ | Domain khác gọi được API, tốn quota miễn phí | Chỉ cho phép domain frontend thật (Vercel URL) và `http://localhost:5173` qua CORS. **v2:** `allowedHeaders: ['Content-Type', 'x-session-id', 'Authorization']` và `exposedHeaders: ['X-Auth-Status']` (thiếu cái sau thì frontend không đọc được trạng thái token hết hạn) |
| Prompt injection | User nhập text cố tình "hack" system prompt của AI | Tách rõ `systemInstruction` trong Gemini SDK vs user input; giới hạn 500 ký tự và lọc bỏ các thẻ giả mạo hệ thống. **v2:** thêm các lớp phòng thủ cho profile/persona ở mục 4.7.5 |
| Lộ mật khẩu | Lưu mật khẩu dạng rõ hoặc hash yếu; log lộ mật khẩu | `bcryptjs` cost 12, mật khẩu 8–72 ký tự; không bao giờ log body của `/api/auth/*`; không trả `password_hash` ra response |
| Dò tài khoản (user enumeration) | Kẻ tấn công đoán username/email nào tồn tại qua thông báo hoặc thời gian phản hồi | Thông báo đăng nhập **chung**; luôn chạy `bcrypt.compare` (với `DUMMY_HASH` nếu không có user) để thời gian giống nhau; `forgot-password` luôn trả `200` |
| Đoán mật khẩu (brute force) | Thử hàng loạt mật khẩu | `express-rate-limit` **5 lần/phút theo cặp IP + username** cho đăng nhập, **5 lần/giờ/IP** cho đăng ký, **3 lần/giờ** cho `forgot-password` |
| Đánh cắp JWT | Token trong `localStorage` bị XSS lấy | Không dùng `dangerouslySetInnerHTML`, CSP cơ bản, token hết hạn sau 7 ngày, payload chỉ có `sub`, `JWT_SECRET` ≥ 32 byte ngẫu nhiên trong biến môi trường. Đăng xuất xóa token **và xoay `session_id`** |
| Giả mạo đăng nhập Google | Client gửi ID token giả hoặc của ứng dụng khác | **Backend** gọi `verifyIdToken` với `audience = GOOGLE_CLIENT_ID`, kiểm tra `email_verified`; không tin bất kỳ trường nào client gửi riêng |
| Token đặt lại mật khẩu bị đánh cắp hoặc đoán | Chiếm tài khoản qua email | 32 byte ngẫu nhiên, DB chỉ lưu **SHA-256**, hết hạn 30 phút, dùng một lần, đổi mật khẩu xong thì vô hiệu mọi token cũ |
| Lộ dữ liệu qua link chia sẻ | Link chuyền tay hoặc bị lộ | Token 128-bit không đoán được, **chỉ đọc**, thu hồi được (token cũ → 404), payload lược bỏ `session_id`/`user_id`/chat/email, `X-Robots-Tag: noindex`, rate limit 30 req/phút/IP |
| Quyền riêng tư dữ liệu | Trí nhớ AI và dữ liệu tài khoản là dữ liệu cá nhân | Chỉ lưu dữ liệu tối thiểu (không lưu SĐT), profile xem/sửa/xóa/tắt được, xóa tài khoản xóa **cascade** toàn bộ chuyến đi và profile |
| Khách lạm dụng quota Gemini | Ai cũng dùng được nên có thể spam | Rate limit theo IP + trần toàn server `geminiBudget`; chat luôn được ưu tiên hơn các lượt gọi phụ |

**Checklist**:

- [x] `.env` + `.gitignore` — không commit API key
- [x] HTTPS tự động từ Vercel/Render
- [ ] Parameterized query (`$1, $2...`) cho mọi truy vấn SQL PostgreSQL
- [ ] Ẩn `GOOGLE_PLACES_API_KEY` khỏi URL ảnh bằng Photo Proxy / Redirect resolver
- [ ] Rate limiting ở backend (`express-rate-limit`: 10 req/phút/IP cho chat)
- [ ] Validate input (frontend + backend) + kiểm tra `x-session-id`
- [ ] Cấu hình CORS chỉ cho domain frontend thật (kèm `Authorization` và `X-Auth-Status`)
- [ ] Prompt injection protection
- [ ] *(P1)* Link chia sẻ: token ngẫu nhiên, chỉ đọc, thu hồi được, payload lược bớt
- [ ] *(P2)* Mật khẩu hash bằng bcrypt, không log, thông báo đăng nhập chung
- [ ] *(P2)* Rate limit đăng nhập / đăng ký / quên mật khẩu
- [ ] *(P2)* `JWT_SECRET` ngẫu nhiên ≥ 32 byte, token có hạn
- [ ] *(P2)* Google ID token được **backend** xác minh
- [ ] *(P2)* Claim chạy trong transaction, `identity` + `ownerClause()` dùng cho mọi route trip
- [ ] *(P3)* `sanitizeProfile()` + whitelist persona; PROFILE bọc `<user_profile>`

---
