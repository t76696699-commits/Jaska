# ============================================================
# 1) Composite action: test.yml'ning backend sozlash qismini o'rash
# ============================================================
# .github/actions/setup-backend/action.yml
name: 'Setup Backend'
description: 'Checkout + Python + bog\'liqliklarni o\'rnatish'
inputs:
  python-version:
    description: 'Python versiyasi'
    required: false
    default: '3.11'
runs:
  using: composite
  steps:
    - uses: actions/setup-python@v5
      with:
        python-version: ${{ inputs.python-version }}
        cache: pip
        cache-dependency-path: backend/requirements.txt
    - name: Install dependencies
      shell: bash
      working-directory: backend
      run: pip install -r requirements.txt
# Composite action ICHIDA "shell: bash" har bir run: step'i uchun ANIQ
# ko'rsatilishi SHART - oddiy workflow'dan farqli, bu yerda standart
# shell avtomatik tanlanmaydi.

# ============================================================
# 2) Composite action'ni test.yml ichida chaqirish
# ============================================================
jobs:
  backend:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: ./.github/actions/setup-backend    # <- BITTA qatorda
        with:                                     #    ikkita step o'rniga
          python-version: "3.11"
      - name: Run tests
        working-directory: backend
        env:
          DATABASE_URL: sqlite+aiosqlite:///./test.db
          SECRET_KEY: test-secret-key-for-ci-only-not-used-in-prod
        run: python -m pytest tests/ -v --tb=short

# ============================================================
# 3) Reusable workflow: butun backend test job'ini funksiya qilish
# ============================================================
# .github/workflows/_reusable-backend-test.yml
name: Reusable Backend Test

on:
  workflow_call:
    inputs:
      python-version:
        type: string
        default: "3.11"
    secrets:
      db-url:
        required: false

jobs:
  test:
    runs-on: ubuntu-latest
    timeout-minutes: 10
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-python@v5
        with:
          python-version: ${{ inputs.python-version }}
          cache: pip
          cache-dependency-path: backend/requirements.txt
      - working-directory: backend
        run: pip install -r requirements.txt
      - working-directory: backend
        env:
          DATABASE_URL: ${{ secrets.db-url || 'sqlite+aiosqlite:///./test.db' }}
          SECRET_KEY: test-secret-key-for-ci-only-not-used-in-prod
        run: python -m pytest tests/ -v --tb=short

# ============================================================
# 4) Reusable workflow'ni chaqiruvchi fayl
# ============================================================
# .github/workflows/test.yml (yangi versiya, reusable bilan)
name: Tests

on:
  push:
    branches: ["**"]
  pull_request:
    branches: [master]

jobs:
  backend:
    uses: ./.github/workflows/_reusable-backend-test.yml
    with:
      python-version: "3.11"
    secrets: inherit    # <- barcha secret'larni avtomatik uzatish

  frontend:
    name: Frontend (Jest)
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: "20"
          cache: npm
          cache-dependency-path: frontend/package-lock.json
      - working-directory: frontend
        run: npm ci --no-audit --no-fund
      - working-directory: frontend
        env:
          CI: "true"
        run: npx react-scripts test --watchAll=false --passWithNoTests
# Diqqat: reusable workflow chaqirilganda "backend" job'i endi
# `uses:` bilan yoziladi, "runs-on:"/"steps:" YO'Q - bular
# _reusable-backend-test.yml ICHIDA yashiringan.
