# ============================================================
# 1) Branch yaratishning "haqiqiy" narxi
# ============================================================
$ time git branch big-history-branch
real    0m0.003s   # 10 000 commit bo'lsa ham natija shu — O(1)

$ cat .git/refs/heads/big-history-branch
c3a1f9e8d7c6b5a4938271605f4e3d2c1b0a9f8e
$ wc -c .git/refs/heads/big-history-branch
41 .git/refs/heads/big-history-branch
# Aynan 41 bayt: 40 belgili SHA-1 + yangi qator (\n).

# ============================================================
# 2) HEAD nima ekanligini isbotlash
# ============================================================
$ cat .git/HEAD
ref: refs/heads/main

$ git switch feature-x
$ cat .git/HEAD
ref: refs/heads/feature-x
# switch shunchaki HEAD faylining matnini o'zgartirdi.

# ============================================================
# 3) Detached HEAD holatini qo'lda hosil qilish
# ============================================================
$ git checkout c3a1f9e
Note: switching to 'c3a1f9e'.
You are in 'detached HEAD' state...
$ cat .git/HEAD
c3a1f9e8d7c6b5a4938271605f4e3d2c1b0a9f8e
# Endi HEAD ref'ga EMAS, to'g'ridan-to'g'ri commit SHA-1'iga ishora qiladi.

$ git commit --allow-empty -m "detached holatda commit"
[detached HEAD f00dbabe] detached holatda commit
$ git branch --contains f00dbabe
# (bo'sh natija) — bu commit hech qanday branch'ga tegishli emas!
$ git reflog | head -3
f00dbabe HEAD@{0}: commit: detached holatda commit
c3a1f9e HEAD@{1}: checkout: moving from feature-x to c3a1f9e
# reflog orqali topib, uni yangi branch'ga biriktirish mumkin:
$ git branch qutqarilgan-branch f00dbabe

# ============================================================
# 4) git switch/checkout uchta amalni bajarishini kuzatish
# ============================================================
$ git switch main
$ cat .git/HEAD                       # 1) HEAD yangilandi
ref: refs/heads/main
$ git status --short                   # 2) index tree bilan mos
# (toza — farq yo'q)
$ ls                                    # 3) working dir yangilandi
README.md  backend/  frontend/

# ============================================================
# 5) packed-refs bilan loose ref orasidagi ustunlik
# ============================================================
$ git pack-refs --all
$ cat .git/packed-refs | grep feature-x
c3a1f9e8d7c6b5a4938271605f4e3d2c1b0a9f8e refs/heads/feature-x
$ ls .git/refs/heads/
main
# feature-x endi loose fayl sifatida yo'q, faqat packed-refs ichida.

$ git commit --allow-empty -m "yangi commit feature-x'da"
$ ls .git/refs/heads/
feature-x  main
# Yangi commit qilinganda Git AVTOMATIK ravishda loose faylni qayta
# yaratdi — packed-refs endi eskirgan, lekin loose versiya ustunlik qiladi.

# ============================================================
# 6) Ikkita branch bir xil eski commit'larni ULASHISHI
# ============================================================
$ git log --oneline main
c3a1f9e uchinchi commit
7b2e8d1 ikkinchi commit
4a1c7f0 birinchi commit
$ git log --oneline feature-x
9d4e2a3 feature ustida ish
7b2e8d1 ikkinchi commit      # <- main bilan BIR XIL obyekt
4a1c7f0 birinchi commit      # <- main bilan BIR XIL obyekt
# 7b2e8d1 va 4a1c7f0 ikkalasida ham bitta marta saqlangan, ikki marta emas.

# ============================================================
# 7) symbolic-ref — HEAD'ni to'g'ridan-to'g'ri boshqarish
# ============================================================
$ git symbolic-ref HEAD
refs/heads/main
$ git symbolic-ref HEAD refs/heads/feature-x
$ cat .git/HEAD
ref: refs/heads/feature-x
# Xuddi git switch qilingandek natija, lekin buyruq darajasida, oshkora.

$ git symbolic-ref HEAD refs/heads/notavjud
$ git status
fatal: not a valid ref: HEAD refers to a nonexistent ref
# symbolic-ref formatni tekshiradi, lekin ref'ning mavjudligini emas —
# shuning uchun ehtiyotkorlik bilan ishlatish kerak.

# ============================================================
# 8) Reflog — .git/logs/ ichida nima bor
# ============================================================
$ cat .git/logs/HEAD | tail -3
7b2e8d1... 9d4e2a3... Dev <dev@example.com> 1706000100 +0500	commit: feature ustida ish
9d4e2a3... c3a1f9e... Dev <dev@example.com> 1706000200 +0500	checkout: moving from feature-x to main
c3a1f9e... 7b2e8d1... Dev <dev@example.com> 1706000300 +0500	commit: uchinchi commit

$ git reflog
7b2e8d1 (HEAD -> main) HEAD@{0}: commit: uchinchi commit
9d4e2a3 HEAD@{1}: checkout: moving from feature-x to main
c3a1f9e HEAD@{2}: commit: feature ustida ish
# git reflog aynan shu faylni o'qib, o'qish uchun qulay formatga o'giradi.

# ============================================================
# 9) Remote-tracking ref — uchinchi ref turi
# ============================================================
$ git fetch origin
$ ls .git/refs/remotes/origin/
main  feature-x
$ cat .git/refs/remotes/origin/main
c3a1f9e8d7c6b5a4938271605f4e3d2c1b0a9f8e
# refs/remotes/origin/main — bu SIZNING main branch'ingiz EMAS, balki
# oxirgi fetch paytida serverda main qayerda turgani haqidagi "xotira".
# git pull = git fetch (bu ref'ni yangilaydi) + git merge (mahalliy
# branch'ga qo'shadi).
