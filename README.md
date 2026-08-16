# ════════════════════════════════════════════════════════════════════
# 5-BOSQICH: Telegram bot - hisobni bog'lash va buyruqlar
# ════════════════════════════════════════════════════════════════════

# ─────────────────────────────────────────────────────────────────────
# 0) studymate/models.py - Profile modeli (telegram maydonlari uchun)
# ─────────────────────────────────────────────────────────────────────

# from django.db import models
# from django.contrib.auth.models import User
#
# class Profile(models.Model):
#     user = models.OneToOneField(User, on_delete=models.CASCADE, related_name='profile')
#     telegram_chat_id = models.BigIntegerField(null=True, blank=True)
#     link_kodi = models.CharField(max_length=10, null=True, blank=True)

# ─────────────────────────────────────────────────────────────────────
# 1) telegram_bot/bot.py - django.setup()
# ─────────────────────────────────────────────────────────────────────

import os
import django

os.environ.setdefault("DJANGO_SETTINGS_MODULE", "studymate.settings")
django.setup()

from studymate.models import Fan, Topshiriq, Profile
from django.contrib.auth.models import User

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
    try:
        user = await User.objects.aget(profile__link_kodi=kod)
    except User.DoesNotExist:
        await message.answer("Kod noto'g'ri yoki eskirgan")
        return

    user.profile.telegram_chat_id = message.chat.id
    user.profile.link_kodi = None
    await user.profile.asave()

    await message.answer(f"✅ Hisobingiz bog'landi, {user.first_name}!")


@dp.message(Command("topshiriqlar"))
async def topshiriqlar_handler(message: types.Message):
    try:
        user = await User.objects.aget(profile__telegram_chat_id=message.chat.id)
    except User.DoesNotExist:
        await message.answer("Avval /link buyrug'i bilan hisobingizni bog'lang")
        return

    topshiriqlar = [t async for t in Topshiriq.objects.filter(
        user=user, bajarilgan=False
    ).select_related('fan')]

    if not topshiriqlar:
        await message.answer("Bajarilmagan topshiriqlar yo'q 🎉")
        return

    matn = "\n".join(f"📌 {t.sarlavha} ({t.fan.nomi}) — {t.muddat_vaqti:%d.%m %H:%M}" for t in topshiriqlar)
    await message.answer(matn)

# ─────────────────────────────────────────────────────────────────────
# Ataylab xato - django.setup()dan OLDIN import (izohda)
# ─────────────────────────────────────────────────────────────────────

# from studymate.models import Fan, Topshiriq   # django.setup()DAN OLDIN!
#
# import os
# import django
# os.environ.setdefault("DJANGO_SETTINGS_MODULE", "studymate.settings")
# django.setup()
# ❌ django.core.exceptions.AppRegistryNotReady: Apps aren't loaded yet.
