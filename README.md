# ════════════════════════════════════════════════════════════════════
# 1-BOSQICH: Loyihalash va repo skeleton
# ════════════════════════════════════════════════════════════════════

# Bu dars kod yozishdan ko'ra REJALASHTIRISHGA bag'ishlangan.
# Quyida - RankVault uchun DB sxemasi va test skeleti:

# ─────────────────────────────────────────────────────────────────────
# schema.sql (hali haqiqiy migratsiya emas - 3-darsda bo'ladi)
# ─────────────────────────────────────────────────────────────────────

# CREATE TABLE users (
#   id SERIAL PRIMARY KEY,
#   username VARCHAR(50) UNIQUE NOT NULL,
#   created_at TIMESTAMP DEFAULT NOW()
# );
#
# CREATE TABLE scores (
#   id SERIAL PRIMARY KEY,
#   user_id INTEGER REFERENCES users(id),
#   points INTEGER NOT NULL,
#   submitted_at TIMESTAMP DEFAULT NOW()
# );

# ─────────────────────────────────────────────────────────────────────
# tests/conftest.py - skelet, 3-darsda to'ldiriladi
# ─────────────────────────────────────────────────────────────────────

import pytest


@pytest.fixture
def client():
    # 3-darsda: ALOHIDA test bazasiga ulanadigan konfiguratsiya
    # shu yerga qo'shiladi (TEST_DATABASE_URL orqali).
    raise NotImplementedError("3-darsda to'ldiriladi")


# ─────────────────────────────────────────────────────────────────────
# Ataylab qiyin - production bazasiga ulanadigan fixture (izohda)
# ─────────────────────────────────────────────────────────────────────

# @pytest.fixture
# def client():
#     app.config['DATABASE_URL'] = os.environ['DATABASE_URL']  # PRODUCTION!
#     return app.test_client()
# Hozircha zararsiz ko'rinadi, lekin testlar ko'paygach flaky bo'ladi.
