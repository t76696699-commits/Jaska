# ════════════════════════════════════════════════════════════════════
# 7-BOSQICH (CAPSTONE YAKUNI): Uchta qismni birga deploy qilish
# ════════════════════════════════════════════════════════════════════

# ─────────────────────────────────────────────────────────────────────
# 1) Xizmat turlari (izohda - deploy platformasi tushunchasi, kod emas)
# ─────────────────────────────────────────────────────────────────────

# django_backend/  -> "Web Service" (HTTP so'rov kutadi, $PORT'da tinglaydi)
# frontend/         -> statik build (Vercel/Netlify, server kerak emas)
# telegram_bot/     -> "Background Worker" (doimiy ishlab turadi, polling)

# ─────────────────────────────────────────────────────────────────────
# 2) Environment o'zgaruvchilari (izohda)
# ─────────────────────────────────────────────────────────────────────

# django_backend/.env.example
# DATABASE_URL=postgresql://user:parol@host:5432/dbnomi
# BOT_TOKEN=...
# FRONTEND_URL=https://studymate.vercel.app

# telegram_bot/.env.example
# DATABASE_URL=postgresql://user:parol@host:5432/dbnomi
# BOT_TOKEN=...
# DJANGO_SETTINGS_MODULE=studymate.settings

# frontend/.env.production
# REACT_APP_API_URL=https://studymate-api.onrender.com

# ─────────────────────────────────────────────────────────────────────
# 3) telegram_bot/bot.py - to'g'ri ishga tushirish (polling)
# ─────────────────────────────────────────────────────────────────────

import asyncio


async def main():
    # ... dp = Dispatcher(), handler'lar ...
    # await dp.start_polling(bot)   # ❗ bu funksiya HECH QACHON qaytmaydi - doim ishlab turadi
    pass


if __name__ == "__main__":
    asyncio.run(main())

# ─────────────────────────────────────────────────────────────────────
# 4) Ataylab xato - botni "Web Service" sifatida sozlash (izohda)
# ─────────────────────────────────────────────────────────────────────

# Agar platform botdan $PORT'da HTTP javob kutsa, lekin bot buni
# hech qachon bermasa:
# ❌ Health check muvaffaqiyatsiz -> platform botni muntazam qayta ishga tushiradi
