
# Uchala variantning ham ulanish/qidiruv kodi qanday ko'rinishini
# solishtirish — konseptual, ishga tushirilmaydi (API kalitlar yo'q),
# lekin har birining haqiqiy Python client kutubxonasi shaklini aks ettiradi.

# ---------------------------------------------------------------------------
# 1) pgvector — oddiy SQL orqali, mavjud SQLAlchemy sessiyasidan foydalanib
# ---------------------------------------------------------------------------

PGVECTOR_EXAMPLE = '''
from sqlalchemy import text

async def search_pgvector(db, query_vector: list[float], k: int = 3):
    rows = await db.execute(
        text(
            "SELECT id, title, embedding <=> :qv AS distance "
            "FROM lesson_embeddings "
            "ORDER BY embedding <=> :qv LIMIT :k"
        ),
        {"qv": str(query_vector), "k": k},
    )
    return rows.fetchall()
'''

# ---------------------------------------------------------------------------
# 2) Chroma — o'z Python client kutubxonasi orqali (chromadb paketi)
# ---------------------------------------------------------------------------

CHROMA_EXAMPLE = '''
import chromadb

client = chromadb.PersistentClient(path="./chroma_data")
collection = client.get_or_create_collection("lessons")

collection.add(
    ids=["lesson_7", "lesson_11"],
    embeddings=[[0.1, 0.2, 0.3], [0.4, 0.1, 0.2]],
    documents=["Class ID haqida dars", "Display Flex haqida dars"],
)

results = collection.query(query_embeddings=[[0.12, 0.19, 0.28]], n_results=3)
'''

# ---------------------------------------------------------------------------
# 3) Pinecone — tashqi bulutli xizmat, o'z Python SDK'si orqali
# ---------------------------------------------------------------------------

PINECONE_EXAMPLE = '''
from pinecone import Pinecone

pc = Pinecone(api_key="...")
index = pc.Index("lessons-index")

index.upsert(vectors=[
    {"id": "lesson_7", "values": [0.1, 0.2, 0.3]},
    {"id": "lesson_11", "values": [0.4, 0.1, 0.2]},
])

matches = index.query(vector=[0.12, 0.19, 0.28], top_k=3)
'''


def print_comparison() -> None:
    print("=== pgvector (SQL, mavjud Postgres bazasida) ===")
    print(PGVECTOR_EXAMPLE)
    print("=== Chroma (mahalliy, chromadb paketi) ===")
    print(CHROMA_EXAMPLE)
    print("=== Pinecone (bulutli, tashqi xizmat) ===")
    print(PINECONE_EXAMPLE)
    print(
        "DIQQAT: uchala kod ham konseptual — API kalitlari/paketlar "
        "o'rnatilmagan, shuning uchun ishga tushirilmaydi. Maqsad — "
        "SHAKLNI solishtirish: pgvector oddiy SQL, Chroma mahalliy client, "
        "Pinecone tashqi bulutli SDK."
    )


if __name__ == "__main__":
    print_comparison()
