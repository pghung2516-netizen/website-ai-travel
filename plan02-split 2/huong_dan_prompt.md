# Hướng dẫn prompt cho Agent — đọc file nào khi làm việc gì

Hãy nghĩ plan02 như một thư viện: `AGENTS.md` là quầy hướng dẫn, mỗi file trong `docs/` là một kệ sách. Agent chỉ cần đứng đúng kệ của việc đang làm.

## 1. Vai trò của từng nhóm file

| Nhóm | File | Vai trò | Cỡ |
|---|---|---|---|
| **Bản đồ** | `AGENTS.md` | Quy tắc chung và chỉ đường tới file khác. Luôn đọc đầu tiên | ~1 KB |
| **Bối cảnh sản phẩm** | `01`, `02` | Vì sao làm, làm gì, không làm gì, thứ tự P0–P4 | 7 + 7 KB |
| **Hợp đồng kỹ thuật** | `03` | Stack, cây thư mục, **API**, **schema DB**. Là điểm nối giữa FE và BE | 28 KB |
| **AI** | `04` | Prompt, tool schema, luồng chat, Smart Start, Type 1/2 | 22 KB |
| **Giao diện** | `10a`, `10b`, `10c` | Màu/font, wireframe từng màn hình, chat/animation/Settings | 2, 33, 18 KB |
| **Quy tắc chất lượng** | `06`, `07`, `08` | Mã lỗi, bảo mật, test | 4, 5, 7 KB |
| **Code mẫu** | `12-1` … `12-8` | Mỗi file một module: DB, tính cước, Places, trips, seed, danh tính, auth, lớp thích ứng AI | 1–7 KB |
| **Vận hành** | `05`, `09`, `11` | Timeline, deploy/Docker, rủi ro | 7, 6, 3 KB |
| **Trình bày** | `13`, `14`, `15` | Kịch bản demo, checklist, roadmap. Gần như không dùng khi code | 6, 1, 2 KB |

## 2. Cần sửa gì thì đọc chính xác file nào

Luôn đọc `AGENTS.md` trước, rồi đọc thêm:

| Việc | Đọc thêm | Tổng (ước lượng) | **Không đọc** |
|---|---|---|---|
| **FE: dựng/sửa một màn hình** | `10a` + **đúng màn hình đó** trong `10b` + `03` mục §3.4 (API để gọi) | ~10 KB | `04`, `07`, `09`, `11`, tất cả `12-*`, `13–15` |
| **FE: chat, animation, Settings, dark mode** | `10c` + `03` §3.4 | ~22 KB | như trên |
| **BE: viết endpoint, sửa DB** | `03` §3.4–3.5 + `12-1` + `12-4` + `06` | ~20 KB | `10*`, `04`, `13–15` |
| **BE: Places, tính cước** | `12-3`, `12-2` (+ `06`) | ~10 KB | gần như mọi thứ khác |
| **Security / Auth** | `07` + `12-6` + `12-7` + `03` §3.4 (phần danh tính, auth, share) | ~15 KB | `10*`, `04`, `05`, `13–15` |
| **AI: chat, tool, prompt** | `04` + `12-8` + `12-3` | ~33 KB | `10*`, `07`, `13–15` |
| **Test** | `08` + file code của module cần test | ~10 KB | còn lại |
| **Deploy, Docker** | `09` + `12-5` | ~13 KB | còn lại |

## 3. Hai file lớn nên đọc một phần

- `10b` (33 KB) là wireframe của 9 màn hình. Bảo Agent: "chỉ đọc mục **Màn hình N**" (tìm theo tiêu đề rồi đọc đoạn đó), đừng nạp cả file.
- `03` (28 KB) trộn cả FE và BE. FE chỉ cần §3.4 (API). BE cần thêm §3.5 (schema). Cũng đọc theo mục.

## 4. Cách ra lệnh cho Agent để nó tiết kiệm

Ghi rõ file trong câu lệnh. Nếu không ghi, Agent sẽ tự đoán và có thể mở nhiều file hơn mức cần.

Ví dụ:

> Làm màn hình My Trips (P1). Đọc `AGENTS.md`, `10a`, mục Màn hình 7 trong `10b`, và §3.4 trong `03`. Không đọc file khác trừ khi thiếu thông tin.

Mức tiết kiệm: một việc FE hay BE thường chỉ cần 10–20 KB, tức khoảng 5–10% bản đầy đủ 213 KB. Đây là ước lượng gần đúng, không đo chính xác theo token.
