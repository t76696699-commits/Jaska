# ============================================================
# Vague vs actionable izoh - bir xil xato uchun ikki xil yozuv
# ============================================================

CODE_UNDER_REVIEW = """
def get_first_answer(student_answer):
    return student_answer.split(",")[0]
"""

# --- VAGUE (noaniq) izoh ---
VAGUE_COMMENT = "Bu funksiya yaxshi emas."

# --- ACTIONABLE (aniq) izoh: NIMA + NEGA + NIMA QILISH ---
ACTIONABLE_COMMENT = """
get_first_answer bo'sh string (student_answer = "") kelsa
IndexError beradi, chunki split(",") natijasi [""] bo'ladi, lekin
[0] baribir ishlaydi - aslida muammo student_answer=None kelganda:
None.split() AttributeError beradi.

Taklif:
    def get_first_answer(student_answer):
        if not student_answer:
            return ""
        return student_answer.split(",")[0]
"""

# --- nit: prefiksi bilan bloklamaydigan izoh ---
NIT_COMMENT = "nit: `get_first_answer` o'rniga `get_first_selected_option` " \
              "nomi maqsadni aniqroq ifodalaydi (bloklamaydi, ixtiyoriy)."

# --- Ijobiy izoh (mustahkamlash) ---
POSITIVE_COMMENT = "Bu yerda `is_multiple_select` tekshiruvi chiroyli - " \
                    "chekka holatni aniq ajratgan."

print("=== Vague ===")
print(VAGUE_COMMENT)
print("\n=== Actionable ===")
print(ACTIONABLE_COMMENT)
print("\n=== Nit (bloklamaydi) ===")
print(NIT_COMMENT)
print("\n=== Ijobiy ===")
print(POSITIVE_COMMENT)


# ============================================================
# Izohni avtomatik "sifat darajasi"ga baholovchi sodda tekshiruv -
# uch mezon: aniq qator/holat, sabab, va ohang
# ============================================================
import re


def score_comment_quality(comment: str) -> dict:
    """Juda sodda evristika - real jamoada bu inson qarori, lekin
    asosiy signal turlarini ko'rsatish uchun foydali."""
    has_specific_reference = bool(re.search(r"\d+-qator|`\w+`", comment))
    has_reasoning = any(word in comment.lower() for word in ["chunki", "sabab", "agar"])
    is_harsh = any(word in comment.lower() for word in ["yomon", "yoqmadi"])
    return {
        "aniq_qatorga_ishora": has_specific_reference,
        "sabab_tushuntirilgan": has_reasoning,
        "keskin_ohang": is_harsh,
        "actionable_hisoblanadimi": has_specific_reference and has_reasoning and not is_harsh,
    }


for label, comment in [("Vague", VAGUE_COMMENT), ("Actionable", ACTIONABLE_COMMENT)]:
    print(f"\n{label}: {score_comment_quality(comment)}")
