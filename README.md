# ════════════════════════════════════════════════════════════════════
# 2-BOSQICH: Flask API + TDD asoslari
# ════════════════════════════════════════════════════════════════════

# ─────────────────────────────────────────────────────────────────────
# 1) tests/test_scores.py - RED, keyin triangulation uchun 2-test
# ─────────────────────────────────────────────────────────────────────

def test_post_score_returns_created_score(client):
    response = client.post('/scores', json={'user_id': 1, 'points': 100})
    assert response.status_code == 201
    assert response.get_json()['points'] == 100


def test_post_score_with_different_values(client):
    response = client.post('/scores', json={'user_id': 2, 'points': 250})
    assert response.status_code == 201
    assert response.get_json()['points'] == 250


# ─────────────────────────────────────────────────────────────────────
# 2) app/routes.py - GREEN: haqiqiy, umumiy kod
# ─────────────────────────────────────────────────────────────────────

from flask import request, jsonify
from app import app, db
from app.models import Score


@app.route('/scores', methods=['POST'])
def create_score():
    data = request.get_json()
    score = Score(user_id=data['user_id'], points=data['points'])
    db.session.add(score)
    db.session.commit()
    return jsonify({'id': score.id, 'points': score.points}), 201


# ─────────────────────────────────────────────────────────────────────
# 3) Ataylab xato - "firib berish" (izohda)
# ─────────────────────────────────────────────────────────────────────

# @app.route('/scores', methods=['POST'])
# def create_score():
#     return jsonify({'id': 1, 'points': 100}), 201   # qattiq yozilgan!
#     # request.get_json() umuman o'qilmaydi, DB'ga hech narsa yozilmaydi.
#
# Yagona test bilan bu "yashil" o'tadi - lekin ikkinchi, boshqa
# qiymatlar bilan test qo'shilsa, albatta muvaffaqiyatsiz bo'ladi.
