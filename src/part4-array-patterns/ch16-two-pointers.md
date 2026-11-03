# Chapter 16: Two Pointers

## 16.0 At a glance

- **What this pattern solves:** pair, triplet and "best container" problems on
  **sorted** (or end-symmetric) data. Two indices move toward each other, and each step
  eliminates a whole row of candidates, turning O(n²) pair checks into O(n).
- **Prerequisites:** Ch 13 (index-based in-place work), Ch 14 (Two Sum with a hash map),
  Ch 10 (counting pointer moves).
- **Typical complexity:** O(n) time per scan, O(1) auxiliary space. With a sort first,
  O(n log n). k-sum costs O(n^(k−1)).
- **Tools used in this chapter:**
  - `sorted`, `bisect` **[KNOW-HOW]**
  - opposite-end pointers, duplicate skipping, k-sum reduction and the
    "discard the shorter side" argument **[HAND-WRITE]**

## 16.1 Recognition signals

- The input is **sorted**, or sorting doesn't destroy what you need
- "find **a pair / triplet / quadruplet** with sum = target", "**no duplicate**
  triplets"
- "**O(1) extra space**" on a pair problem (rules out the Ch 14 hash map)
- "**container / area / water** between two lines": the answer depends on both ends
- "**reverse**", "**palindrome**", "**squares of a sorted array**": the extreme values sit
  at the ends

**Look-alikes:**

| Looks like | Actually | Tell |
|---|---|---|
| Two pointers, opposite ends | Hash map (Ch 14) | **Unsorted** input and original indices needed (1 Two Sum) |
| Two pointers, opposite ends | Sliding window (Ch 17) | Both pointers move in the **same** direction over a contiguous range |
| Two pointers, opposite ends | Write pointer (Ch 13) | Read/write indices that both start at 0 |
| Trapping water with pointers | Monotonic stack (Ch 21) | Same problem, solved by layers. Both are valid; pointers use O(1) space |

## 16.2 Core idea

Brute force checks all n(n − 1)/2 pairs. On a sorted array, most of those checks are
predictable. Put `left` at the smallest value and `right` at the largest:

- If `a[left] + a[right] < target`, then `a[left]` paired with **anything** is too small:
  every other partner is ≤ `a[right]`. So discard `left`.
- If the sum is too big, `a[right]` paired with anything is too big. So discard `right`.

```text
sorted:  [ 2,  7, 11, 15 ]   target 9
          L            R      2 + 15 = 17 > 9  -> R can't be in any pair:  R -= 1
          L        R          2 + 11 = 13 > 9  -> R -= 1
          L    R              2 +  7 =  9      -> found
```

Each step removes one candidate for good, so there are at most n − 1 steps: O(n). The
same "one side can be discarded" argument powers 3Sum (fix one value, then two pointers),
the container problem, and trapping rain water.

## 16.3 The template

```python
def pair_with_sum(a: list[int], target: int) -> tuple[int, int] | None:
    """a sorted ascending; returns 0-based indices (i, j), i < j, or None."""
    left, right = 0, len(a) - 1
    while left < right:
        total = a[left] + a[right]
        if total == target:
            return left, right
        if total < target:
            left += 1
        else:
            right -= 1
    return None


pair_with_sum([1, 3, 4, 6, 9], 10)  # -> (0, 4)
pair_with_sum([1, 3, 4, 6, 9], 2) is None  # -> True
```

- **Invariant:** if a valid pair exists, both of its indices lie in `[left, right]`.
  Every pointer move preserves this, because the discarded index can't be part of any
  valid pair with the remaining indices.
- **Complexity:** O(n) time (each iteration shrinks `right − left` by 1); O(1) auxiliary
  space.

## 16.4 Dry run

11 Container With Most Water on `[1, 8, 6, 2, 5, 4, 8, 3, 7]` (16.6.3). Area =
(right − left) × min(height). Move the **shorter** side.

| left | right | h[left] | h[right] | area | best | move |
|---|---|---|---|---|---|---|
| 0 | 8 | 1 | 7 | 8 × 1 = 8 | 8 | left (1 < 7) |
| 1 | 8 | 8 | 7 | 7 × 7 = 49 | 49 | right (7 ≤ 8) |
| 1 | 7 | 8 | 3 | 6 × 3 = 18 | 49 | right |
| 1 | 6 | 8 | 8 | 5 × 8 = 40 | 49 | right (tie) |
| 1 | 5 | 8 | 4 | 4 × 4 = 16 | 49 | right |
| 1 | 4 | 8 | 5 | 3 × 5 = 15 | 49 | right |
| 1 | 3 | 8 | 2 | 2 × 2 = 4 | 49 | right |
| 1 | 2 | 8 | 6 | 1 × 6 = 6 | 49 | right |

The pointers meet after 8 steps. The answer, 49, was found at step 2.

## 16.5 Variations

| Variant | What changes | Example |
|---|---|---|
| Symmetric operation | Swap or compare `a[left]`, `a[right]`, move both | 344 Reverse String (Ch 1) |
| Sorted pair sum | Move by comparing the sum with the target | 167 Two Sum II |
| Fill output from the back | The largest result sits at one of the two ends | 977 Squares of a Sorted Array |
| k-sum | Sort; fix k − 2 values with loops; two pointers for the last two; skip duplicates at every level | 15 3Sum, 18 4Sum |
| Maximize an area | Move the **shorter** side | 11 Container With Most Water |
| Running maxima from both ends | Move the side with the smaller running max | 42 Trapping Rain Water |

## 16.6 Worked problems

### 16.6.1 167 Two Sum II - Input Array Is Sorted (Medium): <https://leetcode.com/problems/two-sum-ii-input-array-is-sorted/>

**Problem.** A sorted array and a target; exactly one pair sums to the target. Return its
two indices **1-based**, smaller first, using only constant extra space.

**Signatures.** LeetCode: `def twoSum(self, numbers: List[int], target: int) -> List[int]:`.
Book: `list[int]`.

**Constraints → target.** n ≤ 3 × 10⁴. Constant space rules out Ch 14's hash map. O(n)
with two pointers.

**Approach 1: binary search for each complement.**

<!-- file: code/ch16_two_pointers/two_sum_sorted.py -->

```python
class SolutionBinarySearch:
    def twoSum(self, numbers: list[int], target: int) -> list[int]:
        # O(n log n) time: one binary search per element. O(1) auxiliary space.
        for i, x in enumerate(numbers):
            j = bisect_left(numbers, target - x, i + 1)
            if j < len(numbers) and numbers[j] == target - x:
                return [i + 1, j + 1]
        return []
```

It meets the space limit, but it doesn't use the sortedness of **both** ends.

**Approach 2: two pointers (optimal).**

<!-- file: code/ch16_two_pointers/two_sum_sorted.py -->

```python
class Solution:
    def twoSum(self, numbers: list[int], target: int) -> list[int]:
        # O(n) time: each step moves one pointer inward. O(1) auxiliary space.
        left, right = 0, len(numbers) - 1
        while left < right:
            total = numbers[left] + numbers[right]
            if total == target:
                return [left + 1, right + 1]       # 1-based
            if total < target:
                left += 1                          # need a bigger sum
            else:
                right -= 1                         # need a smaller sum
        return []
```

Why it works: the invariant from 16.3. (The O(n²) brute force is the one from Ch 14.)

**Edge cases tested:**
- the minimum length of 2, equal values
- ±1000
- n = 3 × 10⁴
- 500 random cases checked for a valid 1-based pair

**Common bugs:**
- Returning 0-based indices.
- `while left <= right`, which can pair an element with itself.
- Using a hash map, which works but violates the space limit.

**Built-in shortcut:** `bisect` **[KNOW-HOW]** (approach 1): O(n log n).

### 16.6.2 15 3Sum (Medium): <https://leetcode.com/problems/3sum/>

**Problem.** Return every distinct triplet of values (from three different positions)
that sums to 0. No triplet may appear twice; any order.

**Signatures.** LeetCode: `def threeSum(self, nums: List[int]) -> List[List[int]]:`. Book:
`list[list[int]]`.

**Constraints → target.** n ≤ 3000. All triples: n³/6 ≈ 4.5 × 10⁹: hopeless. O(n²)
≈ 4.5 × 10⁶ steps: fine.

**Approach 1: all triples.**

<!-- file: code/ch16_two_pointers/three_sum.py -->

```python
class SolutionBruteForce:
    def threeSum(self, nums: list[int]) -> list[list[int]]:
        # O(n^3) time: every triple. O(t) space for t distinct triplets.
        found = {tuple(sorted(c)) for c in combinations(nums, 3) if sum(c) == 0}
        return [list(t) for t in found]
```

**Approach 2: fix one value, then Two Sum with a set.**

<!-- file: code/ch16_two_pointers/three_sum.py -->

```python
class SolutionHashSet:
    def threeSum(self, nums: list[int]) -> list[list[int]]:
        # O(n^2) average time: fix i, then 1 Two Sum (Ch 14) on the rest.
        # O(n) auxiliary for the seen-set, plus a set of results for de-duplication.
        nums = sorted(nums)
        found: set[tuple[int, int, int]] = set()
        for i in range(len(nums) - 2):
            if i > 0 and nums[i] == nums[i - 1]:
                continue
            seen: set[int] = set()
            for j in range(i + 1, len(nums)):
                need = -nums[i] - nums[j]
                if need in seen:
                    found.add((nums[i], need, nums[j]))
                seen.add(nums[j])
        return [list(t) for t in found]
```

**Approach 3: sort, fix one value, two pointers (optimal).**

<!-- file: code/ch16_two_pointers/three_sum.py -->

```python
class Solution:
    def threeSum(self, nums: list[int]) -> list[list[int]]:
        # O(n^2) time: sort O(n log n), then for each i a two-pointer scan O(n).
        # O(n) auxiliary space: the sorted copy (Timsort may also use O(n)).
        nums = sorted(nums)
        n = len(nums)
        result: list[list[int]] = []
        for i in range(n - 2):
            if nums[i] > 0:
                break                                # smallest is positive: done
            if i > 0 and nums[i] == nums[i - 1]:
                continue                             # same first value: same triplets
            left, right = i + 1, n - 1
            while left < right:
                total = nums[i] + nums[left] + nums[right]
                if total < 0:
                    left += 1
                elif total > 0:
                    right -= 1
                else:
                    result.append([nums[i], nums[left], nums[right]])
                    left += 1
                    right -= 1
                    while left < right and nums[left] == nums[left - 1]:
                        left += 1                    # skip duplicate second values
        return result
```

Why the duplicate skipping is enough: in sorted order, equal first values generate
exactly the same triplets, so the second copy is skipped. Within one `i`, after a match,
advancing `left` past copies of the same value prevents the same (second, third) pair.
`right` doesn't need its own skip: once the second value has changed, the third value
must also change for the sum to stay zero. Approach 2 dedups with a set of tuples
instead, which is simpler to get right but measured slower (16.7).

**Edge cases tested:**
- `[0, 0, 0]` and `[0, 0, 0, 0]` (one triplet, not four)
- no solution
- a repeated middle value
- n = 3000
- 400 random cases checked against brute force, including a check for no duplicates

**Common bugs:**
- Skipping duplicates with `nums[i] == nums[i + 1]`, which drops valid triplets like
  `[-1, -1, 2]`.
- Forgetting to move both pointers after a match, which loops forever.
- Returning index triplets instead of values.

**Built-in shortcut:** a `set` of tuples for de-duplication (approach 2). It's acceptable;
state that it costs memory and time.

### 16.6.3 11 Container With Most Water (Medium): <https://leetcode.com/problems/container-with-most-water/>

**Problem.** Vertical lines with the given heights stand at positions 0..n−1. Choose two
lines that, with the x-axis, hold the most water: width × the shorter height.

**Signatures.** LeetCode: `def maxArea(self, height: List[int]) -> int:`. Book:
`list[int]`.

**Constraints → target.** n ≤ 10⁵: O(n²) is out; target O(n).

**Approach 1: all pairs.**

<!-- file: code/ch16_two_pointers/container_most_water.py -->

```python
class SolutionBruteForce:
    def maxArea(self, height: list[int]) -> int:
        # O(n^2) time: every pair of lines. O(1) auxiliary space.
        best = 0
        for i in range(len(height)):
            for j in range(i + 1, len(height)):
                best = max(best, (j - i) * min(height[i], height[j]))
        return best
```

**Approach 2: two pointers, move the shorter line (optimal).**

<!-- file: code/ch16_two_pointers/container_most_water.py -->

```python
class Solution:
    def maxArea(self, height: list[int]) -> int:
        # O(n) time: each step discards one line for good. O(1) auxiliary space.
        # Discarding the shorter line is safe: any container that keeps it and
        # uses a line further inside is narrower and no taller than its height.
        left, right = 0, len(height) - 1
        best = 0
        while left < right:
            width = right - left
            if height[left] < height[right]:
                best = max(best, width * height[left])
                left += 1
            else:
                best = max(best, width * height[right])
                right -= 1
        return best
```

Why discarding the shorter line is safe: say `height[left] < height[right]`. Every
container that uses `left` with some other line j < right is narrower than the current
one, and its height is at most `height[left]`. So none of them can beat the area just
recorded, and `left` can never help again. Moving the taller line instead could only
make things worse. Traced in 16.4.

**Edge cases tested:**
- the minimum length of 2, all zeros
- equal tall ends
- n = 10⁵ of equal height
- 500 random cases checked against brute force

**Common bugs:**
- Moving the taller side.
- Computing width as `right - left + 1`: the width is the distance between the lines.
- Trying to sort, which destroys positions.

**Built-in shortcut:** none. The argument is the whole problem (**[HAND-WRITE]**).

### 16.6.4 42 Trapping Rain Water (Hard): <https://leetcode.com/problems/trapping-rain-water/>

**Problem.** Bars of width 1 have the given heights. Compute how much rainwater is
trapped between them.

**Signatures.** LeetCode: `def trap(self, height: List[int]) -> int:`. Book: `list[int]`.

**Constraints → target.** n ≤ 2 × 10⁴. O(n²) is 4 × 10⁸ steps: too slow in Python. Target
O(n), and ideally O(1) space.

**Key fact.** The water above bar i is
`min(max of bars 0..i, max of bars i..n−1) − height[i]`: the level is set by the lower of
the two tallest walls around it.

**Approach 1: compute both maxima for every bar.**

<!-- file: code/ch16_two_pointers/trapping_rain_water.py -->

```python
class SolutionBruteForce:
    def trap(self, height: list[int]) -> int:
        # O(n^2) time: two scans per bar. O(1) auxiliary space.
        total = 0
        for i in range(len(height)):
            left_max = max(height[: i + 1])
            right_max = max(height[i:])
            total += min(left_max, right_max) - height[i]
        return total
```

**Approach 2: prefix and suffix maxima (Ch 15 with `max` instead of `+`).**

<!-- file: code/ch16_two_pointers/trapping_rain_water.py -->

```python
class SolutionPrefixMax:
    def trap(self, height: list[int]) -> int:
        # O(n) time: three passes. O(n) auxiliary space: two max arrays (Ch 15's
        # prefix idea with max instead of +).
        n = len(height)
        left_max = [0] * n
        right_max = [0] * n
        running = 0
        for i in range(n):
            running = max(running, height[i])
            left_max[i] = running
        running = 0
        for i in range(n - 1, -1, -1):
            running = max(running, height[i])
            right_max[i] = running
        return sum(min(left_max[i], right_max[i]) - height[i] for i in range(n))
```

**Approach 3: two pointers (optimal: O(1) space).**

<!-- file: code/ch16_two_pointers/trapping_rain_water.py -->

```python
class Solution:
    def trap(self, height: list[int]) -> int:
        # O(n) time: one pass with two pointers. O(1) auxiliary space.
        # Whichever side has the lower running max is the bottleneck for its
        # pointer's bar: the other side is guaranteed to be at least that tall.
        left, right = 0, len(height) - 1
        left_max = right_max = 0
        total = 0
        while left < right:
            if height[left] < height[right]:
                left_max = max(left_max, height[left])
                total += left_max - height[left]
                left += 1
            else:
                right_max = max(right_max, height[right])
                total += right_max - height[right]
                right -= 1
        return total
```

Why it works: when `height[left] < height[right]`, the right side of `left` contains a
bar at least as tall as `height[right]`, which is taller than `height[left]`. So the
right-hand maximum for bar `left` is at least `height[right]`, and the water level at
`left` is decided by `left_max` alone. The bar is settled and the pointer advances. The
mirror argument covers the other case.

**Edge cases tested:**
- a single bar, all equal, strictly increasing (no basin)
- `[5, 0, 5]`
- n = 2 × 10⁴ with walls of 10⁵
- 500 random cases checked across all three approaches

**Common bugs:**
- Updating the running max **after** adding water, which can add a negative amount.
- Comparing `left_max` with `right_max` but forgetting to update them consistently. (The
  version above compares heights; comparing the maxima is also valid if done
  consistently.)
- An O(n²) slice-based max, as in approach 1.

**Built-in shortcut:** none. Ch 21 solves the same problem with a monotonic stack.

## 16.7 Python-specific notes

**Same complexity, different constants.** 3Sum at its maximum size, n = 3000
(`bench/ch16_three_sum.py`, best of 3, two runs):

| Input | Two pointers (approach 3) | Hash set (approach 2) |
|---|---|---|
| Random values in ±10⁵ | 311–617 ms | 574–1,043 ms |
| Dense range −1500..1499 | 421–734 ms | 1,955–2,078 ms |

Both are O(n²). The two-pointer version measured 2–5× faster: it does integer
comparisons and index moves, while the set version hashes, inserts and builds tuples on
every step. Both stay under a second here, but O(n²) Python at n = 3000 doesn't leave a
lot of margin (Ch 10's budget).

**Sorting is free in comparison.** Ch 11 measured `sorted` at 20–26 ms for 10⁵ values;
scaled down to 3000 values, that's under a millisecond, against hundreds for the O(n²) loop. Sorting
first is never the bottleneck in k-sum.

**Big values don't overflow.** 18 4Sum sums four values up to 10⁹, which overflows 32-bit
ints in C++ and Java. Python ints don't overflow (Ch 3).

## 16.8 Practice set

Hints are in Appendix D, under "Chapter 16 practice". Reference solutions:
`code/ch16_two_pointers/practice_solutions.py` (and `code/ch01_anatomy/reverse_string.py`
for 344).

| # | Title | Difficulty | Link | Hint ref |
|---|---|---|---|---|
| 344 | Reverse String | Easy | <https://leetcode.com/problems/reverse-string/> | D.16.1 |
| 977 | Squares of a Sorted Array | Easy | <https://leetcode.com/problems/squares-of-a-sorted-array/> | D.16.2 |
| 18 | 4Sum | Medium | <https://leetcode.com/problems/4sum/> | D.16.3 |

## 16.9 Chapter cheat sheet

| | |
|---|---|
| **Signals** | Sorted input; pair/triplet sum; O(1) extra space; area between two ends; extremes at the ends |
| **Template** | `left, right = 0, n - 1`; `while left < right`: compare, move the side that can't be in any answer |
| **Invariant** | Any valid answer lies in `[left, right]` |
| **k-sum** | Sort; fix k − 2 values; two pointers; skip duplicates at **every** level → O(n^(k−1)) |
| **Complexity** | O(n) per scan, O(1) space (+ O(n log n) to sort) |
| **Top 3 bugs** | `<=` instead of `<`; wrong duplicate-skip direction; moving the taller side (11) |
| **CPython** | 3Sum two pointers measured 2–5× faster than the set variant at n = 3000 |
