# ============================================================
# 1) test.yml — ushbu platformaning haqiqiy workflow fayli
#    (.github/workflows/test.yml, to'liq holicha)
# ============================================================
name: Tests

on:
  push:
    branches: ["**"]
  pull_request:
    branches: [master]

jobs:
  backend:
    name: Backend (pytest)
    runs-on: ubuntu-latest
    timeout-minutes: 10

    steps:
      - uses: actions/checkout@v4

      - name: Set up Python
        uses: actions/setup-python@v5
        with:
          python-version: "3.11"
          cache: pip
          cache-dependency-path: backend/requirements.txt

      - name: Install dependencies
        working-directory: backend
        run: pip install -r requirements.txt

      - name: Run tests
        working-directory: backend
        env:
          DATABASE_URL: sqlite+aiosqlite:///./test.db
          SECRET_KEY: test-secret-key-for-ci-only-not-used-in-prod
        run: python -m pytest tests/ -v --tb=short

  frontend:
    name: Frontend (Jest)
    runs-on: ubuntu-latest
    timeout-minutes: 10

    steps:
      - uses: actions/checkout@v4

      - name: Set up Node
        uses: actions/setup-node@v4
        with:
          node-version: "20"
          cache: npm
          cache-dependency-path: frontend/package-lock.json

      - name: Install dependencies
        working-directory: frontend
        run: npm ci --no-audit --no-fund

      - name: Run tests
        working-directory: frontend
        env:
          CI: "true"
        run: npx react-scripts test --watchAll=false --passWithNoTests

# ============================================================
# 2) Anatomiyani qatma-qat o'qish
# ============================================================
# name: Tests                    <- workflow darajasi: GitHub UI'da shu
#                                    nom "Actions" bo'limida ko'rinadi
#
# on: push / pull_request        <- workflow darajasi: qachon ishga tushadi
#                                    (keyingi darsda batafsil)
#
# jobs:                          <- ikkita mustaqil job: backend, frontend
#   backend:                     <- job darajasi
#     runs-on: ubuntu-latest     <- qaysi virtual mashinada
#     timeout-minutes: 10        <- job 10 daqiqadan ko'p ketsa, majburan
#                                    to'xtatiladi (osilib qolgan testni
#                                    abadiy kutmaslik uchun)
#     steps:                     <- step darajasi, KETMA-KET bajariladi
#       - uses: actions/checkout@v4          <- 1-step: tayyor action
#       - name: Set up Python                <- "name" ixtiyoriy — UI'da
#         uses: actions/setup-python@v5           chiroyli ko'rsatish uchun
#       - name: Install dependencies          <- 3-step: run bilan
#         run: pip install -r requirements.txt

# ============================================================
# 3) working-directory — nega backend/frontend prefiksi bor
# ============================================================
# actions/checkout@v4 BUTUN repozitoriyani (backend/ va frontend/ ikkalasini
# ham) runner'ga klonlaydi. Lekin backend job'ining "Install dependencies"
# step'i faqat backend/requirements.txt bilan ishlashi kerak — shuning uchun
# har bir run: qatoriga alohida "working-directory: backend" yozilgan.
# Buni yozmasangiz, `pip install -r requirements.txt` ildiz papkada
# requirements.txt qidiradi va topolmay, xato beradi.

$ ls .github/workflows/
deploy-backend.yml  deploy-frontend.yml  test.yml

# ============================================================
# 4) Workflow'ni GitHub UI'dan kuzatish
# ============================================================
# Reponing "Actions" tabiga o'ting -> "Tests" workflow'ini tanlang ->
# har bir push/PR uchun alohida "run" ro'yxatga olinadi. Har bir run ichida
# backend va frontend job'lari alohida ustunlarda, parallel progress bilan
# ko'rinadi. Muvaffaqiyatsiz step qizil X bilan, muvaffaqiyatli esa yashil
# tik bilan belgilanadi — 11-darsda buni disk qilishni chuqur o'rganamiz.

# ============================================================
# 5) Minimal, o'zingiz sinab ko'rish uchun workflow
# ============================================================
# .github/workflows/hello.yml sifatida saqlab, push qiling:
name: Hello CI

on: [push]

jobs:
  say-hello:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - run: echo "Salom, $GITHUB_ACTOR! Bu commit $GITHUB_SHA."
      - run: ls -la
