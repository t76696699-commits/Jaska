# ============================================================
# 1) test.yml trigger qismi — keng, HAR QANDAY branch
# ============================================================
on:
  push:
    branches: ["**"]
  pull_request:
    branches: [master]
# "**" glob naqshi — feature/xyz, bugfix/abc, hattoki ismsiz branch'lar
# ham mos keladi. pull_request faqat master'ga yo'naltirilgan PR'lar uchun.

# ============================================================
# 2) deploy-backend.yml trigger qismi — tor, faqat backend/** + server branch
# ============================================================
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

# ============================================================
# 3) deploy-frontend.yml trigger qismi — o'ziga xos izoh bilan
# ============================================================
on:
  push:
    branches: [server]
    paths:
      - 'frontend/**'
      - '.github/workflows/deploy-frontend.yml'
  workflow_dispatch:
    # Lets you click "Run workflow" in the Actions tab — handy for forcing
    # a rebuild without pushing a code change (e.g. after rotating a key).

concurrency:
  group: deploy-frontend
  cancel-in-progress: false

# ============================================================
# 4) Amaliy stsenariy: nechta workflow ishga tushadi?
# ============================================================
# Stsenariy A: `feature/login` branch'iga faqat frontend/src/App.js
# o'zgartirilib push qilinsa:
#   - test.yml    -> ISHGA TUSHADI (push: branches ["**"])
#   - deploy-backend.yml -> ISHGA TUSHMAYDI (branch server emas)
#   - deploy-frontend.yml -> ISHGA TUSHMAYDI (branch server emas)
#
# Stsenariy B: `server` branch'iga faqat backend/app/main.py o'zgartirilib
# push qilinsa:
#   - test.yml    -> ISHGA TUSHADI (har qanday branch)
#   - deploy-backend.yml -> ISHGA TUSHADI (branch=server, paths=backend/**)
#   - deploy-frontend.yml -> ISHGA TUSHMAYDI (paths mos kelmadi)
#
# Stsenariy C: `server` branch'iga backend/ VA frontend/ ikkalasi ham
# bitta commit'da o'zgartirilib push qilinsa:
#   - test.yml    -> ISHGA TUSHADI
#   - deploy-backend.yml  -> ISHGA TUSHADI
#   - deploy-frontend.yml -> ISHGA TUSHADI (ikkalasi HAM parallel, lekin
#     ularning ICHIDAGI concurrency group'lari alohida — bir-birini
#     to'sib qo'ymaydi, faqat OʻZ turidagi ikkinchi runni to'sadi)

# ============================================================
# 5) workflow_dispatch'ga kirish parametri qo'shish (kengaytma)
# ============================================================
on:
  workflow_dispatch:
    inputs:
      environment:
        description: 'Qaysi muhitga deploy qilish'
        required: true
        default: 'production'
        type: choice
        options: [production, staging]
# Ishga tushirilganda GitHub UI kirish maydonini so'raydi;
# ${{ github.event.inputs.environment }} orqali workflow ichida o'qiladi.

# ============================================================
# 6) schedule bilan kunlik vazifa (hozircha bu repoda ishlatilmagan)
# ============================================================
on:
  schedule:
    - cron: '0 3 * * *'   # har kuni soat 03:00 UTC da
# Cron formati: daqiqa soat kun-oy oy hafta-kuni.
# GitHub'ning o'z jadvali biroz kechikishi mumkin (yuklama past bo'lganda
# yaqinroq, yuqori bo'lganda 10-15 daqiqagacha kechikishi mumkin) —
# aniq vaqtga tayanadigan muhim vazifalar uchun bu cheklovni bilib qo'ying.
