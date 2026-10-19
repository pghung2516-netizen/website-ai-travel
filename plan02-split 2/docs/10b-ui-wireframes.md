### 10.2. Wireframe chi tiết — Tất cả màn hình & trạng thái

#### Màn hình 1: Landing + Trip Form (Kèm tuỳ chọn "Conversational Mad-Libs")

> **v2:** header có thêm `[My Trips]` và `[Sign in]` (khách) hoặc avatar menu (thành viên). Phần dẫn dắt của AI cho Type 1 / Type 2 xuất hiện **phía trên** form, xem Màn hình 5 và 6.

```
┌─────────────────────────────────────────────────────────┐
│ 📍 HCMC Travel AI          ☀️/🌙  [My Trips]  [Sign in] │  ← Header: logo + dark mode toggle + My Trips + Sign in
│─────────────────────────────────────────────────────────│
│                                                         │
│          ✈️ Plan Your Perfect HCMC Adventure             │  ← Hero heading (H1)
│     AI-powered itineraries tailored to your style       │  ← Subtitle (Text Secondary)
│                                                         │
│  [ Standard Form ]     [ ✨ Natural Sentence Mode ]      │  ← Mode switcher tab
│  ─────────────────     ────────────────────────────     │
│                                                         │
│  ┌─ ✨ Natural Sentence Mode (Giảm ma sát onboarding) ─┐│
│  │ "I want to explore [District 1, HCMC 📍] for        ││
│  │  [2 days 🗓️] with [₫3,000,000 💰], focusing on       ││
│  │  [Street food 🍜] & [Café culture ☕]."              ││
│  │                                                     ││
│  │  Tap any underlined token to toggle or edit!        ││
│  └─────────────────────────────────────────────────────┘│
│                                                         │
│  ┌─ Hoặc điền Standard Form ───────────────────────┐    │
│  │                                                 │    │
│  │  📍 Starting location                           │    │
│  │  ┌─────────────────────────────────────────┐    │    │
│  │  │ District 1, Ho Chi Minh City        🔍  │    │    │  ← Autocomplete input
│  │  └─────────────────────────────────────────┘    │    │
│  │                                                 │    │
│  │  💰 Budget (VND)            📅 Number of days   │    │
│  │  ┌──────────────────┐      ┌───────────────┐   │    │
│  │  │ ₫ 3,000,000      │      │ 2         -/+ │   │    │  ← Stepper cho số ngày
│  │  └──────────────────┘      └───────────────┘   │    │
│  │  Budget per day: ₫1,500,000                    │    │  ← Auto-calculate, Text Secondary
│  │                                                 │    │
│  │  🎯 What do you enjoy?                          │    │
│  │  ┌──────────┐ ┌──────────┐ ┌──────────┐       │    │
│  │  │🍜 Street │ │🏛️ Museums│ │🌙 Night- │       │    │  ← Tag chips, multi-select
│  │  │  food ✓  │ │          │ │  life ✓  │       │    │     Chọn = filled Primary color
│  │  └──────────┘ └──────────┘ └──────────┘       │    │     Chưa chọn = outlined Border
│  │  ┌──────────┐ ┌──────────┐ ┌──────────┐       │    │
│  │  │🛍️ Shop- │ │🌿 Nature │ │🎭 History│       │    │
│  │  │  ping ✓  │ │          │ │          │       │    │
│  │  └──────────┘ └──────────┘ └──────────┘       │    │
│  │                                                 │    │
│  │  ┌─────────────────────────────────────────┐   │    │
│  │  │         🚀 Plan My Trip                 │   │    │  ← CTA button, Accent color
│  │  └─────────────────────────────────────────┘   │    │     Hover: scale(1.02) + darker shade
│  │                                                 │    │
│  └─────────────────────────────────────────────────┘    │
│                                                         │
│  Powered by Google Gemini AI & Google Places            │  ← Footer trust badge
└─────────────────────────────────────────────────────────┘
```

#### Màn hình 2A: AI đang sinh lịch trình (Loading State)

```
┌─────────────────────────────────────────────────────────┐
│  ☀️/🌙  📍 HCMC Travel AI                    [New Trip] │
│─────────────────────────────────────────────────────────│
│                                                         │
│     ┌──────────────────────────────────────────────┐    │
│     │                                              │    │
│     │    ✨ Creating your perfect itinerary...     │    │
│     │                                              │    │
│     │    ● 🔍 Searching for the best spots...     │    │  ← Step 1: xong
│     │    ● 🗓️ Organizing your days...      ←      │    │  ← Step 2: đang chạy (pulse animation)
│     │    ○ 💰 Checking your budget...              │    │  ← Step 3: chưa
│     │    ○ ✨ Finalizing your perfect trip!        │    │  ← Step 4: chưa
│     │                                              │    │
│     │    ┌────────────────────────────────────┐    │    │
│     │    │▓▓▓▓▓▓▓▓▓▓▓▓░░░░░░░░░░░░░░░░░░░░░│    │    │  ← Progress bar (animated)
│     │    └────────────────────────────────────┘    │    │
│     │    This usually takes 10-15 seconds          │    │
│     │                                              │    │
│     └──────────────────────────────────────────────┘    │
│                                                         │
└─────────────────────────────────────────────────────────┘
```

#### Màn hình 2B: Chat + Itinerary (Desktop — Split View)

> **v2:** header có `[My Trips]` + `[Sign in]`; panel lịch trình có thêm thanh hành động `[🔗 Share] [⧉ Duplicate]` (mục 10.4); khách vừa có lịch trình lần đầu sẽ thấy thẻ `SaveTripPrompt` **bên dưới Budget Bar**, không che nội dung (mục 10.12).

```
┌──────────────────────────────────────────────────────────────────────────┐
│ 📍 HCMC Travel AI                     ☀️/🌙  [My Trips]  [Sign in]      │
│──────────────────────────────────────────────────────────────────────────│
│                                                                          │
│  ┌─────────── 40% ──────────┬──────────────── 60% ──────────────────┐   │
│  │  💬 Chat with AI          │  📋 Your Itinerary    💰 ₫1.85M/3M   │   │
│  │                           │  ┌─ Day 1 ─┬─ Day 2 ─┐  [🔗 Share]   │   │  ← Day tabs + hành động
│  │  ┌───────────────────┐   │  │  active  │         │              │   │
│  │  │ 🤖 Sài Gòn Guide │   │  └──────────┴─────────┘              │   │
│  │  │                   │   │                                      │   │
│  │  │ I'd love to help  │   │  ● 09:00 ─── 🏛️ War Remnants Museum │   │  ← Timeline node (Blue)
│  │  │ you explore HCMC! │   │  │  ┌─────────────────────────────┐ │   │
│  │  │ Which area do you │   │  │  │ ⭐ 4.6 · Est. ₫40,000      │ │   │  ← Activity card
│  │  │ prefer?           │   │  │  │ Explore Vietnam War history │ │   │
│  │  │                   │   │  │  │ 📍 28 Vo Van Tan, D3        │ │   │
│  │  │ ┌────────────────┐│   │  │  └─────────────────────────────┘ │   │
│  │  │ │ 🏙️ Central D1  ││   │  │                                  │   │
│  │  │ └────────────────┘│   │  ● 12:00 ─── 🍜 Phở Hòa Pasteur   │   │  ← Timeline node (Amber)
│  │  │ ┌────────────────┐│   │  │  ┌─────────────────────────────┐ │   │
│  │  │ │ 🗺️ Explore D3  ││   │  │  │ ⭐ 4.3 · Est. ₫65,000      │ │   │
│  │  │ └────────────────┘│   │  │  │ Famous beef phở since 1968  │ │   │
│  │  │ ┌────────────────┐│   │  │  │ 📍 260C Pasteur, D3    [📸] │ │   │  ← Thumbnail photo
│  │  │ │ 🌟 Mix both!   ││   │  │  └─────────────────────────────┘ │   │
│  │  │ └────────────────┘│   │  │                                  │   │
│  │  └───────────────────┘   │  ● 15:00 ─── ☕ The Workshop Coffee │   │  ← Timeline node (Purple)
│  │  ← Quick reply chips     │  │  ┌─────────────────────────────┐ │   │
│  │                           │  │  │ ⭐ 4.5 · Est. ₫80,000      │ │   │
│  │  ┌───────────────────┐   │  │  │ Rooftop café with city view │ │   │
│  │  │ 👤 Central D1     │   │  │  │ 📍 27 Ngo Duc Ke, D1  [📸] │ │   │
│  │  │    please          │   │  │  └─────────────────────────────┘ │   │
│  │  └───────────────────┘   │                                      │   │
│  │                           │  ┌──────────────────────────────┐   │   │
│  │  ┌───────────────────┐   │  │  🗺️ Map View                 │   │   │  ← Embedded Google Map
│  │  │ 🤖 Sài Gòn Guide │   │  │  [Google Maps với 3 pins]     │   │   │     Hiển thị pins cho ngày đang xem
│  │  │                   │   │  │  connected by route lines     │   │   │
│  │  │ Great choice! 🎉  │   │  └──────────────────────────────┘   │   │
│  │  │ Here's your Day 1 │   │                                      │   │
│  │  │ plan...           │   │  ── Budget Bar ──────────────────── │   │  ← Progress bar
│  │  └───────────────────┘   │  ▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓░░░░░░░░░░░░  │   │     Green: 62% used
│  │                           │  ₫1,850,000 / ₫3,000,000 (62%)    │   │
│  │  • • •  ← typing         │  ┌ 💾 Save this trip to your     ┐ │   │  ← SaveTripPrompt (khách, 1 lần,
│  │                           │  │ account?  [Sign in] [Not now] │ │   │     không chặn)
│  │  ┌────────────────── ▶ │  │  └───────────────────────────────┘ │   │  ← Input area
│  │  │ Type a message...     │  │                                      │   │     Auto-expand, send button
│  └─────────────────────────┴──────────────────────────────────────┘   │
│                                                                          │
└──────────────────────────────────────────────────────────────────────────┘
```

#### Màn hình 2C: Mobile Layout (375px)

```
┌────────────────────────┐
│ 📍 HCMC Travel AI  ☀️  │
│────────────────────────│
│                        │
│ 🤖 Sài Gòn Guide      │
│ ┌────────────────────┐ │
│ │ I'd love to help   │ │
│ │ you explore HCMC!  │ │
│ │ Which area?        │ │
│ └────────────────────┘ │
│                        │
│ ┌──────┐ ┌──────────┐ │  ← Quick reply chips
│ │🏙️ D1 │ │🗺️ Mix!  │ │     (horizontal scroll nếu nhiều)
│ └──────┘ └──────────┘ │
│                        │
│       ┌──────────────┐ │
│       │ 👤 Central   │ │  ← User message (right-aligned)
│       │ D1 please    │ │
│       └──────────────┘ │
│                        │
│ 🤖 Sài Gòn Guide      │
│ ┌────────────────────┐ │
│ │ Great choice! 🎉   │ │
│ │ Here's your plan.. │ │
│ └────────────────────┘ │
│                        │
│ • • •                  │  ← Typing indicator
│                        │
│ ┌──────────────── ▶ │ │  ← Input bar (fixed bottom)
│ │ Type a message...   │ │
│────────────────────────│
│  💬 Chat  │ 📋 Plan   │  ← Bottom tab bar
│  ═══════    ────────   │     Active = underline Primary
└────────────────────────┘

Khi chuyển sang tab "📋 Plan":

┌────────────────────────┐
│ 📍 HCMC Travel AI  ☀️  │
│────────────────────────│
│ ┌─ Day 1 ─┬─ Day 2 ─┐ │  ← Day tabs (swipe để chuyển)
│ └═════════┴─────────┘ │
│                        │
│ 💰 ₫1.85M / ₫3M       │  ← Budget bar (compact)
│ ▓▓▓▓▓▓▓▓▓▓░░░░░░░░░  │
│                        │
│ ● 09:00 🏛️            │
│ ┌────────────────────┐ │
│ │ War Remnants Museum│ │  ← Card (full width on mobile)
│ │ ⭐ 4.6 · ₫40,000   │ │
│ │ 📍 28 Vo Van Tan   │ │
│ │ [      📸 ảnh     ] │ │  ← Photo hiển thị rõ hơn trên mobile
│ └────────────────────┘ │
│ │                      │
│ ● 12:00 🍜            │
│ ┌────────────────────┐ │
│ │ Phở Hòa Pasteur    │ │
│ │ ⭐ 4.3 · ₫65,000   │ │
│ │ 📍 260C Pasteur    │ │
│ └────────────────────┘ │
│                        │
│────────────────────────│
│  💬 Chat  │ 📋 Plan   │
│  ────────   ═══════    │
└────────────────────────┘
```

> **v2 (mobile):** trên màn hẹp, `My Trips` và `Sign in` nằm trong **menu ☰/avatar** ở góc phải header (không làm chật header); Auth modal mở dạng **bottom sheet** toàn chiều rộng.

#### Màn hình 3: Error & Empty States

```
── Empty State (chưa có lịch trình) ──

┌────────────────────────────────────┐
│                                    │
│         🧳 (illustration)          │  ← Dùng ảnh miễn phí từ undraw.co
│                                    │
│   Ready for an adventure?          │
│   Fill in your trip details        │
│   to get started!                  │
│                                    │
│   [ 🚀 Plan My Trip ]             │  ← CTA button
│                                    │
└────────────────────────────────────┘

── Error State (API lỗi) ──

┌────────────────────────────────────┐  ← Toast notification (top-center)
│ ❌ Oops! AI is busy right now.     │     Auto-dismiss sau 5 giây
│    Please try again in a moment.   │     Trừ khi cần action → giữ lại
│                        [Try Again] │
└────────────────────────────────────┘

── Offline State ──

┌────────────────────────────────────────────────┐  ← Persistent banner (top)
│ 📡 You're offline. Some features unavailable.  │     Background: Amber 100
└────────────────────────────────────────────────┘     Text: Amber 800
```

#### Màn hình 4: Đăng nhập / Đăng ký (Auth Modal) — P2

```
┌──────────────────────────────────────────┐
│  Welcome to HCMC Travel AI           ✕   │
│  [ Sign in ]  [ Sign up ]                │  ← Tab chuyển qua lại
│                                          │
│  ┌────────────────────────────────────┐  │
│  │  G   Continue with Google          │  │  ← Cần Internet (không dùng khi demo offline)
│  └────────────────────────────────────┘  │
│  ──────────────── or ────────────────    │
│  Username                                │
│  ┌────────────────────────────────────┐  │
│  │ linh_nguyen                        │  │  ← autocomplete="username"
│  └────────────────────────────────────┘  │
│  Password                     [👁 Show]  │
│  ┌────────────────────────────────────┐  │
│  │ ••••••••                           │  │  ← autocomplete="current-password"
│  └────────────────────────────────────┘  │
│  Forgot password?                        │  ← Làm sau cùng trong P2
│  ┌────────────────────────────────────┐  │
│  │              Sign in               │  │
│  └────────────────────────────────────┘  │
│  ✔ Your 2 trips on this device will be   │  ← Chỉ hiện khi khách đang có chuyến
│    saved to your account.                │
│                                          │
│          Continue as guest →             │  ← LUÔN có (nguyên tắc 1 và 2)
└──────────────────────────────────────────┘

Tab "Sign up" thêm các phần:
  • Email (optional) — "Needed only to reset your password"
  • Hint dưới ô mật khẩu: "At least 8 characters"
  • Nếu để trống email: "Without an email, we can't recover your password."
  • Lỗi hiện ngay dưới ô (aria-live), giữ nguyên dữ liệu đã nhập
```

#### Màn hình 5: Smart Start cho Type 1 (khách mới / dưới 3 chuyến) — P3

```
┌─────────────────────────────────────────────────────────┐
│ 📍 HCMC Travel AI          ☀️/🌙  [My Trips]  [Sign in] │
│─────────────────────────────────────────────────────────│
│          ✈️ Plan Your Perfect HCMC Adventure             │
│                                                         │
│  ┌─ ✨ Tell me about your trip in one sentence ───────┐ │
│  │ ┌─────────────────────────────────────────┐        │ │
│  │ │ 2 days, around 3 million, love street…  │   [▶]  │ │  ← 1 lần gọi Gemini
│  │ └─────────────────────────────────────────┘        │ │
│  │ Where will you start each day?                     │ │  ← Câu hỏi bổ sung (MẪU, tối đa 1)
│  │ [District 1] [District 3] [District 7] [Other…]    │ │
│  │                    [ Skip, I'll fill the form ]    │ │  ← LUÔN có
│  └────────────────────────────────────────────────────┘ │
│                                                         │
│  ┌─ Standard Form (điền sẵn khi AI hiểu xong) ────────┐ │
│  │ 📍 District 1   💰 ₫3,000,000   📅 2 days           │ │  ← Chỉ điền sẵn, KHÔNG tự gửi
│  │ 🎯 Street food ✓  Café culture ✓                    │ │
│  │            [ 🚀 Plan My Trip ]                      │ │
│  └─────────────────────────────────────────────────────┘ │
└─────────────────────────────────────────────────────────┘
```

#### Màn hình 6: Welcome back cho Type 2 (từ 3 chuyến hoàn tất) — P3

```
┌─────────────────────────────────────────────────────────┐
│ 📍 HCMC Travel AI           ☀️/🌙  [My Trips]  [👤 linh ▾]│
│─────────────────────────────────────────────────────────│
│  ┌─ 🧭 Welcome back! ──────────────────────────────────┐│
│  │ Last time: 2 days of street food around District 1.  ││  ← Theo MẪU từ chuyến gần nhất (0 Gemini)
│  │ Same vibe this time?                                 ││
│  │ [Same vibe as last time]  [Something different]      ││
│  │                       [Not now]  [Don't show again]  ││  ← Hai lối thoát
│  └──────────────────────────────────────────────────────┘│
│                                                         │
│  (Form bên dưới như thường; "Same vibe" sẽ điền sẵn)     │
└─────────────────────────────────────────────────────────┘
```

#### Màn hình 7: My Trips — P1

```
┌────────────────────────────────────────────────────────────┐
│ 📍 HCMC Travel AI                 ☀️/🌙  [Sign in]          │
│────────────────────────────────────────────────────────────│
│ My Trips                                    [ + New trip ] │
│                                                            │
│ ┌────────────────────────┐ ┌────────────────────────┐      │
│ │ 2-day HCMC · Street…   │ │ 3-day HCMC · Museums   │      │
│ │ ₫1.85M / ₫3M · 6 stops │ │ ₫2.1M / ₫4M · 9 stops  │      │
│ │ Created 2 days ago     │ │ Created last week      │      │
│ │ 🔗 Shared              │ │                        │      │
│ │ [ Open ]          [⋯]  │ │ [ Open ]          [⋯]  │      │
│ └────────────────────────┘ └────────────────────────┘      │
│      ⋯ menu: Rename · Duplicate · Share link · Delete      │  ← Delete có xác nhận
│                                                            │
│ ┌────────────────────────────────────────────────────────┐ │
│ │ 💾 Trips are saved on this device only. Sign in to     │ │  ← Banner khách, đóng được
│ │ keep them across devices.       [Sign in]  [Not now]   │ │
│ └────────────────────────────────────────────────────────┘ │
│                                                            │
│ Empty state: 🧳 "No trips yet" + [ 🚀 Plan My Trip ]        │
└────────────────────────────────────────────────────────────┘
```

#### Màn hình 8: Chia sẻ lịch trình — P1

```
Share dialog:                               Trang công khai /s/:token (chỉ xem):
┌──────────────────────────────────────┐   ┌──────────────────────────────────────┐
│ Share this trip                   ✕  │   │ 📍 HCMC Travel AI          [Sign in] │
│ Anyone with the link can VIEW        │   │ Shared trip · 2 days · ₫1.85M        │
│ this trip (not edit).                │   │ ┌─ Day 1 ─┬─ Day 2 ─┐                │
│ ┌──────────────────────────┐ [Copy]  │   │ └─────────┴─────────┘                │
│ │ https://app/s/Xk3vT9mQ…  │         │   │ ● 09:00 🏛️ War Remnants Museum       │
│ └──────────────────────────┘         │   │ ● 12:00 🍜 Phở Hòa Pasteur           │
│ [ Stop sharing ]                     │   │ (timeline + map, KHÔNG có chat)      │
└──────────────────────────────────────┘   │                                      │
  ← "Stop sharing" thu hồi link ngay;      │ [ 📋 Copy to my trips ]              │
    toast "Link copied" khi bấm Copy       └──────────────────────────────────────┘
```

#### Màn hình 9: Settings & Travel Profile — P2/P3

```
┌─────────────────────────────────────────────────────────────┐
│ Settings                                                    │
│ ┌ Account ──────────────────────────────────────────────┐  │
│ │ Username: linh_nguyen     [Change password] [Delete…]  │  │  ← P2
│ └────────────────────────────────────────────────────────┘  │
│ ┌ AI Assistant ──────────────────────────────────────────┐  │
│ │ Interaction:  (•) Guided   ( ) Minimal (fewer questions)│  │  ← P2: mức chủ động
│ │ Welcome messages:  [ON]                                 │  │
│ │ Avatar:  🧭  🛵  🌸  🌿  🏳️‍🌈      Name: Sài Gòn Guide   │  │  ← P3: persona (preset)
│ │ Tone:    (•) Friendly  ( ) Concise  ( ) Playful         │  │
│ └────────────────────────────────────────────────────────┘  │
│ ┌ Travel Profile — what the AI remembers ────────────────┐  │  ← P3 (Type 2)
│ │ [ON] Let the AI remember my preferences                 │  │
│ │ Food:  street food ✕   specialty coffee ✕               │  │
│ │ Pace: relaxed ▾    Crowds: low ▾    Budget/day: ₫1.5M   │  │
│ │ Dislikes: crowded places ✕                              │  │
│ │ [ Clear everything ]                                    │  │
│ └────────────────────────────────────────────────────────┘  │
│ ┌ Appearance ┐  Theme: ( ) Light  ( ) Dark  (•) System      │
│ ┌ Privacy ──────────────────────────────────────────────┐  │
│ │ We store only what you see on this page.               │  │
│ └────────────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────────┘
```
