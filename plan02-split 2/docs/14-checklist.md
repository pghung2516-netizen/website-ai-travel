## 14. Checklist tài liệu nộp bài / thuyết trình

- [ ] Slide tiếng Anh (vấn đề → giải pháp → kiến trúc → **UI/UX showcase** → demo → **Roadmap (mục 15)** → kết luận)
- [ ] Video demo backup (quay màn hình full flow + voice tiếng Anh, đề phòng mạng trường chập chờn)
- [ ] Bản deploy hoạt động trên Vercel + Render (kèm link live trong slide)
- [ ] Source code trên GitHub, kèm README tiếng Anh (hướng dẫn chạy local + ảnh chụp kiến trúc + link demo)
- [ ] Kịch bản demo viết sẵn từng câu tiếng Anh (tránh bị khớp khi nói trực tiếp)
- [ ] Chạy script `seedDemoCache.js` trước buổi thuyết trình để nạp sẵn dữ liệu địa điểm vào `places_cache`
- [ ] *(P3)* Chạy `seedDemoUser.js` (sau `seedDemoCache.js`) và thử đăng nhập `demo_traveler` để thấy lời chào Type 2
- [ ] Mở link web trước 2-3 phút để Render free tier "thức dậy" trước khi vào phòng
- [ ] Test demo flow ít nhất 2 lần trước ngày thuyết trình (kiểm tra API còn quota, link deploy còn hoạt động)
- [ ] Test trên mobile thật (hoặc Chrome DevTools mobile emulator) để tự tin khoe tính năng responsive
- [ ] *(P1)* Test mở link chia sẻ trong **cửa sổ ẩn danh**, rồi thu hồi link và kiểm tra link cũ báo 404
- [ ] *(P2)* Test luồng: khách tạo chuyến → đăng ký → chuyến còn nguyên (`claimed_trips` đúng)
- [ ] *(P2)* Demo offline chỉ dùng **username + tài khoản seed**, không dùng Google / quên mật khẩu
- [ ] *(P2)* Vercel URL đã nằm trong **Authorized JavaScript origins** của OAuth Client (nếu có Google)
- [ ] Đã **feature freeze** từ 02/12; chỉ sửa lỗi
- [ ] Không để lộ `DEMO_USER_PASSWORD`, `JWT_SECRET`, `SMTP_PASS` trong slide, video hay ảnh chụp màn hình

---
