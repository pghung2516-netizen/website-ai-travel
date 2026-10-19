### 12.6. File `server/src/middleware/identity.js` (Danh tính: khách hay thành viên) — mới

```javascript
import jwt from 'jsonwebtoken';

const UUID_RE = /^[0-9a-f]{8}-[0-9a-f]{4}-[1-8][0-9a-f]{3}-[89ab][0-9a-f]{3}-[0-9a-f]{12}$/i;

/**
 * Dựng req.principal từ 2 header:
 *   x-session-id  (bắt buộc cho hầu hết route, UUID)  -> danh tính "khách" / trình duyệt
 *   Authorization (tùy chọn)                           -> danh tính "thành viên"
 *
 * Tuyến công khai (/api/health, /api/shared/:token) được bỏ qua kiểm tra header.
 * Riêng POST /api/shared/:token/clone vẫn bắt buộc x-session-id.
 *
 * LƯU Ý KIẾN TRÚC CHO app.js (An làm chủ):
 * 1. BẮT BUỘC: app.use(cors(corsOptions)) phải đứng TRƯỚC identity.
 *    Nếu không, request preflight OPTIONS của trình duyệt (không mang header x-session-id)
 *    sẽ bị identity trả về 400, làm hỏng toàn bộ CORS giữa Vercel và Render!
 * 2. Dùng req.originalUrl || req.path để dù mount app.use('/api', identity)
 *    hay app.use(identity) toàn cục thì đường dẫn vẫn không bị mất tiền tố '/api'.
 */
export function identity(req, res, next) {
  // Cho phép preflight CORS OPTIONS đi qua
  if (req.method === 'OPTIONS') {
    return next();
  }

  const url = req.originalUrl || req.url || req.path;

  // Bỏ qua route công khai (health check, xem link chia sẻ /api/shared/:token)
  // Mọi method lên /api/shared/:token (trừ /clone) đều bypass identity
  // để request thử PUT/DELETE không có header vẫn lọt tới router trả 404/405 theo TC-12.
  const isPublicRoute =
    url.startsWith('/api/health') ||
    url === '/health' ||
    (url.includes('/api/shared/') && !url.endsWith('/clone'));

  if (isPublicRoute) {
    return next();
  }

  const sessionId = req.header('x-session-id');
  if (!sessionId || !UUID_RE.test(sessionId)) {
    return res.status(400).json({
      error: {
        code: 'VALIDATION_ERROR',
        message: 'Missing or invalid x-session-id header (required UUIDv4)'
      }
    });
  }

  let userId = null;
  const auth = req.header('authorization') || '';
  if (auth.startsWith('Bearer ')) {
    try {
      // Ép thuật toán HS256 để chặn kiểu tấn công "alg: none"
      const payload = jwt.verify(auth.slice(7), process.env.JWT_SECRET, { algorithms: ['HS256'] });
      userId = payload.sub;
    } catch {
      // Token hết hạn/sai: KHÔNG chặn request, coi như khách và báo cho client biết.
      // Cần exposedHeaders: ['X-Auth-Status'] trong CORS để frontend đọc được header này.
      res.setHeader('X-Auth-Status', 'expired');
    }
  }

  req.principal = { sessionId, userId };
  next();
}

/**
 * Điều kiện SQL "chuyến đi này thuộc về ai".
 * Chỉ ghép SỐ THỨ TỰ placeholder ($1, $2...) vào chuỗi; dữ liệu thật đi qua params
 * => không có SQL injection.
 *
 * Ví dụ:
 *   const { clause, params } = ownerClause(req.principal, 2);
 *   pool.query(`SELECT * FROM trips WHERE id = $1 AND ${clause}`, [tripId, ...params]);
 *   // Không có dòng nào => trả 404 (không phải 403) để không lộ chuyến đó có tồn tại.
 */
export function ownerClause(principal, startIndex = 1, alias = '') {
  const p = alias ? `${alias}.` : '';
  if (principal.userId) {
    // Thành viên: chỉ thấy chuyến của chính mình
    return { clause: `${p}user_id = $${startIndex}`, params: [principal.userId] };
  }
  // Khách: chỉ thấy chuyến CHƯA thuộc tài khoản nào VÀ đúng session_id
  return {
    clause: `(${p}user_id IS NULL AND ${p}session_id = $${startIndex})`,
    params: [principal.sessionId]
  };
}
```
