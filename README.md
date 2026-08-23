# ============================================================
# 1) test.yml'dagi o'rnatilgan keshlash (real, ikkala job)
# ============================================================
- name: Set up Python
  uses: actions/setup-python@v5
  with:
    python-version: "3.11"
    cache: pip
    cache-dependency-path: backend/requirements.txt

- name: Set up Node
  uses: actions/setup-node@v4
  with:
    node-version: "20"
    cache: npm
    cache-dependency-path: frontend/package-lock.json
# Ikkalasi ham "cache: <manager>" - GitHub qaysi papkani keshlashni
# (~/.cache/pip yoki node_modules manba kesh papkasini) o'zi biladi,
# faqat cache-dependency-path orqali qaysi fayl KESH KALITINI
# aniqlashini ko'rsatish kifoya.

# ============================================================
# 2) deploy-frontend.yml'dagi xuddi shu naqsh
# ============================================================
- name: Setup Node
  uses: actions/setup-node@v4
  with:
    node-version: '20'
    cache: npm
    cache-dependency-path: frontend/package-lock.json
# deploy-frontend.yml HAM xuddi shu keshlash strategiyasidan foydalanadi -
# har bir deploy'da npm ci'ni tezlashtirish uchun.

# ============================================================
# 3) actions/cache'ni to'g'ridan-to'g'ri ishlatish (qo'lda)
# ============================================================
- name: Cache pip packages manually
  uses: actions/cache@v4
  with:
    path: ~/.cache/pip
    key: ${{ runner.os }}-pip-${{ hashFiles('backend/requirements.txt') }}
    restore-keys: |
      ${{ runner.os }}-pip-
# key: aniq mos kelish uchun - hashFiles() requirements.txt kontentidan
# hash hisoblaydi (git obyektlarining SHA-1 kontent-addressing
# tamoyiliga o'xshash - 112-kurs 0-darsini eslang).
# restore-keys: aniq mos kelmasa (masalan requirements.txt biroz
# o'zgargan bo'lsa), eng yaqin mos keluvchi ESKI kesh'ni tiklaydi -
# to'liq bo'sh boshlashdan ko'ra tezroq.

# ============================================================
# 4) Playwright brauzerlarini keshlash namunasi (frontend E2E uchun)
# ============================================================
- name: Cache Playwright browsers
  uses: actions/cache@v4
  id: playwright-cache
  with:
    path: ~/.cache/ms-playwright
    key: playwright-${{ runner.os }}-${{ hashFiles('frontend/package-lock.json') }}

- name: Install Playwright browsers
  if: steps.playwright-cache.outputs.cache-hit != 'true'
  working-directory: frontend
  run: npx playwright install --with-deps
# id: bilan step natijasiga keyingi step'dan murojaat qilinadi.
# cache-hit != 'true' - agar kesh TOPILGAN bo'lsa, brauzerlarni QAYTA
# yuklab olishning hojati yo'q (yuklab olish o'zi bir necha daqiqa ketadi).

# ============================================================
# 5) Keshning haqiqiy tezlik farqi (namunaviy o'lchov)
# ============================================================
# Kesh YO'Q (MISS):    npm ci -> ~55 soniya (tarmoqdan yuklash)
# Kesh BOR (HIT):      npm ci -> ~8 soniya  (mahalliy nusxadan)
# Kesh YO'Q (MISS):    pip install -r requirements.txt -> ~30 soniya
# Kesh BOR (HIT):      pip install -r requirements.txt -> ~4 soniya
# Sonlar taxminiy - real vaqt tarmoq holati va paket sonига bog'liq,
# lekin nisbat (5-8x tezlashish) odatiy CI loyihalarida barqaror kuzatiladi.

# ============================================================
# 6) gh CLI orqali repozitoriy kesh'larini ko'rish va o'chirish
# ============================================================
$ gh cache list
ID      KEY                                              SIZE     CREATED
123456  Linux-pip-3a7f9e2b1c...                          45 MB    2 hours ago
123457  Linux-npm-9d8c7b6a5f...                          120 MB   1 day ago
123458  playwright-Linux-2e1d0c9b...                     310 MB   3 days ago

$ gh cache delete 123458
# Eskirgan yoki noto'g'ri kesh'ni qo'lda o'chirish - masalan, kesh
# buzilgan (corrupt) bo'lib qolsa yoki 10 GB limitiga yaqinlashilsa.

$ gh cache delete --all
# BARCHA repozitoriy kesh'larini tozalash - keyingi run to'liq MISS
# bo'ladi, lekin bu xavfsiz: build hech qachon shu sababdan sinmaydi.

# ============================================================
# 7) Docker layer keshlash - boshqa turdagi kesh, xuddi shu tamoyil
# ============================================================
- name: Set up Docker Buildx
  uses: docker/setup-buildx-action@v3

- name: Build with layer cache
  uses: docker/build-push-action@v5
  with:
    context: ./backend
    push: false
    cache-from: type=gha
    cache-to: type=gha,mode=max
# type=gha - GitHub Actions'ning o'z kesh backend'idan foydalanish,
# xuddi actions/cache kabi, lekin Docker qatlamlari (layer) uchun
# maxsuslashtirilgan. Har bir o'zgarmagan Dockerfile qatlami qayta
# qurilmaydi - faqat o'zgargan qatlamdan boshlab qayta quriladi.

# ============================================================
# 8) Kesh o'lchamini kuzatish - GitHub UI orqali
# ============================================================
# Repo -> Settings -> Actions -> Caches bo'limida:
#   - Har bir kesh yozuvi: kalit, hajm, oxirgi ishlatilgan vaqt
#   - Umumiy sig'im chizig'i (10 GB'ga nisbatan foiz)
#   - "Delete" tugmasi - qo'lda tozalash uchun
# Bu bo'lim ayniqsa ko'p matrix kombinatsiyasi ishlatilganda foydali -
# har bir kombinatsiya o'z alohida kesh yozuvini yaratishi mumkin, va
# ular jamlanib tez orada 10 GB limitiga yaqinlashishi mumkin.
