schema.sql

SQL
CREATE TABLE IF NOT EXISTS issues (
    id SERIAL PRIMARY KEY,
    title VARCHAR(255) NOT NULL,
    description TEXT,
    status VARCHAR(50) DEFAULT 'open' NOT NULL,
    created_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP
);
src/types/issue.ts

TypeScript
// To'liq Issue modeli
export interface Issue {
  id: number;
  title: string;
  description: string | null;
  status: string;
  created_at: Date;
  updated_at: Date;
}

// Issue turini QAYTA ISHLATMASDAN yaratilgan alohida interfeys
export interface IssueSummary {
  id: number;
  title: string;
  status: string;
}
src/db.ts

TypeScript
import { Pool } from 'pg';

export const pool = new Pool({
  user: process.env.DB_USER || 'postgres',
  host: process.env.DB_HOST || 'localhost',
  database: process.env.DB_NAME || 'issueforge',
  password: process.env.DB_PASSWORD || 'postgres',
  port: Number(process.env.DB_PORT) || 5432,
});
src/routes/issue.routes.ts

TypeScript
import { Router, Request, Response } from 'express';
import { pool } from '../db';
import { Issue, IssueSummary } from '../types/issue';

const router = Router();

// 1. GET /issues/summary — Alohida IssueSummary interfeysi bilan tiplashtirilgan
// MUHIM: Bu marshrut GET /issues/:id dan yuqorida joylashishi shart!
router.get('/summary', async (req: Request, res: Response) => {
  try {
    const { rows } = await pool.query<IssueSummary>(
      'SELECT id, title, status FROM issues ORDER BY id DESC'
    );
    res.json(rows);
  } catch (error) {
    res.status(500).json({ error: "Baza bilan bog'lanishda xatolik yuz berdi" });
  }
});

// 2. GET /issues — Barcha issuelarni olish
router.get('/', async (req: Request, res: Response) => {
  try {
    const { rows } = await pool.query<Issue>('SELECT * FROM issues ORDER BY id DESC');
    res.json(rows);
  } catch (error) {
    res.status(500).json({ error: "Baza bilan bog'lanishda xatolik yuz berdi" });
  }
});

// 3. GET /issues/:id — Bitta issueni id bo'yicha olish
router.get('/:id', async (req: Request, res: Response) => {
  const { id } = req.params;
  try {
    const { rows } = await pool.query<Issue>(
      'SELECT * FROM issues WHERE id = $1',
      [id]
    );

    if (rows.length === 0) {
      return res.status(404).json({ message: "Issue topilmadi" });
    }

    res.json(rows[0]);
  } catch (error) {
    res.status(500).json({ error: "Baza bilan bog'lanishda xatolik yuz berdi" });
  }
});

// 4. POST /issues — Yangi issue yaratish
router.post('/', async (req: Request, res: Response) => {
  const { title, description, status } = req.body;

  if (!title) {
    return res.status(400).json({ error: "Title maydoni majburiy" });
  }

  try {
    const { rows } = await pool.query<Issue>(
      `INSERT INTO issues (title, description, status) 
       VALUES ($1, $2, COALESCE($3, 'open')) 
       RETURNING *`,
      [title, description || null, status || null]
    );

    res.status(201).json(rows[0]);
  } catch (error) {
    res.status(500).json({ error: "Yangi issue qo'shishda xatolik" });
  }
});

// 5. PUT /issues/:id — Issueni yangilash
router.put('/:id', async (req: Request, res: Response) => {
  const { id } = req.params;
  const { title, description, status } = req.body;

  try {
    const { rows } = await pool.query<Issue>(
      `UPDATE issues 
       SET title = COALESCE($1, title), 
           description = COALESCE($2, description), 
           status = COALESCE($3, status),
           updated_at = CURRENT_TIMESTAMP
       WHERE id = $4 
       RETURNING *`,
      [title, description, status, id]
    );

    if (rows.length === 0) {
      return res.status(404).json({ message: "Issue topilmadi" });
    }

    res.json(rows[0]);
  } catch (error) {
    res.status(500).json({ error: "Issueni yangilashda xatolik" });
  }
});

// 6. DELETE /issues/:id — Issueni o'chirish
router.delete('/:id', async (req: Request, res: Response) => {
  const { id } = req.params;

  try {
    const { rows } = await pool.query<Issue>(
      'DELETE FROM issues WHERE id = $1 RETURNING *',
      [id]
    );

    if (rows.length === 0) {
      return res.status(404).json({ message: "Issue topilmadi" });
    }

    res.json({ message: "Issue o'chirildi", deletedIssue: rows[0] });
  } catch (error) {
    res.status(500).json({ error: "Issueni o'chirishda xatolik" });
  }
});

export default router;
src/app.ts

TypeScript
import express from 'express';
import issueRoutes from './routes/issue.routes';

const app = express();
app.use(express.json());

app.use('/issues', issueRoutes);

const PORT = process.env.PORT || 3000;
app.listen(PORT, () => {
  console.log(`Server running on port ${PORT}`);
});
README.md

Markdown
# IssueForge — REST API

## Bosqichlar holati (Checklist)

- [x] **1-bosqich:** Loyiha strukturasi va TypeScript konfiguratsiyasi
- [x] **2-bosqich:** In-Memory CRUD va marshrutlash
- [x] **3-bosqich:** PostgreSQL integratsiyasi (Joriy)
  - [x] `schema.sql` yaratildi
  - [x] `pool.query<Issue>()` yordamida to'liq tiplashtirilgan CRUD yozildi
  - [x] Barcha SQL so'rovlarda parametrlashtirish (`$1`, `$2`) qo'llanildi
  - [x] `IssueSummary` alohida interfeysi va `GET /issues/summary` endpoint'i yaratildi
