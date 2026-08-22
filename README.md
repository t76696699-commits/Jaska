# ============================================================
# 1) Bu platformaning haqiqiy monorepo tuzilishi
# ============================================================
$ ls
alembic/  alembic.ini  backend/  docs/  frontend/  .github/
$ du -sh */
340M    backend/
180M    frontend/
2.1M    docs/
8.4M    alembic/

# ============================================================
# 2) Partial clone — faqat commit/tree, blob'lar keyinroq
# ============================================================
$ git clone --filter=blob:none --sparse \
    https://github.com/team/student_platform.git thin-clone
Cloning into 'thin-clone'...
remote: Enumerating objects: 15234, done.
remote: Total 15234 (delta 8821), reused 15100 (delta 8750)
Receiving objects: 100% (15234/15234), 12.4 MiB | ...
# Diqqat: 12.4 MiB — TO'LIQ clone bo'lsa 500+ MiB bo'lardi.
# Commit/tree tarixi TO'LIQ, lekin fayl kontenti (blob) hali yuklanmagan.

$ cd thin-clone
$ ls
# (bo'sh yoki minimal — sparse hali sozlanmagan)

# ============================================================
# 3) sparse-checkout: faqat kerakli papkalarni tanlash
# ============================================================
$ git sparse-checkout init --cone
$ git sparse-checkout set backend frontend
$ ls
backend/  frontend/
# docs/ va alembic/ diskda YO'Q, lekin hali obyektlar bazasida mavjud.

$ du -sh .git
45M .git
# To'liq clone'dagi bir necha yuz MB'ga solishtiring.

# ============================================================
# 4) Blob'lar "lazy" ravishda yuklanishi
# ============================================================
$ cat backend/app/models/course.py
remote: Enumerating objects: 3, done.
Receiving objects: 100% (3/3), 2.1 KiB | 850 KiB/s, done.
class Course(Base):
    __tablename__ = "courses"
    ...
# Fayl birinchi marta o'qilganda Git uni serverdan "on-demand" yuklab
# oldi — bu paytgacha faqat SHA-1 ma'lum edi, kontent yo'q edi.

# ============================================================
# 5) Kerak bo'lganda docs/ ni qaytarib qo'shish
# ============================================================
$ git sparse-checkout add docs
$ ls
backend/  docs/  frontend/
# Qayta klonlash SHART EMAS — docs/ obyektlari allaqachon .git/objects
# ichida bor edi (chunki partial clone faqat BLOB'larni kechiktirgan
# edi, tree'larni emas).

# ============================================================
# 6) Cone mode qanday sozlanganini tekshirish
# ============================================================
$ cat .git/info/sparse-checkout
/*
!/*/
/backend/
/docs/
/frontend/
$ git config core.sparseCheckoutCone
true

# ============================================================
# 7) To'liq clone bilan solishtirish jadvali
# ============================================================
# | Usul                                          | Vaqt | Hajm  |
# |------------------------------------------------|------|-------|
# | git clone (to'liq)                             | 45s  | 520MB |
# | --filter=blob:none --sparse + backend/frontend | 6s   | 45MB  |

# ============================================================
# 8) Shallow clone — tarix chuqurligini cheklash
# ============================================================
$ git clone --depth=1 https://github.com/team/student_platform.git shallow-clone
Cloning into 'shallow-clone'...
remote: Total 234 (delta 12)
Receiving objects: 100% (234/234), 8.2 MiB | ...

$ cd shallow-clone
$ git log --oneline
a1b2c3d (HEAD -> main) oxirgi commit
# ATIGI BITTA commit — qolgan tarix UMUMAN yuklanmagan.

$ git log --oneline HEAD~5
fatal: ambiguous argument 'HEAD~5': unknown revision
# Eski commit'larga umuman kira olmaymiz — bu partial clone'dan farq
# qiladi, u yerda tarix TO'LIQ, faqat blob kechiktiriladi.

# ============================================================
# 9) Shallow'ni keyinroq to'liq tarixga aylantirish
# ============================================================
$ git fetch --unshallow
remote: Enumerating objects: 15234, done.
Receiving objects: 100% (15000/15000), 480 MiB | ...
$ git log --oneline | wc -l
15234
# Endi to'liq tarix mavjud — lekin bu katta, sekin operatsiya (butun
# tarixni bir zumda yuklaydi).

# ============================================================
# 10) Qaysi birini tanlash — jadval
# ============================================================
# | Ehtiyoj                                    | Yechim          |
# |-----------------------------------------------|-------------------|
# | CI'da faqat oxirgi kodni build qilish          | shallow clone     |
# | Faqat backend/ bilan ishlash, TO'LIQ tarix     | partial+sparse    |
# | git blame/log to'liq tarix kerak               | partial clone     |
# | Disk joyi eng muhim, tarix umuman kerak emas   | shallow clone     |
