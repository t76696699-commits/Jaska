name: CI/CD Pipeline

on: [push]

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - name: Kodni yuklab olish
        uses: actions/checkout@v4

      - name: Python o'rnatish
        uses: actions/setup-python@v5
        with:
          python-version: '3.10'

      - name: Kutubxonalarni o'rnatish
        run: |
          python -m pip install --upgrade pip
          if [ -f requirements.txt ]; then pip install -r requirements.txt; fi

      - name: Testlarni ishga tushirish
        run: |
          chmod +x run_tests.sh
          ./run_tests.sh

  deploy:
    needs: test
    runs-on: ubuntu-latest
    steps:
      - name: Deploy jarayoni
        run: |
          echo "Testlar muvaffaqiyatli o'tdi! Serverga deploy qilinmoqda..." 


name: Test

on: [push]

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-python@v5
        with:
          python-version: '3.10'
      - run: pip install -r requirements.txt
      - run: |
          chmod +x run_tests.sh
          ./run_tests.sh

          name: Deploy

on:
  workflow_run:
    workflows: ["Test"]
    types: [completed]

jobs:
  deploy:
    runs-on: ubuntu-latest
    if: ${{ github.event.workflow_run.conclusion == 'success' }}
    steps:
      - run: echo "Deploying to production..."

      #!/bin/bash
set -o pipefail

pytest --cov=app --cov-fail-under=80 | tee test_output.log
echo "Test skripti chiqish kodi: $?"

venv/
__pycache__/
*.pyc
.pytest_cache/
.coverage
test_output.log
.env
instance/

git add .
git commit -m "feat: Add CI/CD workflows and run_tests script"
git push origin main
