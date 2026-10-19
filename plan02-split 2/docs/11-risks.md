## 11. Quản lý Rủi ro & Kế hoạch Dự phòng (Project Risk & Contingency Plan)

| Rủi ro dự án | Mức độ | Khả năng | Kế hoạch dự phòng (Contingency) |
| --- | --- | --- | --- |
| **Gemini function calling bị lỗi hoặc loop** | Cao | Thấp | Đã gộp batch `searchPlaces` và tách tool `saveItinerary`. Nếu API gặp sự cố, fallback sang chế độ **JSON Mode (Structured Outputs)** hoặc bật cờ `DEMO_FALLBACK_MODE=true` dùng dữ liệu địa phương có sẵn trong cache/seed data. |
| **Google Places API phát sinh chi phí** | Cao | Thấp | Bật **Budget Alert $5** trên Google Cloud Console + set quota cứng 150 request/ngày. Tận dụng triệt để `places_cache` trong database và chạy `seedDemoCache.js` trước buổi demo. |
| **Render Free Tier "ngủ đông" trước giờ demo** | Cao | Cao | 1. Đặt lịch gọi ping tự động (cronjob) hoặc chủ động mở link web trước 3-5 phút trước khi thuyết trình.<br>2. **Backup Plan**: Quay sẵn 1 video demo chất lượng cao (screen recording có voice tiếng Anh).<br>3. Chạy sẵn bản local trên laptop (`npm run dev` / `docker compose`) để sẵn sàng switch nếu mất mạng trường. |
| **Trễ deadline tuần 3-4 do tích hợp AI** | Trung bình | Trung bình | Đã dựng UI tĩnh từ Tuần 2. **Không bắt đầu P1** nếu chưa qua Gate A. Cắt theo thứ tự ngược: Roadmap → P3 → P2 → P1 → P4; tập trung 100% hoàn thiện Core MVP (Form → Chat → Itinerary Timeline). |
| **Chuyển đổi SQLite → PostgreSQL trên deploy gặp trục trặc** | Thấp (Đã triệt tiêu) | Rất thấp | **Đã giải quyết**: Dùng **PostgreSQL ngay từ đầu** (chạy container `postgres:16-alpine` local qua Docker, production dùng Supabase PostgreSQL). Cùng chung engine và file `schema.sql`, không cần migration phức tạp. |
| **Phình phạm vi do tài khoản và AI thích ứng** *(mới)* | Cao | Cao | Ba tầng phòng thủ: (1) Gate A/B và Cut line (mục 5.2), (2) ước lượng công sức công khai (mục 5.1), (3) **Feature freeze 02/12**. Mặc định kỳ vọng dừng ở Cut line 2. |
| **Claim làm mất chuyến của khách khi đăng ký** *(mới)* | Cao | Thấp | Claim chỉ **gán `user_id`**, không xóa dữ liệu; chạy trong cùng transaction với đăng ký/đăng nhập (lỗi thì rollback, vẫn là khách); có `TC-16`. |
| **Rò rỉ mật khẩu hoặc dữ liệu cá nhân** *(mới)* | Cao | Thấp | bcrypt cost 12, không log body auth, thông báo lỗi chung, rate limit, không lưu SĐT, xóa tài khoản cascade (mục 7). |
| **Hết quota Gemini do tính năng phụ** *(mới)* | Trung bình | Trung bình | Lời chào Type 2 dùng mẫu (0 lượt), Smart Start 1 lượt, tóm tắt profile chạy nền và bị bỏ qua khi thiếu slot; `geminiBudget` ưu tiên chat (mục 4.7.1). |
| **AI "nhớ" sai hoặc gây khó chịu** *(mới)* | Trung bình | Trung bình | AI luôn nêu giả định trong **một câu** để sửa ngay; profile xem/sửa/xóa/tắt được; trò chuyện ngoài lề tối thiểu và tắt được (mục 10.12). |
| **Đăng nhập Google / email lỗi lúc demo** *(mới)* | Trung bình | Trung bình | Demo dùng **username + tài khoản `demo_traveler`** (không cần Internet ngoài Gemini/Places). Google và quên mật khẩu chỉ mô tả bằng ảnh chụp/video. |
| **Injection qua trí nhớ của AI** *(mới)* | Trung bình | Thấp | Giá trị profile lấy từ **từ vựng cố định** (không có văn bản tự do), bọc `<user_profile>`, persona qua whitelist (mục 4.7.5). Có `TC-24`. |
| **Có tài khoản username nhưng chưa có quên mật khẩu** *(mới)* | Trung bình | Cao nếu cắt P2 một phần | Hiện cảnh báo "Password cannot be recovered yet" và ẩn ô email; không để người dùng tưởng có khôi phục (mục 5.2). |

---
