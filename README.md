from aiogram import Router, F
from aiogram.filters import Command
from aiogram.types import Message, LabeledPrice
from sqlalchemy.exc import IntegrityError

router = Router()


@router.message(F.successful_payment)
async def handle_successful_payment(message: Message, db_session):
    payment = message.successful_payment

    try:
        await save_payment_record(
            db_session,
            charge_id=payment.telegram_payment_charge_id,
            invoice_payload=payment.invoice_payload,
            amount=payment.total_amount,
            currency=payment.currency,
        )
    except IntegrityError:
        # charge_id UNIQUE ustunga to'qnashdi — bu update allaqachon qayta ishlangan,
        # xizmatni ikkinchi marta bermaslik uchun shu yerda to'xtaymiz
        await db_session.rollback()
        return

    _, product_id, user_id = payment.invoice_payload.split(":")
    await grant_access(db_session, user_id=int(user_id), product_id=int(product_id))
    await message.answer("To'lovingiz uchun rahmat! Xizmat faollashtirildi.")


async def save_payment_record(db_session, *, charge_id: str, invoice_payload: str, amount: int, currency: str):
    ...  # INSERT ... charge_id UNIQUE bo'lgan jadvalga


async def grant_access(db_session, *, user_id: int, product_id: int):
    ...  # foydalanuvchiga xizmatni yoqish


# --- Telegram Stars orqali invoice (raqamli tovar uchun) ---
@router.message(Command("buy_stars"))
async def cmd_buy_with_stars(message: Message):
    await message.bot.send_invoice(
        chat_id=message.chat.id,
        title="Premium obuna (1 oy)",
        description="Reklamasiz, cheksiz so'rovlar",
        payload=f"stars_order:premium_1m:{message.from_user.id}",
        provider_token="",       # Stars uchun bo'sh
        currency="XTR",
        prices=[LabeledPrice(label="Premium 1 oy", amount=100)],  # 100 Stars
    )


# --- Stars to'lovini qaytarish ---
async def refund_stars_payment(bot, user_id: int, charge_id: str) -> bool:
    """Faqat XTR (Stars) to'lovlar uchun ishlaydi — fiat uchun provayder
    paneli/API orqali qaytariladi, bu metod orqali emas."""
    return await bot.refund_star_payment(user_id=user_id, telegram_payment_charge_id=charge_id)
