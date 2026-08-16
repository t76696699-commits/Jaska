# ════════════════════════════════════════════════════════════════════
# 6-BOSQICH: Avtomatik bildirishnomalar
# ════════════════════════════════════════════════════════════════════

# ─────────────────────────────────────────────────────────────────────
# 1) studymate/management/commands/send_reminders.py
# ─────────────────────────────────────────────────────────────────────

import requests
from django.core.management.base import BaseCommand
from django.utils import timezone
from datetime import timedelta
from studymate.models import Topshiriq

BOT_TOKEN = "..."  # environment o'zgaruvchisidan olinadi


class Command(BaseCommand):
    help = "Muddati yaqinlashgan topshiriqlar uchun Telegram orqali eslatma yuboradi"

    def handle(self, *args, **options):
        hozir = timezone.now()
        chegara = hozir + timedelta(hours=24)

        topshiriqlar = Topshiriq.objects.filter(
            bajarilgan=False,
            muddat_vaqti__gte=hozir,
            muddat_vaqti__lte=chegara,
        ).exclude(
            user__profile__telegram_chat_id__isnull=True
        ).select_related('user__profile', 'fan')

        for t in topshiriqlar:
            self.xabar_yuborish(t)

    def xabar_yuborish(self, topshiriq):
        chat_id = topshiriq.user.profile.telegram_chat_id
        matn = f"⏰ Eslatma: '{topshiriq.sarlavha}' ({topshiriq.fan.nomi}) muddati yaqinlashmoqda!"
        requests.post(
            f"https://api.telegram.org/bot{BOT_TOKEN}/sendMessage",
            json={"chat_id": chat_id, "text": matn},
        )

# ─────────────────────────────────────────────────────────────────────
# 2) crontab (izohda - server sozlamasi, Python emas)
# ─────────────────────────────────────────────────────────────────────

# 0 * * * * cd /path/to/django_backend && python manage.py send_reminders

# ─────────────────────────────────────────────────────────────────────
# 3) Ataylab xato - filtrlashni unutish (izohda)
# ─────────────────────────────────────────────────────────────────────

# def handle_xato(self, *args, **options):
#     topshiriqlar = Topshiriq.objects.filter(
#         bajarilgan=False,
#         muddat_vaqti__lte=timezone.now() + timedelta(hours=24),
#     ).select_related('user__profile', 'fan')   # .exclude(...) YO'Q!
#     for t in topshiriqlar:
#         chat_id = t.user.profile.telegram_chat_id   # None bo'lishi mumkin!
#         requests.post(url, json={"chat_id": chat_id, "text": "..."})
