# ════════════════════════════════════════════════════════════════════
# 5-BOSQICH: Telegram bot - tezkor xarajat va hisob bog'lash
# ════════════════════════════════════════════════════════════════════

import sys
import os
from datetime import date

sys.path.insert(0, os.path.join(os.path.dirname(__file__), '..', 'flask_backend'))

from app import create_app, db
from app.models import User, Category, Expense

app = create_app()

from aiogram import Bot, Dispatcher, types
from aiogram.filters import Command

bot = Bot(token=os.environ["BOT_TOKEN"])
dp = Dispatcher()


@dp.message(Command("link"))
async def link_handler(message: types.Message):
    qismlar = message.text.split()
    if len(qismlar) != 2:
        await message.answer("Foydalanish: /link KOD")
        return

    kod = qismlar[1]
    with app.app_context():
        user = User.query.filter_by(link_kodi=kod).first()
        if user is None:
            await message.answer("Kod noto'g'ri yoki eskirgan")
            return

        user.telegram_chat_id = message.chat.id
        user.link_kodi = None
        db.session.commit()

    await message.answer(f"✅ Hisobingiz bog'landi, {user.ism}!")


@dp.message()
async def tezkor_xarajat_handler(message: types.Message):
    qismlar = message.text.split(maxsplit=1)
    if len(qismlar) != 2 or not qismlar[0].isdigit():
        await message.answer("Format: SUMMA TAVSIF (masalan: 50000 ovqat)")
        return

    summa, tavsif = qismlar

    with app.app_context():
        user = User.query.filter_by(telegram_chat_id=message.chat.id).first()
        if user is None:
            await message.answer("Avval /link buyrug'i bilan hisobingizni bog'lang")
            return

        category = Category.query.filter_by(user_id=user.id, nomi=tavsif).first()
        if category is None:
            category = Category(nomi=tavsif, user_id=user.id)
            db.session.add(category)
            db.session.flush()

        xarajat = Expense(
            summa=summa, tavsif=tavsif, sana=date.today(),
            category_id=category.id, user_id=user.id,
        )
        db.session.add(xarajat)
        db.session.commit()

    await message.answer(f"✅ {summa} so'm '{tavsif}' uchun yozildi")

# ─────────────────────────────────────────────────────────────────────
# Ataylab xato - app_context()siz so'rov yuborish (izohda)
# ─────────────────────────────────────────────────────────────────────

# @dp.message(Command("link"))
# async def link_handler_xato(message: types.Message):
#     kod = message.text.split()[1]
#     user = User.query.filter_by(link_kodi=kod).first()   # app_context() YO'Q!
# ❌ RuntimeError: Working outside of application context.
