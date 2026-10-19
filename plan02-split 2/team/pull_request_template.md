<!-- Đặt file này ở .github/pull_request_template.md -->

## Việc này làm gì?
<!-- 1–2 câu. Liên kết mục plan02 (ví dụ §3.4.2, P1 My Trips). -->

## Loại thay đổi
- [ ] Tính năng mới (P0 / P1 / P2 / P3)
- [ ] Sửa lỗi
- [ ] Tài liệu / cấu hình

## Có đổi hợp đồng không?
- [ ] Không
- [ ] Có đổi API: đã sửa `team/openapi.yaml` trong PR này, đã nhờ An/Hưng (BE/AI) và Hạo (FE) duyệt
- [ ] Có đổi DB: đã báo Lam (chủ `schema.sql`) và đã sửa `docs/12-1`
- [ ] Có đổi prompt / tool schema: đã báo Hưng

## Cách kiểm tra
<!-- Các bước để người duyệt chạy lại. Có ảnh/video nếu là UI. -->

## Checklist
- [ ] Chạy được trên máy khác, không phụ thuộc cấu hình riêng
- [ ] Có test hoặc đã nhờ Đạt (QA) kiểm tra (bao gồm contract test nếu đổi API)
- [ ] CI xanh (lint, spectral, contract test)
- [ ] Không commit `.env` / API key
- [ ] Tài liệu trong `docs/` đã cập nhật nếu cần
