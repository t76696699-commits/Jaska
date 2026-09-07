R1-Takrorlash: dual-mode Client, handlerlar, filters va sessiya xavfsizligi
Урок 9 из 14
· 3 раздела
📝
Matn
Matn
#1
Bu dars — sof takrorlash, yangi mavzu yo'q
Bu qisqa takrorlash darsi ataylab yengil — yangi API yoki kontseptsiya kiritilmaydi, faqat 1-8 darslarda o'rganilganlar amaliy loyihaga birlashtiriladi. Shuning uchun matn qisqa: asosiy e'tibor pastdagi amaliy vazifada.

Qisqacha xotira jadvali
Dars	Asosiy g'oya
1-2	Pyrogram MTProto ustida ishlaydi; bitta Client bot yoki userbot bo'lishi mumkin
3	@app.on_message dekoratorlari, group= va continue_propagation()
4	filters moduli, & / | / ~ bilan birlashtirish, filters.create()
5	Sessiya fayl = auth_key; workdir, in_memory, session_string xavfsizligi
6	InlineKeyboardMarkup, parse_mode, send_media_group
7	callback_query.answer() majburiy; inline_query.answer(results)
8	async for + get_chat_history/get_chat_members, FloodWait bilan ishlash
Quyidagi amaliy vazifa aynan shu yettita bo'lakni bitta kichik, lekin to'liq ishlaydigan botga birlashtirishni talab qiladi.

💻
Kod
Kod
#2
python
 Nusxalash
# Takrorlash darsi — yangi kod yo'q. Amaliy vazifa (pastda) barcha
# o'rganilgan qismlarni (dual-mode, handlerlar, filters, klaviaturalar,
# callback, async iteratsiya) birlashtiradi.
🎯
Mashqlar
#3
🎯
2 заданий
 11 pts всего
#1
Medium
 6 pts
 +6 pts
🔘 Выбор ответа
R1: dual-mode va filters bo'yicha bilim tekshiruvi
Client'ni bot rejimida ishga tushirish uchun qaysi parametr beriladi va admin-tekshiruvi uchun qaysi funksiya ishlatiladi? To'g'ri juftlikni tanlang.

A
bot_token va filters.create()
✓

B
phone_number va filters.command()

C
session_string va filters.text

D
api_hash va filters.user()
💡 Показать подсказку
🎉
Правильно! Отличная работа!
AI Feedback
To'g'ri!
✓ Выполнено
#2
Easy
 5 pts
✏️ Заполни пропуск
R1: majburiy callback metodi
Har qanday callback_query handlerida chaqirilishi shart bo'lgan metod: callback_query.
1
()
💡 Показать подсказку
✅ Проверить ответ
