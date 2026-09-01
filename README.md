from aiogram import Router
from aiogram.types import Message, PreCheckoutQuery, LabeledPrice
from aiogram.filters import Command

router = Router()

PROVIDER_TOKEN = "381764678:TEST:12345"  # BotFather orqali olingan, .env'da saqlanadi
CURRENCY = "UZS"


def to_minor_units(sum_uzs: int) -> int:
    """Provayder talab qiladigan eng kichik birlikka o'tkazadi.
    Aniq koeffitsientni ishlatilayotgan provayderning hujjatidan tasdiqlang."""
    return sum_uzs * 100


@router.message(Command("buy"))
async def cmd_buy(message: Message, db_session):
    product = await get_product(db_session, sku="premium_1m")
    if product.stock <= 0:
        await message.answer("Kechirasiz, mahsulot tugagan.")
        return

    await message.bot.send_invoice(
        chat_id=message.chat.id,
        title=product.title,
        description=product.description,
        payload=f"order:{product.id}:{message.from_user.id}",
        provider_token=PROVIDER_TOKEN,
        currency=CURRENCY,
        prices=[LabeledPrice(label=product.title, amount=to_minor_units(product.price_uzs))],
    )


@router.pre_checkout_query()
async def handle_pre_checkout(pre_checkout_query: PreCheckoutQuery, db_session):
    _, product_id, user_id = pre_checkout_query.invoice_payload.split(":")
    product = await get_product(db_session, id=int(product_id))

    if product is None or product.stock <= 0:
        await pre_checkout_query.bot.answer_pre_checkout_query(
            pre_checkout_query.id, ok=False,
            error_message="Kechirasiz, mahsulot tugab qoldi. Pul yechilmadi.",
        )
        return

    expected_amount = to_minor_units(product.price_uzs)
    if pre_checkout_query.total_amount != expected_amount:
        await pre_checkout_query.bot.answer_pre_checkout_query(
            pre_checkout_query.id, ok=False, error_message="Narx nomuvofiqligi aniqlandi.",
        )
        return

    await pre_checkout_query.bot.answer_pre_checkout_query(pre_checkout_query.id, ok=True)


async def get_product(db_session, **filters):
    ...  # repository qatlamidan haqiqiy so'rov
