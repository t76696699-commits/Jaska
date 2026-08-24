# ============================================================
# 1) Log'dagi ::error:: annotatsiyasini o'qish
# ============================================================
# deploy-frontend.yml'dagi haqiqiy misol:
- name: Verify build artefact
  run: |
    test -f frontend/build/index.html || { echo "::error::build/index.html missing — aborting deploy"; exit 1; }

# Agar bu step muvaffaqiyatsiz bo'lsa, Actions UI'da:
#   [ERROR] build/index.html missing — aborting deploy
# qatori QIZIL rangda, alohida "Annotations" bo'limida ko'rinadi -
# butun log ichida qidirishning hojati yo'q.

# ============================================================
# 2) working-directory unutilgan xato (keng tarqalgan #1)
# ============================================================
# XATO (working-directory yo'q):
jobs:
  backend:
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-python@v5
        with:
          python-version: "3.11"
      - run: pip install -r requirements.txt   # <- YO'Q: working-directory!
# Xato xabari:
#   ERROR: Could not open requirements file: [Errno 2] No such file or
#   directory: 'requirements.txt'

# TO'G'RI:
      - working-directory: backend
        run: pip install -r requirements.txt

# ============================================================
# 3) env o'zgaruvchisi CI'da yo'qligi (keng tarqalgan #2)
# ============================================================
# Mahalliyda ishlaydi (.env fayli orqali DATABASE_URL bor), lekin CI'da
# .env fayli YO'Q (odatda .gitignore'da) - shuning uchun test.yml buni
# ANIQ env: orqali beradi:
- name: Run tests
  working-directory: backend
  env:
    DATABASE_URL: sqlite+aiosqlite:///./test.db
    SECRET_KEY: test-secret-key-for-ci-only-not-used-in-prod
  run: python -m pytest tests/ -v --tb=short
# Agar bu env: bloki YO'QOLSA, xato odatda:
#   KeyError: 'DATABASE_URL'  yoki  sqlalchemy.exc.ArgumentError

# ============================================================
# 4) SSH secret xatosi (keng tarqalgan #3)
# ============================================================
$ ssh -i ~/.ssh/deploy_key -o StrictHostKeyChecking=yes user@host "..."
# Muvaffaqiyatsiz bo'lsa:
Permission denied (publickey).
# Eng ko'p uchraydigan sabablar:
#  a) SSH_PRIVATE_KEY secret'i noto'liq nusxalangan (masalan oxirgi
#     bo'sh qator yo'qolgan) - printf '%s\n' ishlatilishi aynan shu
#     muammoni oldini olish uchun (deploy-backend.yml'da ko'rgandek)
#  b) Serverdagi ~/.ssh/authorized_keys hali yangi PUBLIC kalitni
#     o'z ichiga OLMAGAN
#  c) Xato foydalanuvchi nomi (SSH_USER) yoki xato host (SSH_HOST)

# ============================================================
# 5) gh CLI orqali muvaffaqiyatsiz run'ni tekshirish va qayta ishga tushirish
# ============================================================
$ gh run list --workflow=test.yml --limit 5
STATUS  TITLE                    WORKFLOW   BRANCH   EVENT  ID
X       fix: auth bug            Tests      server   push   123456789

$ gh run view 123456789 --log-failed
# Faqat MUVAFFAQIYATSIZ step'larning to'liq logini ko'rsatadi -
# muvaffaqiyatli step'larni skroll qilishga hojat yo'q.

$ gh run rerun 123456789 --failed
# Faqat muvaffaqiyatsiz job'larni qayta ishga tushiradi (muvaffaqiyatli
# job'larni qayta bajarmaydi - vaqt tejaydi).

# ============================================================
# 6) YAML'ni push qilishdan OLDIN mahalliyda tekshirish
# ============================================================
$ pip install yamllint
$ yamllint .github/workflows/test.yml
.github/workflows/test.yml
  12:9      error    wrong indentation: expected 10 but found 8  (indentation)

# Yoki tezroq, o'rnatishsiz:
$ python -c "import yaml; yaml.safe_load(open('.github/workflows/test.yml'))"
# Xato bo'lsa, aniq qator raqami bilan yaml.scanner.ScannerError chiqadi -
# bu push qilishdan oldin, hattoki GitHub'ga yuborishdan oldin xatoni
# ushlaydi.

$ gh workflow view test.yml
# Workflow GitHub tomonidan qabul qilinganini (sintaksis to'g'ri
# ekanini) tasdiqlaydi - agar fayl noto'g'ri bo'lsa, bu buyruq workflow
# ro'yxatda "disabled" yoki umuman ko'rinmasligi mumkinligini bildiradi.
