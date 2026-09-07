8-Katta hajmdagi ma'lumotlar: chat tarixi va a'zolar bo'yicha async iteratsiya
Урок 8 из 14
· 3 раздела
✓ Пройден
📝
Matn
Matn
#1
Bot API'da yo'q, MTProto'da bor: chuqur tarix
Bot API orqali botlar chatning eski xabarlarini "orqaga qarab" o'qiy olmaydi — faqat kelayotgan update'larni ko'radi. Pyrogram MTProto darajasida ishlagani uchun bu cheklovga ega emas: get_chat_history() orqali istalgan chatning butun (ruxsat etilgan) tarixini orqaga qarab sahifalab o'qish mumkin — bu foydalanuvchi rejimida ayniqsa kuchli (chunki foydalanuvchi hisobi allaqachon o'sha chatda "bo'lgan").

async for — Pyrogram'ning sahifalashni yashiruvchi uslubi
Katta ro'yxatlar (tarix, a'zolar, dialoglar) uchun Pyrogram alohida "keyingi sahifa" so'rovi yozishga majburlamaydi — buning o'rniga async generator qaytaradi, va siz oddiy async for bilan iteratsiya qilaverasiz, Pyrogram orqa fonda avtomatik ravishda keyingi sahifalarni so'rab turadi:

async for message in client.get_chat_history(chat_id, limit=1000):
    print(message.id, message.text)
limit=0 (yoki umuman ko'rsatilmasa) — cheksiz, ya'ni butun tarix oxirigacha o'qiladi. Katta kanallar uchun bu millionlab xabar bo'lishi mumkin, shuning uchun har doim oqilona limit qo'yish yoki natijani real vaqtda qayta ishlab, xotirada to'plamaslik kerak.

Kanal/guruh a'zolarini ro'yxatlash
async for member in client.get_chat_members(chat_id):
    print(member.user.id, member.user.username, member.status)
Katta ochiq kanallar (o'nlab minglab a'zo) uchun bu operatsiya sezilarli vaqt olishi va Telegram'ning flood-limitlariga tegishi mumkin — ayniqsa userbot rejimida tez-tez qo'llanilsa. filter= parametri orqali faqat ma'lum status (masalan ChatMembersFilter.ADMINISTRATORS) bo'yicha cheklash tezlikni oshiradi.

FloodWait: MTProto'ning tabiiy rate-limit signali
Ko'p so'rov yuborilganda Telegram FloodWait xatosini qaytaradi — unda qancha soniya kutish kerakligi ko'rsatiladi. To'g'ri yondashuv — buni try/except bilan ushlab, ko'rsatilgan vaqt davomida asyncio.sleep() qilib, keyin qayta urinish:

from pyrogram.errors import FloodWait
import asyncio

async def safe_iterate(client, chat_id):
    while True:
        try:
            async for msg in client.get_chat_history(chat_id, limit=5000):
                process(msg)
            break
        except FloodWait as e:
            await asyncio.sleep(e.value)
ha

yo'q

FloodWait xatosi

async for x in client.get_chat_history(...)

Pyrogram bitta 'sahifa'ni
so'raydi (ichkarida)

Har bir elementni
navbat bilan qaytaradi

Yana sahifa bormi?

Iteratsiya tugaydi

asyncio.sleep(e.value)

Diagram async for ortida yashiringan sahifalash mexanizmini va FloodWait uchrasa qayta urinish oqimini ko'rsatadi — kod darajasida siz bularning hech birini qo'lda yozmaysiz, faqat oddiy for-loop ko'rasiz.

Xotira bo'yicha eng ko'p uchraydigan xato
Yangi boshlovchilar ko'pincha results = [m async for m in get_chat_history(...)] kabi ro'yxat qurib, keyin uni qayta ishlaydi — bu katta kanallar uchun butun tarixni xotiraga yuklashga urinishni anglatadi va xotira tugashiga olib kelishi mumkin. To'g'ri yondashuv — har bir elementni async for ichida darhol qayta ishlash (yozish, hisoblash, filtrlash), ro'yxatga yig'masdan.

get_dialogs(): barcha suhbatlar ro'yxati (faqat foydalanuvchi rejimida)
Yana bir Bot API'da mavjud bo'lmagan imkoniyat — get_dialogs() orqali hisobingiz a'zo bo'lgan barcha shaxsiy chat, guruh va kanallar ro'yxatini async iteratsiya qilish. Bu faqat foydalanuvchi (userbot) rejimida ma'noga ega — bot faqat o'zi qo'shilgan chatlarni "biladi", lekin "barcha dialoglarim" degan tushunchaga ega emas, chunki u hech qachon o'z xohishi bilan biror joyga qo'shilmagan:

async for dialog in app.get_dialogs():
    print(dialog.chat.title or dialog.chat.first_name, dialog.unread_messages_count)
Bu real hayotda, masalan, "500 tadan ortiq o'qilmagan xabarli barcha kanallarni arxivlash" kabi avtomatlashtirish skriptlarida ishlatiladi.

💻
Kod
Kod
#2
python
 Nusxalash
import asyncio
from pyrogram import Client
from pyrogram.errors import FloodWait
from pyrogram.enums import ChatMembersFilter

app = Client("history_demo")


async def count_messages_with_word(client: Client, chat_id: int, word: str, limit: int = 5000) -> int:
    """Xotirada ro'yxat yig'masdan, har bir xabarni darhol tekshiradi."""
    count = 0
    while True:
        try:
            async for message in client.get_chat_history(chat_id, limit=limit):
                if message.text and word.lower() in message.text.lower():
                    count += 1
            break
        except FloodWait as e:
            await asyncio.sleep(e.value)
    return count


async def list_admins(client: Client, chat_id: int) -> list[str]:
    admins = []
    async for member in client.get_chat_members(chat_id, filter=ChatMembersFilter.ADMINISTRATORS):
        admins.append(member.user.username or str(member.user.id))
    return admins


async def main():
    async with app:
        chat_id = -1001234567890
        total = await count_messages_with_word(app, chat_id, "pyrogram")
        print(f"'pyrogram' so'zi {total} marta uchradi")

        admins = await list_admins(app, chat_id)
        print("Adminlar:", admins)


if __name__ == "__main__":
    asyncio.run(main())
