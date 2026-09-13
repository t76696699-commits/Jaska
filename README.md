
# Konseptual (haqiqiy DB'da ishga TUSHIRILMAYDI) — pgvector sozlash va
# ishlatishning to'liq Python + SQLAlchemy shakli. Buni O'Z loyihangizning
# migratsiya faylida yoki alohida sozlash skriptida ishlating.

PGVECTOR_SETUP_SQL = """
-- 1) Kengaytmani yoqish (server darajasida kutubxona o'rnatilgan bo'lishi kerak)
CREATE EXTENSION IF NOT EXISTS vector;

-- 2) Vektor ustunli jadval
CREATE TABLE IF NOT EXISTS lesson_embeddings (
    id SERIAL PRIMARY KEY,
    lesson_id INTEGER NOT NULL REFERENCES lessons(id) ON DELETE CASCADE,
    chunk_text TEXT NOT NULL,
    chunk_heading VARCHAR(500),
    embedding vector(384) NOT NULL
);

-- 3) ANN indeks (katta miqyos uchun; kichik jadvalda shart emas)
CREATE INDEX IF NOT EXISTS lesson_embeddings_hnsw_idx
    ON lesson_embeddings USING hnsw (embedding vector_cosine_ops);
"""


async def insert_chunk_embedding(db, lesson_id: int, chunk_text: str, heading: str, embedding: list[float]) -> None:
    """O'z loyihangizda: bitta chunk + uning embeddingini saqlaydi.
    `db` — mavjud AsyncSession (bu platformadagi barcha skriptlar
    ishlatadigan xuddi shu pattern)."""
    from sqlalchemy import text
    await db.execute(
        text(
            "INSERT INTO lesson_embeddings (lesson_id, chunk_text, chunk_heading, embedding) "
            "VALUES (:lesson_id, :chunk_text, :heading, :embedding)"
        ),
        {
            "lesson_id": lesson_id,
            "chunk_text": chunk_text,
            "heading": heading,
            "embedding": str(embedding),  # pgvector matn shaklidagi '[0.1,0.2,...]'ni kutadi
        },
    )


async def search_lesson_embeddings(db, query_vector: list[float], k: int = 5) -> list:
    """Eng mos k ta chunk'ni qaytaradi, masofa (distance) bo'yicha
    o'sish tartibida (kichikroq masofa = yaqinroq ma'no)."""
    from sqlalchemy import text
    rows = await db.execute(
        text(
            "SELECT lesson_id, chunk_text, chunk_heading, "
            "embedding <=> :qv AS distance "
            "FROM lesson_embeddings "
            "ORDER BY embedding <=> :qv "
            "LIMIT :k"
        ),
        {"qv": str(query_vector), "k": k},
    )
    return rows.fetchall()


if __name__ == "__main__":
    print("Bu modul faqat namuna kodini o'z ichiga oladi — DB'ga ulanmaydi.")
    print("O'Z loyihangizda ishlatish uchun PGVECTOR_SETUP_SQL'ni migratsiya sifatida ishga tushiring.")
    print(PGVECTOR_SETUP_SQL)
