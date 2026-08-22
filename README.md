**# ============================================================
# Yakuniy amaliyot: to'rt mavzuni bitta jamoaviy stsenariyda
# ============================================================

# 1) Shoshilinch hotfix uchun worktree (6-dars)
$ git worktree add ../hotfix main
$ cd ../hotfix
$ git switch -c hotfix/vendor-update

# 2) vendor/ submodule'ini yangilash (7-dars)
$ cd vendor/ui-kit
$ git pull origin main
$ cd ../..
$ git add vendor/ui-kit
$ git commit -m "vendor/ui-kit yangilandi"

# 3) pre-commit hook maxfiy kalitni tekshiradi (8-dars)
$ git commit -am "config.py yangilandi"
XATO: staged o'zgarishlarda maxfiy kalit topildi!
$ vim config.py   # kalitni .env'ga ko'chiramiz
$ git commit -am "config.py yangilandi (kalit .env'ga ko'chirildi)"
[hotfix/vendor-update abc123] config.py yangilandi

# 4) main'ga birlashtirishda avvalgi konflikt qayta chiqadi, rerere yechadi (9-dars)
$ git switch main
$ git merge hotfix/vendor-update
Auto-merging config.py
Resolved 'config.py' using previous resolution.
Auto-merging vendor/ui-kit
Merge made by the 'ort' strategy.

# 5) Push'dan oldin mahalliy test, keyin server tekshiruvi
$ git push origin main
pre-push: backend testlari (test.yml kabi)...
3 passed
# ... push davom etadi, GitHub serverida .github/workflows/test.yml
# QAYTA, mustaqil ravishda ishga tushadi — bu ikkinchi, majburiy qatlam.

# 6) Tozalash
$ git worktree remove ../hotfix

# ============================================================
# O'z-o'zini tekshirish: interaktiv jadval
# ============================================================
# | Savol                                      | Javob qayerda?     |
# |----------------------------------------------|---------------------|
# | worktree nima saqlaydi, nima ulashadi?       | 6-dars              |
# | submodule vs subtree fayl saqlash farqi?     | 7-dars              |
# | --no-verify nima uchun xavfli?               | 8-dars              |
# | rerere ikkinchi safar nima deydi?            | 9-dars              |

# ============================================================
# Qo'shimcha tekshiruv: worktree + submodule + hook birgalikda
# ============================================================
$ git worktree list --porcelain | head -6
worktree /home/user/repo
HEAD 3f2e1d0c9b8a7d6e5f4a3b2c1d0e9f8a7b6c5d4e
branch refs/heads/main

worktree /home/user/hotfix
HEAD a8b7c6d5e4f3a2b1c0d9e8f7a6b5c4d3e2f1a0b9

$ cd ../hotfix
$ git submodule foreach 'git log -1 --oneline'
Entering 'vendor/ui-kit'
9a8b7c1 v2.2.0 relizi

$ git config core.hooksPath scripts/git-hooks
$ git commit -am "config.py yangilandi"
# pre-commit hook ishga tushadi, tekshiradi, muammo bo'lmasa o'tkazadi

$ cd ../repo
$ git merge hotfix/vendor-update
Resolved 'config.py' using previous resolution.
$ git rerere status
# (bo'sh — barcha konfliktlar allaqachon yechilgan)

$ git submodule foreach 'git status --short'
Entering 'vendor/ui-kit'
Entering 'vendor/charts'
# (bo'sh ikkalasida ham — ikkala submodule ham toza, saqlanmagan
# o'zgarishlarsiz)

$ git worktree list
/home/user/repo      3f2e1d0 [main]
/home/user/hotfix     a8b7c6d [hotfix/vendor-update]
$ git worktree remove ../hotfix
# Ish kuni tugadi: worktree yopildi, submodule yangilandi, hook ishladi,
# rerere konfliktni avtomatik yechdi — to'rtta vosita, bitta izchil ish
# oqimida.**
