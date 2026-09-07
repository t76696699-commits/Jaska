from pyrogram import Client, filters
from pyrogram.types import (
    InlineKeyboardMarkup, InlineKeyboardButton,
    InlineQueryResultArticle, InputTextMessageContent,
)

app = Client("callback_inline_demo")

_PRODUCTS = {"1": "Kitob", "2": "Ruchka", "3": "Daftar"}


@app.on_message(filters.command("shop"))
async def shop(client, message):
    buttons = [
        [InlineKeyboardButton(name, callback_data=f"prod:view:{pid}")]
        for pid, name in _PRODUCTS.items()
    ]
    await message.reply_text("Mahsulotlar:", reply_markup=InlineKeyboardMarkup(buttons))


@app.on_callback_query(filters.regex(r"^prod:view:"))
async def view_product(client, callback_query):
    _, _, product_id = callback_query.data.split(":")
    name = _PRODUCTS.get(product_id, "Noma'lum")
    await callback_query.answer()  # majburiy — "yuklanmoqda"ni olib tashlaydi
    buttons = [[InlineKeyboardButton("Xarid qilish", callback_data=f"prod:buy:{product_id}")]]
    await callback_query.message.edit_text(
        f"Siz tanladingiz: {name}", reply_markup=InlineKeyboardMarkup(buttons)
    )


@app.on_callback_query(filters.regex(r"^prod:buy:"))
async def buy_product(client, callback_query):
    await callback_query.answer("Buyurtma qabul qilindi!", show_alert=True)


@app.on_inline_query()
async def search_products(client, inline_query):
    q = inline_query.query.lower()
    results = [
        InlineQueryResultArticle(
            title=name,
            input_message_content=InputTextMessageContent(f"{name} — narxini so'rash uchun /shop"),
        )
        for pid, name in _PRODUCTS.items()
        if q in name.lower()
    ]
    await inline_query.answer(results, cache_time=1)


if __name__ == "__main__":
    app.run()
