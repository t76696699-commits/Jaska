# ════════════════════════════════════════════════════════════════════
# 3-BOSQICH: PostgreSQL CRUD + Fixture'lar
# ════════════════════════════════════════════════════════════════════

# ─────────────────────────────────────────────────────────────────────
# 1) tests/conftest.py - alohida test bazasi + avtomatik tozalash
# ─────────────────────────────────────────────────────────────────────

import os
import pytest
from app import create_app, db as _db
from app.models import Score, User


@pytest.fixture
def app():
    app = create_app(database_url=os.environ['TEST_DATABASE_URL'])
    with app.app_context():
        _db.create_all()
        yield app
        _db.drop_all()


@pytest.fixture
def client(app):
    return app.test_client()


@pytest.fixture(autouse=True)
def clean_tables(app):
    yield
    with app.app_context():
        _db.session.query(Score).delete()
        _db.session.query(User).delete()
        _db.session.commit()


# ─────────────────────────────────────────────────────────────────────
# 2) tests/test_scores.py - GET /scores, toza holatga tayanib
# ─────────────────────────────────────────────────────────────────────

def test_get_scores_empty_list(client):
    response = client.get('/scores')
    assert response.status_code == 200
    assert response.get_json() == []


def test_get_scores_after_post(client):
    client.post('/scores', json={'user_id': 1, 'points': 100})
    response = client.get('/scores')
    assert len(response.get_json()) == 1


# ─────────────────────────────────────────────────────────────────────
# 3) Ataylab xato - tozalashsiz conftest.py (izohda)
# ─────────────────────────────────────────────────────────────────────

# @pytest.fixture
# def app():
#     app = create_app(database_url=os.environ['TEST_DATABASE_URL'])
#     with app.app_context():
#         _db.create_all()
#         yield app
#         _db.drop_all()   # faqat BUTUN sessiya oxirida!
# # clean_tables fixture'i UMUMAN YO'Q - testlar orasida ma'lumot qoladi,
# # natija test ishga tushirish TARTIBIGA qarab o'zgaradi (flaky).
