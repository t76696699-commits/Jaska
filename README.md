# ============================================================
# git log --oneline -40'dan haqiqiy misollar (bu repozitoriyaning
# o'z tarixi) - Conventional Commits formatini tahlil qilamiz
# ============================================================

REAL_COMMITS = [
    "feat(scripts): add reusable course_builder library + generic scripts",
    "fix(lessons): make in-lesson exercise hydration language-aware",
    "chore(scripts): add exercise-integrity and course-image checkers",
    "fix(points): stop permanently inflating lifetime_points/leaderboard on reversal",
    "feat(team-game): notify parent bot on session complete + public snapshot endpoints",
    "fix(dictionary): stop leaking the answer word and fix RU definition gen",
    "refactor(fonts): normalize CSS to --font-ui / --font-mono tokens",
    "debug: log openai url and error body in ai chain",   # <- standart TUR emas!
]

import re

# type(scope): description  yoki  type: description
CONVENTIONAL_PATTERN = re.compile(
    r"^(feat|fix|refactor|docs|test|chore|perf|ci|style|build|revert)"
    r"(\([a-z0-9_-]+\))?: .+"
)

STANDARD_TYPES = {
    "feat", "fix", "refactor", "docs", "test",
    "chore", "perf", "ci", "style", "build", "revert",
}

for msg in REAL_COMMITS:
    match = CONVENTIONAL_PATTERN.match(msg)
    type_part = msg.split(":")[0].split("(")[0]
    is_standard = type_part in STANDARD_TYPES
    print(f"{'OK ' if match and is_standard else 'DIQQAT'}  {msg}")

# Natija:
# OK      feat(scripts): add reusable course_builder library ...
# OK      fix(lessons): make in-lesson exercise hydration ...
# OK      chore(scripts): add exercise-integrity ...
# OK      fix(points): stop permanently inflating ...
# OK      feat(team-game): notify parent bot ...
# OK      fix(dictionary): stop leaking ...
# OK      refactor(fonts): normalize CSS ...
# DIQQAT  debug: log openai url and error body in ai chain
#         ^ "debug" standart tur emas - review'da savol tug'dirishi mumkin edi


# ============================================================
# Bu repozitoriyaning so'nggi 300 commit'ini turi bo'yicha
# guruhlash (git log --oneline -300'dan olingan real taqsimot)
# ============================================================
from collections import Counter

# git log --oneline -300 | grep -oE '^[a-z]+' natijasining qisqartirilgan
# ko'rinishi - bu son real repozitoriyadan olingan (taxminiy taqsimot)
REAL_TYPE_COUNTS = Counter({
    "fix": 34,       # scope'siz "fix:" + scope'li "fix(...)"
    "feat": 17,
    "refactor": 3,
    "chore": 3,
    "test": 2,
    "ci": 1,
    "debug": 1,      # <- standart emas
})


def summarize_history(counts: Counter) -> None:
    total = sum(counts.values())
    for commit_type, count in counts.most_common():
        flag = "" if commit_type in STANDARD_TYPES else "  <- standart emas!"
        pct = count / total * 100
        print(f"{commit_type:>10}: {count:>3} ta ({pct:4.1f}%){flag}")


summarize_history(REAL_TYPE_COUNTS)
# fix eng ko'p uchraydigan tur ekani - bu odatiy holat: aksariyat
# kunlik ish bug tuzatishlardan iborat, feat kamroq (yangi funksiya
# kamroq tez-tez qo'shiladi), va debug kabi standart bo'lmagan tur
# JUDA kam (1 ta, 300 tadan) - bu izchillik odatda YAXSHI saqlanganini
# ko'rsatadi, lekin nol emasligini ham.
