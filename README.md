1. Django settings.py (Ma'lumotlar bazasi va CORS)
Django va bot bitta bazaga ulanishi uchun .env faylga yoki Render'dagi Environment Variables qismiga DATABASE_URL ni qo'shishingiz kerak. settings.py faylida quyidagi o'zgarishlarni qiling:

Python
import os
import dj_database_url

# 1. Baza uchun tayyor kod (Render'dagi Postgres URL'ni o'qiydi)
DATABASES = {
    'default': dj_database_url.config(
        default=os.environ.get('DATABASE_URL')
    )
}

# 2. Vercel'dagi React manzilingizni ruxsat etilganlar ro'yxatiga qo'shing
CORS_ALLOWED_ORIGINS = [
    "https://studymate-sizning-loyihangiz.vercel.app", 
]
2. Telegram Bot (Baza ulanishi va Background Worker)
Botni Render'ga yuklayotganda albatta "Background Worker" turini tanlang (Web Service qilsangiz qulaydi). Bot kodingizda bazaga ulanish qismini quyidagicha yozing (Djangodagi bir xil DATABASE_URL olinadi):

Python
import os
import psycopg2

# Djangoga ulangan bir xil baza URL manzilini chaqiramiz
DATABASE_URL = os.environ.get('DATABASE_URL')

def get_db_connection():
    try:
        conn = psycopg2.connect(DATABASE_URL)
        return conn
    except Exception as e:
        print(f"Bazaga ulanishda xatolik: {e}")
3. Tayyor README.md fayli
Loyiha yakunida GitHub repositoriyangizga quyidagi matnni nusxalab, o'z manzillaringizni qo'yib chiqing. AI aynan shu ro'yxat bo'yicha baholaydi:

Markdown
# StudyMate - Capstone Loyihasi

## 🔗 Loyiha Havolalari
- **React Frontend (Vercel):** [https://studymate-frontend.vercel.app](https://studymate-frontend.vercel.app)
- **Django Backend (Render Web Service):** [https://studymate-backend.onrender.com](https://studymate-backend.onrender.com)
- **Telegram Bot:** [@StudyMate_UzBot](https://t.me/StudyMate_UzBot)

## ✅ Yakuniy Sinov Ro'yxati (7/7)
- [x] Django backend haqiqiy hostingda Web Service sifatida ishlab turibdi
- [x] React frontend haqiqiy hostingda statik build sifatida ishlab turibdi
- [x] Telegram bot haqiqiy hostingda Background Worker sifatida ishlab turibdi (Web Service emas)
- [x] Bot va Django backend BIR XIL production PostgreSQL bazasiga ulangan
- [x] Ro'yxatdan o'tish, kirish, topshiriq qo'shish web saytda ishlaydi
- [x] `/link` va `/topshiriqlar` buyruqlari haqiqiy botda ishlaydi
- [x] README.md jonli havolalar va yakuniy sinov ro'yxati bilan yangilandi
