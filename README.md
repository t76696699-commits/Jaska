# ============================================================
# Bitta o'zgarish (2096b0e), ikki tavsif - solishtiring
# ============================================================

# --- HAQIQIY TAVSIF (bu repozitoriyaning o'z commit tarixidan) ---
REAL_DESCRIPTION = """
feat(scripts): add reusable course_builder library + generic scripts

Replaces the "copy a 2,000+ line seed script per course" pattern with a
shared library plus small, generic, per-concern scripts driven by a
course spec module (pure data - no DB code):

  create_course.py       - Course row (idempotent by title)
  create_lessons.py      - Lesson rows (idempotent by course_id+order)
  create_exercises.py    - Exercise rows + sections_json exercise stubs
  create_samples.py      - LessonSample rows (namuna)
  set_submission_tasks.py - UPDATE task_* columns (not a separate table)
  translate_lessons_ru.py / translate_exercises_ru.py - RU content
  build_course.py         - runs all of the above in the correct order,
                            then check_exercise_integrity + check_ru_coverage

Exercises are stored in sections_json as bare {"id": N} stubs rather than
full snapshots: a full embed is a frozen copy that silently goes stale,
a stub is always hydrated fresh at request time.

Verified end-to-end via build_course.py against production (committed,
checked, then cleaned up).
"""

# --- ZAIFLASHTIRILGAN TAVSIF (HAQIQIY EMAS - faqat solishtirish uchun,
#     xuddi shu o'zgarish uchun ko'p ko'riladigan zaif variant) ---
WEAK_DESCRIPTION = """
refactor: update course scripts

Cleaned up the scripts folder a bit. Added some new files for building
courses. Should work now.
"""

# ============================================================
# Reviewer nuqtai nazaridan farq:
#
# REAL_DESCRIPTION o'qigandan keyin reviewer BILADI: (1) qanday muammo
# hal qilinmoqda (2000+ qatorli nusxalash), (2) qaysi fayllar qo'shilgan
# va har biri nima qiladi, (3) NEGA stub yondashuvi tanlangan (frozen
# copy emas, har doim yangi hydrate), (4) qanday tasdiqlangan (production
# build orqali).
#
# WEAK_DESCRIPTION o'qigandan keyin reviewer HECH NARSA bilmaydi: "bir
# oz" tozalangan - qancha? "ba'zi" fayllar - qaysi va nima uchun? "ishlashi
# kerak" - qanday tekshirilgan, umuman tekshirilganmi? Reviewer endi BUTUN
# diff'ni o'zi, hech qanday yo'l-yo'riqsiz o'qib chiqishga majbur.
# ============================================================
