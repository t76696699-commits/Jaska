# ════════════════════════════════════════════════════════════════════
# 6-BOSQICH: HashMap cache + Mocking
# ════════════════════════════════════════════════════════════════════

# ─────────────────────────────────────────────────────────────────────
# 1) app/cache.py - HashMap orqali O(1) rank cache
# ─────────────────────────────────────────────────────────────────────

class RankCache:
    def __init__(self):
        self._cache = {}

    def get(self, username):
        return self._cache.get(username)

    def set_all(self, ranked_list):
        self._cache = {
            entry.username: i + 1
            for i, entry in enumerate(ranked_list)
        }


# ─────────────────────────────────────────────────────────────────────
# 2) app/notifications.py - xatoni TUTADIGAN, xavfsiz versiya
# ─────────────────────────────────────────────────────────────────────

import requests


def notify_top_10_safe(username):
    try:
        response = requests.post(
            'https://notify.example.com/send',
            json={'username': username, 'message': "Siz TOP 10'ga kirdingiz!"},
            timeout=5,
        )
        response.raise_for_status()
        return True
    except requests.exceptions.RequestException:
        return False   # xato tutildi - dastur ishlashda davom etadi


# ─────────────────────────────────────────────────────────────────────
# 3) tests/test_notifications.py - HAM muvaffaqiyat, HAM xato holati
# ─────────────────────────────────────────────────────────────────────

from unittest.mock import patch


@patch('app.notifications.requests.post')
def test_notify_top_10_success(mock_post):
    mock_post.return_value.status_code = 200
    assert notify_top_10_safe('ali') is True


@patch('app.notifications.requests.post')
def test_notify_top_10_service_down(mock_post):
    mock_post.side_effect = requests.exceptions.Timeout()
    assert notify_top_10_safe('ali') is False


# ─────────────────────────────────────────────────────────────────────
# 4) Ataylab xato - faqat muvaffaqiyat mock qilingan, try/except yo'q (izohda)
# ─────────────────────────────────────────────────────────────────────

# def notify_top_10(username):
#     response = requests.post(..., timeout=5)
#     response.raise_for_status()   # try/exceptSIZ!
#     return True
#
# @patch('app.notifications.requests.post')
# def test_notify_top_10(mock_post):
#     mock_post.return_value.status_code = 200   # FAQAT muvaffaqiyat!
#     assert notify_top_10('ali') is True
# Xizmat XATOSI HECH QACHON sinalmagan - production'da xizmat
# ishlamasa, POST /scores butunlay 500 bilan yiqiladi.
