# ============================================================
# 1) Branch protection qoidasini GitHub CLI orqali o'rnatish
#    (test.yml'ning HAQIQIY job nomlari bilan)
# ============================================================
$ gh api repos/{owner}/{repo}/branches/master/protection \
  --method PUT \
  --field required_status_checks='{"strict":true,"contexts":["Backend (pytest)","Frontend (Jest)"]}' \
  --field enforce_admins=true \
  --field required_pull_request_reviews='{"required_approving_review_count":1}' \
  --field restrictions=null
# required_status_checks.strict: true - bu "Require branches to be up
# to date before merging" ga mos keladi.
# contexts - test.yml'dagi job "name:" maydonlaridan OLINGAN ANIQ matn -
# nomi bir harf farq qilsa ham, GitHub qoidani mos kelmagan deb hisoblaydi.

# ============================================================
# 2) .github/CODEOWNERS - muayyan yo'llar uchun majburiy tekshiruvchi
# ============================================================
# Global standart - reponing istalgan qismi uchun
* @backend-team-lead

# Backend model o'zgarishlari - faqat bazaviy tuzilishni biladigan odam
/backend/app/models/ @db-schema-owner

# Deploy workflow'lari - faqat DevOps mas'uli
/.github/workflows/deploy-*.yml @devops-lead

# Frontend komponentlar - frontend jamoasi
/frontend/src/components/ @frontend-team

# ============================================================
# 3) required_status_checks kontekstini test.yml'dan aniq olish
# ============================================================
# test.yml'dagi:
#   backend:
#     name: Backend (pytest)     <- bu "context" nomi
#   frontend:
#     name: Frontend (Jest)      <- bu ham "context" nomi
#
# Branch protection sozlamasida ANIQ shu ikki nom kiritilishi kerak -
# agar "name:" o'zgartirilsa (masalan "Backend Tests" deb), eski qoidada
# saqlangan "Backend (pytest)" endi HECH QACHON topilmaydi, va PR
# ABADIY "kutilmoqda" holatida qolib ketadi (bu keng tarqalgan xato -
# 11-darsda batafsil ko'ramiz).

# ============================================================
# 4) Rebase talab qilinishi - 112-kurs bilimi amaliyotda
# ============================================================
$ git fetch origin
$ git rebase origin/master
# Agar "Require branches to be up to date" yoqilgan bo'lsa, GitHub
# feature branch'ni PR oynasida "This branch is out-of-date with the
# base branch" deb ko'rsatadi - "Update branch" tugmasi (yoki qo'lda
# rebase) bosilmaguncha, hatto testlar avval o'tgan bo'lsa ham, Merge
# tugmasi ishlamaydi.

# ============================================================
# 5) Bypass qilishni butunlay o'chirish (eng qattiq siyosat)
# ============================================================
$ gh api repos/{owner}/{repo}/branches/master/protection \
  --method PUT \
  --field enforce_admins=true
# enforce_admins: true - "Do not allow bypassing the above settings"ga
# mos keladi. Bu yoqilgach, repo egasi HAM qoidalarni chetlab o'ta
# olmaydi - favqulodda holatda ham avval qoidani VAQTINCHA o'chirish
# kerak bo'ladi, bu esa qasddan qiyinlashtirilgan (audit uchun).
