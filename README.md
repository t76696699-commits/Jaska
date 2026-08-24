# ============================================================
# 1) git bisect'ni workflow_dispatch orqali avtomatlashtirish
# ============================================================
name: Automated Bisect

on:
  workflow_dispatch:
    inputs:
      good_ref:
        description: "Oxirgi ma'lum YAXSHI commit/tag (masalan v1.2.0)"
        required: true
      bad_ref:
        description: "Ma'lum YOMON commit (masalan HEAD)"
        required: true
        default: HEAD

jobs:
  bisect:
    runs-on: ubuntu-latest
    timeout-minutes: 30
    steps:
      - uses: actions/checkout@v4
        with:
          fetch-depth: 0     # <- BUTUN tarix kerak, bisect uchun shart
                              #    (112-kurs 2-darsi: packfile/gc'ni eslang)

      - uses: actions/setup-python@v5
        with:
          python-version: "3.11"
          cache: pip
          cache-dependency-path: backend/requirements.txt

      - name: Install dependencies
        working-directory: backend
        run: pip install -r requirements.txt

      - name: Run automated bisect
        run: |
          git bisect start
          git bisect bad ${{ inputs.bad_ref }}
          git bisect good ${{ inputs.good_ref }}
          # git bisect run - har bir qadamda BUYRUQNI avtomatik bajaradi;
          # buyruq 0 qaytarsa "good", boshqa kod qaytarsa "bad" deb belgilaydi.
          git bisect run bash -c "cd backend && python -m pytest tests/test_regression.py -x -q"
          echo "Bisect natijasi:"
          git bisect log
          git bisect reset

# ============================================================
# 2) pre-commit hook + CI'dagi bir xil tekshiruv (112-kurs 8-darsi amaliyoti)
# ============================================================
# .pre-commit-config.yaml (mahalliy, --no-verify bilan chetlab o'tilishi mumkin)
repos:
  - repo: https://github.com/psf/black
    rev: 24.4.2
    hooks:
      - id: black
        language_version: python3.11

# test.yml'ga QO'SHILGAN yangi job (chetlab bo'lmaydigan ikkinchi qatlam):
jobs:
  format-check:
    name: Black formatting (chetlab bo'lmaydigan)
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-python@v5
        with:
          python-version: "3.11"
      - run: pip install black==24.4.2
      - working-directory: backend
        run: black --check .
# Mahalliy hook TEZ, lekin --no-verify bilan o'tkazib yuborilishi mumkin.
# CI job xuddi shu buyruqni ishlatadi, lekin HECH KIM uni chetlab o'ta
# olmaydi (8-darsdagi required status check bilan birlashtirilsa).

# ============================================================
# 3) Faqat SEMVER tag push qilinganda deploy (annotated tag amaliyoti)
# ============================================================
name: Deploy Backend On Release Tag

on:
  push:
    tags:
      - 'v[0-9]+.[0-9]+.[0-9]+'   # <- faqat v1.2.0 kabi ANIQ SEMVER teglar

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Confirm this is an annotated tag
        run: |
          git cat-file -t "$GITHUB_REF_NAME" || echo "Lightweight tag (obyekt emas)"
          # 112-kurs 0-darsi: annotated tag ALOHIDA obyekt, lightweight
          # tag esa shunchaki ref - shu farq shu yerda amaliy tekshiriladi.
      - name: Deploy (namuna)
        run: echo "Deploying release $GITHUB_REF_NAME to production"

# ============================================================
# 4) Monorepo: paths + matrix birlashuvi (kontseptual loyiha)
# ============================================================
jobs:
  detect-changes:
    runs-on: ubuntu-latest
    outputs:
      backend_changed: ${{ steps.filter.outputs.backend }}
      frontend_changed: ${{ steps.filter.outputs.frontend }}
    steps:
      - uses: actions/checkout@v4
      - uses: dorny/paths-filter@v3
        id: filter
        with:
          filters: |
            backend:
              - 'backend/**'
            frontend:
              - 'frontend/**'

  test-backend:
    needs: detect-changes
    if: needs.detect-changes.outputs.backend_changed == 'true'
    strategy:
      matrix:
        python-version: ["3.10", "3.11", "3.12"]
    runs-on: ubuntu-latest
    steps:
      - run: echo "Faqat backend/ o'zgarganda, faqat shu versiyada test"
# Bu - 1-darsdagi paths (faqat o'zgargan qismni aniqlash) va 3-darsdagi
# matrix (bir nechta versiyada test)ning MONOREPO SHAROITIDA birlashuvi -
# 112-kurs 11-darsidagi sparse-checkout g'oyasining CI strategiyasidagi
# ekvivalenti: "faqat kerakli qismga e'tibor qaratish".
