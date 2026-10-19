### 12.7. File `server/src/services/authService.js` (Đăng ký / đăng nhập / Google / Claim) — mới

```javascript
import bcrypt from 'bcryptjs';
import jwt from 'jsonwebtoken';
import { OAuth2Client } from 'google-auth-library';
import pool from '../db/connection.js';

const BCRYPT_COST = 12;
// Hash "giả": khi username không tồn tại ta VẪN tốn thời gian bcrypt y như khi có.
// Nếu không, kẻ tấn công đo thời gian phản hồi là biết username nào tồn tại.
const DUMMY_HASH = bcrypt.hashSync('timing-equalizer', BCRYPT_COST);
const googleClient = new OAuth2Client(process.env.GOOGLE_CLIENT_ID);

function authError(status, code, message) {
  const e = new Error(message);
  e.status = status;
  e.code = code;
  return e;
}

export function signToken(userId) {
  return jwt.sign({ sub: userId }, process.env.JWT_SECRET, {
    algorithm: 'HS256',
    expiresIn: process.env.JWT_EXPIRES_IN || '7d'
  });
}

/**
 * Gán chuyến đi (và hồ sơ) của KHÁCH cho tài khoản.
 * PHẢI gọi bên trong transaction của đăng ký / đăng nhập.
 *
 * Trace: khách có session S, đã tạo trip A và B (user_id = NULL).
 *   Đăng ký xong, có user U.
 *   UPDATE ... WHERE session_id = S AND user_id IS NULL
 *     => A và B có user_id = U, rowCount = 2 (trả về làm claimed_trips).
 *   Từ giờ ownerClause cho U thấy A, B; khách với session S không còn thấy (user_id khác NULL).
 *   Gọi lại lần nữa => rowCount = 0 (idempotent), không hỏng gì.
 */
export async function claimGuestData(client, userId, sessionId) {
  const trips = await client.query(
    `UPDATE trips SET user_id = $1, updated_at = NOW()
     WHERE session_id = $2 AND user_id IS NULL`,
    [userId, sessionId]
  );
  // Hồ sơ du lịch của khách: chuyển sang tài khoản nếu tài khoản CHƯA có hồ sơ
  await client.query(
    `UPDATE travel_profiles SET owner_key = $1
     WHERE owner_key = $2
       AND NOT EXISTS (SELECT 1 FROM travel_profiles WHERE owner_key = $1)`,
    [`u:${userId}`, `s:${sessionId}`]
  );
  // Nếu tài khoản đã có hồ sơ thì hồ sơ khách còn sót lại -> xóa để không bị mồ côi
  await client.query(`DELETE FROM travel_profiles WHERE owner_key = $1`, [`s:${sessionId}`]);
  return trips.rowCount;
}

export async function register({ username, password, email }, sessionId) {
  const passwordHash = await bcrypt.hash(password, BCRYPT_COST);
  const client = await pool.connect();
  try {
    await client.query('BEGIN');
    const ins = await client.query(
      `INSERT INTO users (username, password_hash, email, display_name)
       VALUES ($1, $2, $3, $1)
       RETURNING id, username, email, display_name, settings`,
      [username, passwordHash, email || null]
    );
    const user = ins.rows[0];
    const claimed = await claimGuestData(client, user.id, sessionId);
    await client.query('COMMIT');
    return { token: signToken(user.id), user, claimed_trips: claimed };
  } catch (err) {
    await client.query('ROLLBACK'); // lỗi ở bất kỳ bước nào => vẫn là khách, KHÔNG mất chuyến đi nào
    // 23505 = unique_violation: trùng LOWER(username) hoặc LOWER(email)
    if (err.code === '23505') {
      throw authError(409, 'AUTH_USERNAME_TAKEN', 'Username or email already in use');
    }
    throw err;
  } finally {
    client.release();
  }
}

export async function login({ username, password }, sessionId) {
  const found = await pool.query(
    `SELECT id, username, email, display_name, settings, password_hash
     FROM users WHERE LOWER(username) = LOWER($1)`,
    [username]
  );
  const row = found.rows[0];

  // Luôn chạy compare (kể cả khi không có user) để thời gian phản hồi giống nhau
  const ok = await bcrypt.compare(password, row?.password_hash ?? DUMMY_HASH);
  if (!row || !ok) {
    // Thông báo CHUNG: không nói sai username hay sai mật khẩu
    throw authError(401, 'AUTH_INVALID_CREDENTIALS', 'Invalid username or password');
  }

  const client = await pool.connect();
  try {
    await client.query('BEGIN');
    await client.query(`UPDATE users SET last_login_at = NOW() WHERE id = $1`, [row.id]);
    const claimed = await claimGuestData(client, row.id, sessionId);
    await client.query('COMMIT');
    const { password_hash, ...user } = row; // KHÔNG trả password_hash ra ngoài
    return { token: signToken(row.id), user, claimed_trips: claimed };
  } catch (err) {
    await client.query('ROLLBACK');
    throw err;
  } finally {
    client.release();
  }
}

export async function loginWithGoogle(idToken, sessionId) {
  // 1. BACKEND xác minh ID token (chữ ký, hạn dùng, audience). Không tin gì client tự gửi.
  let payload;
  try {
    const ticket = await googleClient.verifyIdToken({
      idToken,
      audience: process.env.GOOGLE_CLIENT_ID
    });
    payload = ticket.getPayload();
  } catch {
    throw authError(401, 'GOOGLE_TOKEN_INVALID', 'Google sign-in failed');
  }
  if (!payload?.sub || !payload.email_verified) {
    throw authError(401, 'GOOGLE_TOKEN_INVALID', 'Google sign-in failed');
  }

  const client = await pool.connect();
  try {
    await client.query('BEGIN');
    // 2. Tạo mới hoặc đăng nhập theo google_sub. (xmax = 0) là mẹo nhận biết dòng vừa được INSERT.
    const up = await client.query(
      `INSERT INTO users (google_sub, email, display_name)
       VALUES ($1, $2, $3)
       ON CONFLICT (google_sub) DO UPDATE SET last_login_at = NOW()
       RETURNING id, username, email, display_name, settings, (xmax = 0) AS is_new`,
      [payload.sub, payload.email, payload.name || null]
    );
    const user = up.rows[0];
    const claimed = await claimGuestData(client, user.id, sessionId);
    await client.query('COMMIT');
    const { is_new, ...safeUser } = user;
    return { token: signToken(user.id), user: safeUser, claimed_trips: claimed, is_new };
  } catch (err) {
    await client.query('ROLLBACK');
    // Email này đã thuộc một tài khoản username: v2 KHÔNG tự gộp (Out of Scope, mục 2.2)
    if (err.code === '23505') {
      throw authError(409, 'AUTH_USERNAME_TAKEN', 'This email is already used by another account');
    }
    throw err;
  } finally {
    client.release();
  }
}
```
