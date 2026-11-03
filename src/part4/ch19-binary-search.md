# Chapter 19: Binary Search

## 19.0 At a glance

- **What this pattern solves:** finding a boundary in anything **monotone**: a sorted
  array, a rotated sorted array, or a yes/no question whose answer flips exactly once
  as a number grows ("can it be done with capacity c?"). Each step halves the search
  space.
- **Prerequisites:** Ch 8 (`bisect`), Ch 9 (lower/upper bound by hand), Ch 10
  (logarithms).
- **Typical complexity:** O(log n) comparisons. Binary search on the answer costs
  O(log R) checks for a range of size R, each check usually O(n).
- **Tools used in this chapter:**
  - `bisect_left`, `bisect_right` **[KNOW-HOW]**
  - lower bound, "first true" on a monotone predicate, rotated-array search, peak
    finding and binary search on the answer **[HAND-WRITE]**

## 19.1 Recognition signals

- "**sorted**" and "**O(log n)**" in the same statement
- "**first / last position**", "**insert position**", "how many are < x"
- "**rotated** sorted array"
- "**minimum k such that** ... is possible" / "**maximum** x such that ...", with a check
  that gets easier (or harder) as k grows → binary search on the **answer**
- A huge numeric answer range (up to 10⁹) and a cheap feasibility check
- "**any peak**", "local maximum" in O(log n)

**Look-alikes:**

| Looks like | Actually | Tell |
|---|---|---|
| Binary search | Two pointers (Ch 16) | Pair conditions on a sorted array: O(n) total beats n binary searches |
| Binary search on the answer | Greedy or DP | The feasibility check isn't monotone in the answer |
| Search in a sorted matrix | Staircase search (Ch 22, 240) | Rows and columns are sorted, but rows don't continue each other |

## 19.2 Core idea

A linear scan asks "is it here?" n times. Binary search asks a question whose answer
**rules out half** of the remaining candidates. It works whenever the candidates can be
laid out so that a predicate is `False, False, ..., False, True, ..., True`. The task is
to find the **first True**.

```text
index:      0      1      2      3      4      5
nums:       5      7      7      8      8     10      target 8
nums[i]>=8: F      F      F      T      T      T
                                 ↑ first True = lower bound = 3
```

Everything follows from that picture:

| Question | Predicate (`False…True`) | Answer |
|---|---|---|
| First position of x | `nums[i] >= x` | first True, if `nums[that] == x` |
| Last position of x | `nums[i] >= x + 1` (ints), or `nums[i] > x` | first True − 1 |
| Insert position | `nums[i] >= x` | first True |
| Min speed k to finish in h hours | `hours(k) <= h` | first True k |

**How never to write an infinite loop:** keep a half-open search range `[lo, hi)` or a
"the answer is in `[lo, hi]`" invariant, always make the range strictly smaller, and
use `mid = (lo + hi) // 2` with `hi = mid` / `lo = mid + 1`. With `lo < hi`, `mid` is
always `< hi`, so `hi = mid` shrinks the range, and `lo = mid + 1` also shrinks it. The
classic infinite loop is `lo = mid` with `mid = (lo + hi) // 2`: when `hi = lo + 1`,
`mid` equals `lo` forever.

## 19.3 The template

**First True** on any monotone predicate over `[lo, hi]`:

```python
def first_true(lo: int, hi: int, pred) -> int:
    """Smallest x in [lo, hi] with pred(x) True; hi + 1 if none.
    Requires pred to be False...False True...True on [lo, hi]."""
    hi += 1                                   # search [lo, hi) for the boundary
    while lo < hi:
        mid = (lo + hi) // 2
        if pred(mid):
            hi = mid                          # mid is True: answer <= mid
        else:
            lo = mid + 1                      # mid is False: answer > mid
    return lo


nums = [5, 7, 7, 8, 8, 10]
first_true(0, len(nums) - 1, lambda i: nums[i] >= 8)  # -> 3
first_true(0, len(nums) - 1, lambda i: nums[i] >= 11)  # -> 6
first_true(1, 100, lambda k: k * k >= 50)  # -> 8
```

- **Invariant:** the answer always lies in `[lo, hi]` (where `hi` may mean "none").
  Everything below `lo` is False, and everything from `hi` on is True.
- **Termination:** the length `hi − lo` strictly decreases every iteration, as argued in
  19.2.
- **Complexity:** O(log(hi − lo)) predicate calls; O(1) auxiliary space.

**Python note:** `(lo + hi) // 2` can't overflow, because Python ints are unbounded
(Ch 3). The Java/C++ idiom `lo + (hi - lo) / 2` exists to avoid 32-bit overflow. It's
harmless in Python, but unnecessary.

## 19.4 Dry run

875 Koko Eating Bananas (19.6.4): `piles = [3, 6, 7, 11]`, `h = 8`. The predicate
`hours(k) <= 8`, where `hours(k) = Σ ceil(p / k)`.

| lo | hi | mid | hours(mid) | ≤ 8? | update |
|---|---|---|---|---|---|
| 1 | 11 | 6 | 1 + 1 + 2 + 2 = 6 | yes | hi = 6 |
| 1 | 6 | 3 | 1 + 2 + 3 + 4 = 10 | no | lo = 4 |
| 4 | 6 | 5 | 1 + 2 + 2 + 3 = 8 | yes | hi = 5 |
| 4 | 5 | 4 | 1 + 2 + 2 + 3 = 8 | yes | hi = 4 |

`lo == hi == 4`: the minimum speed is 4. That took four checks for a range of 11.
For a range of 10⁹ it would take about 30 (Ch 10).

## 19.5 Variations

| Variant | What changes | Example |
|---|---|---|
| Exact match | Closed interval `[lo, hi]`, return on equality | 704 Binary Search |
| Lower bound | First index with `nums[i] >= x` | 35 Search Insert Position |
| First and last | Two lower bounds: x and x + 1 | 34 Find First and Last Position |
| 2D as 1D | Index k ↔ `(k // n, k % n)` | 74 Search a 2D Matrix |
| Rotated: find min | Compare `nums[mid]` with `nums[hi]` | 153 Find Minimum in Rotated Sorted Array |
| Rotated: find target | One half is always sorted; test whether target is in it | 33 Search in Rotated Sorted Array |
| Peak | Climb toward the larger neighbor | 162 Find Peak Element |
| Search the answer | First k with feasible(k) | 875 Koko Eating Bananas, 1011 Capacity To Ship Packages |
| Search a partition | Binary search how many items to take from one array | 4 Median of Two Sorted Arrays |

## 19.6 Worked problems

### 19.6.1 34 Find First and Last Position of Element in Sorted Array (Medium): <https://leetcode.com/problems/find-first-and-last-position-of-element-in-sorted-array/>

**Problem.** In a non-decreasing array, return the first and last index of `target`, or
`[-1, -1]` if it's absent. Required: O(log n).

**Signatures.** LeetCode: `def searchRange(self, nums: List[int], target: int) -> List[int]:`.
Book: `list[int]`.

**Constraints → target.** n ≤ 10⁵ (possibly 0); O(log n) is required by the statement.

**Approach 1: linear scan.**

<!-- file: code/ch19_binary_search/search_range.py -->

```python
class SolutionLinear:
    def searchRange(self, nums: list[int], target: int) -> list[int]:
        # O(n) time: scan. O(1) space. Violates the O(log n) requirement.
        first = last = -1
        for i, x in enumerate(nums):
            if x == target:
                if first == -1:
                    first = i
                last = i
        return [first, last]
```

**Approach 2: two library binary searches.**

<!-- file: code/ch19_binary_search/search_range.py -->

```python
class SolutionBisect:
    def searchRange(self, nums: list[int], target: int) -> list[int]:
        # O(log n): two library binary searches ([KNOW-HOW]).
        lo = bisect_left(nums, target)
        if lo == len(nums) or nums[lo] != target:
            return [-1, -1]
        return [lo, bisect_right(nums, target) - 1]
```

**Approach 3: one hand-written lower bound, used twice (optimal).**

<!-- file: code/ch19_binary_search/search_range.py -->

```python
class Solution:
    def searchRange(self, nums: list[int], target: int) -> list[int]:
        # O(log n) time: two hand-written bound searches. O(1) space.
        first = self._lower_bound(nums, target)
        if first == len(nums) or nums[first] != target:
            return [-1, -1]
        return [first, self._lower_bound(nums, target + 1) - 1]

    def _lower_bound(self, nums: list[int], x: int) -> int:
        """First index i with nums[i] >= x (len(nums) if none)."""
        lo, hi = 0, len(nums)                 # answer always in [lo, hi]
        while lo < hi:
            mid = (lo + hi) // 2
            if nums[mid] < x:
                lo = mid + 1                  # mid and everything left are < x
            else:
                hi = mid                      # mid could be the answer
        return lo
```

Why `target + 1` works: for integers, "the first index with a value ≥ target + 1" is
one past the last occurrence of `target`. One function, written and tested once, gives
both ends. That's less code to get wrong in an interview.

**Edge cases tested:**
- an empty array, a single element, all equal
- a target beyond the end
- ±10⁹
- 1,000 random cases checked across all three approaches

**Common bugs:**
- `hi = len(nums) - 1` with `while lo < hi`, which can never return `len(nums)` (the
  "absent, larger than everything" case).
- Using `nums[first]` without checking `first < len(nums)`: an `IndexError`.
- Expanding outward linearly from a found index, which is O(n) for all-equal arrays.

**Built-in shortcut:** approach 2, `bisect` **[KNOW-HOW]**.

### 19.6.2 33 Search in Rotated Sorted Array (Medium): <https://leetcode.com/problems/search-in-rotated-sorted-array/>

**Problem.** A sorted array of **distinct** values was rotated at an unknown pivot (for
example `[4, 5, 6, 7, 0, 1, 2]`). Return the index of `target`, or −1. Required: O(log n).

**Signatures.** LeetCode: `def search(self, nums: List[int], target: int) -> int:`. Book:
`list[int]`.

**Constraints → target.** n ≤ 5000; O(log n) is required.

**Approach 1: linear scan.**

<!-- file: code/ch19_binary_search/rotated_search.py -->

```python
class SolutionLinear:
    def search(self, nums: list[int], target: int) -> int:
        # O(n) time. Violates the requirement.
        for i, x in enumerate(nums):
            if x == target:
                return i
        return -1
```

**Approach 2: binary search on the sorted half (optimal).**

<!-- file: code/ch19_binary_search/rotated_search.py -->

```python
class Solution:
    def search(self, nums: list[int], target: int) -> int:
        # O(log n) time, O(1) space. In [lo, hi], at least one half around mid is
        # sorted; check whether target lies in that sorted half's value range.
        lo, hi = 0, len(nums) - 1
        while lo <= hi:
            mid = (lo + hi) // 2
            if nums[mid] == target:
                return mid
            if nums[lo] <= nums[mid]:                 # left half [lo, mid] sorted
                if nums[lo] <= target < nums[mid]:
                    hi = mid - 1
                else:
                    lo = mid + 1
            else:                                     # right half [mid, hi] sorted
                if nums[mid] < target <= nums[hi]:
                    lo = mid + 1
                else:
                    hi = mid - 1
        return -1
```

Why it works: the rotation point lies in at most one of the two halves, so the other
half is an ordinary sorted range. For a sorted range, "is the target inside?" is two
comparisons. If it's inside, search there; otherwise search the other half. Either way,
half the range is discarded. `nums[lo] <= nums[mid]` (with `<=`) handles `lo == mid`,
where the left half is a single element and trivially sorted.

**Edge cases tested:**
- a single element, present and absent
- an unrotated array, a rotation by 1
- 2,000 random rotations of random sorted arrays, checked against linear search

**Common bugs:**
- `nums[lo] < nums[mid]` (strict), which mishandles the two-element case.
- Closed vs open bounds mixed up (`target <= nums[mid]` when `nums[mid]` was already
  checked).
- Applying this to arrays **with duplicates**: when `nums[lo] == nums[mid]`, you can't
  tell which half is sorted, and the worst case degrades to O(n).

**Built-in shortcut:** find the rotation point (153), then `bisect` in the correct
half: also O(log n).

### 19.6.3 162 Find Peak Element (Medium): <https://leetcode.com/problems/find-peak-element/>

**Problem.** Adjacent values differ, and the positions just outside the array count as
−∞. Return the index of **any** element strictly greater than both neighbors. Required:
O(log n).

**Signatures.** LeetCode: `def findPeakElement(self, nums: List[int]) -> int:`. Book:
`list[int]`.

**Constraints → target.** n ≤ 1000. The point is the O(log n) argument; the array isn't
sorted at all.

**Approach 1: first descent.**

<!-- file: code/ch19_binary_search/peak_element.py -->

```python
class SolutionLinear:
    def findPeakElement(self, nums: list[int]) -> int:
        # O(n): the first i with nums[i] > nums[i + 1] is a peak (everything before
        # it was rising); if the array only rises, the last index is a peak.
        for i in range(len(nums) - 1):
            if nums[i] > nums[i + 1]:
                return i
        return len(nums) - 1
```

**Approach 2: climb uphill by halves (optimal).**

<!-- file: code/ch19_binary_search/peak_element.py -->

```python
class Solution:
    def findPeakElement(self, nums: list[int]) -> int:
        # O(log n) time, O(1) space. If nums[mid] < nums[mid + 1], climbing right
        # must reach a peak in (mid, hi] (the array ends in -inf); otherwise
        # [lo, mid] contains one. The search space halves while always holding a peak.
        lo, hi = 0, len(nums) - 1
        while lo < hi:
            mid = (lo + hi) // 2
            if nums[mid] < nums[mid + 1]:
                lo = mid + 1
            else:
                hi = mid
        return lo
```

Why binary search works **without sorting**: the predicate isn't "the value is big",
it's "**a peak exists in this range**". If `nums[mid] < nums[mid + 1]`, walk right from
`mid + 1`: the values either rise until the −∞ boundary forces a fall, or fall at some
point. Either way you hit a peak to the right. The mirror argument covers the left. The
invariant "[lo, hi] contains a peak" survives each halving, and a range of one element
must be that peak.

**Edge cases tested:**
- a single element, strictly rising, strictly falling
- 2,000 random arrays with no equal neighbors, checking that the returned index really
  is a peak (any peak is accepted)

**Common bugs:**
- Comparing with `nums[mid - 1]` without a bounds check.
- Expecting a specific peak in your tests: the problem accepts any, so test the
  property instead.

**Built-in shortcut:** none (**[HAND-WRITE]**).

### 19.6.4 875 Koko Eating Bananas (Medium): <https://leetcode.com/problems/koko-eating-bananas/>

**Problem.** There are piles of bananas. Each hour, Koko picks one pile and eats up to k
bananas from it (if the pile has fewer, she finishes it and waits out the hour). Return
the minimum integer speed k that finishes every pile within h hours.

**Signatures.** LeetCode: `def minEatingSpeed(self, piles: List[int], h: int) -> int:`. Book:
`list[int]`.

**Constraints → target.** n ≤ 10⁴ piles of up to 10⁹; h ≤ 10⁹. Trying every k is up to 10⁹
checks of O(n) each: hopeless. Binary search over k needs about 30 checks: 30 × 10⁴ =
3 × 10⁵ steps.

**Approach 1: try speeds in increasing order.**

<!-- file: code/ch19_binary_search/koko_bananas.py -->

```python
class SolutionLinearSpeeds:
    def minEatingSpeed(self, piles: list[int], h: int) -> int:
        # O(n * max(piles)) time: try k = 1, 2, ... in order. Up to 10^4 * 10^9:
        # hopeless at full scale. O(1) space. Test-only.
        k = 1
        while hours_needed(piles, k) > h:
            k += 1
        return k
```

**Approach 2: binary search on the answer (optimal).**

<!-- file: code/ch19_binary_search/koko_bananas.py -->

```python
def hours_needed(piles: list[int], k: int) -> int:
    # ceil(p / k) per pile, in pure integer arithmetic (Ch 8: -(-p // k)).
    return sum(-(-p // k) for p in piles)
```

<!-- file: code/ch19_binary_search/koko_bananas.py -->

```python
class Solution:
    def minEatingSpeed(self, piles: list[int], h: int) -> int:
        # Binary search on the ANSWER: feasible(k) = hours_needed(k) <= h is
        # monotone (false ... false true ... true) in k. O(n log max(piles)) time:
        # about 30 checks of O(n) each. O(1) space.
        lo, hi = 1, max(piles)                 # k = max(piles) always works (h >= n)
        while lo < hi:
            mid = (lo + hi) // 2
            if hours_needed(piles, mid) <= h:
                hi = mid                       # mid works; maybe smaller does too
            else:
                lo = mid + 1                   # mid too slow
        return lo
```

Why it works: a higher speed never needs more hours, so `hours_needed(k) <= h` flips from
False to True exactly once as k grows. The answer is the "first True" of 19.3, over
`k ∈ [1, max(piles)]`. The upper bound works because at speed `max(piles)` each pile
takes one hour, and h ≥ n. Traced in 19.4.

**Edge cases tested:**
- h = n (the speed must equal the largest pile)
- a single pile, all-equal piles with plenty of time
- 10⁴ piles of 10⁹ with h = 10⁴
- one pile of 10⁹ with h = 10⁹
- 500 random cases checked against the linear search

**Common bugs:**
- `math.ceil(p / k)` with floats: fine at these sizes, but integer ceiling division is
  exact for any size (Ch 3).
- `lo = 0`, which divides by zero.
- Searching the wrong direction (finding the maximum infeasible speed instead of the
  minimum feasible one).

**Built-in shortcut:** `bisect_left(range(1, max(piles) + 1), True, key=lambda k:
hours_needed(piles, k) <= h) + 1` (3.10+, `key=`). It's correct and O(n log max), since a
`range` supports O(1) indexing. But it's harder to read than the loop. Use it only as an
aside.

## 19.7 Python-specific notes

- **No overflow in `mid`.** `(lo + hi) // 2` is exact for any size (Ch 3). You'll see
  `lo + (hi - lo) // 2` copied from Java; it's harmless.
- **`bisect` on a `range`** works without building a list, because `range` indexing is
  O(1) (Ch 7, Ch 12). With `key=` (3.10+) it can binary-search answers. That's handy, but
  it's **[KNOW-HOW]**: write the loop in interviews.
- **Integer ceiling:** `-(-a // b)` or `(a + b - 1) // b` for positive values; no floats
  needed.
- **Recursion isn't needed.** Every search here is a `while` loop: O(1) space, and no
  recursion limit (Ch 10, 10.2.8: recursive slicing even turns O(log n) into O(n)).

## 19.8 Practice set

Hints are in Appendix D, under "Chapter 19 practice". Reference solutions:
`code/ch19_binary_search/practice_solutions.py`.

| # | Title | Difficulty | Link | Hint ref |
|---|---|---|---|---|
| 704 | Binary Search | Easy | <https://leetcode.com/problems/binary-search/> | D.19.1 |
| 35 | Search Insert Position | Easy | <https://leetcode.com/problems/search-insert-position/> | D.19.2 |
| 74 | Search a 2D Matrix | Medium | <https://leetcode.com/problems/search-a-2d-matrix/> | D.19.3 |
| 153 | Find Minimum in Rotated Sorted Array | Medium | <https://leetcode.com/problems/find-minimum-in-rotated-sorted-array/> | D.19.4 |
| 1011 | Capacity To Ship Packages Within D Days | Medium | <https://leetcode.com/problems/capacity-to-ship-packages-within-d-days/> | D.19.5 |
| 4 | Median of Two Sorted Arrays (stretch) | Hard | <https://leetcode.com/problems/median-of-two-sorted-arrays/> | D.19.6 |

## 19.9 Chapter cheat sheet

| | |
|---|---|
| **Signals** | Sorted + O(log n); first/last/insert position; rotated; "minimum k such that"; any peak |
| **Template** | `while lo < hi: mid = (lo + hi) // 2; if pred(mid): hi = mid else: lo = mid + 1` → first True |
| **Invariant** | Below `lo`: False; from `hi` on: True; the range shrinks every step |
| **Bounds** | Lower bound searches `[0, len]`; answer search uses `[smallest possible, largest possible]` |
| **Rotated** | One half is always sorted; compare with `nums[hi]` to find the minimum |
| **Complexity** | O(log n), or O(log R) checks × the check's cost |
| **Top 3 bugs** | `lo = mid` (infinite loop); `hi = len - 1` when the answer can be `len`; non-monotone predicates |
