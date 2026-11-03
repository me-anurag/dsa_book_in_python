# Chapter 15: Prefix Sums & Difference Arrays

## 15.0 At a glance

- **What this pattern solves:** questions about **sums (or products, or counts) over
  contiguous ranges**, answered in O(1) each after one O(n) pass, plus "many range
  updates, then read everything" in O(n + updates).
- **Prerequisites:** Ch 3 (`%` with negatives), Ch 14 (hash maps), Ch 10 (complexity).
- **Typical complexity:** O(n) time to build, O(1) per range query, O(n) auxiliary
  space. With a hash map: O(n) average time to count subarrays.
- **Tools used in this chapter:**
  - `list`, `dict` **[USE-FREELY]**
  - `itertools.accumulate` **[KNOW-HOW]**
  - prefix arrays, prefix + hash map, prefix remainders, prefix/suffix products and
    difference arrays **[HAND-WRITE]**

## 15.1 Recognition signals

- "sum of elements **between indices i and j**", asked **many times** (303)
- "**contiguous subarray** whose sum **equals k** / **is divisible by k** / is a multiple
  of k" (560, 974, 523), **especially when values can be negative**
- "left sum equals right sum" (724)
- "product of everything **except** this element", **without division** (238)
- "add v to every position in **[l, r)**" many times, then read the result (1094)

**Look-alikes:**

| Looks like | Actually | Tell |
|---|---|---|
| Prefix sum + hash map | Sliding window (Ch 17) | All values **non-negative** and the condition is monotone ("sum ≥ k", "at most k") → a window works. **Negatives**, or "**equals** k" with negatives → prefix + map |
| Range sum queries | Fenwick / segment tree | The array **changes** between queries (out of scope for this book) |
| Difference array | Sweep over sorted events | Coordinates huge (10⁹) → sort events instead of indexing an array |

## 15.2 Core idea

Summing a range from scratch costs O(length). If you answer q queries that way, you pay
O(n · q). The waste is re-adding the same elements over and over.

**Precompute running totals once.** With `prefix[i]` = sum of the first i elements
(`prefix[0] = 0`):

```text
nums     =    [ 3,  1,  4,  1,  5 ]
index         0   1   2   3   4
prefix   = [0,  3,  4,  8,  9, 14]       prefix has n + 1 entries
              ↑                 ↑
           prefix[0]         prefix[5]

sum(nums[i:j]) = prefix[j] - prefix[i]       e.g. sum(nums[1:4]) = 9 - 3 = 6
```

That identity drives every variant:

- **Counting subarrays with sum k:** a subarray `nums[i:j]` sums to k exactly when
  `prefix[i] = prefix[j] − k`. Scan j left to right, and count how many earlier prefixes
  equal `prefix[j] − k` with a hash map.
- **Divisibility:** `prefix[j] − prefix[i]` is divisible by k exactly when the two
  prefixes have the **same remainder** mod k.
- **Products:** "product except self" = (product of everything before) × (product of
  everything after).
- **Difference arrays (the inverse):** to add v to every position in [l, r), record
  `diff[l] += v` and `diff[r] -= v`. The prefix sum of `diff` gives the final values.

## 15.3 The template

**Prefix array and range sum:**

```python
def build_prefix(nums: list[int]) -> list[int]:
    prefix = [0]
    for x in nums:
        prefix.append(prefix[-1] + x)
    return prefix


def range_sum(prefix: list[int], i: int, j: int) -> int:
    """Sum of nums[i:j] (half-open, like slicing)."""
    return prefix[j] - prefix[i]


p = build_prefix([3, 1, 4, 1, 5])
p  # -> [0, 3, 4, 8, 9, 14]
range_sum(p, 1, 4)  # -> 6
range_sum(p, 0, 5)  # -> 14
range_sum(p, 2, 2)  # -> 0
```

- **Invariant:** `prefix[i] == sum(nums[:i])` for every 0 ≤ i ≤ n.
- **Complexity:** O(n) build time (one pass), O(1) per query (one subtraction), O(n)
  auxiliary space (n + 1 entries).

**Prefix + hash map** (count subarrays whose sum is k):

```python
def count_sum_k(nums: list[int], k: int) -> int:
    seen = {0: 1}                 # one empty prefix, sum 0
    prefix = count = 0
    for x in nums:
        prefix += x
        count += seen.get(prefix - k, 0)
        seen[prefix] = seen.get(prefix, 0) + 1
    return count


count_sum_k([1, 1, 1], 2)  # -> 2
```

- **Invariant:** before processing index j, `seen` maps each prefix sum value to the
  number of indices i ≤ j with that value.
- **Complexity:** O(n) average time, O(n) auxiliary space.

**Difference array** (range add, then read):

```python
def apply_range_adds(n: int, updates: list[tuple[int, int, int]]) -> list[int]:
    """Each update (l, r, v) adds v to positions l..r-1."""
    diff = [0] * (n + 1)
    for left, right, v in updates:
        diff[left] += v
        diff[right] -= v
    values, running = [], 0
    for i in range(n):
        running += diff[i]
        values.append(running)
    return values


apply_range_adds(5, [(1, 4, 2), (0, 2, 1)])  # -> [1, 3, 2, 2, 0]
```

- **Complexity:** O(n + u) time for u updates, instead of O(n · u). O(n) auxiliary space.

## 15.4 Dry run

`count_sum_k([3, 4, 7, 2, -3, 1, 4, 2], 7)`. Answer: 4 (the subarrays `[3, 4]`, `[7]`,
`[7, 2, -3, 1]` and `[1, 4, 2]`).

| j | x | prefix | prefix − k | `seen[prefix − k]` | count | `seen` after (new entries) |
|---|---|---|---|---|---|---|
| start | | 0 | | | 0 | {0: 1} |
| 0 | 3 | 3 | −4 | 0 | 0 | 3: 1 |
| 1 | 4 | 7 | 0 | 1 | 1 | 7: 1 |
| 2 | 7 | 14 | 7 | 1 | 2 | 14: 1 |
| 3 | 2 | 16 | 9 | 0 | 2 | 16: 1 |
| 4 | −3 | 13 | 6 | 0 | 2 | 13: 1 |
| 5 | 1 | 14 | 7 | 1 | 3 | 14: 2 |
| 6 | 4 | 18 | 11 | 0 | 3 | 18: 1 |
| 7 | 2 | 20 | 13 | 1 | 4 | 20: 1 |

Each hit is one subarray ending at j. At j = 5, `prefix − k = 7` was seen once (after
index 1), which identifies `nums[2:6] = [7, 2, -3, 1]`.

## 15.5 Variations

| Variant | What changes | Example |
|---|---|---|
| Static range sums | Build once in `__init__`, answer in O(1) | 303 Range Sum Query - Immutable |
| Balance point | `left` running sum; `right = total − left − x` | 724 Find Pivot Index |
| Products | Prefix product × suffix product; no division | 238 Product of Array Except Self |
| Count subarrays with sum = k | Prefix + map of **counts**, seeded `{0: 1}` | 560 Subarray Sum Equals K |
| Count subarrays with sum % k == 0 | Map (or k-slot list) of **remainder counts** | 974 Subarray Sums Divisible by K |
| Exists a subarray of length ≥ 2 with sum % k == 0 | Map of remainder → **first index**, seeded `{0: -1}` | 523 Continuous Subarray Sum |
| Many range adds | Difference array, then one prefix pass | 1094 Car Pooling |

## 15.6 Worked problems

### 15.6.1 238 Product of Array Except Self (Medium): <https://leetcode.com/problems/product-of-array-except-self/>

**Problem.** Return an array where position i holds the product of all elements except
`nums[i]`. Don't use division; run in O(n). The follow-up asks for O(1) extra space,
not counting the output.

**Signatures.** LeetCode:
`def productExceptSelf(self, nums: List[int]) -> List[int]:`. Book: `list[int]`.

**Constraints → target.** n ≤ 10⁵, so O(n²) is 10¹⁰: hopeless. Values are in [−30, 30],
and every answer fits in 32 bits. Target: O(n) time. Why not "total product ÷ nums[i]"?
Division is forbidden, and it also breaks on zeros.

**Approach 1: brute force.**

<!-- file: code/ch15_prefix_sums/product_except_self.py -->

```python
class SolutionBruteForce:
    def productExceptSelf(self, nums: list[int]) -> list[int]:
        # O(n^2) time: a full product for each index. O(1) auxiliary space.
        n = len(nums)
        answer = []
        for i in range(n):
            product = 1
            for j in range(n):
                if j != i:
                    product *= nums[j]
            answer.append(product)
        return answer
```

**Approach 2: prefix and suffix product arrays.**

<!-- file: code/ch15_prefix_sums/product_except_self.py -->

```python
class SolutionPrefixSuffixArrays:
    def productExceptSelf(self, nums: list[int]) -> list[int]:
        # O(n) time: three passes. O(n) auxiliary space: two helper arrays.
        n = len(nums)
        prefix = [1] * n           # prefix[i] = product of nums[:i]
        suffix = [1] * n           # suffix[i] = product of nums[i + 1:]
        for i in range(1, n):
            prefix[i] = prefix[i - 1] * nums[i - 1]
        for i in range(n - 2, -1, -1):
            suffix[i] = suffix[i + 1] * nums[i + 1]
        return [prefix[i] * suffix[i] for i in range(n)]
```

**Approach 3: output array + one running suffix (optimal, meets the follow-up).**

<!-- file: code/ch15_prefix_sums/product_except_self.py -->

```python
class Solution:
    def productExceptSelf(self, nums: list[int]) -> list[int]:
        # O(n) time: two passes. O(1) auxiliary space besides the output: the
        # output holds the prefix products, and one running variable supplies the
        # suffix products on the way back.
        n = len(nums)
        answer = [1] * n
        for i in range(1, n):
            answer[i] = answer[i - 1] * nums[i - 1]
        suffix = 1
        for i in range(n - 1, -1, -1):
            answer[i] *= suffix
            suffix *= nums[i]
        return answer
```

Why it works: after the first pass, `answer[i]` is the product of everything **left**
of i. The backward pass multiplies in the product of everything **right** of i, which
the variable `suffix` accumulates without needing an array. Zeros need no special
case.

**Edge cases tested:**
- one zero, two zeros
- length 2, all-equal negatives
- n = 10⁵
- 300 random arrays with values in [−3, 3] (many zeros), checked against brute force

**Common bugs:**
- Using division and special-casing zeros. That's forbidden, and it's easy to get the
  two-zeros case wrong.
- Off-by-one in the prefix definition: `nums[:i]` excludes i.
- Multiplying `suffix` before using it in the backward pass, which includes `nums[i]`
  itself.

**Built-in shortcut:** none. `math.prod` would need division or O(n²).

### 15.6.2 560 Subarray Sum Equals K (Medium): <https://leetcode.com/problems/subarray-sum-equals-k/>

**Problem.** Count the contiguous, non-empty subarrays whose elements sum to k. Values
can be negative.

**Signatures.** LeetCode: `def subarraySum(self, nums: List[int], k: int) -> int:`. Book:
`list[int]`.

**Constraints → target.** n ≤ 2 × 10⁴; values in [−1000, 1000]. O(n²) is 2 × 10⁸
inner-loop steps: tens of seconds of Python (Ch 10). Target: O(n). **Negatives rule out
the sliding window**: extending a window can decrease its sum, so "shrink when too big"
has no valid rule.

**Approach 1: every start, running sum.**

<!-- file: code/ch15_prefix_sums/subarray_sum_k.py -->

```python
class SolutionBruteForce:
    def subarraySum(self, nums: list[int], k: int) -> int:
        # O(n^2) time: every start i, extend j while keeping a running sum.
        # (Summing each subarray from scratch would be O(n^3).) O(1) auxiliary.
        count = 0
        for i in range(len(nums)):
            total = 0
            for j in range(i, len(nums)):
                total += nums[j]
                if total == k:
                    count += 1
        return count
```

**Approach 2: prefix sum + hash map (optimal).**

<!-- file: code/ch15_prefix_sums/subarray_sum_k.py -->

```python
class Solution:
    def subarraySum(self, nums: list[int], k: int) -> int:
        # O(n) average time: one pass with O(1) average dict operations.
        # O(n) auxiliary space: up to n + 1 distinct prefix sums.
        # sum(nums[i:j]) == prefix[j] - prefix[i] == k  <=>  prefix[i] == prefix[j] - k
        seen = {0: 1}               # the empty prefix: lets subarrays start at index 0
        prefix = 0
        count = 0
        for x in nums:
            prefix += x
            count += seen.get(prefix - k, 0)
            seen[prefix] = seen.get(prefix, 0) + 1
        return count
```

Why it works: each subarray is identified by its end j and its start i. For a fixed end,
the valid starts are exactly the earlier prefix positions whose sum is `prefix[j] − k`,
and the map holds their **count**. Traced in 15.4.

**Edge cases tested:**
- a single element with no match
- all zeros with k = 0 (6 subarrays from 3 zeros)
- negatives cancelling out
- n = 2 × 10⁴, with an answer of 10⁸ (a count, not a list)
- 500 random cases checked against brute force

**Common bugs:**
- Forgetting the `{0: 1}` seed, which misses subarrays that start at index 0.
- Storing a set instead of counts, which undercounts repeated prefix sums.
- Updating `seen[prefix]` **before** the lookup: with k = 0, that counts the empty
  subarray.
- Reaching for a sliding window despite negative values.

**Built-in shortcut:** none. This is the canonical **[HAND-WRITE]** prefix + map problem.

### 15.6.3 974 Subarray Sums Divisible by K (Medium): <https://leetcode.com/problems/subarray-sums-divisible-by-k/>

**Problem.** Count the non-empty contiguous subarrays whose sum is divisible by k
(k ≥ 2). Values can be negative.

**Signatures.** LeetCode: `def subarraysDivByK(self, nums: List[int], k: int) -> int:`.
Book: `list[int]`.

**Constraints → target.** n ≤ 3 × 10⁴: O(n²) is too slow in Python; target O(n).

**Approach 1: every start, running sum.**

<!-- file: code/ch15_prefix_sums/subarray_div_k.py -->

```python
class SolutionBruteForce:
    def subarraysDivByK(self, nums: list[int], k: int) -> int:
        # O(n^2) time: running sum from each start. O(1) auxiliary.
        count = 0
        for i in range(len(nums)):
            total = 0
            for j in range(i, len(nums)):
                total += nums[j]
                if total % k == 0:
                    count += 1
        return count
```

**Approach 2: count prefix remainders (optimal).**

<!-- file: code/ch15_prefix_sums/subarray_div_k.py -->

```python
class Solution:
    def subarraysDivByK(self, nums: list[int], k: int) -> int:
        # O(n) time: one pass. O(k) auxiliary: one counter per remainder.
        # sum(nums[i:j]) % k == 0  <=>  prefix[i] % k == prefix[j] % k.
        # Python's % already returns a value in [0, k) for k > 0, even for a
        # negative prefix (Ch 3), so no "+ k" fix is needed, unlike in C/Java.
        remainder_count = [0] * k
        remainder_count[0] = 1          # the empty prefix
        prefix = 0
        count = 0
        for x in nums:
            prefix += x
            r = prefix % k
            count += remainder_count[r]
            remainder_count[r] += 1
        return count
```

Why it works: two prefixes with equal remainders differ by a multiple of k, so the
subarray between them is divisible by k. Each new prefix pairs with every earlier prefix
that has the same remainder.

**Python note:** in C++ or Java, `-1 % 5` is −1, and solutions add k to fix negative
remainders. Python's `%` already returns 4 (Ch 3, 3.2.6), so translating a C++ solution
with that fix still works, but the fix is unnecessary here. Writing
`(prefix % k + k) % k` in Python is a sign you've memorized another language's
workaround.

**Edge cases tested:**
- a negative prefix (`[-1, 2, 9]`, k = 2)
- zeros, a single divisible element
- n = 3 × 10⁴ of all-equal values: n(n + 1)/2 subarrays
- 500 random cases checked against brute force

**Common bugs:**
- Missing the `remainder_count[0] = 1` seed.
- Using `prefix % k` in a language where it can be negative (not a bug in Python).
- Recomputing `prefix` from scratch inside the loop.

**Built-in shortcut:** none needed.

### 15.6.4 1094 Car Pooling (Medium): <https://leetcode.com/problems/car-pooling/>

**Problem.** A car with a given capacity drives east past locations 0..1000. Each trip
`[passengers, start, end]` picks riders up at `start` and drops them off at `end`.
Riders leave at `end` before new riders board there. Can every trip be served without
exceeding capacity?

**Signatures.** LeetCode:
`def carPooling(self, trips: List[List[int]], capacity: int) -> bool:`. Book:
`list[list[int]]`.

**Constraints → target.** T ≤ 1000 trips; locations 0..1000. Even O(T · L) = 10⁶ passes,
but the difference array gives O(T + L) and generalizes to much larger inputs.

**Approach 1: add each trip to every location it covers.**

<!-- file: code/ch15_prefix_sums/car_pooling.py -->

```python
class SolutionSimulate:
    def carPooling(self, trips: list[list[int]], capacity: int) -> bool:
        # O(T * L) time: each trip adds its riders to every location it covers.
        # O(L) auxiliary space.
        load = [0] * 1001
        for passengers, start, end in trips:
            for x in range(start, end):           # aboard on [start, end)
                load[x] += passengers
        return max(load) <= capacity
```

**Approach 2: sort pickup and drop-off events.**

<!-- file: code/ch15_prefix_sums/car_pooling.py -->

```python
class SolutionEvents:
    def carPooling(self, trips: list[list[int]], capacity: int) -> bool:
        # O(T log T) time: sort 2T events. O(T) auxiliary space.
        # At equal positions, drop-offs (negative deltas) sort first.
        events = []
        for passengers, start, end in trips:
            events.append((start, passengers))
            events.append((end, -passengers))
        load = 0
        for _, delta in sorted(events):
            load += delta
            if load > capacity:
                return False
        return True
```

This is the version to use when coordinates are huge (10⁹), because it doesn't index by
location. Sorting the tuples puts a negative delta before a positive one at the same
position, which encodes "drop off first".

**Approach 3: difference array (optimal for small coordinates).**

<!-- file: code/ch15_prefix_sums/car_pooling.py -->

```python
class Solution:
    def carPooling(self, trips: list[list[int]], capacity: int) -> bool:
        # O(T + L) time: O(1) per trip to mark, one prefix-sum pass over L.
        # O(L) auxiliary space: the difference array.
        diff = [0] * 1002
        for passengers, start, end in trips:
            diff[start] += passengers      # they board at start ...
            diff[end] -= passengers        # ... and are gone from end onward
        load = 0
        for delta in diff:
            load += delta                  # prefix sum of diff = load at x
            if load > capacity:
                return False
        return True
```

Why it works: the load at location x is the sum of every boarding at or before x, minus
every drop-off at or before x. That's exactly the prefix sum of `diff`. Marking the
drop-off at `end` (not `end + 1`) encodes the half-open interval [start, end), so
riders are gone when new ones board at the same place.

**Edge cases tested:**
- the two statement examples (capacity 4 → false, 5 → true)
- a drop-off and a pickup at the same location
- a single trip, 0 → 1000 at the maximum count
- identical trips
- 500 random cases checked across all three approaches

**Common bugs:**
- `diff[end + 1] -= passengers`, which keeps riders aboard at `end` (closed-interval
  thinking).
- Sorting events with pickups first at equal positions.
- Sizing `diff` too small for `end = 1000`.

**Built-in shortcut:** `itertools.accumulate(diff)` **[KNOW-HOW]** for the prefix pass.

## 15.7 Python-specific notes

**`accumulate` vs a loop.** Building a prefix array of 10⁶ ints
(`bench/ch15_prefix_build.py`, best of 5, five runs): `list(accumulate(nums,
initial=0))` beat the Python loop in 4 of 5 runs, by 1.2–1.4×, and lost in one run. A
small, noisy gain. Use whichever reads better; `initial=0` (3.8+) gives the leading
zero directly.

**Never sum a slice in a loop.** `sum(nums[i:j])` per query is O(j − i), plus an O(j − i)
copy (Ch 11 measured slicing in loops at ~550× the cost of indexing). Build the prefix
array once.

**Python ints don't overflow.** Prefix sums of 10⁵ values up to 10⁹ reach 10¹⁴, which
overflows 32-bit ints in C++ and Java (hence `long` in those solutions). It's a
non-issue in Python (Ch 3).

**`%` with negatives** works for remainder bucketing without adjustment (15.6.3).

**303 is a design problem.** LeetCode's template is `class NumArray` with `__init__` and
`sumRange`, not `class Solution`. The judge constructs it once and calls `sumRange` up to
10⁴ times. That's exactly why precomputing in `__init__` pays off.

## 15.8 Practice set

Hints are in Appendix D, under "Chapter 15 practice". Reference solutions:
`code/ch15_prefix_sums/practice_solutions.py`.

| # | Title | Difficulty | Link | Hint ref |
|---|---|---|---|---|
| 303 | Range Sum Query - Immutable | Easy | <https://leetcode.com/problems/range-sum-query-immutable/> | D.15.1 |
| 724 | Find Pivot Index | Easy | <https://leetcode.com/problems/find-pivot-index/> | D.15.2 |
| 523 | Continuous Subarray Sum | Medium | <https://leetcode.com/problems/continuous-subarray-sum/> | D.15.3 |

## 15.9 Chapter cheat sheet

| | |
|---|---|
| **Signals** | Range sums asked repeatedly; "contiguous subarray" + "sum = k / divisible by k" with **negatives**; product except self; many range updates |
| **Identity** | `sum(nums[i:j]) = prefix[j] - prefix[i]`, with `prefix[0] = 0` |
| **Templates** | Prefix array; prefix + map of counts seeded `{0: 1}`; remainder counts; difference array `diff[l] += v; diff[r] -= v` |
| **Complexity** | O(n) build, O(1) query; O(n) average counting with a map; O(n + u) for u range updates |
| **Top 3 bugs** | Missing the `{0: 1}` seed; updating the map before the lookup; closed vs half-open range ends |
| **Python** | `%` is already non-negative; no overflow; never `sum(slice)` in a loop |
