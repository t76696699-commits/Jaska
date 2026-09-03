# review_flow.py -- Mini App + initData + Payments -- bitta qisqa zanjir
from aiogram import Bot, F, Router
from aiogram.types import Message

router = Router()

# To'liq HMAC-SHA256 validatsiya kodi 2-darsda berilgan edi -- bu yerda
# faqat uning oqim ichidagi o'rni ko'rsatilgan.
from security import verify_init_data  # 2-darsdagi funksiya


async def handle_mini_app_order(bot: Bot, chat_id: int, init_data: str, product_id: str) -> None:
    user = verify_init_data(init_data, bot_token=bot.token)
    if user is None:
        raise PermissionError("initData yaroqsiz -- so'rov rad etildi")

    price = get_real_price(product_id)  # faqat serverdagi narx, mijozdan emas
    await bot.send_invoice(
        chat_id=chat_id,
        title="Buyurtma",
        description=f"Mahsulot #{product_id}",
        payload=f"order:{user['id']}:{product_id}",
        provider_token="PROVIDER_TOKEN",
        currency="UZS",
        prices=[{"label": "Narx", "amount": price}],
    )


@router.pre_checkout_query()
async def confirm_pre_checkout(pre_checkout_query) -> None:
    await pre_checkout_query.answer(ok=True)


@router.message(F.successful_payment)
async def mark_order_paid(message: Message) -> None:
    payload = message.successful_payment.invoice_payload
    mark_paid_in_db(payload, charge_id=message.successful_payment.telegram_payment_charge_id)
