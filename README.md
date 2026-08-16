1.PostgreSQL bazasini yagona qilish:Render yoki Railway'da.Ikkala qism (Django va Bot) uchun faqat bitta ma'lumotlar bazasi bo'lishi shart. Render yoki Railway'da bitta PostgreSQL yarating. Uning External Database URL (yoki DATABASE_URL) manzilini nusxalab oling. Buni ham backend, ham bot uchun muhit o'zgaruvchisi (Environment Variable) sifatida ishlatsangizgina ma'lumotlar sinxron bo'ladi.2.Django Backend'ni Web Service sifatida yuklash:CORS sozlamalarini tekshiring.Django loyihangizdagi settings.py faylida bazani o'qitish uchun dj-database-url paketidan foydalaning.Eng muhimi: CORS_ALLOWED_ORIGINS ro'yxatiga Vercel yoki Netlify'da yaratiladigan React frontend manzilingizni qo'shishni unutmang. Render'da buni odatdagidek "Web Service" sifatida deploy qiling va domen oling.3.Telegram Bot'ni Background Worker sifatida yuklash:Eng ko'p xato qilinadigan bosqich.Telegram bot HTTP so'rovlarni qabul qiluvchi server emas, shuning uchun uni Web Service qilib yuklasangiz, hosting port topolmay xato beradi (crash bo'ladi).Render yoki Railway'da yangi loyiha qo'shayotganda "Background Worker" ni tanlang. U faqat orqa fonda ishlab, Telegram serverlari bilan long-polling (yoki webhook) orqali muloqot qiladi. Bot ham xuddi Django ishlatayotgan DATABASE_URL ga ulanishi shart (psycopg2 yoki botning o'z ORM sozlamalari orqali).4.React Frontend'ni statik qilib yuklash:Vercel yoki Netlify.React kodingizdagi barcha API so'rovlar (fetch/axios) URL manzilini tekshiring. Ular endi localhost:8000 ga emas, Render'dagi Django Web Service manzilingizga qaratilgan bo'lishi kerak. Buni React'ning .env faylida (masalan, REACT_APP_API_URL yoki VITE_API_URL) ko'rsating, build qiling va Vercel'ga yuklang. Saytda ro'yxatdan o'tish va topshiriq qo'shish jarayonini jonli sinab ko'ring.5.README.md faylini yangilash:AI tekshiruvi uchun.Faqat GitHub URL jo'natishingiz sababli, AI butun tizim qanday ishlashini README orqali tushunadi. Faylga barcha jonli manzillarni va talab qilingan 7/7 checklistni qo'shing.README.md uchun namunaRepository'ning asosiy README.md faylini quyidagi struktura asosida yangilang:Markdown# StudyMate - Capstone Project

## 🔗 Jonli Havolalar (Live Links)
- **Frontend (Web):** [https://studymate-yourname.vercel.app](https://studymate-yourname.vercel.app)
- **Backend (API):** [https://studymate-api.onrender.com](https://studymate-api.onrender.com)
- **Telegram Bot:** [@StudyMate_UzBot](https://t.me/StudyMate_UzBot)

## ✅ 7/7 Bosqich Checklist
- [x] Django backend haqiqiy hostingda Web Service sifatida ishlab turibdi
- [x] React frontend haqiqiy hostingda statik build sifatida ishlab turibdi
- [x] Telegram bot haqiqiy hostingda Background Worker sifatida ishlab turibdi
- [x] Bot va Django backend BIR XIL production PostgreSQL bazasiga ulangan
- [x] Ro'yxatdan o'tish, kishish, topshiriq qo'shish web saytda ishlaydi
- [x] `/link` va `/topshiriqlar` buyruqlari haqiqiy botda ishlaydi
- [x] README.md barcha havolalar va sinov ro'yxati bilan yangilangan

## 🧪 Sinov Ro'yxati (Test Cases)
1. Web saytga kirib yangi foydalanuvchi yarating.
2. Tizimga kirib yangi topshiriq (Task) qo'shing.
3. Telegram botga o'tib `/link` orqali akkauntni ulang.
4. Botda `/topshiriqlar` buyrug'ini bering — web saytda qo'shgan topshirig'ingiz ko'rinishi ke
