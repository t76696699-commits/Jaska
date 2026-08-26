# ============================================================
# Sintez: xayoliy "profilda ballar noto'g'ri ko'rsatilmoqda" bug'i
# uchun TO'LIQ ish jarayoni - 0-4-darslarning barchasi
# ============================================================

# --- 0-dars: muammo nega review talab qiladi ---
# CI (test.yml) mavjud testlarni tekshiradi, lekin "Rejected holatidagi
# loyihalar ball hisoblanishi kerakmi" degan yangi savolni CI o'zi
# o'ylab topmaydi - buni inson ko'rib chiqishi kerak.

# --- 3-dars: ikkita atomik commit (git add -p orqali ajratilgan) ---
# $ git add -p backend/app/api/v1/endpoints/students.py
# $ git commit -m "fix(students): filter project points by approved status"
#
# $ git add -p backend/app/services/exercise_service.py
# $ git commit -m "fix(scoring): guard multiple_choice grading against \
#   comma-containing single-select answers"

# --- 2-dars: xabarlar Conventional Commits formatiga mos ---
COMMITS = [
    "fix(students): filter project points by approved status",
    "fix(scoring): guard multiple_choice grading against comma-containing single-select answers",
]
for msg in COMMITS:
    assert ": " in msg and msg.split("(")[0] in {"fix", "feat", "chore", "refactor"}
print("Ikkala commit ham Conventional Commits formatiga mos:", COMMITS)

# --- 1-dars: PR tavsifi to'rt bo'lim bilan ---
PR_DESCRIPTION = """
## Kontekst
Talabalar profilida ko'rsatilgan ball haqiqiy hisoblangan balldan farq
qilishi haqida shikoyat tushdi.

## Nima o'zgardi
students.py'dagi profil statistikasi endi faqat Approved/Reviewed
holatidagi loyihalar ballarini hisoblaydi.

## Nega aynan shu yechim
Rejected loyihaning ball_earned qiymati hech qachon hamyonga
qo'shilmaydi - shu sababli ko'rsatilgan raqam ham shunga mos bo'lishi
kerak.

## Qanday tekshirish mumkin
pytest tests/test_students.py ishga tushiring - yangi test Rejected
loyiha ball hisobiga kirmasligini tasdiqlaydi.
"""

# --- 4-dars: reviewer to'rt ustuvorlik bo'yicha tekshiradi ---
REVIEW_RESULT = {
    "1_correctness": True,   # filtr to'g'ri holatlarni qamrab oladi
    "2_security": True,      # foydalanuvchi kiritishi ishtirok etmaydi
    "3_tests": True,         # yangi test qo'shilgan
    "4_readability": True,   # nomlar aniq
}
print("Review natijasi: Approved" if all(REVIEW_RESULT.values()) else "Changes requested")
