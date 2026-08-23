# ============================================================
# 1) deploy-backend.yml — secret'lardan foydalanish (to'liq oqim)
# ============================================================
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
               systemctl restart \"$SERVICE_NAME\""

      - name: Cleanup SSH key
        if: always()
        run: rm -f ~/.ssh/deploy_key
# "if: always()" — bu step HAR DOIM bajariladi, hattoki oldingi step'lar
# muvaffaqiyatsiz bo'lsa ham. Kalit fayli runner o'chganda baribir yo'qoladi
# (runner ephemeral), lekin aniq o'chirish - "belt-and-suspenders" ehtiyot
# chorasi, izohda aytilganidek.

# ============================================================
# 2) deploy-frontend.yml — oddiy (secret bo'lmagan) env o'zgaruvchilari
# ============================================================
- name: Build production bundle
  working-directory: frontend
  env:
    REACT_APP_API_URL: https://tech.gennis.uz/
    # CRA treats warnings as errors when CI=true. Runners set CI=true by
    # default, so we override.
    CI: 'false'
    NODE_OPTIONS: --max-old-space-size=4096
    GENERATE_SOURCEMAP: 'false'
  run: npm run build
# Bu qiymatlar YAML faylida OCHIQ - GitHub log'ida to'liq ko'rinadi,
# hech narsa maskировка qilinmaydi, chunki bular maxfiy emas.

# ============================================================
# 3) Secret qiymatini GitHub CLI orqali qo'shish
# ============================================================
$ gh secret set SSH_PRIVATE_KEY < ~/.ssh/id_deploy
$ gh secret set SSH_HOST --body "5.129.242.151"
$ gh secret list
NAME               UPDATED
BACKEND_DIR        2 days ago
SERVICE_NAME       2 days ago
SSH_HOST           2 days ago
SSH_PRIVATE_KEY    2 days ago
SSH_PORT           2 days ago
SSH_USER           2 days ago
# gh secret list qiymatlarni HECH QACHON ko'rsatmaydi - faqat nom va
# oxirgi yangilanish sanasi, bu qasddan shunday (secret write-only).

# ============================================================
# 4) GITHUB_TOKEN'ning avtomatik namunasi (bu repoda ishlatilmagan,
#    lekin har qanday workflow'da mavjud)
# ============================================================
jobs:
  comment-on-pr:
    runs-on: ubuntu-latest
    permissions:
      pull-requests: write   # GITHUB_TOKEN'ning ruxsatlarini cheklash/kengaytirish
    steps:
      - uses: actions/github-script@v7
        with:
          github-token: ${{ secrets.GITHUB_TOKEN }}
          script: |
            github.rest.issues.createComment({
              owner: context.repo.owner,
              repo: context.repo.repo,
              issue_number: context.issue.number,
              body: 'CI testlari muvaffaqiyatli o\'tdi!'
            })
# permissions: bloki bilan GITHUB_TOKEN'ning nima qila olishini
# minimal qilib cheklash - xavfsizlik amaliyoti (least privilege).

# ============================================================
# 5) Environment (production) bilan required reviewer namunasi
# ============================================================
jobs:
  deploy:
    runs-on: ubuntu-latest
    environment: production   # <- Settings > Environments'da sozlangan
                               #    "production" muhitiga bog'laydi
    steps:
      - run: echo "Bu step faqat tasdiqlangandan keyin ishga tushadi"
