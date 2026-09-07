from pyrogram import Client, filters
from pyrogram.types import (
    InlineKeyboardMarkup, InlineKeyboardButton, InputMediaPhoto,
)
from pyrogram.enums import ParseMode

app = Client("rich_messages_demo")


@app.on_message(filters.command("menu"))
async def show_menu(client, message):
    keyboard = InlineKeyboardMarkup([
        [InlineKeyboardButton("Katalog", callback_data="menu_catalog"),
         InlineKeyboardButton("Buyurtmalarim", callback_data="menu_orders")],
        [InlineKeyboardButton("Bizning sayt", url="https://example.com")],
    ])
    await message.reply_text(
        "**Asosiy menyu**\nKerakli bo'limni tanlang:",
        parse_mode=ParseMode.MARKDOWN,
        reply_markup=keyboard,
    )


@app.on_message(filters.command("album"))
async def send_album(client, message):
    await client.send_media_group(
        message.chat.id,
        [
            InputMediaPhoto("images/product1.jpg", caption="Yangi kolleksiya — 3 ta mahsulot"),
            InputMediaPhoto("images/product2.jpg"),
            InputMediaPhoto("images/product3.jpg"),
        ],
    )


# file_id'ni keshlash — qayta yuklashdan qochish
_cached_banner_id: str | None = None


@app.on_message(filters.command("banner"))
async def send_banner(client, message):
    global _cached_banner_id
    if _cached_banner_id:
        await message.reply_photo(_cached_banner_id)
        return
    sent = await message.reply_photo("images/banner.jpg", caption="Bizning banner")
    _cached_banner_id = sent.photo.file_id


if __name__ == "__main__":
    app.run()
