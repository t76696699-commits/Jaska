# ============================================================
# 1) rerere'ni yoqish va birinchi konfliktni yechish
# ============================================================
$ git config rerere.enabled true
$ git rerere status
# hali hech narsa yo'q

$ git merge feature-a
Auto-merging config.py
CONFLICT (content): Merge conflict in config.py
Recorded preimage for 'config.py'
# "Recorded preimage" — rerere konfliktning KO'RINISHINI eslab qolayapti.

$ cat config.py
<<<<<<< HEAD
TIMEOUT = 30
=======
TIMEOUT = 60
>>>>>>> feature-a
$ vim config.py    # qo'lda TIMEOUT = 45 deb yechamiz
$ git add config.py
$ git commit
Recorded resolution for 'config.py'.
# "Recorded resolution" — bu safar YECHIMNI ham eslab qoldi.

# ============================================================
# 2) Xuddi shu konflikt qayta chiqsa — AVTOMATIK yechiladi
# ============================================================
$ git rebase main   # feature-a ni main ustiga qayta joylashtiramiz
Auto-merging config.py
CONFLICT (content): Merge conflict in config.py
Resolved 'config.py' using previous resolution.
# rerere DARHOL avvalgi yechimni qo'lladi — qayta qo'lda tuzatish shart emas!
$ git status --short
# (config.py allaqachon avtomatik yechilgan holda staged)
$ git rebase --continue

# ============================================================
# 3) rerere keshini ko'rish
# ============================================================
$ ls .git/rr-cache/
a3f291e8d7c6b5a4938271605f4e3d2c1b0a9f8/
$ ls .git/rr-cache/a3f291e8d7c6b5a4938271605f4e3d2c1b0a9f8/
postimage  preimage

# ============================================================
# 4) .gitattributes orqali custom merge driver
# ============================================================
$ cat .gitattributes
package-lock.json merge=ours
*.generated.json merge=ours

$ git config merge.ours.driver true
# "true" buyrug'i har doim 0 (muvaffaqiyat) qaytaradi -> HECH QANDAY
# birlashtirish qilinmaydi, joriy branch versiyasi saqlanadi.

$ git merge feature-b
Auto-merging package-lock.json
Merge made by the 'ort' strategy.
# package-lock.json'da konflikt BO'LSA HAM, u ko'rsatilmaydi —
# HAR DOIM bizning (HEAD) versiyamiz saqlanadi, boshqa tomon e'tiborsiz.

# ============================================================
# 5) -X ours vs merge=ours — MUHIM farq
# ============================================================
$ git merge -X ours feature-c
# config.py'da konflikt bo'lsa — FAQAT konflikt qatorlarida bizning
# versiyamiz tanlanadi, LEKIN feature-c'dagi BOSHQA (konfliktsiz)
# o'zgarishlar hali ham qo'shiladi:
$ git diff HEAD~1 --stat
config.py       | 2 +-
new_feature.py  | 15 +++++++++++++++    # <- bu hali ham qo'shildi!

# merge=ours drayveri esa BUTUN faylni e'tiborsiz qoldiradi:
$ git merge feature-d   # package-lock.json uchun merge=ours ishlaydi
$ git diff HEAD~1 -- package-lock.json
# (bo'sh — fayl UMUMAN o'zgarmadi, feature-d'dagi o'zgarishlar yo'qoldi)

# ============================================================
# 6) rerere'ni tozalash (eskirgan yechimlar to'planganda)
# ============================================================
$ git rerere gc
# gc.rerereResolved (odatda 60 kun) va gc.rerereUnresolved (15 kun)
# muddatidan o'tgan yozuvlarni tozalaydi.

# ============================================================
# 7) diff3 konflikt uslubi — umumiy ajdodni ko'rish
# ============================================================
$ git config merge.conflictStyle diff3
$ git merge feature-a
CONFLICT (content): Merge conflict in config.py

$ cat config.py
<<<<<<< HEAD
TIMEOUT = 45
||||||| a1b2c3d (umumiy ajdod)
TIMEOUT = 30
=======
TIMEOUT = 60
>>>>>>> feature-a
# Endi UCHTA versiya ko'rinadi: HEAD (45), asl ajdod (30), feature-a (60).
# Buni ko'rib, "ikkalasi ham asl 30'dan boshqacha yo'nalishda o'zgartirgan"
# ekanini tushunish osonlashadi — oddiy ikki tomonlama diff bunday
# kontekstni bermas edi.

# ============================================================
# 8) git mergetool — vizual yechim
# ============================================================
$ git config merge.tool vimdiff
$ git mergetool
Merging:
config.py

Normal merge conflict for 'config.py':
  {local}: modified file
  {base}: modified file
  {remote}: modified file
Hit return to start merge resolution tool (vimdiff):
# vimdiff to'rt panelli ko'rinishda ochiladi: LOCAL | BASE | MERGED | REMOTE
# Qo'lda kerakli qatorlarni tanlab, :wqa bilan saqlab chiqiladi.

$ git status --short
M  config.py    # mergetool orqali yechilgan, endi staged
$ git commit --no-edit
Recorded resolution for 'config.py'.
# rerere BU YERDA ham ishlaydi — mergetool orqali yechilgan konflikt ham
# keyingi safar avtomatik eslab qolinadi.
