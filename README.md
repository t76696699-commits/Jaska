# ════════════════════════════════════════════════════════════════════
# 2-BOSQICH: Django backend API - Fan va Topshiriq uchun CRUD
# ════════════════════════════════════════════════════════════════════

# ─────────────────────────────────────────────────────────────────────
# 1) studymate/models.py
# ─────────────────────────────────────────────────────────────────────

from django.db import models
from django.contrib.auth.models import User


class Fan(models.Model):
    nomi = models.CharField(max_length=100)
    user = models.ForeignKey(User, on_delete=models.CASCADE, related_name='fanlar')

    def __str__(self):
        return self.nomi


class Topshiriq(models.Model):
    sarlavha = models.CharField(max_length=200)
    matn = models.TextField(blank=True)
    muddat_vaqti = models.DateTimeField()
    bajarilgan = models.BooleanField(default=False)
    fan = models.ForeignKey(Fan, on_delete=models.CASCADE, related_name='topshiriqlar')
    user = models.ForeignKey(User, on_delete=models.CASCADE, related_name='topshiriqlar')
    yaratilgan_vaqt = models.DateTimeField(auto_now_add=True)

    def __str__(self):
        return self.sarlavha

# ─────────────────────────────────────────────────────────────────────
# 2) studymate/views.py
# ─────────────────────────────────────────────────────────────────────

import json
from django.http import JsonResponse
from django.views.decorators.csrf import csrf_exempt
from django.views.decorators.http import require_http_methods


def topshiriq_to_dict(t):
    return {
        "id": t.id, "sarlavha": t.sarlavha, "matn": t.matn,
        "muddat_vaqti": t.muddat_vaqti.isoformat(),
        "bajarilgan": t.bajarilgan, "fan_nomi": t.fan.nomi,
    }


@require_http_methods(["GET", "POST"])
@csrf_exempt
def topshiriqlar_view(request):
    if request.method == "GET":
        topshiriqlar = Topshiriq.objects.filter(user=request.user).select_related('fan')
        natija = [topshiriq_to_dict(t) for t in topshiriqlar]
        return JsonResponse(natija, safe=False)

    ma_lumot = json.loads(request.body)
    yangi = Topshiriq.objects.create(
        sarlavha=ma_lumot["sarlavha"], matn=ma_lumot.get("matn", ""),
        muddat_vaqti=ma_lumot["muddat_vaqti"], fan_id=ma_lumot["fan_id"],
        user=request.user,
    )
    return JsonResponse(topshiriq_to_dict(yangi), status=201)

# ─────────────────────────────────────────────────────────────────────
# 3) studymate/urls.py (izohda)
# ─────────────────────────────────────────────────────────────────────

# from django.urls import path
# from . import views
#
# urlpatterns = [
#     path('api/topshiriqlar/', views.topshiriqlar_view, name='topshiriqlar'),
# ]

# ─────────────────────────────────────────────────────────────────────
# 4) Ataylab xato - safe=False'ni unutish (izohda)
# ─────────────────────────────────────────────────────────────────────

# def topshiriqlar_view_xato(request):
#     topshiriqlar = Topshiriq.objects.filter(user=request.user)
#     natija = [topshiriq_to_dict(t) for t in topshiriqlar]
#     return JsonResponse(natija)   # safe=False YO'Q!
# ❌ TypeError: In order to allow non-dict objects to be serialized set the
#    safe parameter to False
