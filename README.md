1-Telegram Mini Apps (WebApp): web_app tugmasi va Telegram.WebApp JS API
Урок 1 из 14
· 3 раздела
📝
Matn
Matn
#1
🛠️ Muhit: bu backend kursi uchun PyCharm 2023.3 dan foydalaning. Terminalni ochish: Alt + F12.

Oddiy bot doirasidan tashqariga chiqish
48-kursda siz reply va inline klaviaturalar bilan ishladingiz — foydalanuvchi tugmani bosadi, bot matn yoki callback_data oladi. Bu yetarli bo'lgan holatlar ko'p, lekin forma to'ldirish, katalog ko'rish, xarita tanlash kabi vazifalarda matn-asosli interfeys tezda noqulay bo'lib qoladi. Telegram buning uchun Mini App (WebApp) deb ataladigan mexanizmni taqdim etadi — bot ichida ochiladigan to'liq huquqli veb-sahifa, HTML/CSS/JS bilan yozilgan, lekin Telegram'ning o'zi bilan integratsiyalashgan.

web_app tugmasi: ikki xil joyda, ikki xil xulq-atvor
Mini App'ni ochish uchun ikkita joy bor, va ular bir xil emas:

KeyboardButton(text="...", web_app=WebAppInfo(url="...")) — reply klaviaturada, faqat shaxsiy chatda ishlaydi. Sahifa Telegram.WebApp.sendData(text) chaqirsa, bot tomonga oddiy message update keladi, unda content_type == "web_app_data" va message.web_app_data.data ichida yuborilgan matn bo'ladi.
InlineKeyboardButton(text="...", web_app=WebAppInfo(url="...")) — botning o'z xabarida ishlaydi (guruhda ham), lekin natijani qaytarish uchun boshqa mexanizm kerak — inline rejimda answerWebAppQuery orqali.
Amalda ko'p holatda kerak bo'ladigani — birinchisi: shaxsiy chatda "Katalog ochish" tugmasi, foydalanuvchi tanlov qiladi, sahifa sendData chaqiradi, bot javob beradi.

Telegram.WebApp JS API — sahifa ichida nima mavjud
Mini App ochilganda, sahifangizga <script src="https://telegram.org/js/telegram-web-app.js"></script> ulanadi va global window.Telegram.WebApp obyekti paydo bo'ladi:

Metod / xususiyat	Vazifasi
WebApp.ready()	Sahifa tayyor ekanini Telegram'ga bildiradi — splash screen yopiladi
WebApp.expand()	Sahifani to'liq balandlikka kengaytiradi
WebApp.close()	Mini App'ni yopadi
WebApp.MainButton	Pastki katta tugma — setText(), show(), onClick()
WebApp.BackButton	Yuqori chap orqaga tugmasi
WebApp.themeParams / colorScheme	Foydalanuvchi Telegram mavzusiga (dark/light) moslashtirish uchun ranglar
WebApp.initData / initDataUnsafe	Foydalanuvchi haqida imzolangan ma'lumot — keyingi darsda tekshiramiz
WebApp.sendData(data)	Ma'lumotni botga web_app_data sifatida yuboradi va sahifani yopadi
WebApp.HapticFeedback	Qurilmada mayin tebranish (vibratsiya) effektlari
Nega initDataUnsafe nomida "Unsafe" so'zi bor
Bu ataylab shunday nomlangan — initDataUnsafe allaqachon JavaScript obyektiga parslangan, lekin hali tasdiqlanmagan ma'lumot. Har qanday foydalanuvchi brauzer konsolida shu obyektni o'zgartirishi mumkin. Backend'ingiz unga ishonib, masalan, user.id asosida ma'lumotlar bazasidan yozuv qaytarsa — boshqa birov o'zini xohlagan foydalanuvchi qilib ko'rsatishi mumkin. Xom initData qatorini imzo (hash) bilan tekshirish — bu keyingi darsning butun mavzusi.

Arxitektura: qayerda nima yashaydi
Mini App uchun kamida ikkita qism kerak: HTTPS orqali xizmat qiladigan statik sahifa (Telegram faqat https:// manzillarni qabul qiladi, http:// yoki localhost ishlamaydi — test uchun ngrok kabi tunnel kerak bo'ladi) va aiogram tomonidagi bot kodi, u tugmani ro'yxatga oladi va qaytgan ma'lumotni qayta ishlaydi. Ikkalasi bitta serverda ham, alohida serverlarda ham bo'lishi mumkin — muhimi, HTTPS.

Foydalanuvchi: 'Katalog' tugmasini bosadi

Telegram: WebView'da sahifani ochadi

Sahifa JS: WebApp.ready() + WebApp.expand()

Foydalanuvchi: mahsulot tanlaydi

Sahifa JS: WebApp.sendData(JSON)

Telegram: sahifani yopadi

Bot: message update, content_type=web_app_data

aiogram handler: message.web_app_data.data ni o'qiydi

Diagramma shuni ko'rsatadi: sendData chaqirilgach, Telegram avtomatik ravishda sahifani yopadi va botga oddiy xabar sifatida ma'lumot yuboradi — alohida webhook yoki API chaqiruvi shart emas.

Fetch orqali muloqot — sendData'dan farqli yo'l
sendData faqat bitta marta, sahifa yopilishidan oldin ishlaydi. Agar sahifa ochiq turgan holda backend bilan doimiy almashinuv kerak bo'lsa (masalan, katalogni yuklash, buyurtma holatini kuzatish), oddiy fetch() orqali o'z backend API'ingizga so'rov yuborilaveradi — bu holatda foydalanuvchini aniqlash uchun initData'ni so'rov sarlavhasida yuborish va uni backend'da tekshirish kerak bo'ladi (keyingi darsning mavzusi).

💻
Kod
Kod
#2
python
 Nusxalash
from aiogram import Router, F
from aiogram.filters import Command
from aiogram.types import Message, KeyboardButton, ReplyKeyboardMarkup, WebAppInfo

router = Router()

MINI_APP_URL = "https://mysite.example.com/catalog"  # HTTPS shart


def catalog_keyboard() -> ReplyKeyboardMarkup:
    return ReplyKeyboardMarkup(
        keyboard=[[KeyboardButton(text="Katalogni ochish", web_app=WebAppInfo(url=MINI_APP_URL))]],
        resize_keyboard=True,
    )


@router.message(Command("shop"))
async def cmd_shop(message: Message):
    await message.answer(
        "Katalogni ko'rish uchun quyidagi tugmani bosing:",
        reply_markup=catalog_keyboard(),
    )


@router.message(F.web_app_data)
async def handle_web_app_data(message: Message, db_session):
    import json

    raw = message.web_app_data.data
    try:
        payload = json.loads(raw)
    except json.JSONDecodeError:
        await message.answer("Noto'g'ri ma'lumot formati.")
        return

    product_id = payload.get("product_id")
    quantity = payload.get("quantity", 1)
    await message.answer(
        f"Buyurtma qabul qilindi: mahsulot #{product_id}, {quantity} dona."
    )
    # bu yerda haqiqiy buyurtma yozuvini DB'ga saqlash kerak bo'ladi


# index.html / app.js (Mini App sahifasi) — to'liq versiyasi "sample" bo'limida.
# Sahifa tomonidagi asosiy chaqiruv shunday ko'rinadi:
#
#     Telegram.WebApp.sendData(JSON.stringify({"product_id": 42, "quantity": 2}));
#
# sendData chaqirilgach, Telegram sahifani avtomatik yopadi va
# yuqoridagi handle_web_app_data handleri ishga tushadi.
