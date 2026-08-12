# ════════════════════════════════════════════════════════════════════
# DARS 1: Pytest bilan tanishuv
# ════════════════════════════════════════════════════════════════════

# ─── kalkulyator.py ───
def qoshish(a, b):
    return a + b


def ayirish(a, b):
    return a - b


# ─── test_kalkulyator.py ───
# from kalkulyator import qoshish, ayirish

def test_qoshish():
    assert qoshish(2, 3) == 5


def test_qoshish_manfiy_sonlar():
    assert qoshish(-2, -3) == -5


def test_qoshish_nol_bilan():
    assert qoshish(5, 0) == 5


def test_ayirish():
    assert ayirish(10, 4) == 6


# ─────────────────────────────────────────────────────────────────────
# Ataylab xato — 'test_' prefiksisiz funksiya (izohda, pytet topmaydi)
# ─────────────────────────────────────────────────────────────────────

# def qoshishni_tekshir():  # ❌ 'test_' bilan boshlanmagani uchun pytest
#     assert qoshish(2, 2) == 5  # bu qatorni HECH QACHON ishga tushirmaydi!


# Terminal:
#   pip install pytest
#   pytest                              # barcha testlar
#   pytest test_kalkulyator.py          # bitta fayl
#   pytest test_kalkulyator.py::test_qoshish  # bitta test
