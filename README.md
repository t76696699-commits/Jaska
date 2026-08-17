# ════════════════════════════════════════════════════════════════════
# 5-BOSQICH: Test Coverage + Binary Search
# ════════════════════════════════════════════════════════════════════

# ─────────────────────────────────────────────────────────────────────
# 1) app/ranking.py - to'g'ri binary search (<=  bilan)
# ─────────────────────────────────────────────────────────────────────

def find_rank_by_points(sorted_points_desc, target_points):
    low, high = 0, len(sorted_points_desc) - 1
    while low <= high:
        mid = (low + high) // 2
        if sorted_points_desc[mid] == target_points:
            while mid > 0 and sorted_points_desc[mid - 1] == target_points:
                mid -= 1
            return mid + 1
        elif sorted_points_desc[mid] > target_points:
            low = mid + 1
        else:
            high = mid - 1
    return None


# ─────────────────────────────────────────────────────────────────────
# 2) tests/test_ranking.py - chekka holatlarni ANIQ sinovdan o'tkazish
# ─────────────────────────────────────────────────────────────────────

def test_find_rank_empty_list():
    assert find_rank_by_points([], 100) is None


def test_find_rank_single_element_found():
    assert find_rank_by_points([100], 100) == 1


def test_find_rank_single_element_not_found():
    assert find_rank_by_points([100], 50) is None


def test_find_rank_target_not_in_list():
    assert find_rank_by_points([300, 200, 100], 250) is None


def test_find_rank_middle_of_large_list():
    scores = [500, 400, 300, 200, 100]
    assert find_rank_by_points(scores, 300) == 3


# ─────────────────────────────────────────────────────────────────────
# 3) Ataylab xato - off-by-one, faqat happy path sinalgan (izohda)
# ─────────────────────────────────────────────────────────────────────

# def find_rank_by_points(sorted_points_desc, target_points):
#     low, high = 0, len(sorted_points_desc) - 1
#     while low < high:                    # <= o'rniga < !
#         ...
#     return None
#
# Faqat katta ro'yxat bilan sinalsa - 100% coverage, lekin
# find_rank_by_points([100], 100) NOTO'G'RI None qaytaradi.
