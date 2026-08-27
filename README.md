# ============================================================
# Capstone: 112 + bu kurs + 117 - to'liq zanjirni ko'rsatuvchi
# skript (kontseptual - real repo'da qo'llash uchun)
# ============================================================

# --- Vazifa 1: to'liq zanjir (bu kursning o'z darslari) ---
ATOMIC_COMMITS = [
    "fix(scoring): guard multiple_choice grading against comma-containing answers",
    "test(scoring): add regression test for comma-in-answer edge case",
]
PR_DESCRIPTION_SECTIONS = ["Kontekst", "Nima o'zgardi", "Nega aynan shu yechim", "Qanday tekshirish mumkin"]
MERGE_STRATEGY = "squash"          # 8-dars
VERSION_BUMP = ("1.0.0", "patch", "1.0.1")   # 9-dars
CHANGELOG_SECTION = "Fixed"        # 10-dars
RELEASE_TAG = "v1.0.1"             # 11-dars (annotated)

# --- Vazifa 2: bisect + atomik commit madaniyati (112-kurs) ---
# $ git bisect start
# $ git bisect bad HEAD
# $ git bisect good v1.0.0
# Git avtomatik oraliq commit'larni taklif qiladi; har birida:
# $ pytest tests/test_scoring.py -k comma_edge_case
# $ git bisect good   # yoki bad
# Natijada: "abc1234 is the first bad commit" - ANIQ bitta atomik
# commit ko'rsatiladi, aralash commit bo'lganida bu ANIQLIK yo'qolardi.

# --- Vazifa 3: uch qatlamli himoya ---
LOCAL_PRE_COMMIT_HOOK = "black --check backend/ && ruff check backend/"
CI_JOB_SAME_CHECK = """
  lint:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - run: pip install black ruff
      - run: black --check backend/ && ruff check backend/
"""
BRANCH_PROTECTION_REQUIRED_CHECK = "lint"   # 117-kurs: required status check nomi

# --- Vazifa 4: reliz tegini CI trigger'iga ulash (117-kurs) ---
RELEASE_TRIGGER_YAML = """
on:
  push:
    tags:
      - 'v*.*.*'
"""

print("=== Capstone zanjiri ===")
print("1) Atomik commit'lar:", ATOMIC_COMMITS)
print("2) PR bo'limlari:", PR_DESCRIPTION_SECTIONS)
print("3) Merge strategiyasi:", MERGE_STRATEGY)
print("4) Versiya:", VERSION_BUMP)
print("5) Changelog bo'limi:", CHANGELOG_SECTION)
print("6) Reliz tegi:", RELEASE_TAG)
print("7) Required CI check (117-kurs):", BRANCH_PROTECTION_REQUIRED_CHECK)
print("8) Reliz trigger (117-kurs):", RELEASE_TRIGGER_YAML)
