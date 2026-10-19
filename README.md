# 🗺️ Website AI Lên Lịch Trình Du Lịch TP.HCM (AI Travel Planner)

[![Template Repository](https://img.shields.io/badge/GitHub-Template%20Repository-blue.svg)](https://github.com/pghung2516-netizen/website-ai-travel)
[![Public Repository](https://img.shields.io/badge/Visibility-Public-green.svg)](https://github.com/pghung2516-netizen/website-ai-travel)

> **Dự án**: Website AI tự động lập lịch trình du lịch thông minh tại TP.HCM kết hợp Gemini AI và Google Places API.  
> **Template**: Repository này được cấu hình làm **Template Repository** (Public) để mọi người có thể nhấn nút **"Use this template"** và khởi tạo dự án mới dễ dàng.

---

## 📌 Tổng quan dự án

Website hỗ trợ người dùng tạo lịch trình du lịch chi tiết tại TP.HCM theo ngân sách, sở thích ẩm thực/tham quan và thời gian thực tế:
- **AI Đàm thoại & Đề xuất**: Tự động tương tác hỏi - đáp và tinh chỉnh lịch trình linh hoạt theo phản hồi.
- **Dữ liệu thực tế**: Tích hợp Google Places API (New) để lấy địa điểm, hình ảnh, đánh giá và tuyến đường.
- **Hệ thống thích ứng**: Nhận biết người dùng mới (dẫn dắt chi tiết) và người dùng quen thuộc (thao tác nhanh chóng, gợi ý cá nhân hóa).
- **Chia sẻ & Lưu trữ**: Hỗ trợ chế độ khách (Guest) dùng ngay không cần đăng nhập, My Trips và chia sẻ lịch trình qua link.

---

## 📁 Cấu trúc thư mục

```text
.
├── .githooks/
│   ├── pre-commit                    # Git hook chặn commit nhầm .codegraph
│   └── pre-push                      # Git hook chặn push nhầm .codegraph
├── .gitignore                        # Cấu hình bỏ qua .codegraph, .DS_Store, v.v.
├── De_Xuat_Cai_Thien_Project_Plan.md # Bản phân tích và đề xuất cải thiện
├── plan02-split 2/                   # Bộ tài liệu kiến trúc & kế hoạch chi tiết
│   ├── AGENTS.md                     # Hướng dẫn quy tắc dành cho AI Agent
│   ├── huong_dan_prompt.md           # Hướng dẫn prompt cho các giai đoạn
│   ├── plan02-full.md                # Toàn bộ nội dung kế hoạch v2
│   ├── docs/                         # Tài liệu chi tiết từ mục 01 - 15
│   └── team/                         # Phân công nhóm, OpenAPI spec, PR template
└── README.md
```

---

## 🛡️ Cơ chế bảo vệ chống push nhầm `.codegraph`

Dự án được trang bị **nhiều lớp bảo vệ tự động** nhằm đảm bảo thư mục `.codegraph` không bao giờ bị đẩy lên GitHub:

1. **Lớp 1 - `.gitignore`**: Khai báo loại trừ `.codegraph/`, `.codegraph/**`, `*.codegraph*`.
2. **Lớp 2 - Git Pre-commit Hook (`.githooks/pre-commit`)**: Tự động kiểm tra trước mỗi lần `git commit`. Nếu phát hiện bất kỳ file nào thuộc `.codegraph` đang ở staging (kể cả khi dùng `git add -f`), commit sẽ ngay lập tức bị huỷ bỏ với thông báo lỗi.
3. **Lớp 3 - Git Pre-push Hook (`.githooks/pre-push`)**: Kiểm tra toàn bộ commit trước khi đẩy lên remote. Nếu phát hiện bất kỳ file nào thuộc `.codegraph`, lệnh `git push` sẽ lập tức bị chặn.
4. **Lớp 4 - `.git/info/exclude`**: Đăng ký loại trừ trực tiếp ở cấp độ repository local.

> 💡 **Kích hoạt Hooks khi clone dự án**:  
> Sau khi clone hoặc tạo repo từ template, bạn chỉ cần chạy lệnh sau để kích hoạt toàn bộ hook bảo vệ:
> ```bash
> git config core.hooksPath .githooks
> ```

---

## 🚀 Cách sử dụng Repository làm Template

1. Bấm vào nút màu xanh lá **"Use this template"** ở đầu trang repository trên GitHub.
2. Chọn **"Create a new repository"**.
3. Điền tên repository mới của bạn và bắt đầu phát triển!

---

## 👥 Tài liệu nhóm & Hướng dẫn đóng góp

- **Hợp đồng API**: [`plan02-split 2/team/openapi.yaml`](plan02-split%202/team/openapi.yaml)
- **Phân công & Quy tắc làm việc**: [`plan02-split 2/team/TEAM_EDITION.md`](plan02-split%202/team/TEAM_EDITION.md)
- **Hướng dẫn cho AI Agents**: [`plan02-split 2/AGENTS.md`](plan02-split%202/AGENTS.md)
