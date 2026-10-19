## 13. Kịch bản Thuyết trình & Demo Tiếng Anh (5-Minute Live Pitch & Q&A Defense)

> **Cách dùng:** kịch bản có phần lõi (P0) và các đoạn chèn theo tính năng đã hoàn thành. **Chỉ nói những gì đã chạy được** — đoạn nào thuộc tính năng bị cắt thì bỏ, thay bằng một câu ở slide Roadmap.

### 13.1. Kịch bản Live Demo từng bước (English Script)

- **Phút 0:00 – 0:45 (Problem & Solution):**
  > *"Good morning/afternoon Professor. Planning a trip to Ho Chi Minh City today means jumping across dozens of TikTok videos, blogs, and Google Maps tabs. Generic AI chatbots hallucinate closed venues or schedule illogical routes across traffic-heavy districts. That’s why I built **HCMC Travel AI** — a proactive, location-verified travel agent powered by Google Gemini Function Calling and Google Places."*
- **Phút 0:45 – 1:45 (Onboarding & Proactive Human-in-the-loop):**
  > *"Instead of filling out a boring form, let’s switch to **Natural Sentence Mode**: 'I want to explore District 1 for 2 days with 3,000,000 VND, focusing on Street Food & Culture.' Notice what happens when I click Plan My Trip: instead of blindly guessing, our AI agent **Sài Gòn Guide** calls the `askPreference` tool and renders interactive **Quick Reply Chips** asking if I prefer a District 1 walking tour, District 3 & 4 food alleys, or a mix of both."*
  > ***[Nếu đã làm P3 – Smart Start]** thay câu đầu bằng:* *"Notice there's no account and no form: I just type one sentence, the AI fills the form for me, asks at most one question, and I can always press Skip."*
- **Phút 1:45 – 3:15 (Batch Tool Calling, Timeline, Haversine Transit & Budget Bar):**
  > *"I’ll tap **'Mix of both ✨'**. While our 4-stage thinking indicator updates the user, the backend executes a batch `searchPlaces` call with a custom FieldMask, checks our 7-day PostgreSQL cache, and invokes `saveItinerary`. On the right panel, we now have a verified day-by-day timeline. Between each stop, our Haversine transit calculator automatically estimates whether to walk or take a GrabBike along with the VND fare, and updates our real-time **Budget Bar** at 62%."*
- **Phút 3:15 – 4:00 (Conversational Modification & Mobile Responsiveness):**
  > *"Now let's test conversational editing: 'Can you swap dinner on Day 1 to a craft beer spot?' Within seconds, the AI queries verified craft breweries in District 1, swaps the dinner node inside a PostgreSQL transaction, and recalculates our total budget. Finally, switching to a 375px mobile viewport, the layout seamlessly adapts into a native-like dual-tab interface."*
- **Phút 4:00 – 4:45 (Continuity: không mất chuyến đi, chia sẻ được):**
  > ***[P1]** "I never signed in, yet this trip is saved in **My Trips**. Travelers go in groups, so I tap **Share**, open the link in a private window, and my friend sees a read-only copy with a 'Copy to my trips' button."*
  > ***[P2]** "If I decide to sign up now, one database transaction attaches my existing trips to the new account, so nothing is lost."*
  > ***[P3]** "Now I log in as a returning traveler. The app recognizes by behavior, not by account, that I've completed three trips, greets me with a template, skips the clarifying question, and tells me in one sentence what it assumes. Everything it remembers is visible in Settings, and I can edit, delete or switch it off."*
- **Phút 4:45 – 5:00 (Closing):**
  > *"HCMC Travel AI turns a scattered, risky planning process into a verified, conversational one, and treats guests and returning travelers differently without ever blocking them. Thank you, I'm happy to take questions."*

### 13.2. Bộ câu hỏi phản biện từ Giảng viên (Q&A Defense Cheat Sheet)

| Câu hỏi của Giảng viên | Câu trả lời mẫu (English) |
| --- | --- |
| **Q1: How do you prevent the LLM from hallucinating fake restaurants or wrong coordinates?** | *"The LLM is strictly prohibited from inventing venues. It must call the `searchPlaces` tool, which queries Google Places API (New) and our PostgreSQL `places_cache`. Only verified coordinates, ratings, and opening hours returned by the tool are passed into `saveItinerary`."* |
| **Q2: How do you handle the 15 requests/minute free-tier limit of Gemini API?** | *"First, I designed `searchPlaces` to accept an array of queries in a single batch turn, reducing a 5-turn loop down to just 2 Gemini requests per user message. Second, I implemented `express-rate-limit` at 10 req/min/IP and exponential backoff retry. Third, a server-wide budget reserves slots for the main chat, so secondary features like Smart Start or profile summarizing can never starve it."* |
| **Q3: How do you protect your Google Places API key and avoid unexpected billing?** | *"All external API calls happen strictly on the Express backend. Even place photos are resolved server-side to `lh3.googleusercontent.com` CDN URLs so the API key never appears in frontend `<img>` tags. On Google Cloud, I configured a $5 budget alert, a hard 150 req/day quota cap, and a 7-day PostgreSQL cache that eliminates over 75% of duplicate requests."* |
| **Q4: Why can users plan a trip without creating an account?** | *"Because the value shows up in about 15 seconds, and forcing a sign-up first would lose most travelers. Anonymous trips are tied to a session ID. If the user signs up later, a single transaction attaches those trips to the new account, so nothing is lost. If anything fails, it rolls back and the user simply remains a guest."* |
| **Q5: How do you store and protect passwords?** | *"Only bcrypt hashes with cost 12, never plain text. Login errors are generic and take the same time whether or not the username exists. Login, sign-up and password reset are rate limited, and reset tokens are stored hashed, single-use and expire in 30 minutes. I did not add phone login because SMS codes cost money per message and Google plus username already cover sign-in."* |
| **Q6: How do you decide whether someone is a new or an experienced user?** | *"By behavior, not by account. I count completed trips, with a threshold of three, per account or per anonymous session. So a frequent guest is still treated as an experienced user. New users get guided onboarding with a Skip button; experienced users get fewer questions and one stated assumption."* |
| **Q7: What does the AI remember about users, and can they control it?** | *"A small profile: food styles, favorite districts, pace, crowd tolerance and a typical budget. Every value comes from a fixed vocabulary, so free text never reaches the prompt, which also blocks prompt injection through memory. Users can see exactly what is stored in Settings, edit or delete items, clear everything, or switch memory off."* |
| **Q8: Where are paid plans and custom NPCs?** | *"They are on the roadmap, not implemented. Payment isn't needed to validate the core idea, and custom avatars need image moderation. I designed paid tiers around real travel value, like more saved trips and group sharing, with NPC customization only as a perk."* |

---
