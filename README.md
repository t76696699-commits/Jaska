# ============================================================
# 1) .sample fayllar — nega ISHLAMAYDI
# ============================================================
$ ls .git/hooks/
applypatch-msg.sample  post-update.sample  pre-commit.sample
commit-msg.sample      pre-applypatch.sample  pre-push.sample
...
$ git commit -m "test"
[main abc123] test
# .sample kengaytmasi bo'lgani uchun Git ularni umuman ko'rmaydi.

# ============================================================
# 2) pre-commit hook — maxfiy kalitni tekshirish
# ============================================================
$ cat > .git/hooks/pre-commit << 'EOF'
#!/bin/bash
if git diff --cached | grep -qE "(SECRET_KEY|API_KEY)\s*=\s*['\"][a-zA-Z0-9]"; then
    echo "XATO: staged o'zgarishlarda maxfiy kalit topildi!"
    echo "  .env faylidan foydalaning, kodga yozmang."
    exit 1
fi
exit 0
EOF
$ chmod +x .git/hooks/pre-commit

$ echo 'SECRET_KEY = "abc123supersecret"' >> config.py
$ git add config.py
$ git commit -m "config yangilandi"
XATO: staged o'zgarishlarda maxfiy kalit topildi!
  .env faylidan foydalaning, kodga yozmang.
# Commit BUTUNLAY to'xtatildi — chiqish kodi 1 bo'lgani uchun.

$ git commit -m "config yangilandi" --no-verify
[main def456] config yangilandi
# --no-verify BARCHA mahalliy hook'larni chetlab o'tadi — bu XAVFLI,
# lekin ba'zan qasddan (masalan WIP commit) ishlatiladi.

# ============================================================
# 3) commit-msg hook — Conventional Commits formatini majburlash
# ============================================================
$ cat > .git/hooks/commit-msg << 'EOF'
#!/bin/bash
MSG_FILE=$1
PATTERN="^(feat|fix|refactor|docs|test|chore|perf|ci)(\(.+\))?: .+"
if ! grep -qE "$PATTERN" "$MSG_FILE"; then
    echo "XATO: commit xabari 'feat: ...' yoki 'fix: ...' formatida bo'lishi kerak"
    exit 1
fi
EOF
$ chmod +x .git/hooks/commit-msg

$ git commit -m "narsalarni tuzatdim"
XATO: commit xabari 'feat: ...' yoki 'fix: ...' formatida bo'lishi kerak
$ git commit -m "fix: chegirma hisoblash xatosi tuzatildi"
[main 7c3a1e9] fix: chegirma hisoblash xatosi tuzatildi

# ============================================================
# 4) pre-push hook — shu platformaning test.yml'iga mos avtomatik test
# ============================================================
$ cat > .git/hooks/pre-push << 'EOF'
#!/bin/bash
echo "pre-push: backend testlari ishga tushirilmoqda (test.yml kabi)..."
cd backend && python -m pytest tests/ -q --tb=short
if [ $? -ne 0 ]; then
    echo "XATO: testlar muvaffaqiyatsiz, push bekor qilindi."
    echo "(Bu xuddi GitHub Actions serverda qiladigan tekshiruv — farqi:"
    echo " bu yerda push'dan OLDIN, mahalliy kompyuterda bajarilyapti.)"
    exit 1
fi
EOF
$ chmod +x .git/hooks/pre-push

$ git push origin feature-x
pre-push: backend testlari ishga tushirilmoqda (test.yml kabi)...
FAILED tests/test_payment.py::test_discount_applies
XATO: testlar muvaffaqiyatsiz, push bekor qilindi.
error: failed to push some refs

# ============================================================
# 5) Nega hook'lar versiyalanmaydi — isbot
# ============================================================
$ git status --short
# (bo'sh) — .git/hooks/pre-commit HECH QACHON "git status" da ko'rinmaydi,
# chunki u .gitignore'da emas, u .git/ ICHIDA, umuman kuzatilmaydi.

$ git ls-files | grep hooks
# (bo'sh natija) — hook'lar repo tarixining qismi EMAS.

# ============================================================
# 6) Versiyalanadigan yechim: repo ichidagi skript + o'rnatuvchi
# ============================================================
$ mkdir -p scripts/git-hooks
$ cp .git/hooks/pre-push scripts/git-hooks/pre-push
$ git add scripts/git-hooks/pre-push
$ git commit -m "chore: pre-push hook skripti repo'ga qo'shildi"
# Endi har bir dasturchi buni o'rnatishi mumkin:
$ ln -sf ../../scripts/git-hooks/pre-push .git/hooks/pre-push
# Yoki README'da: "git config core.hooksPath scripts/git-hooks"
$ git config core.hooksPath scripts/git-hooks
# Bu Git'ga hook'larni .git/hooks/ o'rniga scripts/git-hooks/ dan
# o'qishni buyuradi — endi versiyalanadigan papka ISHLAYDI.

# ============================================================
# 7) Server tomonidagi pre-receive hook (illyustrativ misol)
# ============================================================
# Bu skript SERVERDA (masalan GitHub Enterprise yoki o'z Git serveringizda)
# joylashadi, mijoz kompyuterida EMAS — shuning uchun --no-verify unga
# ta'sir qilmaydi.
$ cat /path/to/server-repo.git/hooks/pre-receive
#!/bin/bash
while read oldrev newrev refname; do
    if [[ "$refname" == "refs/heads/main" ]]; then
        echo "XATO: main branch'ga to'g'ridan-to'g'ri push taqiqlangan."
        echo "Iltimos, Pull Request oching."
        exit 1
    fi
done

$ git push origin main
remote: XATO: main branch'ga to'g'ridan-to'g'ri push taqiqlangan.
remote: Iltimos, Pull Request oching.
To github.com:team/repo.git
 ! [remote rejected] main -> main (pre-receive hook declined)
# --no-verify bu yerda HECH QANDAY farq qilmaydi, chunki tekshiruv
# mijoz tomonida emas, SERVER tomonida ishlayapti.

# ============================================================
# 8) commit-msg hook'ni kengaytirish — uzunlik tekshiruvi qo'shish
# ============================================================
$ cat scripts/git-hooks/commit-msg
#!/bin/bash
MSG_FILE=$1
FIRST_LINE=$(head -1 "$MSG_FILE")
if [ ${#FIRST_LINE} -gt 72 ]; then
    echo "XATO: birinchi qator 72 belgidan uzun (${#FIRST_LINE} belgi)"
    exit 1
fi
PATTERN="^(feat|fix|refactor|docs|test|chore|perf|ci)(\(.+\))?: .+"
if ! grep -qE "$PATTERN" "$MSG_FILE"; then
    echo "XATO: 'feat: ...' yoki 'fix: ...' formatida yozing"
    exit 1
fi
