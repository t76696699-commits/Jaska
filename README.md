7-Polish va Deploy (CAPSTONE yakuni)
Урок 7 из 7
· 3 раздела
✓ Пройден
📝
Matn
Текст
#1
7-bosqich (CAPSTONE yakuni): deploy va "tsc muvaffaqiyatli, production buzilgan" xatosi
Dev: ts-node + tsconfig-paths - @shared/types ISHLAYDI

npm run build: tsc

tsc paths xaritasini FAQAT tur tekshirish uchun ishlatadi

dist/server.js: require('@shared/types') - O'ZGARTIRILMAGAN!

node dist/server.js

❌ Cannot find module '@shared/types' - garchi tsc 0 xato bilan tugagan bo'lsa ham!

Node.js/Express kursida CORS'ni va React'ni backend bilan bog'lashni allaqachon o'rgangansiz. Bu — IssueForge'ning so'nggi, yakuniy bosqichi, va bu yerda capstone davomida ko'rgan g'oyaning eng aniq ko'rinishi paydo bo'ladi: bu safar hatto tscning o'zi ham "hammasi joyida" deb hisoblaydi — compile 0 xato bilan tugaydi — lekin production baribir ishlamay qoladi.

🏆 5 daqiqada g'alaba
BLOKA 1 — path alias: nisbiy yo'llar o'rniga qisqa, o'qilishi oson import
// backend/tsconfig.json
{
  "compilerOptions": {
    "baseUrl": ".",
    "paths": {
      "@shared/*": ["../shared/*"]
    }
  }
}
// ODDIY (nisbiy) import - fayl chuqurlashgan sari o'qish qiyinlashadi:
// import { Issue } from '../../../shared/types';

// PATH ALIAS bilan - qisqa va aniq:
import { Issue } from '@shared/types';
BLOKA 2 — development'da path alias'ni ishga tushirish
# package.json
{
  "scripts": {
    "dev": "ts-node -r tsconfig-paths/register src/server.ts"
  }
}

# npm install -D tsconfig-paths
# ts-node -r tsconfig-paths/register - RUNTIME'da @shared/* alias'ini
# haqiqiy fayl yo'liga aylantiradi. Dev'da hammasi MUKAMMAL ishlaydi.
BLOKA 3 — production build: alias'ni ham build vaqtida hal qilish
# npm install -D tsc-alias
# package.json
{
  "scripts": {
    "build": "tsc && tsc-alias"
  }
}

# tsc-alias - tsc chiqargan dist/*.js fayllardagi '@shared/*'
# yozuvlarini HAQIQIY nisbiy yo'llarga QAYTA YOZADI. Shundan keyingina
# 'node dist/server.js' production'da xatosiz ishlaydi.
🐛 Ataylab xato — faqat "tsc" bilan build qilib, alias'ni unutish
# package.json - tsc-alias QO'SHILMAGAN:
{
  "scripts": {
    "build": "tsc"
  }
}

# Lokalda (dev'da) hammasi ishlaydi, chunki ts-node -r tsconfig-paths/register
# RUNTIME'da alias'ni hal qiladi. Shuning uchun bu muammo "sinovda" umuman
# sezilmaydi!

$ npm run build
# ✅ tsc: 0 xato! "Muvaffaqiyatli compile qilindi."
#
# LEKIN dist/server.js faylini ochib qarasangiz:
#   const types_1 = require("@shared/types");   // ❗ O'ZGARTIRILMAGAN!
#
# tsc "paths" xaritasini FAQAT compile vaqtida TUR tekshirish uchun
# ishlatadi - u chiqargan JavaScript'dagi import/require yo'llarini
# HECH QACHON qayta yozmaydi (bu - hujjatlashtirilgan, ataylab qilingan
# tsc xatti-harakati).

$ node dist/server.js
# ❌ Error: Cannot find module '@shared/types'
#    Require stack: - /app/dist/server.js
# Production darhol ishga tushmay qoladi - garchi tsc 0 xato bergan
# bo'lsa ham!
Natija: tsc tsconfig.jsondagi paths xaritasini faqat compile vaqtida turlarni to'g'ri tekshirish uchun ishlatadi — @shared/types qayerga ishora qilishini bilib, shu asosda tur xatolarini topadi. Lekin u chiqargan .js fayllarda @shared/types yozuvi o'zgarishsiz qoladi — chunki bu alias faqat TypeScript'ning o'ziga, compile vaqtida tanish, Node.js'ning require() mexanizmiga esa butunlay notanish. Node ishga tushganda, @shared/types degan haqiqiy npm paketi yoki fayl yo'q — shuning uchun Cannot find module xatosi bilan darhol yiqiladi. Bu — capstone davomida ko'rgan barcha "compile vaqtida OK, runtime'da muammo" xatolarining eng yalang'och shakli: bu safar hatto tscning o'zi ham noto'g'ri signal beradi.

Endi tushuntiramiz
1. Nega path alias (@shared/*) dev'da (ts-node bilan) muammosiz ishlaydi?
ts-node -r tsconfig-paths/register — bu runtimeda ishlaydigan qo'shimcha vosita. U dastur ishga tushgan paytda, har bir @shared/* importini "ushlab", uni haqiqiy fayl yo'liga o'zi aylantiradi. Shuning uchun dev muhitida bu jarayon butunlay ko'rinmas holda, muammosiz ishlab turadi.

2. tsc paths xaritasini nima uchun ishlatadi, va nima uchun ishlatmaydi?
tsc pathsni faqat compile vaqtida, @shared/typesning haqiqatda qaysi fayl/interfeysga mos kelishini bilish uchun ishlatadi — bu unga to'g'ri tur tekshiruvini o'tkazish imkonini beradi. Lekin tscning vazifasi TypeScript'ni JavaScript'ga aylantirish, import yo'llarini qayta yozish emas — shuning uchun u chiqargan .js faylda original @shared/types satri o'zgarishsiz qoladi.

3. Nega bu xato aynan production'da, node dist/server.js ishga tushirilganda paydo bo'ladi?
Production'da, oddatda, ts-node ham, tsconfig-paths/register ham ishlatilmaydi — faqat oldindan compile qilingan, "sof" JavaScript (node dist/server.js) ishga tushiriladi. Node.js'ning standart require() mexanizmi tsconfig.json haqida umuman bilmaydi va @shared/typesni oddiy npm paket nomi deb qabul qiladi — bunday paket node_modulesda yo'qligi uchun xato beradi.

4. Bu muammoning to'g'ri yechimi nima?
tsc-alias kabi vositani build jarayoniga qo'shish — bu vosita tsc chiqargan .js fayllardagi @shared/* kabi alias yozuvlarini haqiqiy nisbiy yo'llarga qayta yozadi, shundan keyin node dist/server.js production'da xatosiz ishlaydi. Muqobil yechim — umuman alias ishlatmasdan, doim nisbiy yo'llardan foydalanish (kamroq qulay, lekin bu muammoni butunlay chetlab o'tadi).

5. Bu xato butun capstone bo'ylab ko'rgan g'oyaning qanday yakuniy ko'rinishi?
1-6-darslarda TypeScript'ning o'zi "aldamadi" — muammo har doim dasturchi compile vaqtidagi ma'lumotga ortiqcha ishonganda paydo bo'lardi. Bu yerda esa hatto tscning muvaffaqiyatli compile xabari ham yetarli emasligini ko'rasiz — bu capstone davomida o'rgangan eng muhim saboqni yakunlaydi: hech qanday compile vaqtidagi "OK" signali, hatto tsc'ning o'zinikidan ham, production'da hamma narsa to'g'ri ishlashini kafolatlamaydi. Faqat haqiqiy, jonli sinov (deploy qilib, ishga tushirib ko'rish) buni tasdiqlay oladi.

📌 Bu darsdan keyin siz bilasizki
✅ Path alias'lar (@shared/*) dev'da tsconfig-paths/register orqali runtime'da hal qilinadi
✅ tsc pathsni faqat tur tekshirish uchun ishlatadi — chiqargan JS'dagi import yo'llarini qayta yozmaydi
✅ Production'da tsc-alias kabi vosita bo'lmasa, node dist/... "Cannot find module" bilan yiqiladi
✅ tscning "0 xato" xabari ham runtime muvaffaqiyatini kafolatlamaydi
✅ Faqat haqiqiy deploy va jonli sinov — compile vaqtidagi har qanday "OK" signalidan ko'ra ishonchliroq tekshiruv
🎉 Tabriklaymiz!
Siz IssueForge'ni 1-bosqichdagi bo'sh repo'dan boshlab, umumiy TypeScript sxemasi, Express + TypeScript backend, PostgreSQL bilan tiplashtirilgan so'rovlar, JWT autentifikatsiyasi, React + Redux Toolkit frontend, testlar va nihoyat to'g'ri, ikki qismli production deploygacha qurdingiz. Bu capstone davomida siz TypeScript Asoslari, Node.js/Express Asoslari va React: Redux Toolkit, TypeScript va Testlash kurslarida alohida o'rgangan bilimlarni bitta, real loyihada birlashtirdingiz — va eng muhimi, TypeScript'ning eng katta haqiqatini yetti xil ko'rinishda ko'rdingiz: u sizga compile vaqtida yordam beradi, lekin runtime'da hech narsani sizning o'rningizga tekshirmaydi.

💻
Kod
Код
#2
typescript
 Nusxalash
// ════════════════════════════════════════════════════════════════════
// 7-BOSQICH (CAPSTONE YAKUNI): Deploy va path alias xatosi
// ════════════════════════════════════════════════════════════════════

// ─────────────────────────────────────────────────────────────────────
// 1) backend/tsconfig.json - path alias sozlash
// ─────────────────────────────────────────────────────────────────────

// {
//   "compilerOptions": {
//     "baseUrl": ".",
//     "paths": { "@shared/*": ["../shared/*"] }
//   }
// }

import { Issue } from '@shared/types';

// ─────────────────────────────────────────────────────────────────────
// 2) package.json - dev va TO'G'RI build (izohda - JSON, kod emas)
// ─────────────────────────────────────────────────────────────────────

// {
//   "scripts": {
//     "dev": "ts-node -r tsconfig-paths/register src/server.ts",
//     "build": "tsc && tsc-alias"
//   }
// }
//
// npm install -D tsconfig-paths tsc-alias

// ─────────────────────────────────────────────────────────────────────
// 3) Ataylab xato - tsc-alias'siz build (izohda)
// ─────────────────────────────────────────────────────────────────────

// {
//   "scripts": { "build": "tsc" }        // tsc-alias YO'Q!
// }
//
// $ npm run build   -> tsc: 0 xato
// $ cat dist/server.js
//   const types_1 = require("@shared/types");   // o'zgartirilmagan!
// $ node dist/server.js
//   -> Error: Cannot find module '@shared/types'
