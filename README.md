# ════════════════════════════════════════════════════════════════════
# 4-BOSQICH: Autentifikatsiya - werkzeug.security va token
# ════════════════════════════════════════════════════════════════════

# ─────────────────────────────────────────────────────────────────────
# 1) app/routes.py - ro'yxatdan o'tish (parolni hash qilib)
# ─────────────────────────────────────────────────────────────────────

from werkzeug.security import generate_password_hash, check_password_hash
import secrets
from flask import request, jsonify
from app import db
from app.models import User


@api.route('/register', methods=['POST'])
def register():
    ma_lumot = request.get_json()
    parol_hash = generate_password_hash(ma_lumot["parol"])

    yangi_user = User(
        ism=ma_lumot["ism"], email=ma_lumot["email"], parol_hash=parol_hash,
    )
    db.session.add(yangi_user)
    db.session.commit()
    return jsonify({"xabar": "Ro'yxatdan o'tish muvaffaqiyatli"}), 201

# ─────────────────────────────────────────────────────────────────────
# 2) app/routes.py - kirish (parolni tekshirib, token yaratish)
# ─────────────────────────────────────────────────────────────────────


@api.route('/login', methods=['POST'])
def login():
    ma_lumot = request.get_json()
    user = User.query.filter_by(email=ma_lumot["email"]).first()

    if user is None or not check_password_hash(user.parol_hash, ma_lumot["parol"]):
        return jsonify({"xato": "Email yoki parol noto'g'ri"}), 401

    user.token = secrets.token_hex(20)
    db.session.commit()
    return jsonify({"token": user.token, "ism": user.ism})

# ─────────────────────────────────────────────────────────────────────
# 3) app/auth_utils.py - himoyalangan endpoint dekoratori
# ─────────────────────────────────────────────────────────────────────

from functools import wraps


def token_talab_qilish(f):
    @wraps(f)
    def wrapper(*args, **kwargs):
        auth_header = request.headers.get('Authorization', '')
        if not auth_header.startswith('Token '):
            return jsonify({"xato": "Token yo'q"}), 401

        token = auth_header.split(' ')[1]
        user = User.query.filter_by(token=token).first()
        if user is None:
            return jsonify({"xato": "Token yaroqsiz"}), 401

        request.joriy_user = user
        return f(*args, **kwargs)
    return wrapper

# ─────────────────────────────────────────────────────────────────────
# 4) Ataylab xato - parolni hash qilishni unutish (izohda)
# ─────────────────────────────────────────────────────────────────────

# @api.route('/register', methods=['POST'])
# def register_xato():
#     ma_lumot = request.get_json()
#     yangi_user = User(
#         ism=ma_lumot["ism"], email=ma_lumot["email"],
#         parol_hash=ma_lumot["parol"],   # generate_password_hash() ISHLATILMAGAN!
#     )
#     db.session.add(yangi_user)
#     db.session.commit()
# ❌ Baza oshkor bo'lsa, barcha parollar OCHIQ MATN sifatida ko'rinadi!
