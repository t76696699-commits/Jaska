# ============================================================
# Real misol: e6c19f2 commit tavsifidan (bu platformaning o'z
# git tarixi) - review nimani ushlab qolishi mumkin edi
# ============================================================

# --- MUAMMOLI KOD (merge qilingunga qadar hech kim savol bermagan) ---
def check_multiple_choice(exercise, student_answer):
    correct = exercise.correct_answers          # masalan: "Konsolga chiqaradi, aniq"
    correct_list = correct.split(",")           # <- HAR DOIM vergul bo'yicha boladi
    student_list = student_answer.split(",")
    return set(correct_list) == set(student_list)

# Agar to'g'ri javob matnining o'zida vergul bo'lsa (yuqoridagi misolda
# "Konsolga chiqaradi, aniq"), u ikkita bo'lakka bo'linadi:
#   ["Konsolga chiqaradi", " aniq"]
# Talaba xuddi shu variantni TO'LIQ tanlagan bo'lsa ham (bitta string
# sifatida), taqqoslash hech qachon mos kelmaydi - javob DOIM "noto'g'ri"
# deb belgilanadi, talaba nima tanlashidan qat'i nazar.

# --- TUZATILGAN KOD (commit e6c19f2'dan keyin) ---
def check_multiple_choice_fixed(exercise, student_answer):
    if exercise.is_multiple_select:
        correct_list = exercise.correct_answers.split(",")
        student_list = student_answer.split(",")
        return set(correct_list) == set(student_list)
    # Bitta tanlovli savolda MATNNI BUTUNLIGICHA solishtiramiz,
    # ichidagi vergulga qaramasdan
    return exercise.correct_answers.strip() == student_answer.strip()

# ============================================================
# Review paytida so'ralishi kerak bo'lgan savol (agar berilganida,
# bu xato ishlab chiqarishga yetib bormagan bo'lardi):
#
#   "correct_answers'ni vergul bo'yicha bo'lish nafaqat ko'p
#    tanlovli (is_multiple_select=True), balki BARCHA holatlar
#    uchun to'g'rimi? Bitta tanlovli javob matnida vergul bo'lsa
#    nima bo'ladi?"
#
# Bu savol CI (test.yml) tomonidan AVTOMATIK berilmaydi - faqat
# mavjud testlar tekshirgan holatlarni tasdiqlaydi. Yangi chekka
# holatni o'ylab topish - aynan inson review'ining vazifasi.
# ============================================================
