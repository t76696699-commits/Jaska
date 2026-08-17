# ════════════════════════════════════════════════════════════════════
# 7-BOSQICH (CAPSTONE YAKUNI): Deploy va nisbiy yo'l xatosi
# ════════════════════════════════════════════════════════════════════

# ─────────────────────────────────────────────────────────────────────
# 1) app.py - statik fayllarni TO'G'RI, mutlaq yo'l bilan berish
# ─────────────────────────────────────────────────────────────────────

import os
from flask import Flask, send_from_directory

BASE_DIR = os.path.dirname(os.path.abspath(__file__))
FRONTEND_DIR = os.path.join(BASE_DIR, "static")

app = Flask(__name__)


@app.route("/")
def index():
    return send_from_directory(FRONTEND_DIR, "index.html")


@app.route("/<path:filename>")
def static_files(filename):
    return send_from_directory(FRONTEND_DIR, filename)


# ─────────────────────────────────────────────────────────────────────
# 2) Xizmat turlari va environment (izohda - deploy tushunchasi, kod emas)
# ─────────────────────────────────────────────────────────────────────

# moneylog-web  -> "Web Service" (Flask: API + frontend, bitta jarayon)
# moneylog-bot  -> "Background Worker" (bot/bot.py: doim ishlab turadi)
#
# .env (ikkalasida HAM bir xil):
# DATABASE_URL=postgresql://user:parol@host:5432/dbnomi
# BOT_TOKEN=...

# ─────────────────────────────────────────────────────────────────────
# 3) Ataylab xato - oddiy nisbiy yo'l (izohda)
# ─────────────────────────────────────────────────────────────────────

# FRONTEND_DIR = "static"          # ❌ joriy ishchi papkaga (cwd) nisbatan!
#
# @app.route("/")
# def index():
#     return send_from_directory(FRONTEND_DIR, "index.html")
#
# Lokalda ishlaydi (cwd == app.py papkasi), production'da esa gunicorn/
# systemd boshqa working directory'dan ishga tushirilsa - 404!
