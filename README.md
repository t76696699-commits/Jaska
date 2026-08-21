# ============================================================
# 1) Muammoni ko'rish: stash'siz branch almashtirib bo'lmaydi
# ============================================================
$ git status --short
 M app/services/payment_service.py
 M app/api/v1/endpoints/payments.py
# feature-payment ustida tugallanmagan ish bor, birdan hotfix kerak:
$ git switch main
error: Your local changes to the following files would be overwritten by checkout:
	app/services/payment_service.py
Please commit your changes or stash them before you switch branches.

# ============================================================
# 2) Worktree bilan yechim — stash umuman kerak emas
# ============================================================
$ git worktree add ../repo-hotfix main
Preparing worktree (checking out 'main')
HEAD is now at 3f2e1d0 fix: oldingi hotfix

$ ls ..
repo/  repo-hotfix/
$ cd ../repo-hotfix
$ git branch --show-current
main
$ git switch -c hotfix/urgent-bug
$ vim app/services/critical.py
$ git commit -am "hotfix: production'dagi kritik xato tuzatildi"
$ git push origin hotfix/urgent-bug

# Bu paytda birinchi papkada HECH NARSA o'zgarmagan:
$ cd ../repo
$ git status --short
 M app/services/payment_service.py
 M app/api/v1/endpoints/payments.py
# Xuddi shu, tegilmagan holatda — hotfix uchun stash/pop kerak bo'lmadi.

# ============================================================
# 3) Ichki mexanizmni tekshirish
# ============================================================
$ cat ../repo-hotfix/.git
gitdir: /home/user/repo/.git/worktrees/repo-hotfix
# Bu FAYL, papka emas — faqat asosiy repo'ga ishora.

$ ls .git/worktrees/
repo-hotfix
$ ls .git/worktrees/repo-hotfix/
HEAD  index  logs/  ORIG_HEAD  commondir  gitdir
$ cat .git/worktrees/repo-hotfix/HEAD
ref: refs/heads/hotfix/urgent-bug
# Har bir worktree o'z HEAD'iga ega, lekin refs/heads/ ULASHILGAN:
$ ls .git/refs/heads/
feature-payment  hotfix/  main

# ============================================================
# 4) Bir xil branch'ni ikki joyda checkout qilib bo'lmasligi
# ============================================================
$ git worktree add ../repo-main2 main
fatal: 'main' is already used by worktree at '/home/user/repo-hotfix'
# Git buni ATAYLAB taqiqlaydi — bitta branch ikki xil ishchi papkada
# bir vaqtda bo'lsa, index'lar mos kelmay qolishi mumkin edi.

# ============================================================
# 5) Faol worktree'larni ko'rish va tozalash
# ============================================================
$ git worktree list
/home/user/repo          3f2e1d0 [feature-payment]
/home/user/repo-hotfix   a1b2c3d [hotfix/urgent-bug]

$ rm -rf ../repo-hotfix          # qo'lda o'chirib yubordik
$ git worktree list
/home/user/repo          3f2e1d0 [feature-payment]
/home/user/repo-hotfix   a1b2c3d [hotfix/urgent-bug]  (o'chirilgan, lekin hali ro'yxatda)

$ git worktree prune
$ git worktree list
/home/user/repo          3f2e1d0 [feature-payment]
# Endi Git bu haqda bilib, yozuvni tozaladi.

# To'g'ri usul (prune shart emas):
$ git worktree add ../repo-review pr-123-branch
$ git worktree remove ../repo-review

# ============================================================
# 6) Worktree'ni qulflash (tashqi disk uchun)
# ============================================================
$ git worktree add /mnt/external-disk/long-task feature-y
$ git worktree lock /mnt/external-disk/long-task --reason "tashqi disk, uzoq muddat"
$ git worktree list
/home/user/repo               3f2e1d0 [main]
/mnt/external-disk/long-task   a8b7c6d [feature-y] locked

$ git worktree prune   # tashqi disk vaqtincha ulanmagan bo'lsa ham xavfsiz
# Locked worktree HECH QACHON prune tomonidan o'chirilmaydi.

$ git worktree unlock /mnt/external-disk/long-task
$ git worktree remove /mnt/external-disk/long-task

# ============================================================
# 7) --porcelain — skript uchun barqaror format
# ============================================================
$ git worktree list --porcelain
worktree /home/user/repo
HEAD 3f2e1d0c9b8a7d6e5f4a3b2c1d0e9f8a7b6c5d4e
branch refs/heads/main

worktree /home/user/repo-hotfix
HEAD a8b7c6d5e4f3a2b1c0d9e8f7a6b5c4d3e2f1a0b9
branch refs/heads/hotfix/urgent-bug

# Har bir worktree haqida uchta qator: worktree (yo'l), HEAD (SHA-1),
# branch (to'liq ref nomi) — bo'sh qator bilan ajratilgan, skript uchun
# qulay, versiyalar orasida barqaror.

# ============================================================
# 8) Worktree'ni boshqa joyga ko'chirish
# ============================================================
$ git worktree move ../repo-hotfix ../renamed-hotfix
$ git worktree list
/home/user/repo             3f2e1d0 [main]
/home/user/renamed-hotfix   a8b7c6d [hotfix/urgent-bug]
# .git/worktrees/ ichidagi ichki yozuvlar ham avtomatik yangilanadi.
