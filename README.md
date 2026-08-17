# ════════════════════════════════════════════════════════════════════
# 7-BOSQICH (CAPSTONE YAKUNI): Deploy va CI chiqish kodi xatosi
# ════════════════════════════════════════════════════════════════════

# ─────────────────────────────────────────────────────────────────────
# 1) .github/workflows/test.yml - har push'da testlarni ishga tushirish
# ─────────────────────────────────────────────────────────────────────

# name: Test
# on: [push]
# jobs:
#   test:
#     runs-on: ubuntu-latest
#     steps:
#       - uses: actions/checkout@v4
#       - run: pip install -r requirements.txt
#       - run: pytest --cov=app --cov-fail-under=80

# ─────────────────────────────────────────────────────────────────────
# 2) .github/workflows/deploy.yml - FAQAT testlar o'tgandan keyin
# ─────────────────────────────────────────────────────────────────────

# jobs:
#   deploy:
#     needs: test
#     runs-on: ubuntu-latest
#     steps:
#       - run: ./deploy.sh

# ─────────────────────────────────────────────────────────────────────
# 3) run_tests.sh - TO'G'RI versiya
# ─────────────────────────────────────────────────────────────────────

#!/bin/bash
set -o pipefail

pytest --cov=app --cov-fail-under=80 | tee test_output.log
echo "Test skripti chiqish kodi: $?"


# ─────────────────────────────────────────────────────────────────────
# 4) Ataylab xato - pipefail'siz skript (izohda)
# ─────────────────────────────────────────────────────────────────────

# #!/bin/bash
# # set -o pipefail YO'Q!
#
# pytest --cov=app --cov-fail-under=80 | tee test_output.log
# echo "Testlar bajarildi"
# # Chiqish kodi HAR DOIM tee'nikiga teng (deyarli har doim 0) -
# # pytest ICHIDA muvaffaqiyatsiz testlar bo'lsa ham CI buni "yashil"
# # deb hisoblaydi va BUZILGAN kodni deploy qiladi.
