# Chapter 20: Kadane & Subarray Optimization

## 20.0 At a glance

- **What this pattern solves:** the best (largest sum, largest product) **contiguous**
  subarray, and close relatives, in one O(n) pass with O(1) memory. It works by keeping
  "the best answer that ends exactly here".
- **Prerequisites:** Ch 15 (prefix sums), Ch 17 (why a window doesn't work with
  negatives), Ch 10 (recurrences).
- **Typical complexity:** O(n) time, O(1) auxiliary space.
- **Tools used in this chapter:**
  - `max`, `min` **[KNOW-HOW]**
  - Kadane's recurrence, max/min tracking and the circular complement trick
    **[HAND-WRITE]**

This chapter is also the book's gentle introduction to **dynamic programming thinking**:
define a quantity for each index, write how it depends on the previous one, and keep
only what you need.

## 20.1 Recognition signals

- "**maximum (or minimum) sum / product of a contiguous subarray**"
- **Negative values present**: if every value were positive, the whole array would win
- "**circular** array" plus "maximum subarray"
- "**buy and sell**" with profit as a sum of differences (122)

**Look-alikes:**

| Looks like | Actually | Tell |
|---|---|---|
| Max subarray sum | Sliding window (Ch 17) | A length or "at most k" constraint, with non-negative values |
| Subarray sum = k | Prefix sum + hash map (Ch 15) | "Equals k" (counting or existence), not "maximum" |
| Max subarray with a length limit | Prefix sums + monotonic deque (Ch 21) | "Length at most k" with negatives |

## 20.2 Core idea

Brute force tries all O(n²) subarrays. Instead, ask a smaller question at each index i:

> **E(i)** = the largest sum of a subarray that **ends exactly at index i**.

Any subarray ending at i either is just `[nums[i]]`, or extends a subarray ending at
i − 1. The best extension uses the best subarray ending at i − 1. So:

```text
E(0) = nums[0]
E(i) = max(nums[i], E(i - 1) + nums[i])        <- the recurrence
answer = max(E(0), E(1), ..., E(n - 1))
```

In words: **if the best sum ending just before i is negative, drop it and start fresh
at i.** Each E(i) needs only E(i − 1), so one variable is enough. That's Kadane's
algorithm: a DP with O(1) state.

```text
nums: -2   1  -3   4  -1   2   1  -5   4
E:    -2   1  -2   4   3   5   6   1   5
best: -2   1   1   4   4   5   6   6   6      -> 6  (subarray [4, -1, 2, 1])
```

## 20.3 The template

```python
def kadane(nums: list[int]) -> int:
    ending_here = best = nums[0]
    for i in range(1, len(nums)):
        ending_here = max(nums[i], ending_here + nums[i])
        best = max(best, ending_here)
    return best


kadane([-2, 1, -3, 4, -1, 2, 1, -5, 4])  # -> 6
kadane([-3, -1, -2])  # -> -1
```

- **Invariant:** after processing index i, `ending_here` = E(i) and
  `best` = max(E(0..i)).
- **Complexity:** O(n) time (one pass); O(1) auxiliary space (two variables).
- Initializing with `nums[0]`, not 0, is what makes all-negative arrays work: the answer
  must be a **non-empty** subarray.

## 20.4 Dry run

152 Maximum Product Subarray (20.6.2) on `[2, 3, -2, 4, -1]`. Track the largest (`hi`)
and smallest (`lo`) product ending here:

| i | x | candidates (x, hi·x, lo·x) | hi | lo | best |
|---|---|---|---|---|---|
| 0 | 2 | (start) | 2 | 2 | 2 |
| 1 | 3 | (3, 6, 6) | 6 | 3 | 6 |
| 2 | −2 | (−2, −12, −6) | −2 | −12 | 6 |
| 3 | 4 | (4, −8, −48) | 4 | −48 | 6 |
| 4 | −1 | (−1, −4, 48) | 48 | −4 | 48 |

The −48 kept at i = 3 (the worst product) became the best at i = 4, once multiplied by
−1. A sum-only Kadane would have thrown it away. Answer: 48 (the whole array).

## 20.5 Variations

| Variant | What changes | Example |
|---|---|---|
| Max sum | Plain Kadane | 53 Maximum Subarray |
| Max sum via prefix minimum | `best = max(prefix[j] - min(prefix[:j]))` (Ch 15 view) | 53 (approach 2) |
| Divide and conquer | Best of left half, right half, and the best sum crossing the middle | 53's follow-up |
| Max product | Track max **and** min ending here; a negative swaps their roles | 152 Maximum Product Subarray |
| Circular | max(normal Kadane, total − minimum subarray), except when all values are negative | 918 Maximum Sum Circular Subarray |
| Sum of positive differences | E(i) collapses: take every upward step | 122 Best Time to Buy and Sell Stock II |

## 20.6 Worked problems

### 20.6.1 53 Maximum Subarray (Medium): <https://leetcode.com/problems/maximum-subarray/>

**Problem.** Return the largest sum of any non-empty contiguous subarray. The follow-up
asks for a divide-and-conquer solution as well.

**Signatures.** LeetCode: `def maxSubArray(self, nums: List[int]) -> int:`. Book:
`list[int]`.

**Constraints → target.** n ≤ 10⁵, values ±10⁴. O(n²) is 5 × 10⁹ steps: hopeless. O(n).

**Approach 1: every start, running sum.**

<!-- file: code/ch20_kadane/max_subarray.py -->

```python
class SolutionBruteForce:
    def maxSubArray(self, nums: list[int]) -> int:
        # O(n^2) time: running sum from each start. O(1) auxiliary space.
        best = nums[0]
        for i in range(len(nums)):
            total = 0
            for j in range(i, len(nums)):
                total += nums[j]
                best = max(best, total)
        return best
```

**Approach 2: prefix sums minus the smallest earlier prefix.**

<!-- file: code/ch20_kadane/max_subarray.py -->

```python
class SolutionPrefixMin:
    def maxSubArray(self, nums: list[int]) -> int:
        # O(n) time: sum(nums[i:j]) = prefix[j] - prefix[i] (Ch 15), so for each j
        # subtract the smallest earlier prefix. O(1) auxiliary space.
        best = nums[0]
        prefix = 0
        min_prefix = 0                       # the empty prefix
        for x in nums:
            prefix += x
            best = max(best, prefix - min_prefix)
            min_prefix = min(min_prefix, prefix)
        return best
```

**Approach 3: divide and conquer (the follow-up).**

<!-- file: code/ch20_kadane/max_subarray.py -->

```python
class SolutionDivideConquer:
    def maxSubArray(self, nums: list[int]) -> int:
        # The follow-up. O(n log n) time: T(n) = 2T(n/2) + O(n) for the crossing
        # sum. O(log n) stack depth (indices, no slicing).
        def solve(lo: int, hi: int) -> int:          # inclusive range
            if lo == hi:
                return nums[lo]
            mid = (lo + hi) // 2
            best_left_end = running = nums[mid]       # best sum ending at mid
            for i in range(mid - 1, lo - 1, -1):
                running += nums[i]
                best_left_end = max(best_left_end, running)
            best_right_start = running = nums[mid + 1]  # best sum starting at mid+1
            for i in range(mid + 2, hi + 1):
                running += nums[i]
                best_right_start = max(best_right_start, running)
            crossing = best_left_end + best_right_start
            return max(solve(lo, mid), solve(mid + 1, hi), crossing)

        return solve(0, len(nums) - 1)
```

The best subarray lies entirely in the left half, entirely in the right half, or
crosses the middle. The crossing case is the best sum ending at `mid` plus the best sum
starting at `mid + 1`, found in O(n).

**Approach 4: Kadane (optimal).**

<!-- file: code/ch20_kadane/max_subarray.py -->

```python
class Solution:
    def maxSubArray(self, nums: list[int]) -> int:
        # Kadane. O(n) time, O(1) auxiliary space.
        # ending_here = best sum of a subarray that ENDS at the current index:
        #   either extend the previous best ending (ending_here + x) or start over (x).
        ending_here = best = nums[0]
        for i in range(1, len(nums)):
            ending_here = max(nums[i], ending_here + nums[i])
            best = max(best, ending_here)
        return best
```

Approaches 2 and 4 are the same idea seen from two angles: E(i) = prefix[i + 1] − (the
minimum earlier prefix).

**Edge cases tested:**
- all negative (the answer is the largest single value)
- a single element, all zeros
- n = 10⁵
- 1,000 random cases across all four approaches

**Common bugs:**
- `best = 0` initialization, which returns 0 for all-negative input (the empty subarray
  isn't allowed).
- Resetting with `if ending_here < 0: ending_here = 0` but updating `best` before adding
  the current value. It's easy to get the order wrong; the `max(x, e + x)` form avoids
  that.
- Slicing in the divide-and-conquer recursion (Ch 10: it turns O(n log n) into more
  memory).

**Built-in shortcut:** none (**[HAND-WRITE]**).

### 20.6.2 152 Maximum Product Subarray (Medium): <https://leetcode.com/problems/maximum-product-subarray/>

**Problem.** Return the largest product of any non-empty contiguous subarray. Values are
in [−10, 10], and every subarray product fits in 32 bits.

**Signatures.** LeetCode: `def maxProduct(self, nums: List[int]) -> int:`. Book:
`list[int]`.

**Constraints → target.** n ≤ 2 × 10⁴. O(n²) is 2 × 10⁸ steps: too slow in Python. O(n).

**Why plain Kadane fails.** With products, the best product ending at i doesn't
necessarily extend the best product ending at i − 1. A **very negative** product times a
negative value becomes the best. So keep both extremes.

**Approach 1: every start, running product.**

<!-- file: code/ch20_kadane/max_product.py -->

```python
class SolutionBruteForce:
    def maxProduct(self, nums: list[int]) -> int:
        # O(n^2) time: running product from each start. O(1) auxiliary space.
        best = nums[0]
        for i in range(len(nums)):
            product = 1
            for j in range(i, len(nums)):
                product *= nums[j]
                best = max(best, product)
        return best
```

**Approach 2: max and min ending here (optimal).**

<!-- file: code/ch20_kadane/max_product.py -->

```python
class Solution:
    def maxProduct(self, nums: list[int]) -> int:
        # O(n) time, O(1) auxiliary. Track BOTH the largest and the smallest
        # product ending here: a negative x turns the smallest (most negative)
        # into the largest. A zero resets both through the "start over at x" option.
        hi = lo = best = nums[0]
        for i in range(1, len(nums)):
            x = nums[i]
            candidates = (x, hi * x, lo * x)
            hi, lo = max(candidates), min(candidates)   # computed from the OLD hi, lo
            best = max(best, hi)
        return best
```

The recurrence: the extreme products ending at i come from x alone, or from x times one
of the two extremes ending at i − 1. Traced in 20.4.

**Edge cases tested:**
- zeros splitting the array
- two negatives, one negative
- all −1s
- n = 2 × 10⁴
- 1,000 random cases checked against brute force

**Common bugs:**
- Updating `hi` first and then computing `lo` from the **new** `hi` (the tuple assignment
  above avoids this; Ch 3).
- Treating zeros specially in complicated ways: the "start over at x" candidate already
  handles them.
- Initializing `best = 0` or `1`.

**Built-in shortcut:** none.

### 20.6.3 918 Maximum Sum Circular Subarray (Medium): <https://leetcode.com/problems/maximum-sum-circular-subarray/>

**Problem.** The array is circular: after the last element comes the first. Return the
largest sum of a non-empty subarray, where a subarray may wrap but each position is used
at most once.

**Signatures.** LeetCode: `def maxSubarraySumCircular(self, nums: List[int]) -> int:`. Book:
`list[int]`.

**Constraints → target.** n ≤ 3 × 10⁴. O(n²) is 9 × 10⁸: too slow. O(n).

**Key observation.** A best subarray either doesn't wrap (normal Kadane), or wraps. A
wrapping subarray is the whole array **minus** a non-wrapping middle piece, so the best
wrapping sum is `total − (minimum subarray sum)`.

```text
[ 5, -3, 5 ]        total 7, minimum subarray [-3] = -3
  ^^      ^^        wrapping answer = 7 - (-3) = 10 = 5 + 5
```

**Approach 1: every start and length, modulo n.**

<!-- file: code/ch20_kadane/circular_subarray.py -->

```python
class SolutionBruteForce:
    def maxSubarraySumCircular(self, nums: list[int]) -> int:
        # O(n^2) time: every start, every length 1..n, using (start + k) % n.
        # O(1) auxiliary space.
        n = len(nums)
        best = nums[0]
        for start in range(n):
            total = 0
            for k in range(n):
                total += nums[(start + k) % n]
                best = max(best, total)
        return best
```

**Approach 2: Kadane for both max and min (optimal).**

<!-- file: code/ch20_kadane/circular_subarray.py -->

```python
class Solution:
    def maxSubarraySumCircular(self, nums: list[int]) -> int:
        # O(n) time, O(1) auxiliary. A wrapping subarray = everything EXCEPT a
        # middle (non-wrapping) subarray, so its best sum is total - (minimum
        # subarray sum). Run Kadane for both the max and the min in one pass.
        total = 0
        cur_max = cur_min = 0
        best_max = best_min = nums[0]
        for x in nums:
            total += x
            cur_max = max(x, cur_max + x)
            best_max = max(best_max, cur_max)
            cur_min = min(x, cur_min + x)
            best_min = min(best_min, cur_min)
        if best_max < 0:
            # All values negative: total - best_min would be the EMPTY subarray (0),
            # which isn't allowed. The answer is the largest single value.
            return best_max
        return max(best_max, total - best_min)
```

Why the all-negative guard matters: when every value is negative, the minimum subarray
is the whole array, so `total − best_min = 0`. That corresponds to choosing nothing,
which is invalid. In that case `best_max` (the largest element) is the answer.
(`cur_max` and `cur_min` start at 0 instead of `nums[0]`. That's equivalent, because the
first `max(x, 0 + x)` is just x.)

**Edge cases tested:**
- all negative, a wrapping answer
- a single element, all equal
- n = 3 × 10⁴ of all −3 × 10⁴
- 1,000 random cases checked against brute force

**Common bugs:**
- Missing the all-negative guard: the solution returns 0.
- Doubling the array and running Kadane on it, which lets a subarray reuse positions
  (longer than n).
- Computing the min and max in two passes: correct, but unnecessary.

**Built-in shortcut:** none.

## 20.7 Python-specific notes

- **`max(a, b)` per element** is a function call each time. An `if` is a bit cheaper in
  CPython (general knowledge, not benchmarked here). At n ≤ 10⁵ either is fine.
- **Tuple assignment** `hi, lo = max(c), min(c)` evaluates the right side first (Ch 3), so
  both updates use the old values, which is exactly what the recurrence needs.
- **Products don't overflow** in Python. The 32-bit guarantee in 152 matters for C++ and
  Java; in Python, a long run of 2s just makes a big int (the test file builds one ≥ 2³¹).
- **A one-line aside with `itertools.accumulate`:** `accumulate(nums, lambda e, x:
  max(x, e + x))` computes every E(i), and `max(...)` of it is Kadane in one line. It's
  correct and O(n), but write the loop in interviews (the aside is **[KNOW-HOW]**).

```python
from itertools import accumulate

nums = [-2, 1, -3, 4, -1, 2, 1, -5, 4]
max(accumulate(nums, lambda e, x: max(x, e + x)))  # -> 6
```

## 20.8 Practice set

Hints are in Appendix D, under "Chapter 20 practice". Reference solutions:
`code/ch20_kadane/practice_solutions.py`.

| # | Title | Difficulty | Link | Hint ref |
|---|---|---|---|---|
| 122 | Best Time to Buy and Sell Stock II | Medium | <https://leetcode.com/problems/best-time-to-buy-and-sell-stock-ii/> | D.20.1 |

(This chapter's list has four problems; the other three are worked above. Re-solve them
from blank files on the day 1/3/7/21 schedule.)

## 20.9 Chapter cheat sheet

| | |
|---|---|
| **Signals** | Max/min contiguous sum or product; negatives present; circular |
| **Recurrence** | `E(i) = max(nums[i], E(i-1) + nums[i])`; answer = max E(i) |
| **Template** | `e = best = nums[0]`; for the rest: `e = max(x, e + x)`; `best = max(best, e)` |
| **Product** | Track max and min ending here; candidates `(x, hi·x, lo·x)` |
| **Circular** | `max(kadane_max, total - kadane_min)`, unless all values are negative |
| **Complexity** | O(n) time, O(1) space; divide and conquer O(n log n) |
| **Top 3 bugs** | Initializing to 0 (empty subarray); updating `lo` from the new `hi`; missing the all-negative guard in 918 |
