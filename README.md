# ════════════════════════════════════════════════════════════════════
# 4-BOSQICH: Autentifikatsiya - token Django'da, React'da ishlatish
# ════════════════════════════════════════════════════════════════════

# ─────────────────────────────────────────────────────────────────────
# 1) studymate/models.py - Token modeli
# ─────────────────────────────────────────────────────────────────────

import secrets
from django.db import models
from django.contrib.auth.models import User


class Token(models.Model):
    key = models.CharField(max_length=40, unique=True)
    user = models.OneToOneField(User, on_delete=models.CASCADE)

    @staticmethod
    def yaratish(user):
        key = secrets.token_hex(20)
        return Token.objects.create(key=key, user=user)

# ─────────────────────────────────────────────────────────────────────
# 2) studymate/views.py - login
# ─────────────────────────────────────────────────────────────────────

import json
from django.contrib.auth import authenticate
from django.http import JsonResponse
from django.views.decorators.csrf import csrf_exempt


@csrf_exempt
def login_view(request):
    ma_lumot = json.loads(request.body)
    user = authenticate(username=ma_lumot["email"], password=ma_lumot["parol"])
    if user is None:
        return JsonResponse({"xato": "Email yoki parol noto'g'ri"}, status=401)

    token, _ = Token.objects.get_or_create(user=user, defaults={"key": secrets.token_hex(20)})
    return JsonResponse({"token": token.key, "ism": user.first_name})

# ─────────────────────────────────────────────────────────────────────
# 3) studymate/auth_utils.py - himoyalangan view dekoratori
# ─────────────────────────────────────────────────────────────────────

from functools import wraps


def token_talab_qilish(view_func):
    @wraps(view_func)
    def wrapper(request, *args, **kwargs):
        auth_header = request.headers.get("Authorization", "")
        if not auth_header.startswith("Token "):
            return JsonResponse({"xato": "Token yo'q"}, status=401)

        key = auth_header.split(" ")[1]
        try:
            token = Token.objects.get(key=key)
        except Token.DoesNotExist:
            return JsonResponse({"xato": "Token yaroqsiz"}, status=401)

        request.user = token.user
        return view_func(request, *args, **kwargs)
    return wrapper

# ─────────────────────────────────────────────────────────────────────
# 4) frontend/src/api/auth.js (izohda - JS)
# ─────────────────────────────────────────────────────────────────────

# export async function kirish(email, parol) {
#   const javob = await fetch(`${API_URL}/api/login/`, {
#     method: 'POST',
#     headers: { 'Content-Type': 'application/json' },
#     body: JSON.stringify({ email, parol }),
#   });
#   const data = await javob.json();
#   localStorage.setItem('token', data.token);
#   return data;
# }
#
# export async function topshiriqlarniOlish() {
#   const token = localStorage.getItem('token');
#   const javob = await fetch(`${API_URL}/api/topshiriqlar/`, {
#     headers: { Authorization: `Token ${token}` },
#   });
#   return await javob.json();
# }

# ─────────────────────────────────────────────────────────────────────
# 5) Ataylab xato - Token.DoesNotExist'ni ushlamaslik (izohda)
# ─────────────────────────────────────────────────────────────────────

# def token_talab_qilish_xato(view_func):
#     def wrapper(request, *args, **kwargs):
#         auth_header = request.headers.get("Authorization", "")
#         key = auth_header.split(" ")[1]
#         token = Token.objects.get(key=key)   # try/except YO'Q!
#         request.user = token.user
#         return view_func(request, *args, **kwargs)
#     return wrapper
# ❌ Noto'g'ri token -> Token.DoesNotExist -> 500 Internal Server Error
