# ============================================================
# 1) Loose obyektlar sonini kuzatish
# ============================================================
$ git count-objects -v
count: 47
size: 188
in-pack: 0
packs: 0
size-pack: 0
prune-packable: 0
garbage: 0
size-garbage: 0
# "count: 47" — 47 ta loose obyekt, hali hech qanday pack yo'q.

$ find .git/objects -type f | grep -v pack | wc -l
47
# Aynan shu son — har bir loose obyekt alohida fayl.

# ============================================================
# 2) git gc ishga tushirish va natijani solishtirish
# ============================================================
$ git gc
Enumerating objects: 47, done.
Counting objects: 100% (47/47), done.
Delta compression using up to 8 threads
Compressing objects: 100% (40/40), done.
Writing objects: 100% (47/47), done.
Total 47 (delta 12), reused 0 (delta 0), pack-reused 0

$ git count-objects -v
count: 0
size: 0
in-pack: 47
packs: 1
size-pack: 62
prune-packable: 0
garbage: 0
size-garbage: 0
# "count: 0" — endi loose obyekt yo'q, hammasi "in-pack: 47" ichida.
# "size-pack: 62" (KB) — 188 KB'dan 62 KB'ga qisqardi (delta+zlib tufayli).

$ ls .git/objects/pack/
pack-3f8a91c2b7d6e5f4a3b2c1d0e9f8a7b6c5d4e3f2.idx
pack-3f8a91c2b7d6e5f4a3b2c1d0e9f8a7b6c5d4e3f2.pack

# ============================================================
# 3) Pack ichidan bitta obyektni topish (git kabi)
# ============================================================
$ git verify-pack -v .git/objects/pack/pack-3f8a91c2*.idx | head -8
9a8b7c1d... blob   1240 512 12
7b2e8d1f... commit 245  180 524
5a3d0b3e... blob   4    32   704 1 4a1c7f0e...
# Oxirgi qatorda "1 4a1c7f0e..." — bu obyekt DELTA sifatida saqlangan,
# ya'ni 4a1c7f0e obyektidan farq sifatida, to'liq nusxa emas.

$ git cat-file -p 5a3d0b3e
# Natija hali ham to'liq, tushunarli kontent — Git delta'ni "shaffof"
# ravishda ORQAGA yig'ib beradi, foydalanuvchi buni sezmaydi.

# ============================================================
# 4) Delta compression samarasini ko'rish (katta fayl misolida)
# ============================================================
$ for i in $(seq 1 20); do
    echo "qator $i: $(date)" >> big_log.txt
    git add big_log.txt
    git commit -q -m "big_log.txt: $i-o'zgarish"
  done
$ du -sh .git/objects   # gc'dan oldin
1.2M    .git/objects
$ git gc
$ du -sh .git/objects   # gc'dan keyin
84K     .git/objects
# 20 ta deyarli bir xil versiya endi bitta bazaviy nusxa + 19 ta kichik
# delta sifatida saqlanadi — sezilarli farq.

# ============================================================
# 5) Nomukammal narsa: dangling (ref'siz) obyektlar
# ============================================================
$ git commit --allow-empty -m "vaqtinchalik"
$ git reset --hard HEAD~1
$ git fsck --unreachable
unreachable commit a1b2c3d4e5f6...
# Bu commit endi hech qanday branch'dan yetib bo'lmaydi, lekin hali
# o'chirilmagan — chunki gc.pruneExpire (odatda 2 hafta) hali o'tmagan.

$ git gc --prune=now
$ git fsck --unreachable
# (bo'sh) — endi butunlay o'chirildi. DIQQAT: bu QAYTARIB BO'LMAYDIGAN amal.

# ============================================================
# 6) git repack — qo'lda maksimal siqish
# ============================================================
$ git repack -a -d --depth=250 --window=250
Enumerating objects: 512, done.
Counting objects: 100% (512/512), done.
Delta compression using up to 8 threads
Compressing objects: 100% (498/498), done.
Writing objects: 100% (512/512), done.
# -a: barcha obyektlarni bitta pack'ga; -d: eski pack fayllarni o'chirish;
# --depth/--window: delta qidiruvni chuqurroq va kengroq qilish (sekinroq,
# lekin yaxshiroq siqish) — odatda faqat release/CI serverida ishlatiladi.

$ du -sh .git/objects/pack/
38K     .git/objects/pack/
# Odatiy git gc'ga nisbatan biroz kichikroq, lekin ancha sekinroq ishladi.

# ============================================================
# 7) Bitmap index bilan pack yaratish
# ============================================================
$ git repack -a -d -b
$ ls .git/objects/pack/
pack-xxxx.bitmap  pack-xxxx.idx  pack-xxxx.pack
# .bitmap fayli — har bir commit uchun "undan yetib bo'ladigan obyektlar"
# ro'yxatini oldindan hisoblab qo'yadi, clone/fetch'ni tezlashtiradi.

# ============================================================
# 8) git prune — faqat yetib bo'lmaydigan loose obyektlarni tozalash
# ============================================================
$ git prune --expire=now
# git gc --prune=now'dan farqi: prune FAQAT tozalaydi, pack yaratmaydi;
# odatda git gc o'z ichida avtomatik chaqiradi, alohida kamdan-kam
# ishlatiladi (masalan disk joyi darhol kerak bo'lganda).
