# ════════════════════════════════════════════════════════════════════
# 1-BOSQICH: Loyihalash va repo skeleton
# ════════════════════════════════════════════════════════════════════

# Bu dars kod yozishdan ko'ra REJALASHTIRISHGA bag'ishlangan.

db_sxemasi = {
    "users": {
        "id": "SERIAL PRIMARY KEY",
        "ism": "VARCHAR(100)",
        "email": "VARCHAR(255) UNIQUE",
        "parol_hash": "VARCHAR(255)",
        "telegram_chat_id": "BIGINT NULL",
        "link_kodi": "VARCHAR(10) NULL",
        "oylik_byudjet": "NUMERIC(10, 2) NULL",   # ❗ FLOAT emas!
    },
    "categories": {
        "id": "SERIAL PRIMARY KEY",
        "nomi": "VARCHAR(100)",
        "user_id": "INTEGER REFERENCES users(id)",
    },
    "expenses": {
        "id": "SERIAL PRIMARY KEY",
        "summa": "NUMERIC(10, 2)",                 # ❗ pul miqdori - aniq tur SHART
        "tavsif": "VARCHAR(200)",
        "sana": "DATE",
        "category_id": "INTEGER REFERENCES categories(id)",
        "user_id": "INTEGER REFERENCES users(id)",
        "yaratilgan_vaqt": "TIMESTAMP DEFAULT NOW()",
    },
}

print(db_sxemasi)

# ─────────────────────────────────────────────────────────────────────
# Repo tuzilmasi (izohda)
# ─────────────────────────────────────────────────────────────────────

# moneylog/
#   flask_backend/
#     app/
#       static/       <- vanilla JS/CSS shu yerda
#       templates/     <- index.html shu yerda
#       models.py
#       routes.py
#     run.py
#   telegram_bot/
#     bot.py
#   README.md

# ─────────────────────────────────────────────────────────────────────
# Ataylab xato - FLOAT bilan pul hisoblash (izohda)
# ─────────────────────────────────────────────────────────────────────

# print(0.1 + 0.2)          # 0.30000000000000004 - aniq emas!
# print(0.1 + 0.2 == 0.3)   # False
