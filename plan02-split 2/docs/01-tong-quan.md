# 📋 Project Plan: Website AI Lên Lịch Trình Du Lịch TP.HCM

Oct 18, 2026 · @Hưng · **v2** (merge `plan01.md` + tài khoản, người dùng thích ứng, My Trips & chia sẻ link)

> **Changelog v1 → v2**
>
> | Thay đổi | Mục |
> | --- | --- |
> | Thêm hệ thống **tài khoản**: username + password (lưu DB), Google Sign-In, quên/đặt lại mật khẩu. **Bỏ** đăng nhập bằng SĐT/OTP. **Giữ** chế độ khách (không cần tài khoản) | 1, 2, 3.4, 3.5, 7 |
> | Thêm **2 loại người dùng theo hành vi** (Type 1 / Type 2) và **lớp AI thích ứng** (Smart Start, lời chào theo mẫu, hồ sơ du lịch có thể xem/sửa/xóa) | 1.2, 2.3, 4.7 |
> | Thêm **My Trips**, **chia sẻ lịch trình bằng link**, **nút "Skip" ở mọi bước dẫn dắt** (không cần tài khoản) | 2, 3.4, 10 |
> | Thêm **Settings** (persona preset, mức chủ động của AI, hồ sơ du lịch, quyền riêng tư) | 2, 10.11 |
> | **Phân lại mức ưu tiên** theo P0–P4 + "Roadmap"; thêm **Gate** và **Cut line** để MVP không bị đe dọa | 2.1, 5 |
> | NPC tùy biến (tự tạo, upload ảnh, nhiều cử động) và **gói trả phí** chuyển sang **Roadmap** (chỉ nêu trong slide) | 15 |
> | Thêm schema, API, prompt, test case, rủi ro, code mẫu tương ứng; sửa lỗi định dạng link trong code mẫu của v1 | 3–13 |

---

## 1. Tổng quan

- **Tên project**: Website AI lên lịch trình du lịch TP.HCM
- **Loại**: Dự án cá nhân, nộp cho giảng viên (thuyết trình + demo bằng tiếng Anh)
- **Mô tả**: Người dùng nhập thông tin chuyến đi (điểm xuất phát, ngân sách, số ngày, sở thích ăn uống/tham quan), AI sẽ sinh lịch trình chi tiết theo ngày, kết hợp dữ liệu địa điểm/ẩm thực thật tại TP.HCM. Điểm nhấn: AI **chủ động** đặt câu hỏi và điều chỉnh lịch trình theo phản hồi của người dùng qua chat, thay vì chỉ trả lời một chiều. **Khách dùng được ngay không cần tài khoản**; người dùng có thể đăng ký sau để giữ chuyến đi trên nhiều thiết bị, và AI sẽ **thích ứng** theo mức độ quen thuộc của họ (người mới được dẫn dắt, người dùng nhiều được phục vụ nhanh và đúng ý hơn).
- **Đối tượng người dùng**: Người có nhu cầu đi du lịch TP.HCM nhanh, không muốn tự tìm kiếm qua nhiều nguồn/mạng xã hội; thường đi theo nhóm nên cần gửi lịch trình cho người khác
- **Mục đích**: Học kỹ năng software engineering + AI integration; nộp làm đồ án cá nhân
- **Thời gian thực hiện**: 8 tuần, làm một mình
- **Ngôn ngữ giao diện**: Tiếng Anh (phù hợp thuyết trình + demo bằng tiếng Anh)

### 1.1. Mục tiêu Kỹ thuật & Giá trị Cốt lõi (Engineering & Product Goals)

| Trụ cột | Vấn đề của các công cụ hiện tại | Giải pháp của HCMC Travel AI | Chỉ số đo lường (Metrics) |
| --- | --- | --- | --- |
| **Chống "Ảo giác" (Anti-Hallucination)** | ChatGPT/LLM thông thường hay bịa địa chỉ, gợi ý quán đã đóng cửa hoặc xếp lịch đi ngược đường | Kết hợp Gemini Function Calling với dữ liệu xác thực từ Google Places API (New) và bộ lọc giờ mở cửa thực tế | 100% địa điểm trong lịch trình có `place_id_google` thật, tọa độ thật và `rating >= 4.0` |
| **Tương tác Chủ động (Proactive Agent)** | Người dùng không biết phải viết prompt dài như thế nào; AI chỉ trả lời một chiều | Cơ chế Human-in-the-loop qua tool `askPreference` kết hợp Quick Reply Chips và Natural Sentence Mode | Giảm thời gian onboarding xuống < 15 giây; tối đa 1 câu hỏi làm rõ trước khi sinh lịch trình |
| **Tối ưu Chi phí & Độ trễ (Cost & Latency)** | Gọi Google Places và LLM tuần tự gây tốn quota (dễ chạm mốc 15 RPM) và chờ đợi lâu (>20s) | Gộp Batch `searchPlaces(queries[])` kèm `X-Goog-FieldMask` + PostgreSQL `places_cache` (TTL 7 ngày) + Toán học Haversine | Giảm >75% request Google Places lặp lại; tối đa 2 Gemini calls/lượt chat; thời gian phản hồi < 6s (cache hit) |
| **Tính Khả thi & Độ tin cậy khi Demo** | Rớt mạng hội trường, Render free tier ngủ đông sau 15 phút, hoặc hết quota API ngay lúc chấm điểm | Kiến trúc đồng nhất PostgreSQL 16 (Docker Local ↔ Supabase Cloud) + Script `seedDemoCache.js` + `seedDemoUser.js` + Fallback Mode | 100% uptime khi thuyết trình (online lẫn offline qua Docker Compose) |
| **Liền mạch & Giữ chân (Continuity)** *(mới)* | Dùng xong là mất: đổi máy/xóa dữ liệu trình duyệt thì hết lịch trình; không gửi được cho bạn đi cùng | Chế độ khách + **My Trips** + **chia sẻ link** + tài khoản tùy chọn; chuyến đi của khách **được giữ nguyên** khi đăng ký/đăng nhập (claim) | 100% chuyến đi của khách còn nguyên sau khi đăng ký; tạo link chia sẻ ≤ 2 thao tác |
| **AI Thích ứng (Adaptive AI)** *(mới)* | Mọi người dùng nhận cùng một trải nghiệm: người mới bị bỏ rơi, người quen bị hỏi lại những điều đã biết | Phân loại **theo hành vi** (Type 1 / Type 2): Type 1 được AI dẫn dắt và tự điền form; Type 2 được AI nhớ sở thích để bớt câu hỏi và đoán đúng ý hơn | Type 1: từ lúc mở web đến form đã điền sẵn < 15s, luôn có nút Skip. Type 2: số câu hỏi làm rõ giảm còn 0–1 |
| **Kiểm soát & Riêng tư (User Control)** *(mới)* | AI "nhớ" nhưng người dùng không biết nó nhớ gì, không xóa được | **Travel Profile** hiển thị rõ AI đang nhớ gì; sửa từng mục, xóa toàn bộ, tắt trí nhớ; mật khẩu chỉ lưu dạng hash | 100% mục trong profile xem/sửa/xóa được; không lưu mật khẩu dạng rõ |

### 1.2. Nguyên tắc sản phẩm (Product Principles)

Sáu nguyên tắc này dùng để quyết định mọi tranh luận về tính năng mới:

| # | Nguyên tắc | Nghĩa là gì trong project này |
| --- | --- | --- |
| 1 | **Giá trị trước, tài khoản sau** | Khách vào là dùng được ngay. Không bao giờ chặn luồng chính bằng đăng ký/đăng nhập. Chỉ nhắc đăng ký **sau khi** đã có lịch trình ("Save this trip to your account?") |
| 2 | **Skip ở mọi nơi** | Mọi bước dẫn dắt của AI (Smart Start, lời chào, câu hỏi làm rõ, nhắc đăng ký) đều có nút bỏ qua. Có tùy chọn tắt hẳn trong Settings |
| 3 | **Phân loại theo hành vi, không theo tài khoản** | Khách dùng nhiều vẫn là Type 2. Người mới đăng ký vẫn là Type 1 |
| 4 | **Cá nhân hóa phải có ích, không phải để dễ thương** | AI nhớ sở thích để **làm đúng ý hơn** (ít hỏi hơn, đoán đúng hơn). Trò chuyện phiếm chỉ ở mức một câu ngắn, liên quan chuyến đi, tắt được |
| 5 | **Người dùng nắm quyền với trí nhớ của AI** | Xem, sửa, xóa, tắt. Không lưu thứ người dùng không nhìn thấy được |
| 6 | **MVP luôn được bảo vệ** | Mọi tính năng mới chỉ làm sau khi MVP qua Gate A, và bị cắt trước nếu trễ (xem mục 5.2) |

---
