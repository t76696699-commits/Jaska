# ============================================================
# 1) .git/ papkasini birinchi marta ko'rish
# ============================================================
$ git init sandbox && cd sandbox
$ echo "salom dunyo" > hello.txt
$ ls -la .git/
# HEAD  config  description  hooks/  info/  objects/  refs/
$ cat .git/HEAD
ref: refs/heads/main
# HEAD hali hech qanday branch fayliga ishora qilmaydi, chunki
# refs/heads/main hali yaratilmagan — birinchi commit'gacha.

# ============================================================
# 2) Blob'ni qo'lda yaratish — git add'siz
# ============================================================
$ git hash-object -w hello.txt
5a3d0b3e6e6b0b3f5e9c8a1d2e4f6a7b8c9d0e1f
# -w kaliti obyektni HAQIQATDA .git/objects/ ichiga yozadi.
# Bu SHA-1 faqat "blob 12\0salom dunyo\n" kontentidan hisoblangan —
# fayl nomi "hello.txt" bu hisoblashda umuman ishtirok etmaydi!

$ git cat-file -t 5a3d0b3
blob
$ git cat-file -p 5a3d0b3
salom dunyo

# ============================================================
# 3) Ikkita bir xil fayl — bitta blob (deduplikatsiya)
# ============================================================
$ mkdir -p a b
$ echo "salom dunyo" > a/hello.txt
$ echo "salom dunyo" > b/hello.txt
$ git hash-object a/hello.txt
5a3d0b3e6e6b0b3f5e9c8a1d2e4f6a7b8c9d0e1f
$ git hash-object b/hello.txt
5a3d0b3e6e6b0b3f5e9c8a1d2e4f6a7b8c9d0e1f
# Ikkalasi AYNAN bir xil SHA-1 — Git ikkalasi uchun BITTA blob saqlaydi,
# ikki marta emas. Kontent bir xil bo'lsa, joylashuv muhim emas.

# ============================================================
# 4) Tree'ni qo'lda qurish
# ============================================================
$ git update-index --add --cacheinfo 100644 5a3d0b3e6e6b0b3f5e9c8a1d2e4f6a7b8c9d0e1f hello.txt
$ git write-tree
def4567890abcdef1234567890abcdef12345678
$ git cat-file -p def4567
100644 blob 5a3d0b3e6e6b0b3f5e9c8a1d2e4f6a7b8c9d0e1f    hello.txt
# Tree — mode + turi + SHA-1 + nom ro'yxati. Nom AYNAN shu yerda saqlanadi,
# blob'da emas — shuning uchun bir xil blob turli nomlar bilan turli
# tree'larda ishlatilishi mumkin.

# ============================================================
# 5) Commit'ni qo'lda qurish
# ============================================================
$ echo "Birinchi commit" | git commit-tree def4567890abcdef1234567890abcdef12345678
abc123def4567890abcdef1234567890abcdef12
$ git cat-file -p abc123d
tree def4567890abcdef1234567890abcdef12345678
author Ism Familiya <email@example.com> 1700000000 +0500
committer Ism Familiya <email@example.com> 1700000000 +0500

Birinchi commit
# E'tibor bering: bu yerda "parent" qatori YO'Q — bu ILDIZ commit.
# Ikkinchi commit qo'shilsa, u "parent abc123d..." qatoriga ega bo'ladi.

# ============================================================
# 6) Bitta bayt o'zgarsa — butunlay boshqa SHA-1
# ============================================================
$ echo "salom dunyo!" > hello.txt   # oxiriga "!" qo'shildi
$ git hash-object hello.txt
f19e02c4a8b7d6e5f4a3b2c1d0e9f8a7b6c5d4e3
# Butunlay boshqa xesh — birgina belgi ham butun SHA-1'ni o'zgartiradi.
# Shu sababli tarixni "orqaga qaytarib tuzatish" darhol sezilib qoladi:
# o'sha commit'dan keyingi HAR BIR commit'ning SHA-1'i ham o'zgaradi.

# ============================================================
# 7) refs/heads/ — branch shunchaki fayl ekanini isbotlash
# ============================================================
$ mkdir -p .git/refs/heads
$ echo "abc123def4567890abcdef1234567890abcdef12" > .git/refs/heads/main
$ cat .git/refs/heads/main
abc123def4567890abcdef1234567890abcdef12
$ git log --oneline
abc123d Birinchi commit
# Endi "main" branch mavjud — biz uni git branch orqali EMAS, oddiy
# `echo` bilan fayl yozib yaratdik. Keyingi darsda buni chuqurroq ko'ramiz.

# ============================================================
# 8) Real loyihada obyektlarni ko'rish
# ============================================================
$ cd /home/user/student_platform
$ git cat-file -p HEAD
tree 7c9e6b4a3d2f1e0c9b8a7d6e5f4a3b2c1d0e9f8a
parent 3f2e1d0c9b8a7d6e5f4a3b2c1d0e9f8a7b6c5d4e
author Dev <dev@example.com> 1706000000 +0500
committer Dev <dev@example.com> 1706000000 +0500

fix: backend/app/models/course.py validatsiya xatosi

$ git cat-file -p 7c9e6b4 | grep app
040000 tree 55ee1122334455667788990011223344556677  backend
$ git cat-file -p 55ee112 | grep app
040000 tree 99aa88bb77cc66dd55ee44ff33gg22hh11ii00jj  app

# ============================================================
# 9) Annotated tag obyekti — to'rtinchi obyekt turi
# ============================================================
$ git tag -a v1.0.0 -m "Birinchi barqaror reliz"
$ git cat-file -t v1.0.0
tag
$ git cat-file -p v1.0.0
object abc123def4567890abcdef1234567890abcdef12
type commit
tag v1.0.0
tagger Ism Familiya <email@example.com> 1706000000 +0500

Birinchi barqaror reliz
# E'tibor bering: tag OBYEKTI commit'ga emas, balki O'ZI alohida obyekt —
# "object" qatori orqali commit'ga ishora qiladi. Yengil (lightweight) teg
# esa umuman bunday obyekt yaratmaydi, faqat oddiy ref (0-darsdagi
# refs/heads/ kabi, lekin refs/tags/ ostida).

$ ls .git/refs/tags/
v1.0.0
$ cat .git/refs/tags/v1.0.0
9f8e7d6c5b4a3928170615f4e3d2c1b0a9f8e7d6
# Bu — annotated tag OBYEKTINING SHA-1'i, commit'ning o'zi EMAS.

# ============================================================
# 10) git fsck — butun obyektlar bazasining yaxlitligini tekshirish
# ============================================================
$ git fsck --full
Checking object directories: 100% (256/256), done.
Checking objects: 100% (52/52), done.
# Muammo bo'lmasa hech qanday xato chiqmaydi. Agar disk buzilib, bitta
# obyekt korruptsiyaga uchrasa:
$ git fsck --full
error: hash mismatch for .git/objects/9f/8e7d6c... (expected 9f8e7d6c...)
# Bu SHA-1'ning content-addressing xususiyati tufayli mumkin bo'lgan
# tekshiruv — agar bayt korruptsiyaga uchrasa, xesh mos kelmay qoladi,
# demak muammo DARHOL aniqlanadi (0-darsdagi "bir bayt o'zgarsa, butun
# SHA-1 o'zgaradi" qoidasi shu yerda amaliy foyda beradi).
