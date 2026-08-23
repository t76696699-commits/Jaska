# ============================================================
# Sintez workflow: 0-4-darslarning HAMMASI bitta faylda
# ============================================================
name: Full CI/CD Recap Demo
# ^ 0-dars: workflow darajasidagi name

on:                                    # <- 1-dars: trigger strategiyasi
  push:
    branches: [server]
    paths:
      - 'backend/**'
  workflow_dispatch:

concurrency:                           # <- 1-dars: bir vaqtda ikkita
  group: recap-deploy                  #    deploy to'qnashmasligi
  cancel-in-progress: false

jobs:
  test-and-deploy:
    name: Test (${{ matrix.python-version }}) then deploy
    runs-on: ubuntu-latest
    timeout-minutes: 15
    strategy:                          # <- 3-dars: matrix build
      fail-fast: false
      matrix:
        python-version: ["3.10", "3.11", "3.12"]

    steps:
      - uses: actions/checkout@v4      # <- 0-dars: uses bilan tayyor action

      - name: Set up Python ${{ matrix.python-version }}
        uses: actions/setup-python@v5
        with:
          python-version: ${{ matrix.python-version }}
          cache: pip                   # <- 4-dars: keshlash
          cache-dependency-path: backend/requirements.txt

      - name: Install dependencies      # <- 0-dars: run bilan shell buyrug'i
        working-directory: backend
        run: pip install -r requirements.txt

      - name: Run tests
        working-directory: backend
        env:
          DATABASE_URL: sqlite+aiosqlite:///./test.db
          SECRET_KEY: test-secret-key-for-ci-only-not-used-in-prod
        run: python -m pytest tests/ -v --tb=short

      - name: Configure SSH             # <- 2-dars: secret vs oddiy env
        if: matrix.python-version == '3.11'   # faqat bitta versiyada deploy
        env:
          SSH_PRIVATE_KEY: ${{ secrets.SSH_PRIVATE_KEY }}
          SSH_HOST:        ${{ secrets.SSH_HOST }}
        run: |
          mkdir -p ~/.ssh && chmod 700 ~/.ssh
          printf '%s\n' "$SSH_PRIVATE_KEY" > ~/.ssh/deploy_key
          chmod 600 ~/.ssh/deploy_key
          ssh-keyscan -H "$SSH_HOST" >> ~/.ssh/known_hosts 2>/dev/null

      - name: Cleanup SSH key
        if: always()
        run: rm -f ~/.ssh/deploy_key

# ============================================================
# Har bir savolga qisqa javob (o'z-o'zini tekshirish uchun)
# ============================================================
# 1) frontend/src/App.js -> server branch:
#    test.yml -> ISHGA TUSHADI (har qanday branch)
#    deploy-backend.yml -> ISHGA TUSHMAYDI (paths mos kelmadi)
#    deploy-frontend.yml -> ISHGA TUSHADI (paths=frontend/** mos keldi)
#
# 2) SSH_PRIVATE_KEY maxfiy (server kirish huquqi), NODE_OPTIONS oshkor
#    bo'lsa ham xavf yo'q (faqat build sozlamasi).
#
# 3) Ha, fail-fast: true (standart) bo'lsa, 3.10 xato bersa, 3.11/3.12
#    HAM darhol bekor qilinadi - shuning uchun ko'p loyihalar
#    fail-fast: false qo'yadi (3-darsni eslang).
#
# 4) Ha, o'zgaradi - kesh kaliti requirements.txt HASH'idan hisoblanadi,
#    fayl o'zgarsa hash ham o'zgaradi, demak eski kesh endi mos kelmaydi
#    (MISS), yangi kesh yaratiladi.
