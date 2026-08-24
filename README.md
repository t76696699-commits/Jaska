# ============================================================
# 1) deploy-backend.yml - to'liq, real fayl
# ============================================================
name: Deploy Backend

on:
  push:
    branches: [server]
    paths:
      - 'backend/**'
      - '.github/workflows/deploy-backend.yml'
  workflow_dispatch:

concurrency:
  group: deploy-backend
  cancel-in-progress: false

jobs:
  deploy:
    name: Pull & restart backend
    runs-on: ubuntu-latest
    timeout-minutes: 15

    steps:
      - name: Configure SSH
        env:
          SSH_PRIVATE_KEY: ${{ secrets.SSH_PRIVATE_KEY }}
          SSH_HOST:        ${{ secrets.SSH_HOST }}
          SSH_PORT:        ${{ secrets.SSH_PORT }}
        run: |
          mkdir -p ~/.ssh && chmod 700 ~/.ssh
          printf '%s\n' "$SSH_PRIVATE_KEY" > ~/.ssh/deploy_key
          chmod 600 ~/.ssh/deploy_key
          PORT="${SSH_PORT:-22}"
          ssh-keyscan -p "$PORT" -H "$SSH_HOST" >> ~/.ssh/known_hosts 2>/dev/null
          chmod 644 ~/.ssh/known_hosts

      - name: Pull latest code and restart service
        env:
          SSH_HOST:     ${{ secrets.SSH_HOST }}
          SSH_USER:     ${{ secrets.SSH_USER }}
          SSH_PORT:     ${{ secrets.SSH_PORT }}
          BACKEND_DIR:  ${{ secrets.BACKEND_DIR }}
          SERVICE_NAME: ${{ secrets.SERVICE_NAME }}
        run: |
          PORT="${SSH_PORT:-22}"
          ssh -i ~/.ssh/deploy_key -p "$PORT" -o StrictHostKeyChecking=yes \
              "$SSH_USER@$SSH_HOST" \
              "set -e
               git -C \"$BACKEND_DIR\" pull origin server
               cd \"$BACKEND_DIR\"
               source venv/bin/activate
               pip install -r requirements.txt --quiet
               systemctl restart \"$SERVICE_NAME\"
               systemctl is-active --quiet \"$SERVICE_NAME\" && echo 'Service is running' || { echo 'Service failed to start'; systemctl status \"$SERVICE_NAME\" --no-pager; exit 1; }"

      - name: Cleanup SSH key
        if: always()
        run: rm -f ~/.ssh/deploy_key

# ============================================================
# 2) deploy-frontend.yml - to'liq, real fayl
# ============================================================
name: Deploy Frontend

on:
  push:
    branches: [server]
    paths:
      - 'frontend/**'
      - '.github/workflows/deploy-frontend.yml'
  workflow_dispatch:

concurrency:
  group: deploy-frontend
  cancel-in-progress: false

jobs:
  build-and-deploy:
    runs-on: ubuntu-latest
    timeout-minutes: 25

    steps:
      - name: Checkout
        uses: actions/checkout@v4

      - name: Setup Node
        uses: actions/setup-node@v4
        with:
          node-version: '20'
          cache: npm
          cache-dependency-path: frontend/package-lock.json

      - name: Install dependencies
        working-directory: frontend
        run: npm ci --no-audit --no-fund

      - name: Build production bundle
        working-directory: frontend
        env:
          REACT_APP_API_URL: https://tech.gennis.uz/
          CI: 'false'
          NODE_OPTIONS: --max-old-space-size=4096
          GENERATE_SOURCEMAP: 'false'
        run: npm run build

      - name: Verify build artefact
        run: |
          test -f frontend/build/index.html || { echo "::error::build/index.html missing — aborting deploy"; exit 1; }
          echo "Build size:"
          du -sh frontend/build
          ls -la frontend/build

      - name: Configure SSH
        env:
          SSH_PRIVATE_KEY: ${{ secrets.SSH_PRIVATE_KEY }}
          SSH_HOST:        ${{ secrets.SSH_HOST }}
          SSH_PORT:        ${{ secrets.SSH_PORT }}
        run: |
          mkdir -p ~/.ssh
          chmod 700 ~/.ssh
          printf '%s\n' "$SSH_PRIVATE_KEY" > ~/.ssh/deploy_key
          chmod 600 ~/.ssh/deploy_key
          PORT="${SSH_PORT:-22}"
          ssh-keyscan -p "$PORT" -H "$SSH_HOST" >> ~/.ssh/known_hosts 2>/dev/null
          chmod 644 ~/.ssh/known_hosts

      - name: Rsync build to prod
        env:
          SSH_HOST: ${{ secrets.SSH_HOST }}
          SSH_USER: ${{ secrets.SSH_USER }}
          SSH_PORT: ${{ secrets.SSH_PORT }}
        run: |
          PORT="${SSH_PORT:-22}"
          rsync -avz --delete \
            -e "ssh -i ~/.ssh/deploy_key -p $PORT -o StrictHostKeyChecking=yes" \
            frontend/build/ \
            "$SSH_USER@$SSH_HOST:/var/www/tech_gennis/frontend/build/"

      - name: Cleanup SSH key
        if: always()
        run: rm -f ~/.ssh/deploy_key

# ============================================================
# 3) Nima uchun rsync'dagi oxirgi slash MUHIM (amaliy tajriba)
# ============================================================
# TO'G'RI (slash BILAN manba yo'lida):
#   rsync frontend/build/ user@host:/var/www/site/build/
#   -> /var/www/site/build/index.html  (TO'G'RI)
#
# NOTO'G'RI (slashSIZ):
#   rsync frontend/build user@host:/var/www/site/build/
#   -> /var/www/site/build/build/index.html  (ICHMA-ICH, NOTO'G'RI!)
#   Sayt "404 Not Found" beradi, chunki server index.html'ni
#   /var/www/site/build/ ichida emas, build/build/ ichida qidiradi.

# ============================================================
# 4) systemctl tekshiruvining ahamiyati - deploy "muvaffaqiyatli"
#    signalini yolg'ondan bermaslik
# ============================================================
$ systemctl restart student-platform-backend
$ systemctl is-active --quiet student-platform-backend && echo OK || echo FAIL
# Agar restart buyrug'i "muvaffaqiyatli" qaytsa-yu, lekin xizmat DARHOL
# yiqilib qolsa (masalan .env faylida SECRET_KEY yo'q bo'lsa), shu
# tekshiruvsiz workflow baribir "yashil" bo'lib qolardi - eng xavfli
# yolg'on signal turi.
