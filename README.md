1. To'g'ri papkada ekanligingizni tekshiring
PyCharm yoki VS Code terminalini oching. Terminal aynan backend, frontend va bot papkalari joylashgan umumiy papkada ochilganiga ishonch hosil qiling.

2. Kodlarning holatini ko'rish
Terminalda shu buyruqni bering:

Bash
git status
Bu ekranda qizil yozuvlar bilan backend/, frontend/, bot/ kabi papkalarni ko'rsatishi kerak. Bu "hali GitHub'ga qo'shilmagan fayllar" degani.

3. Barcha kodlarni qamrab olish (qo'shish)
Barcha fayl va papkalarni Git'ga qo'shish uchun nuqta bilan yozilgan mana bu buyruqni bering:

Bash
git add .
(Nuqta . barcha fayllarni bildiradi)

4. O'zgarishlarni saqlash (Commit)

Bash
git commit -m "Barcha loyiha kodlari qo'shildi"
5. GitHub'ga yuklash (Push)

Bash
git push origin main
