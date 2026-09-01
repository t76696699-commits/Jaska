# test_handlers.py -- hendlerlarni yupqa qilib, biznes-mantiqni alohida testlash
from datetime import datetime
from unittest.mock import AsyncMock

import pytest
from aiogram import Bot, Dispatcher, F, Router
from aiogram.types import Chat, Message, Update, User

router = Router()


# ---- 1) Sof biznes-mantiq -- aiogram'ga bog'liq emas, oddiy funksiya ----
def format_greeting(first_name: str) -> str:
    return f"Salom, {first_name}! Botga xush kelibsiz."


# ---- 2) Yupqa handler -- faqat o'qiydi, chaqiradi, javob beradi ----
@router.message(F.text == "/start")
async def start_handler(message: Message) -> None:
    text = format_greeting(message.from_user.first_name)
    await message.answer(text)


# ---- 3) pytest fixture'lari ----
@pytest.fixture
def dp() -> Dispatcher:
    d = Dispatcher()
    d.include_router(router)
    return d


@pytest.fixture
def bot() -> AsyncMock:
    return AsyncMock(spec=Bot)


def _make_update(text: str) -> Update:
    chat = Chat(id=123, type="private")
    user = User(id=42, is_bot=False, first_name="Aziz")
    message = Message(message_id=1, date=datetime.now(), chat=chat, from_user=user, text=text)
    return Update(update_id=1, message=message)


# ---- 4a) Sof funksiyani testlash -- aiogram umuman ishtirok etmaydi ----
def test_format_greeting() -> None:
    assert format_greeting("Aziz") == "Salom, Aziz! Botga xush kelibsiz."


# ---- 4b) Integratsion test -- Dispatcher orqali to'liq zanjirni tekshirish ----
@pytest.mark.asyncio
async def test_start_handler_replies(dp: Dispatcher, bot: AsyncMock) -> None:
    update = _make_update("/start")

    await dp.feed_update(bot, update)

    bot.send_message.assert_called_once_with(
        chat_id=123, text="Salom, Aziz! Botga xush kelibsiz."
    )


# ---- 5) pytest.ini / pyproject.toml sozlamasi (izoh sifatida) ----
# [tool.pytest.ini_options]
# asyncio_mode = "auto"
# testpaths = ["tests"]
#
# Shu sozlama bilan @pytest.mark.asyncio har bir testga qo'lda yozilmaydi --
# pytest-asyncio barcha "async def test_..." funksiyalarni avtomatik taniydi.
