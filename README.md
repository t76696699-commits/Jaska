// ════════════════════════════════════════════════════════════════════
// 4-BOSQICH: Autentifikatsiya - JWT va tiplashtirilgan payload
// ════════════════════════════════════════════════════════════════════

import jwt from 'jsonwebtoken';
import bcrypt from 'bcrypt';
import { Request, Response, NextFunction } from 'express';

const JWT_SECRET = process.env.JWT_SECRET as string;

interface AuthTokenPayload {
  userId: number;
  role: 'member' | 'admin';
}

// ─────────────────────────────────────────────────────────────────────
// 1) Token chiqarish
// ─────────────────────────────────────────────────────────────────────

export function issueToken(payload: AuthTokenPayload): string {
  return jwt.sign(payload, JWT_SECRET, { expiresIn: '7d' });
}

// ─────────────────────────────────────────────────────────────────────
// 2) Token tekshirish - runtime narrowing bilan XAVFSIZ
// ─────────────────────────────────────────────────────────────────────

function verifyAuthToken(token: string): AuthTokenPayload | null {
  const decoded = jwt.verify(token, JWT_SECRET);

  if (typeof decoded === 'string' || !('userId' in decoded) || !('role' in decoded)) {
    return null;
  }
  return decoded as AuthTokenPayload;
}

// ─────────────────────────────────────────────────────────────────────
// 3) Express Request'ni kengaytirish + middleware
// ─────────────────────────────────────────────────────────────────────

declare global {
  namespace Express {
    interface Request {
      user?: AuthTokenPayload;
    }
  }
}

export function requireAuth(req: Request, res: Response, next: NextFunction) {
  const token = req.headers.authorization?.replace('Bearer ', '');
  const payload = token ? verifyAuthToken(token) : null;
  if (!payload) return res.status(401).json({ error: "Avtorizatsiyadan o'tilmagan" });

  req.user = payload;
  next();
}

// ─────────────────────────────────────────────────────────────────────
// 4) Ataylab xato - tekshiruvsiz cast (izohda)
// ─────────────────────────────────────────────────────────────────────

// function verifyAuthToken(token: string): AuthTokenPayload {
//   const decoded = jwt.verify(token, JWT_SECRET) as AuthTokenPayload;   // tekshiruvsiz!
//   return decoded;
// }
// Boshqa maqsaddagi (masalan parolni tiklash) string-token bilan
// qayta ishlatilsa: payload.userId -> undefined, payload.role -> undefined
