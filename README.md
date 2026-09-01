import hashlib
import hmac
import time
from urllib.parse import parse_qsl


class InitDataError(Exception):
    """initData tekshiruvidan o'tmadi."""


def validate_init_data(init_data: str, bot_token: str, max_age_seconds: int = 86400) -> dict:
    """Telegram Mini App initData'sini HMAC-SHA256 orqali tekshiradi.

    Muvaffaqiyatli bo'lsa parslangan maydonlar lug'atini qaytaradi
    (shu jumladan 'user' -> dict). Muvaffaqiyatsiz bo'lsa InitDataError.
    """
    pairs = dict(parse_qsl(init_data, strict_parsing=True))
    received_hash = pairs.pop("hash", None)
    if not received_hash:
        raise InitDataError("initData ichida 'hash' maydoni yo'q")

    data_check_string = "\n".join(
        f"{key}={value}" for key, value in sorted(pairs.items())
    )

    secret_key = hmac.new(
        key=b"WebAppData", msg=bot_token.encode(), digestmod=hashlib.sha256
    ).digest()
    computed_hash = hmac.new(
        key=secret_key, msg=data_check_string.encode(), digestmod=hashlib.sha256
    ).hexdigest()

    if not hmac.compare_digest(computed_hash, received_hash):
        raise InitDataError("hash mos kelmadi — initData soxta yoki buzilgan")

    auth_date = int(pairs.get("auth_date", 0))
    if time.time() - auth_date > max_age_seconds:
        raise InitDataError(f"initData eskirgan (auth_date {max_age_seconds}s dan katta)")

    import json
    if "user" in pairs:
        pairs["user"] = json.loads(pairs["user"])
    return pairs


# --- FastAPI'da ishlatish ---
from fastapi import FastAPI, Header, HTTPException

app = FastAPI()
BOT_TOKEN = "123456:ABC-DEF..."  # .env'dan olinadi, hech qachon kodga yozilmaydi


@app.get("/api/profile")
async def profile(authorization: str = Header(...)):
    if not authorization.startswith("tma "):
        raise HTTPException(401, "Noto'g'ri Authorization format")
    init_data = authorization.removeprefix("tma ")
    try:
        data = validate_init_data(init_data, BOT_TOKEN)
    except InitDataError as e:
        raise HTTPException(401, str(e))
    user = data["user"]
    return {"telegram_id": user["id"], "first_name": user.get("first_name")}


def _build_test_init_data(bot_token: str, user: dict, auth_date: int) -> str:
    """Faqat testlar uchun: haqiqiy initData'ga o'xshash, to'g'ri imzolangan
    qatorni qo'lda yasab beradi — validate_init_data'ni Telegram'siz sinash uchun."""
    import json
    from urllib.parse import urlencode

    fields = {"auth_date": str(auth_date), "query_id": "AAH1234", "user": json.dumps(user)}
    data_check_string = "\n".join(f"{k}={v}" for k, v in sorted(fields.items()))
    secret_key = hmac.new(b"WebAppData", bot_token.encode(), hashlib.sha256).digest()
    fields["hash"] = hmac.new(secret_key, data_check_string.encode(), hashlib.sha256).hexdigest()
    return urlencode(fields)


def test_validate_init_data_ok():
    token = "TEST:TOKEN"
    raw = _build_test_init_data(token, {"id": 1, "first_name": "Aziz"}, int(time.time()))
    result = validate_init_data(raw, token)
    assert result["user"]["first_name"] == "Aziz"


def test_validate_init_data_tampered():
    token = "TEST:TOKEN"
    raw = _build_test_init_data(token, {"id": 1, "first_name": "Aziz"}, int(time.time()))
    tampered = raw.replace("Aziz", "Hacker")
    try:
        validate_init_data(tampered, token)
        assert False, "bu yerga yetib kelmasligi kerak"
    except InitDataError:
        pass
