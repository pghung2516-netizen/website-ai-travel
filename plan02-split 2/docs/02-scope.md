## 2. Phạm vi (Scope)

### 2.1. Bảng tính năng theo mức ưu tiên

| Tier | Feature | Ghi chú |
| --- | --- | --- |
| **P0 Bắt buộc (MVP)** | Form nhập thông tin chuyến đi (điểm xuất phát, ngân sách, số ngày, sở thích) | Input cơ bản để AI lên plan (kèm chế độ Natural Sentence Mode) |
| **P0 Bắt buộc (MVP)** | AI sinh lịch trình chi tiết theo ngày (địa điểm + ẩm thực, giờ giấc, chi phí ước tính) | Lõi của sản phẩm, tích hợp thuật toán Haversine tự tính thời gian & chi phí di chuyển |
| **P0 Bắt buộc (MVP)** | Chat tương tác với AI để chỉnh sửa lịch trình (AI chủ động hỏi lại, gợi ý thêm) | Điểm nhấn "chủ động" với Quick Reply Chips và nút "Swap this place" |
| **P0 Bắt buộc (MVP)** | Lấy dữ liệu địa điểm/ẩm thực thật từ Google Places API (ảnh, rating, giờ mở cửa) | Thay thế phần crawl mạng xã hội; dùng Places API (New) kèm FieldMask + Photo Proxy |
| **P0 Bắt buộc (MVP)** | Hiển thị lịch trình dạng Vertical Timeline trực quan (Activity Cards, Transit, Budget Bar) | Giao diện cốt lõi để hiển thị và tương tác với lịch trình theo từng ngày |
| **P0 Bắt buộc (MVP)** | Responsive design (hoạt động tốt trên mobile) | Travel app cần dùng được trên điện thoại (Split-view trên Desktop, Bottom Tab trên Mobile) |
| **P1 Liền mạch (không cần tài khoản)** | **My Trips**: danh sách chuyến đã tạo (xem lại, đổi tên, xóa, nhân bản) | Dựa trên `session_id` của khách. Giải quyết lỗ hổng "quay lại không thấy gì" |
| **P1 Liền mạch (không cần tài khoản)** | **Chia sẻ lịch trình bằng link** (chỉ xem, thu hồi được) | Nhu cầu thật nhất vì đi du lịch thường đi nhóm. Nâng từ Bonus của v1 lên P1. Người nhận có thể "Copy to my trips" |
| **P1 Liền mạch (không cần tài khoản)** | **Nguyên tắc Skip**: nút bỏ qua ở mọi bước dẫn dắt + thẻ nhắc đăng ký không chặn luồng | Là quy tắc UI áp dụng cho toàn bộ web (mục 10.12) |
| **P1 Liền mạch (không cần tài khoản)** | Bản đồ tương tác (Google Maps Embed) liên kết hai chiều Card ↔ Pin bản đồ | Tăng trải nghiệm, liên kết tương tác hai chiều Card ↔ Pin bản đồ |
| **P2 Tài khoản** | Đăng ký / đăng nhập bằng **username + password** (lưu PostgreSQL, hash bcrypt) | Hoạt động cả khi demo offline (Docker). Email là tùy chọn nhưng cần nếu muốn khôi phục mật khẩu |
| **P2 Tài khoản** | **Claim**: chuyến đi của khách được gắn vào tài khoản khi đăng ký/đăng nhập | Bắt buộc đi kèm tài khoản. Nếu mất dữ liệu lúc đăng ký thì người dùng sẽ bỏ đi |
| **P2 Tài khoản** | Đăng nhập bằng **Google** ("Continue with Google") | Nhanh hơn nhập mật khẩu; cần Internet nên không dùng cho demo offline |
| **P2 Tài khoản** | Quên mật khẩu / đặt lại mật khẩu qua email | Làm sau cùng trong nhóm P2. Cần Internet (gửi email) |
| **P2 Tài khoản** | Trang **Settings** cơ bản (tài khoản, đổi mật khẩu, xóa tài khoản, giao diện, mức chủ động của AI) | Mục 10.11 |
| **P3 Thích ứng** | **Type 1 – Smart Start**: một ô nhập tự do, AI tự điền form, thiếu gì hỏi lại tối đa 1 câu | Luôn có nút Skip. 1 lần gọi Gemini |
| **P3 Thích ứng** | **Type 2 – Returning traveler**: lời chào theo mẫu, "Giống lần trước?", AI nhớ sở thích để ít hỏi hơn | Làm sau Type 1. Phần hỏi thăm ngoài lề chỉ là một câu ngắn, tắt được |
| **P3 Thích ứng** | **Travel Profile**: xem / sửa / xóa / tắt trí nhớ của AI | Bắt buộc đi kèm Type 2 (nguyên tắc 5) |
| **P3 Thích ứng** | **Persona preset**: chọn avatar + tên hiển thị (nam / nữ / trung tính / Pride…) và giọng điệu | Chỉ là lớp vỏ cho trải nghiệm. Giữ ở mức preset |
| **P4 Bonus** | Dark mode giao diện ban đêm | Dùng Tailwind `dark:`, custom dark map tiles. Dựng CSS tokens ngay từ đầu |
| **P4 Bonus** | Gamification (Trip Readiness Score + "Passport Stamp" Celebration) | Hạ từ "Nên có" của v1: tạo thiện cảm khi demo nhưng ít giữ chân người dùng quay lại |
| **P4 Bonus** | Đánh giá chuyến đi sau khi đi + gợi ý "chuyến tương tự" | Cho AI dữ liệu thật để lần sau gợi ý đúng hơn |
| **P4 Bonus** | Video YouTube giới thiệu địa điểm gắn kèm lịch trình | Chỉ làm khi các mục trên đã ổn |
| **P4 Bonus** | Xuất lịch trình ra PDF | Cách rẻ nhất: print stylesheet + `window.print()` |
| **P4 Bonus** | Streaming phản hồi chat (Server-Sent Events - SSE) | Text hiện dần từng chữ thay vì chờ đợi toàn bộ request |
| **Roadmap (chỉ nêu trong slide)** | NPC tự tạo / upload ảnh NPC / NPC nhiều cử động, gói trả phí | Mục 15. Có thể làm **màn hình mô phỏng** "Upgrade" để minh họa mô hình kinh doanh |

> **Giữ nguyên từ v1**: với làm một mình trong 8 tuần, **6 mục P0 là mục tiêu MVP thực tế**. Mọi thứ từ P1 trở đi chỉ bắt đầu sau khi MVP hoàn chỉnh (Gate A, mục 5.2).

### 2.2. Ngoài phạm vi (Out of Scope) của v2

| Hạng mục | Lý do loại |
| --- | --- |
| Đăng nhập bằng SĐT / OTP | Mỗi tin nhắn OTP tốn tiền; đồ án cá nhân không nên gánh khoản này. Google + username đã đủ |
| Thanh toán thật (VNPay/MoMo/Stripe) | Không cần cho đồ án; chỉ mô phỏng màn hình nâng gói nếu còn thời gian |
| Upload ảnh NPC, NPC tự tạo | Cần kiểm duyệt ảnh và lọc nội dung người dùng đưa vào prompt (mục 15) |
| Tự động gộp tài khoản Google ↔ username | Dễ sinh lỗ hổng chiếm tài khoản; để sau |
| Đồng chỉnh sửa realtime theo nhóm | Phức tạp; link chỉ xem + "Copy to my trips" đã đáp ứng nhu cầu chính |

### 2.3. Hai chiều phân loại người dùng

Có **hai trục độc lập**, đừng nhầm lẫn:

| | **Type 1** (dưới 3 chuyến hoàn tất) | **Type 2** (từ 3 chuyến hoàn tất trở lên) |
| --- | --- | --- |
| **Khách** (không đăng nhập) | Smart Start + Skip. Sau khi có lịch trình: thẻ nhắc lưu vào tài khoản (không chặn) | Gợi ý "Giống lần trước?", AI nhớ theo `session_id`. Nhắc nhẹ: "Đăng ký để AI nhớ bạn trên mọi thiết bị" |
| **Thành viên** (đã đăng nhập) | Smart Start + Skip. Không còn nhắc đăng ký | Đầy đủ: lời chào theo mẫu, Travel Profile, AI bỏ qua câu hỏi đã biết đáp án |

**Quy tắc xác định Type** (tính khi cần, không cần bảng riêng): đếm số chuyến `status = 'completed'` thuộc về người dùng đó (theo `user_id` nếu đã đăng nhập, theo `session_id` nếu là khách). `< 3` là Type 1, `>= 3` là Type 2. Ngưỡng nằm trong hằng số `TIER2_MIN_COMPLETED_TRIPS` để chỉnh khi demo.

---
