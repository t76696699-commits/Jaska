# ════════════════════════════════════════════════════════════════════
# 1-BOSQICH: Loyihalash va repo skeleton
# ════════════════════════════════════════════════════════════════════

# Bu dars kod yozishdan ko'ra REJALASHTIRISHGA bag'ishlangan.
# Quyida - StudyMate uchun DB sxemasining "qog'ozdagi" tasviri:

db_sxemasi = {
    "users": {
        "id": "SERIAL PRIMARY KEY",
        "ism": "VARCHAR(100)",
        "email": "VARCHAR(255) UNIQUE",
        "parol_hash": "VARCHAR(255)",
        "telegram_chat_id": "BIGINT NULL",   # bog'lanmagan bo'lsa NULL
        "link_kodi": "VARCHAR(10) NULL",     # bog'lash jarayoni uchun vaqtinchalik
        "yaratilgan_vaqt": "TIMESTAMP DEFAULT NOW()",
    },
    "fanlar": {
        "id": "SERIAL PRIMARY KEY",
        "nomi": "VARCHAR(100)",
        "user_id": "INTEGER REFERENCES users(id)",
    },
    "topshiriqlar": {
        "id": "SERIAL PRIMARY KEY",
        "sarlavha": "VARCHAR(200)",
        "matn": "TEXT",
        "muddat_vaqti": "TIMESTAMP",
        "bajarilgan": "BOOLEAN DEFAULT false",
        "fan_id": "INTEGER REFERENCES fanlar(id)",
        "user_id": "INTEGER REFERENCES users(id)",
        "yaratilgan_vaqt": "TIMESTAMP DEFAULT NOW()",
    },
}

print(db_sxemasi)

# ─────────────────────────────────────────────────────────────────────
# Repo tuzilmasi (izohda - papka/fayl tuzilmasi, kod emas)
# ─────────────────────────────────────────────────────────────────────

# studymate/
#   django_backend/
#   frontend/
#   telegram_bot/
#   README.md
#   .gitignore

# ─────────────────────────────────────────────────────────────────────
# ENG MUHIM QAROR (izohda)
# ─────────────────────────────────────────────────────────────────────

# telegram_bot/ VA django_backend/ BIR XIL DATABASE_URL'ga ulanadi -
# botning o'zining alohida bazasi BO'LMAYDI!

