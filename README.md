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
