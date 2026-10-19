## 10. UI/UX Design

### 10.1. Design System

#### Bảng màu (Color Palette)

| Token | Light Mode | Dark Mode | Sử dụng |
| --- | --- | --- | --- |
| Primary | `#2563EB` (Blue 600) | `#60A5FA` (Blue 400) | Nút chính, link, focus ring, user chat bubble |
| Accent | `#F97316` (Orange 500) | `#FB923C` (Orange 400) | CTA nổi bật ("Plan My Trip"), highlight giá tiền |
| Background | `#FFFFFF` | `#111827` (Gray 900) | Nền trang |
| Surface | `#F9FAFB` (Gray 50) | `#1F2937` (Gray 800) | Nền card, panel |
| AI Bubble | `#EEF2FF` (Indigo 50) | `#1E293B` (Slate 800) | Nền tin nhắn AI |
| Text Primary | `#111827` (Gray 900) | `#F9FAFB` (Gray 50) | Tiêu đề, nội dung chính |
| Text Secondary | `#6B7280` (Gray 500) | `#9CA3AF` (Gray 400) | Mô tả phụ, timestamp |
| Border | `#E5E7EB` (Gray 200) | `#374151` (Gray 700) | Viền card, divider |
| Success | `#10B981` (Emerald 500) | `#34D399` (Emerald 400) | Under budget, thành công |
| Warning | `#F59E0B` (Amber 500) | `#FBBF24` (Amber 400) | Gần hết budget, cảnh báo |
| Error | `#EF4444` (Red 500) | `#F87171` (Red 400) | Lỗi, over budget |

> **Ý đồ thiết kế**: Blue gợi sự tin tưởng + khám phá, Orange mang hơi ấm Việt Nam. Tổ hợp tương tự Airbnb (coral + neutral) nhưng chuyên nghiệp hơn cho AI tool.

#### Category Colors (dùng cho timeline)

| Category | Màu | Icon |
| --- | --- | --- |
| 🍜 Food/Restaurant | `#F59E0B` (Amber) | 🍜 |
| 🏛️ Sightseeing | `#3B82F6` (Blue) | 🏛️ |
| ☕ Café / Rest | `#8B5CF6` (Purple) | ☕ |
| 🛍️ Shopping | `#EC4899` (Pink) | 🛍️ |
| 🌿 Nature / Park | `#10B981` (Green) | 🌿 |
| 🎭 Entertainment | `#F97316` (Orange) | 🎭 |

#### Typography

| Thành phần | Font | Weight | Size | Ghi chú |
| --- | --- | --- | --- | --- |
| Heading H1 | Inter | 700 (Bold) | 2rem (32px) | Tiêu đề trang |
| Heading H2 | Inter | 700 | 1.5rem (24px) | Tiêu đề section |
| Heading H3 | Inter | 600 (Semi) | 1.25rem (20px) | Tiêu đề card |
| Body | Inter | 400 (Regular) | 1rem (16px) | Nội dung, `line-height: 1.6` |
| Chat message | Inter | 400 | 0.9375rem (15px) | Nhỏ hơn body, tight hơn trong chat |
| Caption / Label | Inter | 500 (Medium) | 0.75rem (12px) | Timestamp, label phụ |
| Cost / Data | JetBrains Mono | 500 | 0.875rem (14px) | Giá tiền VND, rating |

> Font Inter miễn phí trên Google Fonts, đọc tốt trên mọi kích thước, là lựa chọn hàng đầu cho UI hiện đại.

#### Spacing & Border Radius

```
Spacing scale (Tailwind default):
  4px (p-1) → 8px (p-2) → 12px (p-3) → 16px (p-4) → 24px (p-6) → 32px (p-8)

Border radius:
  Card:          12px (rounded-xl)
  Chat bubble:   18px (rounded-2xl)
  Button:        8px  (rounded-lg)
  Input field:   8px  (rounded-lg)
  Tag/Chip:      20px (rounded-full)
  Avatar:        50%  (rounded-full)

Box shadow (card):
  Default:  0 1px 3px rgba(0,0,0,0.1)
  Hover:    0 4px 6px rgba(0,0,0,0.1)   + translateY(-2px)
  Active:   0 1px 2px rgba(0,0,0,0.1)
```
