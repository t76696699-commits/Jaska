# ============================================================
# 1) Submodule qo'shish
# ============================================================
$ git submodule add https://github.com/example/ui-kit.git vendor/ui-kit
Cloning into '/home/user/repo/vendor/ui-kit'...
$ cat .gitmodules
[submodule "vendor/ui-kit"]
	path = vendor/ui-kit
	url = https://github.com/example/ui-kit.git

$ git status --short
A  .gitmodules
A  vendor/ui-kit          # <- 160000 mode, gitlink, oddiy papka EMAS
$ git ls-tree HEAD vendor/
160000 commit 3f2e1d0c9b8a7d6e5f4a3b2c1d0e9f8a7b6c5d4e	vendor/ui-kit
# "160000" — bu maxsus mode, submodule ekanini bildiradi.

$ git commit -am "vendor/ui-kit submodule qo'shildi"

# ============================================================
# 2) Klonlashda submodule'ni unutish xatosi
# ============================================================
$ git clone https://github.com/team/repo.git fresh-clone
$ ls fresh-clone/vendor/ui-kit
# (BO'SH papka!)

$ cd fresh-clone
$ git submodule update --init --recursive
Submodule 'vendor/ui-kit' registered for path 'vendor/ui-kit'
Cloning into 'vendor/ui-kit'...
$ ls vendor/ui-kit
README.md  src/  package.json    # endi to'liq

# Yaxshiroq usul — bir qadamda:
$ git clone --recurse-submodules https://github.com/team/repo.git

# ============================================================
# 3) Submodule'ni yangi commit'ga ko'chirish
# ============================================================
$ cd vendor/ui-kit
$ git log --oneline -1
3f2e1d0 v2.1.0 relizi
$ git pull origin main
$ git log --oneline -1
9a8b7c1 v2.2.0 relizi
$ cd ../..
$ git status --short
 M vendor/ui-kit           # ishorat commit o'zgardi
$ git add vendor/ui-kit
$ git commit -m "vendor/ui-kit ni v2.2.0 ga yangilash"
# Asosiy repo faqat "endi 9a8b7c1'ga ishora qil" deb saqlaydi.

# ============================================================
# 4) Subtree qo'shish — HAQIQIY nusxa
# ============================================================
$ git subtree add --prefix=vendor/ui-kit-subtree \
    https://github.com/example/ui-kit.git main --squash
Squash commit -- not updating HEAD
Merge commit -- not updating HEAD

$ git log --oneline -1
a1b2c3d Merge commit 'xxxx' as 'vendor/ui-kit-subtree'
$ ls vendor/ui-kit-subtree/
README.md  src/  package.json    # DARHOL to'liq — clone/init shart emas

$ git clone https://github.com/team/repo.git fresh2
$ ls fresh2/vendor/ui-kit-subtree/
README.md  src/  package.json    # darhol to'liq, hech qanday qo'shimcha buyruqsiz!

# ============================================================
# 5) Subtree'ni yangilash va o'zgarishni qaytarish
# ============================================================
$ git subtree pull --prefix=vendor/ui-kit-subtree \
    https://github.com/example/ui-kit.git main --squash

$ vim vendor/ui-kit-subtree/src/button.js   # mahalliy tuzatish
$ git commit -am "ui-kit tugmasini tuzatish"
$ git subtree push --prefix=vendor/ui-kit-subtree \
    https://github.com/example/ui-kit.git my-fix-branch
# Endi my-fix-branch orqali original repo'ga PR ochish mumkin.

# ============================================================
# 6) Qaysi mode ekanini tekshirish (submodule vs oddiy papka)
# ============================================================
$ git ls-tree HEAD vendor/
160000 commit 9a8b7c1...  vendor/ui-kit           # <- submodule (gitlink)
040000 tree   b2c3d4e...  vendor/ui-kit-subtree   # <- subtree (oddiy tree)

# ============================================================
# 7) submodule foreach — bir nechta submodule'ni bir vaqtda yangilash
# ============================================================
$ cat .gitmodules
[submodule "vendor/ui-kit"]
	path = vendor/ui-kit
	url = https://github.com/example/ui-kit.git
[submodule "vendor/charts"]
	path = vendor/charts
	url = https://github.com/example/charts.git

$ git submodule foreach 'git checkout main && git pull origin main'
Entering 'vendor/charts'
Already on 'main'
Entering 'vendor/ui-kit'
Already on 'main'
# Ikkala submodule ham BITTA buyruq bilan yangilandi.

$ git submodule foreach 'echo "$name da $(git rev-parse --short HEAD)"'
Entering 'vendor/charts'
vendor/charts da 7c3a1e9
Entering 'vendor/ui-kit'
vendor/ui-kit da 9a8b7c1

# ============================================================
# 8) Submodule'ni to'g'ri olib tashlash
# ============================================================
$ git submodule deinit vendor/ui-kit
Cleared directory 'vendor/ui-kit'
$ ls vendor/ui-kit
# (bo'sh — fayllar o'chirildi, lekin .gitmodules'da yozuv qoladi)

$ git rm vendor/ui-kit
rm 'vendor/ui-kit'
$ git status --short
D  .gitmodules
D  vendor/ui-kit
$ git commit -m "vendor/ui-kit submodule butunlay olib tashlandi"
# Endi .gitmodules, .git/config va .git/modules/ ichidagi barcha
# izlar tozalandi — oddiy rm -rf bilan bu HECH QACHON to'liq bo'lmasdi.
