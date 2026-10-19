## 6. Xử lý lỗi (Error Handling)

| Tình huống | Mã lỗi nội bộ | Cách xử lý |
| --- | --- | --- |
| Gemini API timeout hoặc trả về lỗi | `AI_SERVICE_TIMEOUT` | Retry tự động 1-2 lần với exponential backoff (1.5s, 3s). Nếu vẫn lỗi → trả thông báo thân thiện: "AI đang bận, vui lòng thử lại sau vài giây" |
| Gemini API hết free quota (~15 req/phút) | `AI_RATE_LIMITED` | Rate limiting phía backend (10 req/phút/IP) kết hợp gộp batch `searchPlaces` ngăn chạm trần RPM. Nếu hết quota → thông báo "Đã đạt giới hạn, vui lòng đợi 1 phút" |
| Hết slot Gemini **toàn server** (nhiều người cùng dùng chung 1 key) | `GEMINI_BUDGET_EXHAUSTED` | Chat: "Vui lòng đợi vài giây". Smart Start: **hiện form thường**. Tóm tắt profile: **bỏ qua** (chạy lại ở chuyến sau) |
| Google Places API hết quota hoặc lỗi | `PLACES_FALLBACK_CACHE` | Fallback sang dữ liệu đã cache trong `places_cache` (tìm kiếm mờ bằng `ILIKE`). Thông báo user: "Một số thông tin có thể không phải mới nhất" |
| User nhập địa điểm xuất phát ngoài TP.HCM | `OUT_OF_BOUNDS_ORIGIN` | Chấp nhận + AI tính thêm thời gian di chuyển tới HCM. Hoặc validate + gợi ý: "Bạn có muốn chọn điểm xuất phát trong TP.HCM không?" |
| AI sinh lịch trình vô lý (chi phí vượt ngân sách, giờ giấc trùng nhau) | `BUDGET_OVERFLOW_WARNING` | Validation layer ở backend trong `saveItinerary` kiểm tra tổng chi phí (activities + transit) trước khi lưu DB + trả về frontend. Nếu vượt ngân sách → gắn cờ `over_budget` để Budget Bar hiện màu đỏ kèm gợi ý giảm chi phí |
| AI không gọi tool `saveItinerary` khi user yêu cầu đổi lịch trình | `MISSING_ITINERARY_TOOL` | Nếu phát hiện câu lệnh sửa lịch trình nhưng AI chỉ trả về văn bản thuần → Backend tự động nhắc lại trong vòng lặp tool với hướng dẫn: *"Please invoke `saveItinerary` to persist the updated schedule"* |
| Mất kết nối giữa chat | `NETWORK_DISCONNECTED` | Conversation history và `itinerary_items` lưu trong DB theo `session_id` (khách) hoặc `user_id` (thành viên) → user refresh trang vẫn thấy lại toàn bộ chat + lịch trình |
| Input không hợp lệ (số ngày ≤ 0, ngân sách âm, text quá dài) | `VALIDATION_ERROR` | Validate cả frontend (realtime feedback) + backend (reject 400 Bad Request). Quy tắc: ngày 1-14, ngân sách > 0, text ≤ 500 ký tự |
| Sai username hoặc mật khẩu | `AUTH_INVALID_CREDENTIALS` | `401` với **một thông báo chung** "Invalid username or password", không nói sai cái nào |
| Username hoặc email đã được dùng | `AUTH_USERNAME_TAKEN` | `409` "Username or email already in use". Form giữ nguyên dữ liệu đã nhập |
| Thử mật khẩu liên tục (brute force) | `AUTH_RATE_LIMITED` | `429` sau 5 lần sai/phút cho mỗi cặp IP + username: "Too many attempts, try again in a minute" |
| JWT hết hạn hoặc không hợp lệ | `AUTH_TOKEN_EXPIRED` | Backend coi như khách và trả header `X-Auth-Status: expired`; frontend hiện "Please sign in again". Dữ liệu thành viên **không bị mất**, chỉ ẩn cho đến khi đăng nhập lại |
| Google ID token không hợp lệ | `GOOGLE_TOKEN_INVALID` | `401`; frontend: "Google sign-in failed. Try again or use your username". Không ảnh hưởng chế độ khách |
| Link đặt lại mật khẩu hết hạn hoặc đã dùng | `RESET_TOKEN_INVALID` | `400` "This link is no longer valid. Request a new one" |
| Gửi email thất bại | `EMAIL_SEND_FAILED` | Vẫn trả `200` (không để lộ email có tồn tại hay không), ghi log; frontend nhắc "Check your inbox, or try again in a few minutes" |
| Claim chuyến của khách lỗi giữa chừng | `CLAIM_FAILED` | Toàn bộ transaction **rollback**: người dùng vẫn là khách, **không mất chuyến đi nào**; hiện "Something went wrong, please try again" |
| Link chia sẻ sai hoặc đã thu hồi | `SHARE_TOKEN_INVALID` | `404` chung, không phân biệt "không tồn tại" và "đã thu hồi" |
| Smart Start không phân tích được | `ONBOARDING_PARSE_FAILED` | Hiện **form thường** như chưa có chuyện gì xảy ra; không hiện lỗi đỏ |
| Tóm tắt Travel Profile lỗi hoặc không hợp lệ | `PROFILE_DISTILL_FAILED` | Bỏ qua im lặng, giữ profile cũ, ghi log. Không ảnh hưởng người dùng |

---
