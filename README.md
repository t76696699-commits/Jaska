# ============================================================
# To'liq pipeline: har bir qismi qaysi darsdan (6-11) kelganini
# izohlarda ko'rsatadi
# ============================================================

# --- test.yml (6-dars: artifact + 9-dars: reusable workflow) ---
name: Tests

on:
  push:
    branches: ["**"]
  pull_request:
    branches: [master]

jobs:
  backend:
    uses: ./.github/workflows/_reusable-backend-test.yml   # <- 9-dars
    with:
      python-version: "3.11"
    secrets: inherit

  frontend:
    name: Frontend (Jest)
    runs-on: ubuntu-latest                                   # <- 10-dars: GitHub-hosted
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
      - name: Upload test artifacts on failure                # <- 6-dars
        if: failure()
        uses: actions/upload-artifact@v4
        with:
          name: frontend-test-logs
          path: frontend/coverage/

# --- Branch protection (8-dars, gh CLI orqali sozlangan) ---
# required_status_checks.contexts: ["Backend (pytest)", "Frontend (Jest)"]
# required_pull_request_reviews.required_approving_review_count: 1
# required_status_checks.strict: true (branch up-to-date bo'lishi shart)

# --- deploy-backend.yml (7-dars) ---
name: Deploy Backend
on:
  push:
    branches: [server]
    paths: ['backend/**']
concurrency:
  group: deploy-backend
  cancel-in-progress: false
jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - name: Configure SSH
        env:
          SSH_PRIVATE_KEY: ${{ secrets.SSH_PRIVATE_KEY }}
        run: |
          mkdir -p ~/.ssh && printf '%s\n' "$SSH_PRIVATE_KEY" > ~/.ssh/deploy_key
          chmod 600 ~/.ssh/deploy_key
      - name: Deploy and verify
        env:
          SSH_HOST: ${{ secrets.SSH_HOST }}
        run: |
          ssh -i ~/.ssh/deploy_key "$SSH_HOST" "systemctl restart backend && systemctl is-active --quiet backend"
          # Muvaffaqiyatsizlik holatida - 11-dars: log'ni o'qib,
          # "Permission denied" (secret muammosi) yoki "inactive"
          # (kod muammosi) ekanini aniqlash.

# ============================================================
# O'z-o'zini tekshirish savollariga qisqa javoblar
# ============================================================
# 1) actions/upload-artifact (build job'ida) + actions/download-artifact
#    (deploy job'ida), bir xil name: orqali bog'lanadi.
# 2) Backend serverda ISHLAYDI (systemd) - SSH orqali yangilanadi;
#    frontend STATIK fayl - CI'da qurilib, rsync qilinadi.
# 3) job'ning name: maydonidan; o'zgartirilsa, eski qoida hech qachon
#    mos kelmay, PR abadiy "kutilmoqda" holatida qoladi.
# 4) Composite action - step darajasida (bitta job ichida); reusable
#    workflow - butun job(lar) darajasida, o'z runner'i bilan.
# 5) Ishonchsiz tashqi PR kodi runner tarmog'ida bajarilishi mumkin.
# 6) SSH kaliti noto'g'ri/eskirgan yoki authorized_keys yangilanmagan.
