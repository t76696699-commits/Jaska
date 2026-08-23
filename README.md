# ============================================================
# 1) test.yml'ni matrix bilan kengaytirish (backend job)
# ============================================================
jobs:
  backend:
    name: Backend (pytest)
    runs-on: ubuntu-latest
    timeout-minutes: 10
    strategy:
      fail-fast: false
      matrix:
        python-version: ["3.10", "3.11", "3.12"]

    steps:
      - uses: actions/checkout@v4

      - name: Set up Python ${{ matrix.python-version }}
        uses: actions/setup-python@v5
        with:
          python-version: ${{ matrix.python-version }}
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
# Natijada Actions tabida "backend (3.10)", "backend (3.11)", "backend
# (3.12)" nomli UCHTA alohida job ko'rinadi - barchasi parallel.

# ============================================================
# 2) Ikki o'lchamli matrix: versiya x OS
# ============================================================
jobs:
  cross-platform-test:
    runs-on: ${{ matrix.os }}
    strategy:
      fail-fast: false
      matrix:
        os: [ubuntu-latest, windows-latest, macos-latest]
        python-version: ["3.10", "3.11"]
        exclude:
          # macOS runner'lari qimmatroq va sekinroq - faqat eng yangi
          # versiyani tekshiramiz, vaqt tejash uchun.
          - os: macos-latest
            python-version: "3.10"
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-python@v5
        with:
          python-version: ${{ matrix.python-version }}
      - run: pip install -r backend/requirements.txt
      - run: python -m pytest backend/tests/ -v
# Jami kombinatsiya: 3 OS x 2 versiya = 6, minus 1 exclude = 5 ta parallel job.

# ============================================================
# 3) include - matrix'ga qo'shimcha, boshqacha maydon qo'shish
# ============================================================
jobs:
  test:
    runs-on: ubuntu-latest
    strategy:
      matrix:
        python-version: ["3.10", "3.11", "3.12"]
        include:
          # Faqat 3.12 uchun qo'shimcha "experimental: true" belgisi
          - python-version: "3.12"
            experimental: true
    continue-on-error: ${{ matrix.experimental == true }}
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-python@v5
        with:
          python-version: ${{ matrix.python-version }}
      - run: pip install -r backend/requirements.txt
      - run: python -m pytest backend/tests/ -v
# continue-on-error: true bo'lsa, o'sha kombinatsiya muvaffaqiyatsiz
# bo'lsa ham butun workflow "muvaffaqiyatli" deb belgilanadi - eksperimental
# versiyalar uchun foydali (hali barqaror emas, lekin CI'ni to'xtatmasin).

# ============================================================
# 4) Natijaviy job nomlari GitHub UI'da qanday ko'rinadi
# ============================================================
# Actions tab -> "Tests" workflow run ->
#   backend (3.10)
#   backend (3.11)
#   backend (3.12)
#   frontend
# Har biri alohida progress-bar, alohida log, alohida muvaffaqiyat/
# muvaffaqiyatsizlik belgisi bilan.
