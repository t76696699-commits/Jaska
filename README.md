
# Token byudjetini dinamik boshqarish: chunk'larni cosine balliga ko'ra
# saralab, byudjetga sig'guncha qo'shib borish.

from __future__ import annotations


def estimate_tokens(text: str) -> int:
    """Taxminiy token soni — aniq tokenizator o'rniga tezkor baholash
    (~4 belgi = 1 token). Production'da tiktoken kabi haqiqiy
    tokenizatordan foydalaning; bu FAQAT tezkor taxmin."""
    return max(1, len(text) // 4)


def fit_chunks_to_budget(
    chunks: list[dict],
    *,
    token_budget: int,
    reserved_for_answer: int = 300,
    reserved_for_instruction: int = 60,
) -> list[dict]:
    """Chunk'larni (allaqachon cosine balliga ko'ra saralangan deb
    faraz qilinadi) YUQORIDAN pastga qarab qo'shib boradi, har safar
    joriy token yig'indisini tekshirib. Byudjetga sig'maydigan chunk
    uchrasa — TO'XTAYDI (keyingi, balki balandroq ballli bo'lmagan
    chunk'larni ham sinab ko'rmaydi — bu chunk'lar allaqachon ball
    bo'yicha saralangani uchun keyingilari ham kamroq mos)."""
    available = token_budget - reserved_for_answer - reserved_for_instruction
    if available <= 0:
        raise ValueError("token_budget juda kichik — javob va ko'rsatma uchun joy qolmadi")

    selected: list[dict] = []
    used = 0
    for chunk in chunks:
        chunk_tokens = estimate_tokens(chunk["text"])
        if used + chunk_tokens > available:
            break
        selected.append(chunk)
        used += chunk_tokens
    return selected


if __name__ == "__main__":
    # Cosine balliga ko'ra allaqachon saralangan chunk'lar namunasi:
    ranked_chunks = [
        {"heading": "1", "text": "A" * 800, "score": 0.91},   # ~200 token
        {"heading": "2", "text": "B" * 1200, "score": 0.85},  # ~300 token
        {"heading": "3", "text": "C" * 2000, "score": 0.60},  # ~500 token
        {"heading": "4", "text": "D" * 400, "score": 0.40},   # ~100 token
    ]

    # Kichik byudjet (masalan 700 token) bilan sinaymiz:
    fitted = fit_chunks_to_budget(ranked_chunks, token_budget=700)
    print(f"Byudjet=700 tokenda {len(fitted)} ta chunk sig'di:")
    for c in fitted:
        print(f"  heading={c['heading']} score={c['score']} ~{estimate_tokens(c['text'])} token")

    # Kattaroq byudjet bilan solishtirish:
    fitted_big = fit_chunks_to_budget(ranked_chunks, token_budget=2000)
    print(f"\nByudjet=2000 tokenda {len(fitted_big)} ta chunk sig'di.")
