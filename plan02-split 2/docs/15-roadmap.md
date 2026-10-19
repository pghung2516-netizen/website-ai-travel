## 15. Roadmap — Hướng phát triển (chỉ nêu trong slide, không làm trong v2)

Nói được rằng bạn đã nghĩ đến và đã chừa chỗ trong thiết kế là đủ. Không cần làm.

### 15.1. NPC nâng cao

| Ý tưởng | Giá trị thật cho người dùng | Chi phí / rủi ro | Hướng làm đúng khi có thời gian |
| --- | --- | --- | --- |
| NPC tự tạo (tên, tính cách) | Vừa phải: tạo cảm giác đồng hành | Chuỗi người dùng gõ vào prompt dễ bị injection | Tên theo regex `^[\p{L} ]{1,20}$` + tính cách chọn từ danh sách; không bao giờ đưa văn bản tự do vào prompt |
| Upload ảnh NPC | Thấp: chủ yếu làm đẹp | Ảnh không phù hợp, ảnh người thật; cần lưu trữ file | Moderation API + giới hạn dung lượng/định dạng + Supabase Storage; chỉ chạy khi đã kiểm duyệt xong |
| NPC nhiều cử động | Thấp nhất | Tốn công nhất, giá trị demo thấp | Animation dựng sẵn (Lottie/Rive) theo trạng thái: đang nghĩ, vui, gợi ý |

### 15.2. Gói trả phí (ý tưởng)

Trả tiền phải đổi lấy **chuyến đi tốt hơn**, không chỉ nhân vật đẹp hơn. NPC tùy biến chỉ là phần thưởng đi kèm.

| Tính năng | Free | Plus (ý tưởng) |
| --- | --- | --- |
| Số chuyến lưu | Giới hạn (ví dụ 5) | Không giới hạn |
| Chia sẻ | Link chỉ xem | Nhóm cùng góp ý / chỉnh sửa |
| Xuất lịch trình | In / PDF cơ bản | PDF đẹp, gói offline, xuất lịch `.ics` |
| Trí nhớ của AI | Hồ sơ cơ bản | Hồ sơ dài hạn, nhiều chi tiết hơn |
| NPC | Preset | Tên và ảnh tùy biến, nhiều cử động |

> **Thanh toán thật nằm ngoài phạm vi.** Nếu muốn minh họa mô hình kinh doanh, chỉ làm **một màn hình "Upgrade" mô phỏng**, ghi rõ *"Demo only — no real payment"*, không thu bất kỳ thông tin thẻ nào.

### 15.3. Các hướng khác

| Hướng | Ghi chú |
| --- | --- |
| Xác minh email, liên kết tài khoản Google ↔ username | Chỉ liên kết khi email đã xác minh, tránh chiếm tài khoản |
| Chuyển JWT sang cookie `HttpOnly` cùng domain | An toàn hơn `localStorage` cho sản phẩm thật |
| Đồng chỉnh sửa theo nhóm, nhắc lịch, xuất lịch `.ics` | Mở rộng từ chia sẻ link |
| Vòng phản hồi sau chuyến đi | Đánh giá sau khi đi để AI học sở thích chính xác hơn (P4) |
| Mở rộng ngoài TP.HCM | Cần dữ liệu và luật địa phương cho từng thành phố |
