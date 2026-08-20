// ════════════════════════════════════════════════════════════════════
// 1-BOSQICH: Loyihalash va repo skeleton
// ════════════════════════════════════════════════════════════════════

// Bu dars kod yozishdan ko'ra REJALASHTIRISHGA bag'ishlangan.
// Quyida - IssueForge uchun shared/types.ts faylining to'liq tarkibi:

interface User {
  id: number;
  name: string;
  email: string;
  passwordHash: string;
  createdAt: string;
}

interface Issue {
  id: number;
  title: string;
  description: string;
  status: 'open' | 'in_progress' | 'closed';
  assigneeId: number | null;
  reporterId: number;
  createdAt: string;
}

export type { User, Issue };

// ─────────────────────────────────────────────────────────────────────
// Repo tuzilmasi (izohda - papka/fayl tuzilmasi, kod emas)
// ─────────────────────────────────────────────────────────────────────

// issueforge/
//   backend/
//     src/
//     tsconfig.json
//   frontend/
//     src/
//     tsconfig.json
//   shared/
//     types.ts
//   README.md
//   .gitignore

// ─────────────────────────────────────────────────────────────────────
// Ataylab qiyin - ALOHIDA yozilgan interfeyslar (izohda)
// ─────────────────────────────────────────────────────────────────────

// backend/src/types.ts va frontend/src/types.ts alohida-alohida
// yozilsa, TypeScript compiler ularni HECH QACHON taqqoslamaydi -
// ular ikkita mustaqil modul. shared/types.ts BU muammoni yo'q qiladi.

  
