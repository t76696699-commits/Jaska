# ═══════════════════════════════════════════════════════════════════════
# Strukturaviy logging + Sentry + Prometheus: bitta outer middleware
# ═══════════════════════════════════════════════════════════════════════
import time
import uuid
from typing import Any, Awaitable, Callable, Dict

import structlog
import sentry_sdk
from aiogram import BaseMiddleware
from aiogram.types import TelegramObject, Update
from prometheus_client import Counter, Histogram, start_http_server

# --- structlog konfiguratsiyasi: production'da JSON, dev'da o'qish qulay ---
structlog.configure(
    processors=[
        structlog.contextvars.merge_contextvars,
        structlog.processors.add_log_level,
        structlog.processors.TimeStamper(fmt="iso"),
        structlog.processors.JSONRenderer(),  # dev uchun ConsoleRenderer() ga almashtiring
    ],
)
log = structlog.get_logger()

# --- Sentry: faqat production'da yoqamiz ---
sentry_sdk.init(
    dsn="https://ornak-dsn@sentry.io/000000",
    traces_sample_rate=0.1,
    before_send=lambda event, hint: _strip_sensitive(event),
)


def _strip_sensitive(event: dict) -> dict:
    extra = event.get("extra", {})
    for key in ("password", "payment_token", "init_data"):
        extra.pop(key, None)
    return event


# --- Prometheus metrikalar ---
MESSAGES_TOTAL = Counter(
    "bot_messages_total", "Qayta ishlangan xabarlar soni", ["handler"]
)
HANDLER_DURATION = Histogram(
    "bot_handler_duration_seconds", "Handler bajarilish vaqti", ["handler"]
)


class ObservabilityMiddleware(BaseMiddleware):
    # Outer middleware — HAR BIR update uchun ishga tushadi, handler
    # topilgan-topilmaganidan qat'i nazar. Context bog'lash, metrika va
    # Sentry xatolik ushlash shu yerda birlashtiriladi.

    async def __call__(
        self,
        handler: Callable[[TelegramObject, Dict[str, Any]], Awaitable[Any]],
        event: Update,
        data: Dict[str, Any],
    ) -> Any:
        trace_id = str(uuid.uuid4())
        user = data.get("event_from_user")
        chat = data.get("event_chat")

        structlog.contextvars.clear_contextvars()
        structlog.contextvars.bind_contextvars(
            trace_id=trace_id,
            update_id=event.update_id,
            user_id=user.id if user else None,
            chat_id=chat.id if chat else None,
        )

        handler_name = data.get("handler", {}).__class__.__name__ if data.get("handler") else "unknown"
        started = time.perf_counter()
        log.info("update_received")

        try:
            result = await handler(event, data)
        except Exception as exc:  # noqa: BLE001 — qasddan: har qanday xatolikni ushlaymiz
            log.error("handler_failed", error=str(exc), exc_info=True)
            sentry_sdk.capture_exception(exc)
            raise
        else:
            elapsed = time.perf_counter() - started
            MESSAGES_TOTAL.labels(handler=handler_name).inc()
            HANDLER_DURATION.labels(handler=handler_name).observe(elapsed)
            log.info("update_processed", duration_ms=round(elapsed * 1000, 2))
            return result


def setup_observability(dp) -> None:
    dp.update.outer_middleware(ObservabilityMiddleware())
    start_http_server(9090)  # /metrics — http://localhost:9090/metrics
