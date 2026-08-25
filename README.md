# ============================================================
# Review checklist - real PR uchun to'rt ustuvorlik bo'yicha
# tekshiruv (bu platformaning e6c19f2 xatosi misolida)
# ============================================================

REVIEW_CHECKLIST = {
    "1_correctness": [
        "Chekka holatlar (bo'sh, None, 0, juda katta qiymat) ko'rib chiqilganmi?",
        "Taxmin (masalan 'har doim vergul bo'yicha bo'linadi') HAR DOIM to'g'rimi,"
        " yoki faqat ba'zi holatlarda?",
        "Race condition (bir vaqtda ikki so'rov) bo'lishi mumkinmi?",
    ],
    "2_security": [
        "Foydalanuvchi kiritgan ma'lumot to'g'ridan-to'g'ri SQL/shell buyrug'iga qo'shilmayaptimi?",
        "Maxfiy kalit (API key, parol) kodda qattiq yozilmaganmi?",
        "Foydalanuvchi HTML'i tozalanmasdan chiqarilmayaptimi (XSS)?",
    ],
    "3_tests": [
        "Yangi xatti-harakat uchun test bormi?",
        "Test faqat 'muvaffaqiyatli' holatni emas, chekka holatni ham tekshiradimi?",
        "Agar bu funksiya buzilsa, test buni ushlab qoladimi?",
    ],
    "4_readability": [
        "O'zgaruvchi va funksiya nomlari mazmunga mos keladimi?",
        "Funksiya bir vaqtning o'zida bir nechta ishni qilmayaptimi?",
        "(BU band uslub emas - agar faqat qavs/bo'sh joy bo'lsa, nit: bilan belgilang)",
    ],
}


def review_pr(diff_summary: dict) -> str:
    """Sodda simulyatsiya: birinchi uchta band bo'yicha muammo topilsa,
    PR bloklanadi; to'rtinchisi faqat nit: sifatida qoldiriladi."""
    for priority in ("1_correctness", "2_security", "3_tests"):
        if diff_summary.get(priority) is False:
            return f"Changes requested - {priority} bo'yicha muammo bor"
    if diff_summary.get("4_readability") is False:
        return "Approved (nit: readability bo'yicha ixtiyoriy izoh qoldirildi)"
    return "Approved"


# e6c19f2'dagi grading bug'i review qilinganda TO'G'RILIK bandida
# aniqlanishi kerak edi:
example_diff = {"1_correctness": False, "2_security": True, "3_tests": True, "4_readability": True}
print(review_pr(example_diff))
# Natija: "Changes requested - 1_correctness bo'yicha muammo bor"
