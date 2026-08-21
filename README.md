# ============================================================
# 1) Interaktiv rebase'ni boshlash
# ============================================================
$ git log --oneline -4
d4e5f6a yana typo
c3b4a5d login validatsiya
b2a3c4e typo tuzatish
a1b2c3d login forma

$ git rebase -i HEAD~4
# Tahrirlovchida ochiladi:
pick a1b2c3d login forma
pick b2a3c4e typo tuzatish
pick c3b4a5d login validatsiya
pick d4e5f6a yana typo

# Rebase 4 commit'ni to'xtatadi (edit, drop, squash, fixup, break, ...)
# Yuqoridagi qatorlar TARTIBI - bu keyingi tarix tartibi.

# ============================================================
# 2) Qo'lda tahrirlash: fixup + qayta tartiblash
# ============================================================
pick a1b2c3d login forma
fixup b2a3c4e typo tuzatish
pick c3b4a5d login validatsiya
fixup d4e5f6a yana typo
# Faylni saqlab yopamiz. Git avtomatik ravishda:
$ git rebase -i HEAD~4
Successfully rebased and updated refs/heads/feature-login.

$ git log --oneline -2
9f8e7d6 login validatsiya
1a2b3c4 login forma
# 4 ta commit endi 2 taga aylandi — typo'lar jimgina yutildi.

# ============================================================
# 3) edit bilan o'rtadagi commit'ni tahrirlash
# ============================================================
$ git rebase -i HEAD~2
pick 1a2b3c4 login forma
edit 9f8e7d6 login validatsiya
# saqlaymiz -> Git birinchi commit'ni qo'llaydi, ikkinchisida to'xtaydi:
Stopped at 9f8e7d6...  login validatsiya
You can amend the commit now, with

  git commit --amend

$ echo "qo'shimcha validatsiya qatori" >> validators.py
$ git add validators.py
$ git commit --amend --no-edit
$ git rebase --continue
Successfully rebased and updated refs/heads/feature-login.
# 9f8e7d6 endi YANGI SHA-1'ga ega — chunki kontenti o'zgardi.

# ============================================================
# 4) --autosquash bilan avtomatlashtirish
# ============================================================
$ git commit --fixup=1a2b3c4 -m "kichik tuzatish"
[feature-login e5f6a7b] fixup! login forma

$ git log --oneline -3
e5f6a7b fixup! login forma
9f8e7d6 login validatsiya
1a2b3c4 login forma

$ git rebase -i --autosquash HEAD~3
# Todo ro'yxati Git tomonidan AVTOMATIK shunday tuziladi:
pick 1a2b3c4 login forma
fixup e5f6a7b fixup! login forma
pick 9f8e7d6 login validatsiya
# Qo'lda ko'chirish shart bo'lmadi — Git "fixup!" prefiksini tanib,
# to'g'ri joyga qo'ydi.

# ============================================================
# 5) drop bilan commit'ni butunlay olib tashlash
# ============================================================
$ git rebase -i HEAD~3
pick 1a2b3c4 login forma
drop 9f8e7d6 login validatsiya   # <- bu qatorni butunlay o'chiramiz
pick e5f6a7b fixup! login forma
# saqlab yopamiz -> login validatsiya commit'i BUTUNLAY yo'qoladi,
# uning kod o'zgarishlari HAM yo'qoladi (drop != revert).

# ============================================================
# 6) Xavfsiz force-push
# ============================================================
$ git push --force-with-lease origin feature-login
# Agar boshqa dasturchi orada push qilgan bo'lsa:
To github.com:team/repo.git
 ! [rejected]  feature-login -> feature-login (stale info)
error: failed to push some refs
# --force-with-lease buni oldini oladi; oddiy --force esa bosib o'tib
# hamkasbning ishini yo'qotib qo'yardi.

# ============================================================
# 7) Rebase to'xtab qolsa — abort bilan orqaga qaytish
# ============================================================
$ git rebase -i HEAD~3
# konflikt yuzaga keldi, chalkashib ketdik:
$ git rebase --abort
# Repo REBASE BOSHLANISHDAN OLDINGI holatga to'liq qaytadi — hech qanday
# o'zgarish saqlanmaydi, xavfsiz "orqaga" tugmasi.

# ============================================================
# 8) rebase --onto — faqat oraliqni ko'chirish
# ============================================================
$ git log --oneline --all --graph
* d4e5f6a (feature-x) feature commit 2
* c3b4a5d feature commit 1
* b2a3c4e (old-base) eski, kerak bo'lmagan asos
* a1b2c3d (new-base) yangi, to'g'ri asos
* 9f8e7d6 umumiy ajdod

$ git rebase --onto new-base old-base feature-x
Successfully rebased and updated refs/heads/feature-x.

$ git log --oneline --all --graph
* f1e2d3c (feature-x) feature commit 2
* e0d1c2b feature commit 1
| * b2a3c4e (old-base) eski, kerak bo'lmagan asos
|/
* a1b2c3d (new-base) yangi, to'g'ri asos
* 9f8e7d6 umumiy ajdod
# feature-x'ning IKKALA commit'i ham endi new-base ustida, old-base
# butunlay chetlab o'tildi — uning tarixiga hech qanday ta'sir bo'lmadi.

# ============================================================
# 9) Rebase paytida konflikt — continue/skip/abort uchligi
# ============================================================
$ git rebase main
Auto-merging config.py
CONFLICT (content): Merge conflict in config.py
error: could not apply 7c3a1e9... login validatsiya

$ cat config.py
<<<<<<< HEAD
TIMEOUT = 30
=======
TIMEOUT = 60
>>>>>>> 7c3a1e9 (login validatsiya)
$ vim config.py && git add config.py
$ git rebase --continue
# YOKI, agar bu commit umuman kerak bo'lmasa:
$ git rebase --skip
# YOKI, agar butunlay chalkashib ketgan bo'lsangiz:
$ git rebase --abort   # boshlanishdan oldingi holatga to'liq qaytadi
