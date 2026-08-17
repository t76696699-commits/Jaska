# ════════════════════════════════════════════════════════════════════
# 6-BOSQICH: Oylik hisobot va byudjet ogohlantirishi
# ════════════════════════════════════════════════════════════════════

# ─────────────────────────────────────────────────────────────────────
# 1) app/commands.py - Flask CLI komandasi
# ─────────────────────────────────────────────────────────────────────

import click
from flask import current_app
from sqlalchemy import func
from datetime import date
import requests
from app import db
from app.models import User, Expense

BOT_TOKEN = "..."  # environment o'zgaruvchisidan olinadi


@current_app.cli.command('send-budget-alerts')
def send_budget_alerts():
    bugun = date.today()
    oy_boshi = bugun.replace(day=1)

    foydalanuvchilar = User.query.filter(User.telegram_chat_id.isnot(None)).all()

    for user in foydalanuvchilar:
        jami = db.session.query(func.sum(Expense.summa)).filter(
            Expense.user_id == user.id,
            Expense.sana >= oy_boshi,
        ).scalar() or 0

        if user.oylik_byudjet and jami > user.oylik_byudjet:
            xabar_yuborish(user.telegram_chat_id, jami, user.oylik_byudjet)


def xabar_yuborish(chat_id, jami, byudjet):
    matn = (
        f"⚠️ Diqqat! Bu oy siz {jami} so'm sarfladingiz, "
        f"byudjetingiz esa {byudjet} so'm edi."
    )
    requests.post(
        f"https://api.telegram.org/bot{BOT_TOKEN}/sendMessage",
        json={"chat_id": chat_id, "text": matn},
    )

# ─────────────────────────────────────────────────────────────────────
# 2) crontab (izohda - server sozlamasi, Python emas)
# ─────────────────────────────────────────────────────────────────────

# 0 20 * * * cd /path/to/flask_backend && flask send-budget-alerts

# ─────────────────────────────────────────────────────────────────────
# 3) Ataylab xato - agregatda user filtrisiz (izohda)
# ─────────────────────────────────────────────────────────────────────

# def send_budget_alerts_xato():
#     foydalanuvchilar = User.query.filter(User.telegram_chat_id.isnot(None)).all()
#     jami = db.session.query(func.sum(Expense.summa)).scalar() or 0   # filter YO'Q!
#     for user in foydalanuvchilar:
#         if user.oylik_byudjet and jami > user.oylik_byudjet:   # 'jami' HAMMA uchun bir xil!
#             xabar_yuborish(user.telegram_chat_id, jami, user.oylik_byudjet)
