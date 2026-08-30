# ============================================================
# Bot API (aiogram) vs MTProto (Telethon) -- bir vazifa, ikki yondashuv
# ============================================================

# --- 1) aiogram (Bot API) -- 48-kursdan tanish shakl -----------------
from aiogram import Bot, Dispatcher, Router
from aiogram.filters import Command
from aiogram.types import Message

bot = Bot(token="123456:BOT_TOKEN")          # bot HISOBINI aniqlaydi
dp = Dispatcher()
router = Router()


@router.message(Command("ping"))
async def aiogram_ping(message: Message) -> None:
    # Bu handler faqat botga yuborilgan /ping buyrug'iga javob beradi.
    # Bot faqat: (a) unga yuborilgan, (b) u qo'shilgan guruhdagi
    # xabarlarni ko'radi -- guruhning eski tarixini o'qiy olmaydi.
    await message.answer("pong (Bot API orqali)")


dp.include_router(router)
# asyncio.run(dp.start_polling(bot))


# --- 2) Telethon (MTProto) -- shu kursning shakli --------------------
from telethon import TelegramClient, events

api_id = 123456          # my.telegram.org'dan -- ILOVANI aniqlaydi
api_hash = "abcdef0123456789abcdef0123456789"

# "session_name" -- diskdagi session fayl nomi (2-darsda batafsil).
# Birinchi ishga tushirishda telefon raqami + kod so'raladi -- shundan
# keyin bu HAQIQIY FOYDALANUVCHI HISOBI nomidan ishlaydigan client.
client = TelegramClient("session_name", api_id, api_hash)


@client.on(events.NewMessage(pattern="/ping"))
async def telethon_ping(event: events.NewMessage.Event) -> None:
    # Bu handler HAR QANDAY chatda ishlaydi -- shaxsiy, guruh, kanal --
    # chunki bu endi "botga kelgan update" emas, balki "hisobim
    # ishtirok etayotgan har qanday suhbatdagi yangi xabar" hodisasi.
    await event.reply("pong (Telethon/MTProto orqali)")


# with client:
#     client.run_until_disconnected()


# --- 3) Imkoniyatlar solishtiruvi -- shu darsning asosiy xulosasi ----
BOT_API_CAPABILITIES = {
    "faqat o'ziga tegishli xabarlarni ko'radi": True,
    "guruhning eski tarixini o'qiydi (admin bo'lmasa)": False,
    "username orqali kanalga o'zi qo'shiladi": False,
    "kontaktlar/dialoglar ro'yxatiga kiradi": False,
    "odam sifatida ko'rinadi (Bot belgisisiz)": False,
}

USERBOT_CAPABILITIES = {
    "faqat o'ziga tegishli xabarlarni ko'radi": False,  # barchasini ko'radi
    "guruhning eski tarixini o'qiydi (admin bo'lmasa)": True,
    "username orqali kanalga o'zi qo'shiladi": True,
    "kontaktlar/dialoglar ro'yxatiga kiradi": True,
    "odam sifatida ko'rinadi (Bot belgisisiz)": True,
}


def print_comparison() -> None:
    print(f"{'Imkoniyat':<55}{'Bot API':>10}{'Userbot':>10}")
    for key in BOT_API_CAPABILITIES:
        print(f"{key:<55}{str(BOT_API_CAPABILITIES[key]):>10}{str(USERBOT_CAPABILITIES[key]):>10}")


if __name__ == "__main__":
    print_comparison()
