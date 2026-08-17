# ════════════════════════════════════════════════════════════════════
# 2-BOSQICH: Flask backend API - Category va Expense uchun CRUD
# ════════════════════════════════════════════════════════════════════

# ─────────────────────────────────────────────────────────────────────
# 1) app/models.py
# ─────────────────────────────────────────────────────────────────────

from app import db


class Category(db.Model):
    id = db.Column(db.Integer, primary_key=True)
    nomi = db.Column(db.String(100), nullable=False)
    user_id = db.Column(db.Integer, db.ForeignKey('user.id'), nullable=False)


class Expense(db.Model):
    id = db.Column(db.Integer, primary_key=True)
    summa = db.Column(db.Numeric(10, 2), nullable=False)
    tavsif = db.Column(db.String(200))
    sana = db.Column(db.Date, nullable=False)
    category_id = db.Column(db.Integer, db.ForeignKey('category.id'), nullable=False)
    user_id = db.Column(db.Integer, db.ForeignKey('user.id'), nullable=False)
    category = db.relationship('Category')

    def to_dict(self):
        return {
            "id": self.id,
            "summa": float(self.summa),
            "tavsif": self.tavsif,
            "sana": self.sana.isoformat(),
            "category_nomi": self.category.nomi,
        }

# ─────────────────────────────────────────────────────────────────────
# 2) app/routes.py - Blueprint orqali JSON API
# ─────────────────────────────────────────────────────────────────────

from flask import Blueprint, jsonify, request

api = Blueprint('api', __name__, url_prefix='/api')


@api.route('/expenses', methods=['GET'])
def expenses_royxati():
    xarajatlar = Expense.query.filter_by(user_id=1).all()
    return jsonify([x.to_dict() for x in xarajatlar])


@api.route('/expenses', methods=['POST'])
def expense_yaratish():
    ma_lumot = request.get_json()
    yangi = Expense(
        summa=ma_lumot["summa"], tavsif=ma_lumot.get("tavsif", ""),
        sana=ma_lumot["sana"], category_id=ma_lumot["category_id"], user_id=1,
    )
    db.session.add(yangi)
    db.session.commit()
    return jsonify(yangi.to_dict()), 201

# ─────────────────────────────────────────────────────────────────────
# 3) app/__init__.py (izohda - Application Factory)
# ─────────────────────────────────────────────────────────────────────

# from flask import Flask
# from flask_sqlalchemy import SQLAlchemy
#
# db = SQLAlchemy()
#
# def create_app():
#     app = Flask(__name__)
#     app.config['SQLALCHEMY_DATABASE_URI'] = '...'
#     db.init_app(app)
#     from app.routes import api
#     app.register_blueprint(api)
#     return app

# ─────────────────────────────────────────────────────────────────────
# 4) Ataylab xato - model obyektini to'g'ridan-to'g'ri jsonify() (izohda)
# ─────────────────────────────────────────────────────────────────────

# @api.route('/expenses/<int:id>')
# def expense_korish_xato(id):
#     xarajat = Expense.query.get_or_404(id)
#     return jsonify(xarajat)   # to_dict() ISHLATILMAGAN!
# ❌ TypeError: Object of type Expense is not JSON serializable
