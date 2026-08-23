# ============================================================
# 1) deploy-frontend.yml'ning haqiqiy verifikatsiya step'i
#    (artifact ISHLATILMAYDI, chunki bitta job ichida ketma-ket)
# ============================================================
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
# Bitta job ichida: build -> verify -> rsync. Artifact YO'Q, chunki
# hammasi bitta runner faylizmida, ketma-ket.

# ============================================================
# 2) Agar build va deploy IKKI job'ga bo'linsa - artifact KERAK bo'ladi
# ============================================================
jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: '20'
          cache: npm
          cache-dependency-path: frontend/package-lock.json
      - working-directory: frontend
        run: npm ci --no-audit --no-fund
      - working-directory: frontend
        env:
          REACT_APP_API_URL: https://tech.gennis.uz/
          CI: 'false'
          GENERATE_SOURCEMAP: 'false'
        run: npm run build
      - name: Verify build artefact
        run: test -f frontend/build/index.html || { echo "::error::missing"; exit 1; }

      # <- YANGI qadam: build natijasini artifact sifatida yuklash
      - name: Upload build artifact
        uses: actions/upload-artifact@v4
        with:
          name: frontend-build
          path: frontend/build/
          retention-days: 1

  deploy:
    needs: build          # <- deploy job build job tugashini kutadi
    runs-on: ubuntu-latest
    steps:
      # <- YANGI qadam: build job'ining artifact'ini shu job'ga tiklash
      - name: Download build artifact
        uses: actions/download-artifact@v4
        with:
          name: frontend-build
          path: frontend/build/

      - name: Configure SSH
        env:
          SSH_PRIVATE_KEY: ${{ secrets.SSH_PRIVATE_KEY }}
          SSH_HOST: ${{ secrets.SSH_HOST }}
        run: |
          mkdir -p ~/.ssh && chmod 700 ~/.ssh
          printf '%s\n' "$SSH_PRIVATE_KEY" > ~/.ssh/deploy_key
          chmod 600 ~/.ssh/deploy_key
          ssh-keyscan -H "$SSH_HOST" >> ~/.ssh/known_hosts 2>/dev/null

      - name: Rsync build to prod
        env:
          SSH_HOST: ${{ secrets.SSH_HOST }}
          SSH_USER: ${{ secrets.SSH_USER }}
        run: |
          rsync -avz --delete \
            -e "ssh -i ~/.ssh/deploy_key -o StrictHostKeyChecking=yes" \
            frontend/build/ "$SSH_USER@$SSH_HOST:/var/www/tech_gennis/frontend/build/"
# needs: build - artifact tayyor bo'lguncha deploy BOSHLANMAYDI.
# Bu ikki job'li bo'linish foydali: masalan, "build" job'ini alohida
# saqlab, bir necha marta "deploy"ni QAYTA ishga tushirish mumkin (11-dars
# - re-run) - build'ni har safar QAYTA yig'ish shart emas.

# ============================================================
# 3) Test hisobotini artifact sifatida saqlash (pytest bilan)
# ============================================================
- name: Run tests with report
  working-directory: backend
  run: python -m pytest tests/ -v --tb=short --junitxml=test-report.xml

- name: Upload test report
  if: always()          # <- testlar MUVAFFAQIYATSIZ bo'lsa ham hisobot saqlansin
  uses: actions/upload-artifact@v4
  with:
    name: pytest-report
    path: backend/test-report.xml
    retention-days: 14
# if: always() - test.yml'dagi "Cleanup SSH key" step'idagi bilan bir xil
# naqsh (2-dars): hisobot HAR DOIM saqlanishi kerak, test o'tdimi
# yoki yo'qmi - ayniqsa muvaffaqiyatsizlikni keyinroq tahlil qilish uchun.
