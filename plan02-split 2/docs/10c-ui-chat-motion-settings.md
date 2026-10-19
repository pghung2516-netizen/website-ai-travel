### 10.3. AI Chat UX — Khiến AI cảm thấy "sống"

#### AI Persona

| Thuộc tính | Chi tiết |
| --- | --- |
| Tên | **Sài Gòn Guide** (mặc định; đổi được bằng preset ở bên dưới) |
| Avatar | 🧭 Icon la bàn hoặc custom SVG (xe máy cách điệu) |
| Giọng điệu | Thân thiện, nhiệt tình nhưng không quá casual. "Great choice! Ben Thanh Market is amazing for street food!" thay vì "Acknowledged. Ben Thanh Market has been added." |
| Ngôn ngữ | Phản hồi cùng ngôn ngữ user dùng (Việt hoặc Anh) |

#### Persona preset (P3): hai lựa chọn độc lập

Persona **chỉ là lớp vỏ**: không đổi luật, không đổi dữ liệu địa điểm. Tách *avatar + tên* khỏi *giọng điệu* để giọng điệu không bị gắn với giới tính của nhân vật. Mọi giá trị đều đi qua **whitelist** (mục 4.7.4).

| ID avatar | Avatar | Tên hiển thị | Ghi chú |
| --- | --- | --- | --- |
| `saigon_guide` | 🧭 | Sài Gòn Guide | Mặc định, trung tính |
| `anh_hai` | 🛵 | Anh Hai | Hình tượng nam |
| `chi_ba` | 🌸 | Chị Ba | Hình tượng nữ |
| `ban_tu` | 🌿 | Bạn Tư | Hình tượng trung tính / phi nhị giới |
| `rainbow_guide` | 🏳️‍🌈 | Rainbow Guide | Pride-friendly |

| ID giọng điệu | Mô tả (được đưa vào prompt) |
| --- | --- |
| `friendly` *(mặc định)* | Warm, enthusiastic local insider, light use of emojis |
| `concise` | Short sentences, minimal emojis, straight to the point |
| `playful` | Light humour, a touch of Saigon slang used sparingly |

> Tên tự đặt và ảnh tự tải lên **không** nằm trong v2 (xem Roadmap, mục 15) vì phải lọc nội dung và chống injection.

#### Chat UI Patterns

| Thành phần | Specification |
| --- | --- |
| **AI bubble** | Left-aligned, nền `AI Bubble` color, `border-radius: 18px 18px 18px 4px`, `max-width: 75%` |
| **User bubble** | Right-aligned, nền `Primary` color + text trắng, `border-radius: 18px 18px 4px 18px`, `max-width: 75%` |
| **Typing indicator** | 3 dots bouncing animation (400ms delay giữa mỗi dot). CSS: `animation: bounce 1.4s infinite ease-in-out` |
| **Quick reply chips** | Hiện dưới AI message khi AI hỏi preference. Rounded full (`border-radius: 20px`), outlined Primary color, `padding: 8px 16px`. Click → gửi như user message + chips biến mất |
| **Timestamp** | Hiện giữa 2 nhóm message cách nhau >2 phút. Format: "just now", "2 min ago". Font: Caption size, Text Secondary color |
| **Auto-scroll** | Luôn scroll xuống tin mới nhất. NẾU user cuộn lên đọc history → tạm dừng auto-scroll + hiện nút "↓ New messages" ở bottom |
| **Input area** | `textarea` auto-expand (max 4 dòng). Send button disable khi input rỗng. Enter gửi, Shift+Enter xuống dòng |
| **Confirmation** | Trước khi đổi lịch trình lớn, AI gửi: "I'll swap X for Y on Day 2. Sound good?" + chips [✅ Yes] [❌ No, keep it] |
| **Nêu giả định (Type 2)** | Khi dùng Travel Profile, AI nói **một câu** nêu giả định ("I'll keep it relaxed like last time. Say 'mix it up' to change.") để người dùng sửa được ngay |

### 10.4. Itinerary Visualization

#### Vertical Timeline Design

```
Timeline structure:

  ● ── Node (circle 12px, màu theo category)
  │     Line (2px solid Border color)
  │
  ├── Activity Card ──────────────────────────────┐
  │   │ Place Name (H3, bold)           [📸 thumb] │
  │   │ ⭐ 4.6 · ₫40,000 · 9:00 - 11:00          │
  │   │ Brief description from AI                   │
  │   │ 📍 Address                      [▼ Details] │  ← Click mở rộng
  │   └─────────────────────────────────────────────┘
  │
  │   ↕ Travel: ~15 min by taxi (₫30,000)           ← Khoảng cách giữa 2 điểm
  │
  ● ── Next activity
```

#### Card mở rộng (Expanded State)

```
┌────────────────────────────────────────────────┐
│ 🏛️ War Remnants Museum               [📸 ảnh] │
│ ⭐ 4.6 · ₫40,000 · 9:00 - 11:00               │
│ Explore Vietnam War history through exhibits   │
│                                                │
│ ── Expanded details ──────────────────────────│  ← Animate mở (framer-motion)
│ 📍 28 Vo Van Tan, District 3                   │
│ 🕐 Open: 7:30 - 17:00 daily                   │
│ 📞 +84 28 3930 5587                            │
│ 💬 "Must-visit place..." — Google Review       │
│                                                │
│ [🗺️ Open in Maps]  [🔄 Swap this place]       │  ← Action buttons
│                                        [▲ Less]│
└────────────────────────────────────────────────┘
```

#### Budget Summary Bar

```
Under budget (< 80%):
  ▓▓▓▓▓▓▓▓▓▓▓▓░░░░░░░░░░░░░░░░  ₫1.85M / ₫3M (62%)
  └── Success color (Green) ──┘

Near budget (80-100%):
  ▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓░░░  ₫2.7M / ₫3M (90%)
  └── Warning color (Amber) ──┘

Over budget (> 100%):
  ▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓ ₫3.5M / ₫3M (117%) ⚠️
  └── Error color (Red) ───────┘  + warning tooltip
```

#### Thanh hành động của chuyến đi (P1, mới)

| Nút | Tác dụng | Ghi chú |
| --- | --- | --- |
| 🔗 **Share** | Mở `ShareDialog`: tạo / sao chép / thu hồi link chỉ xem | Không cần tài khoản |
| ⧉ **Duplicate** | Nhân bản chuyến để thử biến thể | |
| 🖨️ **Print / PDF** *(P4)* | `window.print()` với print stylesheet | Cách rẻ nhất để có PDF |

### 10.5. Micro-interactions & Animations

| Interaction | Animation | Timing | Library |
| --- | --- | --- | --- |
| Button hover | `scale(1.02)` + shadow tăng | `0.2s ease-out` | CSS transition |
| Button press | `scale(0.98)` | `0.1s ease` | CSS transition |
| Card hover | `translateY(-2px)` + shadow tăng | `0.2s ease` | CSS transition |
| Input focus | Border color `Border → Primary` | `0.15s ease` | CSS transition |
| **Card ↔ Map Pin Sync** | Hover card: Pin trên bản đồ scale 1.3x & pulse.<br>Click pin: Timeline tự động scroll mượt đến card tương ứng + flash viền sáng | `0.25s cubic-bezier(0.34, 1.56, 0.64, 1)` | Google Maps API + CSS |
| Page transition (Form → Chat) | Fade + slide up: `opacity: 0, y: 20 → opacity: 1, y: 0` | `0.3s ease-out` | framer-motion |
| Chat message appear | Slide up + fade: `opacity: 0, y: 10 → opacity: 1, y: 0` | `0.25s ease-out` | framer-motion |
| Quick reply chips | Stagger appear: mỗi chip delay thêm 50ms | `0.2s ease` + `staggerChildren: 0.05` | framer-motion |
| Card expand/collapse | `AnimatePresence` height auto | `0.3s ease-in-out` | framer-motion |
| Card delete / swap | Trượt sang trái `-20px`, mờ dần và sụp chiều cao về 0 | `0.3s ease-out` | framer-motion |
| Itinerary complete! | Confetti burst (subtle, 1 giây) | — | canvas-confetti |
| Skeleton shimmer | Gradient chạy ngang | `1.5s infinite` | CSS animation |
| Scroll-triggered elements | Fade in + slide up khi vào viewport | `0.4s ease-out` | Intersection Observer |
| Typing indicator dots | Sequential bounce | `1.4s infinite, delay 0-0.16-0.32s` | CSS keyframes |
| *(mới)* Auth modal | Desktop: fade + scale `0.96 → 1`. Mobile: bottom sheet trượt lên | `0.25s ease-out` | framer-motion |
| *(mới)* Thẻ nhắc / lời chào | Slide-down nhẹ; khi bấm "Not now" thì gập chiều cao về 0 | `0.3s ease-out` | framer-motion |
| *(mới)* Copy link chia sẻ | Nút đổi thành "Copied ✓" kèm toast | `0.2s ease` | CSS + sonner |

> **Nguyên tắc**: Animation < 0.3s cho interaction trực tiếp (click, hover). Animation 0.3-0.5s cho transition giữa các trạng thái. Không animation > 0.5s (trừ loading indicator). Tôn trọng `prefers-reduced-motion`: tắt hiệu ứng trượt/confetti khi người dùng bật giảm chuyển động.

### 10.6. Responsive Design (Mobile-First)

#### Breakpoints (Tailwind)

| Breakpoint | Size | Layout |
| --- | --- | --- |
| Default (Mobile) | < 640px | Single column, bottom tab bar |
| `sm` | ≥ 640px | Vẫn single column, form rộng hơn |
| `md` | ≥ 768px | Tablet — có thể side-by-side nhưng hẹp |
| `lg` | ≥ 1024px | Desktop — split view Chat (40%) + Itinerary (60%) |
| `xl` | ≥ 1280px | Desktop rộng — thêm map panel nếu có |

#### Mobile-specific patterns

| Pattern | Chi tiết |
| --- | --- |
| **Bottom tab bar** | 2 tab: 💬 Chat / 📋 Plan (kèm badge số địa điểm `Day 1: 5 spots`). Fixed bottom, height 56px. Active tab = Primary underline |
| **Touch targets** | Minimum 44×44px cho mọi element tương tác (nút bấm, chip, tabs) |
| **Spacing** | Minimum 8px gap giữa các tappable elements |
| **Input bar** | Fixed bottom (trên tab bar), không bị virtual keyboard che trên iOS/Android (xử lý qua `visualViewport`) |
| **Swipe** | Swipe trái/phải để chuyển Day tabs trong itinerary |
| **Photo** | Hiển thị thumbnail tỷ lệ 16:9 rõ nét trên mobile (card chiếm full width) |
| **Quick replies** | Horizontal scroll mượt mà nếu có nhiều options (không bị gãy dòng) |
| *(mới)* **Header gọn** | `My Trips` / `Sign in` / `Settings` nằm trong menu avatar; header chỉ còn logo + theme |
| *(mới)* **Auth & Share** | Mở dạng bottom sheet full width; trường nhập đủ lớn để không bị zoom trên iOS (font ≥ 16px) |
| *(mới)* **My Trips** | Lưới 1 cột trên mobile, 2 cột từ `md`, 3 cột từ `xl` |

### 10.7. Accessibility (a11y)

| Yêu cầu | Implementation |
| --- | --- |
| **Semantic HTML** | Dùng `<main>`, `<nav>`, `<article>`, `<section>`, `<button>` (không `<div onClick>`), `<form>`, `<label>` |
| **Keyboard navigation** | Tab qua mọi interactive element. Enter/Space để click. Escape đóng modal/expanded card |
| **ARIA labels** | `aria-label="Send message"` cho icon button. `aria-live="polite"` trên chat area (screen reader đọc tin mới). `role="status"` trên typing indicator |
| **Color contrast** | Đạt WCAG AA (4.5:1 normal text, 3:1 large text). Palette đã chọn đạt chuẩn này |
| **Focus rings** | `focus:ring-2 focus:ring-blue-500 focus:ring-offset-2` (Tailwind) trên mọi interactive element |
| **Alt text** | Mọi ảnh từ Google Places có `alt="Photo of {place_name}"` |
| **Font sizing** | Dùng `rem` unit, respect browser font size settings |
| *(mới)* **Form tài khoản** | Mỗi ô có `<label>` thật; `autocomplete="username"`, `current-password`, `new-password` để trình quản lý mật khẩu hoạt động; nút 👁 Show có `aria-pressed`; lỗi hiện trong vùng `aria-live="assertive"` |
| *(mới)* **Modal** | Focus trap trong modal, trả focus về nút mở khi đóng, Escape đóng; `role="dialog"` + `aria-modal="true"` + `aria-labelledby` |
| *(mới)* **Avatar persona** | Emoji kèm `aria-label` mô tả ("Compass avatar"), không dùng màu làm tín hiệu duy nhất |

### 10.8. Gamification & Delight — Giữ chân người dùng

| Element | Mô tả | Khi nào hiện | Mức ưu tiên |
| --- | --- | --- | --- |
| **Progress indicator 4 bước** | "🔍 Searching spots..." → "📊 Checking hours..." → "🗺️ Optimizing route..." → "✨ Finalizing!" | Khi AI đang sinh lịch trình | P0 Bắt buộc |
| **Trip Readiness Score** | Vòng tròn % độ hoàn thiện (0% → 100% khi đủ hoạt động sáng/trưa/chiều/tối) | Trên đầu bảng lịch trình | **P4 Bonus** *(v1: Nên có)* |
| **"Passport Stamp" Celebration** | Hiệu ứng đóng dấu mộc cổ điển: *"SAIGON APPROVED • 2 DAYS"* kèm pháo hoa nhẹ (`canvas-confetti`) | Khi lịch trình đạt 100% hoặc hoàn tất | **P4 Bonus** *(v1: Nên có)* |
| **Trip summary card** | "Your 2-Day HCMC Adventure · ₫3M budget · 6 places · 4 meals" — thiết kế như vé máy bay / boarding pass | Đầu trang itinerary | P0 Bắt buộc |
| **Transit friction indicator** | Hiển thị rõ: `🚶 8 min walk (600m) via Le Loi` hoặc `🚕 10 min Grab (~35,000 VND)` giữa các điểm | Giữa 2 card địa điểm trên timeline | P0 Bắt buộc |
| **Budget tracking realtime** | Progress bar đổi màu xanh/vàng/đỏ khi user thêm/đổi/xóa địa điểm | Luôn hiển thị trên itinerary panel | P0 Bắt buộc |
| **"Swap this place"** | Nút trên mỗi card cho phép đổi nhanh — AI tìm quán/địa điểm tương đương | Mỗi activity card | P0 Bắt buộc |
| **Data transparency** | "Based on Google Reviews (⭐ 4.5)" dưới mỗi gợi ý | Trong activity card | P0 Bắt buộc |

> **Vì sao hạ hai mục gamification xuống P4:** chúng tạo thiện cảm khi demo nhưng không làm người dùng quay lại dùng lần hai. Phần giữ chân thật nằm ở **My Trips, chia sẻ link và AI nhớ sở thích** (P1 đến P3).

### 10.9. Dark Mode

| Khía cạnh | Chi tiết |
| --- | --- |
| **Cách làm** | Dùng Tailwind `dark:` variant. Store preference vào `localStorage` (khách) hoặc `users.settings.theme` (thành viên). Default theo system `prefers-color-scheme` |
| **Toggle** | Icon ☀️/🌙 trên header. Transition `0.2s ease` khi chuyển |
| **Mức ưu tiên** | **P4 Bonus** — làm sau MVP. Thiết kế CSS custom properties ngay từ đầu để bật dark mode nhanh chóng |
| **Palette** | Xem bảng màu ở mục 10.1 — cột Dark Mode |
| **Bản đồ Dark Mode** | Tùy biến Google Map JSON style (water màu navy đậm `#17263c`, landmass `#1d2c4d`) để không bị chói mắt ban đêm |
| **Lưu ý** | Ảnh từ Google Places áp dụng `brightness(0.9)` trong dark mode. Card shadow dùng `rgba(0,0,0,0.3)` |

### 10.10. Thư viện UI khuyến nghị

| Mục đích | Thư viện | Lý do chọn |
| --- | --- | --- |
| Styling | Tailwind CSS | Utility-first, responsive built-in, dark mode dễ |
| Animations | framer-motion | API đơn giản, `AnimatePresence` cho mount/unmount mượt mà |
| Toast notifications | sonner hoặc react-hot-toast | Nhẹ (<5KB), hiển thị đẹp mắt, tự động dismiss |
| Icons | lucide-react | Consistent, nhẹ, tree-shakeable |
| Confetti | canvas-confetti | Chỉ 6KB, tạo hiệu ứng chúc mừng sinh động |
| Google Maps | @react-google-maps/api | Official React wrapper, hooks-based |
| Font | Inter & Plus Jakarta Sans & JetBrains Mono | Google Fonts miễn phí, tối ưu hiển thị số tiền VND |
| *(mới)* Routing | react-router-dom v6 | Cần nhiều trang (`/trips`, `/s/:token`, `/settings`...) |
| *(mới)* Google Sign-In | @react-oauth/google | Nút "Continue with Google" chính chủ, trả về ID token |

### 10.11. Settings & Travel Profile (mới)

| Nhóm | Mục | Tier | Ghi chú |
| --- | --- | --- | --- |
| **Account** | Tên đăng nhập, đổi mật khẩu, xóa tài khoản | P2 | Xóa tài khoản yêu cầu nhập lại mật khẩu (hoặc đăng nhập Google lại) và cảnh báo xóa cả chuyến đi |
| **AI Assistant** | Mức chủ động: **Guided** / **Minimal** | P2 | *Minimal* = AI không hỏi thêm trừ khi bắt buộc, không lời chào. Đây là công tắc "bỏ qua" toàn cục |
| **AI Assistant** | Bật / tắt lời chào | P3 | Là lối thoát vĩnh viễn của nút "Don't show again" |
| **AI Assistant** | Persona: avatar + tên (preset), giọng điệu | P3 | Mục 10.3 |
| **Travel Profile** | Bật / tắt trí nhớ | P3 | Tắt = AI không học thêm và không dùng profile |
| **Travel Profile** | Xem, xóa từng mục, sửa `pace` / `crowd_tolerance`, xóa toàn bộ | P3 | Nguyên tắc 5. Hiển thị **đúng thứ AI đang dùng** |
| **Appearance** | Theme Light / Dark / System | P4 | |
| **Privacy** | Giải thích ngắn dữ liệu được lưu và cách xóa | P2 | Một đoạn văn bản, không cần trang riêng |

> Khách **không có trang Settings đầy đủ**: cài đặt persona/theme của khách nằm trong `localStorage` và được gửi kèm chat qua `client_settings` (mục 3.4.2). Travel Profile của khách vẫn xem/xóa được qua `/api/me/profile` theo `session_id`.

### 10.12. Nguyên tắc "Skip" và dẫn dắt không gây phiền (mới)

Quy tắc UI áp dụng cho **mọi** phần tử mà AI hoặc hệ thống chủ động hiện ra:

| Quy tắc | Chi tiết |
| --- | --- |
| **Luôn có lối thoát** | Mọi thẻ dẫn dắt (Smart Start, Welcome back, SaveTripPrompt, banner khách ở My Trips) có nút **Skip / Not now** và nút đóng |
| **Tối đa một thứ chủ động tại một thời điểm** | Không hiện đồng thời lời chào, thẻ nhắc đăng ký và banner |
| **Không bao giờ chặn luồng chính** | Không modal bắt buộc, không "đăng nhập để tiếp tục". Auth luôn kèm **Continue as guest** |
| **Nhắc đăng ký có tần suất** | `SaveTripPrompt` hiện **một lần** sau lịch trình đầu tiên của khách. "Not now" → im 7 ngày (lưu `localStorage`). "Don't ask again" → vĩnh viễn |
| **Lời chào ngắn dần** | Smart Start thu gọn thành một dòng sau lần dùng đầu. "Not now" ẩn đến hết phiên; "Don't show again" ghi vào Settings |
| **Có công tắc toàn cục** | Settings → AI Assistant → **Minimal** tắt toàn bộ lời chào và hạn chế câu hỏi làm rõ |
| **Không làm hỏng khi lỗi** | Smart Start/greeting lỗi thì biến mất im lặng và hiện form thường. Không toast lỗi đỏ cho tính năng phụ |
| **Trò chuyện ngoài lề tối thiểu** | Tối đa **một câu**, liên quan đến chuyến đi trước (ví dụ nhắc đến chuyến vừa rồi), không hỏi chuyện đời sống |

---
