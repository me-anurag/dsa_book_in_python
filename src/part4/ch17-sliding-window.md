# Chapter 17: Sliding Window

## 17.0 At a glance

- **What this pattern solves:** optimizing or counting over **contiguous** subarrays (or
  substrings) whose validity can be maintained **incrementally**. A window
  `[left, right]` grows on the right and shrinks on the left, and each element enters
  and leaves at most once.
- **Prerequisites:** Ch 10 (amortized pointer counting), Ch 14 (counting with dicts),
  Ch 15 (when prefix sums are needed instead).
- **Typical complexity:** O(n) time, O(1) or O(k) auxiliary space.
- **Tools used in this chapter:**
  - `dict`, `enumerate` **[USE-FREELY]**
  - `heapq`, `bisect`, `itertools.accumulate` **[KNOW-HOW]**
  - fixed windows, shrink-while-invalid, record-while-valid and
    exactly-k = at-most-k − at-most-(k−1) **[HAND-WRITE]**

## 17.1 Recognition signals

- "**contiguous subarray / substring**" plus "**longest / shortest / maximum / count**"
- "**of size k**" (a fixed window)
- "**at most k** zeros / distinct values / replacements"
- All values **non-negative**, with a sum condition ("sum ≥ target")
- "**exactly k** distinct": rewrite it as a difference of two at-most counts

**When the window does NOT work:**

| Situation | Why it fails | Use instead |
|---|---|---|
| Negative values with a sum condition | Growing the window can **decrease** the sum, so shrinking has no valid rule | Prefix sum + hash map (Ch 15) |
| "Sum = k exactly" with negatives | Same | Ch 15 (560) |
| Non-contiguous choices (subsequences, subsets) | No window shape | DP, sorting, hashing |
| Maximum **inside** each fixed window | A plain counter can't undo a max when an element leaves | Monotonic deque (Ch 21) |

The window works when validity is **monotone**: if a window is valid, every smaller
window inside it is too (or, for "at least" conditions, every larger window containing
it is). That's what lets one pointer chase the other without ever moving back.

## 17.2 Core idea

Brute force examines all O(n²) subarrays, often recomputing each one's sum or count from
scratch, which makes it O(n³). But the subarray `[i, j+1]` differs from `[i, j]` by one
element. A window **updates** its summary (sum, zero count, value counts) in O(1) as
elements enter and leave.

```text
shortest subarray with sum >= 7 in [2, 3, 1, 2, 4, 3]:

grow:   [2 3 1 2] 4 3     sum 8 >= 7 -> record len 4, shrink
shrink:  2[3 1 2] 4 3     sum 6 < 7  -> grow
grow:    2[3 1 2 4] 3     sum 10     -> record 4, shrink
shrink:  2 3[1 2 4] 3     sum 7      -> record 3, shrink
shrink:  2 3 1[2 4] 3     sum 6      -> grow
grow:    2 3 1[2 4 3]     sum 9      -> record 3, shrink
shrink:  2 3 1 2[4 3]     sum 7      -> record 2  <- answer
```

`right` moves n times, and `left` moves at most n times in total, so the whole run is
O(n) even though a `while` sits inside the `for` (Ch 10, 10.2.4).

## 17.3 The template

**Variable window, "longest valid":**

```python
def longest_valid(nums: list[int], k: int) -> int:
    """Longest subarray with at most k zeros (shrink while invalid)."""
    best = zeros = left = 0
    for right, x in enumerate(nums):
        zeros += x == 0                     # 1. add nums[right]
        while zeros > k:                    # 2. shrink until valid again
            zeros -= nums[left] == 0
            left += 1
        best = max(best, right - left + 1)  # 3. window is valid: record
    return best


longest_valid([1, 0, 1, 1, 0, 1], 1)  # -> 4
```

**Variable window, "shortest valid":** record **inside** the shrink loop, while the window
is still valid (see 209 in 17.6.2).

**Fixed window of size k:**

```python
def max_window_sum(nums: list[int], k: int) -> int:
    window = sum(nums[:k])
    best = window
    for right in range(k, len(nums)):
        window += nums[right] - nums[right - k]   # one in, one out
        best = max(best, window)
    return best


max_window_sum([1, 12, -5, -6, 50, 3], 4)  # -> 51
```

- **Invariant (variable window):** after the shrink loop, `nums[left:right + 1]` is the
  longest valid window ending at `right`, for "longest" problems. (For "shortest"
  problems, the shrink loop examines every valid window ending at `right`, from the
  longest down.)
- **Complexity:** O(n) time, because each index is added once and removed at most once.
  O(1) auxiliary space for a scalar summary; O(k) for a dict of counts.

## 17.4 Dry run

1004 Max Consecutive Ones III on `[1, 1, 0, 0, 1, 1, 0]`, k = 1 (at most one zero in the
window):

| right | x | zeros after add | shrink? | left | window | length | best |
|---|---|---|---|---|---|---|---|
| 0 | 1 | 0 | no | 0 | `[1]` | 1 | 1 |
| 1 | 1 | 0 | no | 0 | `[1 1]` | 2 | 2 |
| 2 | 0 | 1 | no | 0 | `[1 1 0]` | 3 | 3 |
| 3 | 0 | 2 | yes: drop 1, 1, 0 | 3 | `[0]` | 1 | 3 |
| 4 | 1 | 1 | no | 3 | `[0 1]` | 2 | 3 |
| 5 | 1 | 1 | no | 3 | `[0 1 1]` | 3 | 3 |
| 6 | 0 | 2 | yes: drop 0 | 4 | `[1 1 0]` | 3 | 3 |

Answer 3. `left` moved 4 times in total and `right` 7 times: linear.

## 17.5 Variations

| Variant | What changes | Example |
|---|---|---|
| Fixed size k | Add `nums[right]`, remove `nums[right - k]` | 643 Maximum Average Subarray I |
| Running minimum (window from the best buy day) | Track the minimum so far; no explicit `left` | 121 Best Time to Buy and Sell Stock |
| Shortest with sum ≥ target (positives) | Record inside the shrink loop | 209 Minimum Size Subarray Sum |
| Longest with ≤ k "bad" elements | Count the bad elements; shrink while count > k | 1004 Max Consecutive Ones III |
| Longest with ≤ k distinct | Dict of counts; delete keys that reach 0 | 904 Fruit Into Baskets (k = 2) |
| Count subarrays with ≤ k ... | Add `right - left + 1` per step | 992 (as a helper) |
| Exactly k | `at_most(k) - at_most(k - 1)` | 992 Subarrays with K Different Integers |
| Max/min inside each fixed window | Heap O(n log n), or monotonic deque O(n) (Ch 21) | 239 Sliding Window Maximum |

## 17.6 Worked problems

### 17.6.1 643 Maximum Average Subarray I (Easy): <https://leetcode.com/problems/maximum-average-subarray-i/>

**Problem.** Find the contiguous subarray of length exactly k with the largest average,
and return that average. Answers within 10⁻⁵ are accepted.

**Signatures.** LeetCode: `def findMaxAverage(self, nums: List[int], k: int) -> float:`.
Book: `list[int]`, `-> float`.

**Constraints → target.** n ≤ 10⁵. O(n · k) is up to 10¹⁰ when k ≈ n/2: too slow. Target
O(n).

**Approach 1: sum every window.**

<!-- file: code/ch17_sliding_window/max_average.py -->

```python
class SolutionBruteForce:
    def findMaxAverage(self, nums: list[int], k: int) -> float:
        # O(n * k) time: each window summed from scratch (slice + sum).
        # O(k) auxiliary space: the slice copy.
        best = max(sum(nums[i:i + k]) for i in range(len(nums) - k + 1))
        return best / k
```

**Approach 2: rolling sum (optimal).**

<!-- file: code/ch17_sliding_window/max_average.py -->

```python
class Solution:
    def findMaxAverage(self, nums: list[int], k: int) -> float:
        # O(n) time: the window sum is updated in O(1) per step (add the entering
        # element, subtract the leaving one). O(1) auxiliary space.
        # Compare integer sums; divide once at the end (Ch 3: fewer float roundings).
        window = sum(nums[:k])
        best = window
        for right in range(k, len(nums)):
            window += nums[right] - nums[right - k]
            best = max(best, window)
        return best / k
```

Why the improvement works: consecutive windows share k − 1 elements, so updating costs
O(1) instead of O(k). Comparing integer sums instead of averages keeps every
comparison exact.

**Edge cases tested:**
- k = n, k = 1
- a single element
- all negatives, all equal
- n = 10⁵
- 500 random cases checked against brute force

**Common bugs:**
- Initializing `best = 0`, which is wrong when every window sum is negative.
- Comparing float averages: harmless here, but a habit that bites (Ch 3).
- An off-by-one in `nums[right - k]`.

**Built-in shortcut:** a prefix array via `accumulate` **[KNOW-HOW]**, then
`max(p[i + k] - p[i])`. Also O(n), but with O(n) auxiliary space.

### 17.6.2 209 Minimum Size Subarray Sum (Medium): <https://leetcode.com/problems/minimum-size-subarray-sum/>

**Problem.** All values are positive. Return the length of the shortest contiguous
subarray whose sum is at least `target`, or 0 if none exists.

**Signatures.** LeetCode: `def minSubArrayLen(self, target: int, nums: List[int]) -> int:`.
Book: `list[int]`.

**Constraints → target.** n ≤ 10⁵ and all values ≥ 1. Target O(n). The follow-up also
asks for an O(n log n) solution.

**Approach 1: every start, extend until the sum is reached.**

<!-- file: code/ch17_sliding_window/min_size_subarray.py -->

```python
class SolutionBruteForce:
    def minSubArrayLen(self, target: int, nums: list[int]) -> int:
        # O(n^2) time: running sum from each start. O(1) auxiliary space.
        best = 0
        for i in range(len(nums)):
            total = 0
            for j in range(i, len(nums)):
                total += nums[j]
                if total >= target:
                    if best == 0 or j - i + 1 < best:
                        best = j - i + 1
                    break                      # longer subarrays from i only grow
        return best
```

**Approach 2: prefix sums + binary search (the follow-up's O(n log n)).** All values are
positive, so the prefix array is strictly increasing, and for each start you can
binary-search the first end that reaches the target:

<!-- file: code/ch17_sliding_window/min_size_subarray.py -->

```python
class SolutionBinarySearch:
    def minSubArrayLen(self, target: int, nums: list[int]) -> int:
        # The follow-up: O(n log n) time. Prefix sums are strictly increasing
        # (all values positive), so for each start i binary-search the first end j
        # with prefix[j] - prefix[i] >= target. O(n) auxiliary: the prefix array.
        prefix = list(accumulate(nums, initial=0))
        best = 0
        for i in range(len(nums)):
            j = bisect_left(prefix, prefix[i] + target)
            if j <= len(nums) and (best == 0 or j - i < best):
                best = j - i
        return best
```

**Approach 3: variable window, record while valid (optimal).**

<!-- file: code/ch17_sliding_window/min_size_subarray.py -->

```python
class Solution:
    def minSubArrayLen(self, target: int, nums: list[int]) -> int:
        # O(n) time: right moves n times, left moves at most n times in total.
        # O(1) auxiliary space.
        best = 0
        window = 0
        left = 0
        for right, x in enumerate(nums):
            window += x
            while window >= target:            # valid: record, then try shrinking
                if best == 0 or right - left + 1 < best:
                    best = right - left + 1
                window -= nums[left]
                left += 1
        return best
```

Why it works: for a fixed `right`, shrinking from the left while the sum stays ≥ target
finds the shortest valid window ending at `right`. Once the sum drops below the target,
no window starting at the current `left` can end at `right` and still be valid. Later
right ends only add positive values, so `left` never needs to move back. **This relies on
positivity**: with negatives, use Ch 15's prefix techniques. The trace is in 17.2.

**Edge cases tested:**
- no valid subarray (returns 0)
- a single element that meets the target
- the whole array needed
- n = 10⁵ with target 10⁹
- 500 random cases checked across all three approaches

**Common bugs:**
- Recording outside the shrink loop, which gives the length before shrinking.
- `best = 0` treated as a real minimum (use a "not found" check, or `math.inf`).
- Applying it with negative values.

**Built-in shortcut:** `bisect` + `accumulate` **[KNOW-HOW]** (approach 2).

### 17.6.3 1004 Max Consecutive Ones III (Medium): <https://leetcode.com/problems/max-consecutive-ones-iii/>

**Problem.** A binary array and an integer k. Return the longest run of 1s you can get
by flipping at most k zeros, which is the same as the longest window containing at most
k zeros.

**Signatures.** LeetCode: `def longestOnes(self, nums: List[int], k: int) -> int:`. Book:
`list[int]`.

**Constraints → target.** n ≤ 10⁵, so O(n).

**Approach 1: extend from every start.**

<!-- file: code/ch17_sliding_window/max_ones.py -->

```python
class SolutionBruteForce:
    def longestOnes(self, nums: list[int], k: int) -> int:
        # O(n^2) time: extend from each start until a (k+1)-th zero. O(1) space.
        best = 0
        for i in range(len(nums)):
            zeros = 0
            for j in range(i, len(nums)):
                zeros += nums[j] == 0
                if zeros > k:
                    break
                best = max(best, j - i + 1)
        return best
```

**Approach 2: shrink while invalid (optimal).**

<!-- file: code/ch17_sliding_window/max_ones.py -->

```python
class Solution:
    def longestOnes(self, nums: list[int], k: int) -> int:
        # O(n) time: each index enters and leaves the window once. O(1) space.
        # Invariant after the while loop: nums[left:right + 1] has <= k zeros.
        best = 0
        zeros = 0
        left = 0
        for right, x in enumerate(nums):
            if x == 0:
                zeros += 1
            while zeros > k:                   # invalid: shrink from the left
                if nums[left] == 0:
                    zeros -= 1
                left += 1
            best = max(best, right - left + 1)
        return best
```

Why reframing works: "flip at most k zeros" is a statement about the **window's zero
count**, a summary that updates in O(1). Thinking about which zeros to flip leads to
exponential search. Traced in 17.4.

**Edge cases tested:**
- k = 0 with all zeros
- all ones
- k larger than the number of zeros
- a single element
- n = 10⁵
- 500 random cases checked against brute force

**Common bugs:**
- Using `if` instead of `while` for shrinking. (In this specific problem, `if` still
  happens to give the right length, which is a well-known "non-shrinking window" trick,
  but it's fragile. Use `while` until you can prove the variant.)
- Counting ones instead of zeros.

**Built-in shortcut:** none (**[HAND-WRITE]**).

### 17.6.4 992 Subarrays with K Different Integers (Hard): <https://leetcode.com/problems/subarrays-with-k-different-integers/>

**Problem.** Count the contiguous subarrays containing exactly k distinct values.

**Signatures.** LeetCode:
`def subarraysWithKDistinct(self, nums: List[int], k: int) -> int:`. Book: `list[int]`.

**Constraints → target.** n ≤ 2 × 10⁴. O(n²) is 2 × 10⁸ steps: too slow in Python
(Ch 10). Target O(n).

**Why a direct window fails.** "Exactly k" isn't monotone: shrinking a window with
exactly k distinct values can drop it to k − 1, and growing it can raise it to k + 1. For
a fixed `right`, the valid starts form a **range** of lefts, not a single one. But
"at most k" **is** monotone, and

> exactly(k) = atMost(k) − atMost(k − 1).

**Approach 1: every start, extend with a set.**

<!-- file: code/ch17_sliding_window/k_distinct.py -->

```python
class SolutionBruteForce:
    def subarraysWithKDistinct(self, nums: list[int], k: int) -> int:
        # O(n^2) time: extend from each start with a running set. O(n) space.
        count = 0
        for i in range(len(nums)):
            seen: set[int] = set()
            for j in range(i, len(nums)):
                seen.add(nums[j])
                if len(seen) == k:
                    count += 1
                elif len(seen) > k:
                    break
        return count
```

**Approach 2: two at-most windows (optimal).**

<!-- file: code/ch17_sliding_window/k_distinct.py -->

```python
class Solution:
    def subarraysWithKDistinct(self, nums: list[int], k: int) -> int:
        # O(n) time: two O(n) window passes. O(k) auxiliary: the count dict.
        return self._at_most(nums, k) - self._at_most(nums, k - 1)

    def _at_most(self, nums: list[int], k: int) -> int:
        """Number of subarrays with at most k distinct values."""
        counts: dict[int, int] = {}
        left = 0
        total = 0
        for right, x in enumerate(nums):
            counts[x] = counts.get(x, 0) + 1
            while len(counts) > k:
                y = nums[left]
                counts[y] -= 1
                if counts[y] == 0:
                    del counts[y]              # keep len(counts) = distinct count
                left += 1
            total += right - left + 1          # every start in [left, right] works
        return total
```

Why `total += right - left + 1`: after shrinking, every subarray ending at `right` that
starts anywhere in `[left, right]` has at most k distinct values (it's inside a valid
window), and there are `right - left + 1` such starts. Summing over every `right`
counts each valid subarray exactly once, by its end.

**Edge cases tested:**
- a single element, all equal (k = 1 → n(n + 1)/2)
- k equal to the number of distinct values
- n = 2 × 10⁴, all distinct
- 500 random cases checked against brute force

**Common bugs:**
- Not deleting zero-count keys, so `len(counts)` overcounts the distinct values.
- Trying a single window for "exactly", which misses subarrays.
- `at_most(k - 1)` with k = 1, which is `at_most(0)`: this correctly returns 0 here,
  because the window can never be valid.

**Built-in shortcut:** `Counter` **[KNOW-HOW]** instead of the dict; same logic.

## 17.7 Python-specific notes

- **Never `sum(nums[i:i + k])` per window.** That's O(k) per window plus an O(k) copy.
  Ch 11 measured slicing in a loop at ~550× the cost of indexing at n = 10⁴.
- **Delete zero-count keys** (`del counts[y]`) so that `len(counts)` means "distinct in
  the window". `Counter` doesn't remove zero counts on decrement either.
- **`collections.deque`** is the window container when the window must hold its elements
  (Ch 8). Ch 21 uses it for 239.
- **`max(best, ...)` in a hot loop** costs a function call per step. An `if` comparison
  is a little cheaper in CPython (general knowledge, not benchmarked here). Choose
  readability unless you're at the time limit.

## 17.8 Practice set

Hints are in Appendix D, under "Chapter 17 practice". Reference solutions:
`code/ch17_sliding_window/practice_solutions.py`.

| # | Title | Difficulty | Link | Hint ref |
|---|---|---|---|---|
| 121 | Best Time to Buy and Sell Stock | Easy | <https://leetcode.com/problems/best-time-to-buy-and-sell-stock/> | D.17.1 |
| 904 | Fruit Into Baskets | Medium | <https://leetcode.com/problems/fruit-into-baskets/> | D.17.2 |
| 239 | Sliding Window Maximum | Hard | <https://leetcode.com/problems/sliding-window-maximum/> | D.17.3 |

## 17.9 Chapter cheat sheet

| | |
|---|---|
| **Signals** | Contiguous + longest/shortest/count; size k; at most k; non-negative sums |
| **Template** | `for right`: add; `while invalid`: remove `nums[left]`, `left += 1`; record |
| **Longest vs shortest** | Longest: record after shrinking. Shortest: record inside the shrink loop |
| **Exactly k** | `at_most(k) - at_most(k - 1)`; count with `right - left + 1` |
| **Complexity** | O(n) time (amortized pointer moves); O(1) or O(k) space |
| **Top 3 bugs** | Window with negative values; not deleting zero-count keys; recording at the wrong point |
| **Not a window** | Negatives with a sum condition (Ch 15); max inside the window (Ch 21) |
