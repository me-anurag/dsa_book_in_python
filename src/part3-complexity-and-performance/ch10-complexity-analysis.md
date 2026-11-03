# Chapter 10: Complexity Analysis, Properly

## 10.0 At a glance

Complexity analysis answers one question before you write code: **will this approach
finish in time for the largest input the constraints allow?** This chapter covers:

- what O, Θ and Ω mean
- how to count operations, including loops that look quadratic but aren't
- amortized costs
- recursion and space, including the hidden cost of slicing in recursion
- a constraints → target-complexity table, built from **measured** Python speed on this
  book's machine rather than folklore

The measured budget: CPython 3.13.2 ran **5–11 million iterations per second** of a
realistic loop body (read, compare, two updates), and 18–32 million for a one-operation
body (`bench/ch10_python_budget.py`). Everything in 10.2.9 follows from that.

## 10.1 Complete reference

### Notation

| Symbol | Plain meaning | Example |
|---|---|---|
| O(f(n)) | Grows **at most** like f(n), up to a constant factor | Linear search is O(n) |
| Ω(f(n)) | Grows **at least** like f(n) | Any comparison sort is Ω(n log n) in the worst case |
| Θ(f(n)) | Both: grows **exactly** like f(n) | Summing a list is Θ(n) |
| Best / average / worst case | Cost on the most favorable, a typical, the least favorable input of size n | Quicksort: Θ(n log n) average, Θ(n²) worst |
| Amortized | Average cost per operation over **any** sequence of operations | `list.append`: O(1) amortized, O(n) for one resizing append |

In interviews, "O" is used loosely to mean a tight bound (Θ). Say "worst case" or
"average case" explicitly when they differ: dict lookup is O(1) average, O(n) worst.

### Growth classes

The last column was measured, not just derived: `code/ch10_complexity/verify_snippets.py`
doubles n for 20 snippets and checks how their work grows (10.5).

| Class | Typical source | Work when n doubles |
|---|---|---|
| O(1) | Index, dict/set lookup (average), arithmetic | Same |
| O(log n) | Halving: binary search, `while n: n //= 2` | +1 step |
| O(√n) | `while i * i <= n` | × 1.41 |
| O(n) | One pass; two pointers; sliding window | × 2 |
| O(n log n) | Sorting; n binary searches; merge sort | × ~2.1–2.2 |
| O(n²) | All pairs; `x in list` inside a loop; `pop(0)` in a loop | × 4 |
| O(n³) | All triples | × 8 |
| O(2ⁿ) | All subsets; naive two-branch recursion | × 2 per **+1** in n |
| O(n!) | All orderings | × (n + 1) per +1 in n |

### Constraints → target complexity (CPython)

Budget assumption: about **10⁷ simple loop iterations per second**. That's the measured
5–11 M/s for a realistic body, rounded up. LeetCode doesn't publish its time limits, so
treat "about one second" as a working assumption, not a fact.

| Largest n | Target | Steps at the limit | Python verdict |
|---|---|---|---|
| ≤ 10 | O(n!), O(n · n!) | 3.6 × 10⁶ – 3.6 × 10⁷ | OK with `itertools.permutations` (C); a pure-Python generator is borderline at n = 10 |
| ≤ 20 | O(2ⁿ), O(n · 2ⁿ) | 10⁶ – 2 × 10⁷ | 2ⁿ fine; n · 2ⁿ at n = 20 is 2–4 s: borderline |
| ≤ 100 | O(n³) | 10⁶ | Fine |
| ≤ 500 | O(n² log n), O(n³) only with tiny constants | n³ = 1.25 × 10⁸ | **O(n³) is too slow:** 12–25 s at the measured speed |
| ≤ 3,000 | O(n²) | 9 × 10⁶ | About 1–2 s: fine to borderline |
| ≤ 5,000 | O(n²) intended by setters | 2.5 × 10⁷ | 2.5–5 s in pure Python: **likely TLE**. Prefer O(n log n), or push the inner loop into C |
| ≤ 10⁵ | O(n log n) | ~1.7 × 10⁶ | Fine |
| ≤ 10⁶ | O(n), or O(n log n) with C-level sorting | 10⁶ – 2 × 10⁷ | O(n) fine; Python-level n log n borderline |
| ≥ 10⁹ (a value) | O(log n), O(√n) | 30; 3 × 10⁴ | Fine |

"Push the inner loop into C" means replacing a Python-level loop with a built-in that
loops internally (`sum`, `sorted`, `set` operations, slicing, `str` methods). Ch 11
measures this.

## 10.2 Explanations by group

### 10.2.1 O, Θ, Ω in plain terms

Big-O ignores constant factors and lower-order terms because, as n grows, they stop
mattering. 3n² + 100n + 7 is O(n²): at n = 10⁴, the n² term is 3 × 10⁸, and the rest is
about 10⁶.

Constants still matter **in practice**. Ch 4 measured up to a 3.5× gap between two O(n)
ways to build a list, and Ch 5 a ~2.5× gap between two O(n) ways to build a string.
Big-O tells you which approaches are candidates; constants decide between them. A
different class is a different league: Ch 6 measured `pop(0)` (O(n²) overall) at 1,400×
the time of `pop()` (O(n)) for 10⁵ items.

### 10.2.2 Best, average and worst case

| Algorithm | Best | Average | Worst | Worst-case input |
|---|---|---|---|---|
| Linear search for x | O(1) | O(n) | O(n) | x absent |
| `x in set` / dict lookup | O(1) | O(1) | O(n) | Every key hash-collides |
| Quicksort, random pivot | O(n log n) | O(n log n) | O(n²) | Extremely unlucky pivots |
| Timsort (`sorted`) | O(n) | O(n log n) | O(n log n) | Random data |
| Insertion into a sorted list (`insort`) | O(log n) search + O(1) | O(n) | O(n) | Insert at the front |

On LeetCode, assume **worst case** for your own code (hidden tests include adversarial
inputs) and **average case** for hash tables (Python randomizes string hashing, and
real collision attacks are out of scope for the judge's tests).

### 10.2.3 Counting operations

Three rules cover most code:

| Code shape | Cost |
|---|---|
| Statements one after another | Add, and keep the largest term: O(n) + O(n log n) = O(n log n) |
| A loop of k iterations around a body of cost c | k · c |
| A loop whose range depends on the outer variable | Sum the inner counts |

```python
def count_pairs(n: int) -> int:
    pairs = 0
    for i in range(n):
        for j in range(i + 1, n):
            pairs += 1
    return pairs


count_pairs(100)  # -> 4950
```

The inner loop runs n − 1, n − 2, ..., 0 times: n(n − 1)/2 in total, which is
O(n²). Halving the work doesn't change the class.

**Calls cost what they do.** A single line can hide O(n): `x in some_list`, `s[i:]`,
`a + b`, `sorted(a)`, `a.pop(0)`, `"".join(parts)`. Ch 12 lists the cost of every common
operation.

### 10.2.4 Nested loops that are not O(n²)

A `while` inside a `for` is not automatically quadratic. Count **how many times each
pointer can move in total**, not per outer iteration:

<!-- file: code/ch10_complexity/snippets.py -->

```python
def s09(nums, k):                   # longest window with sum <= k, nums >= 0
    best = window = left = 0
    for right in range(len(nums)):
        window += nums[right]
        while window > k:
            window -= nums[left]
            left += 1
        best = max(best, right - left + 1)
    return best
```

`right` moves n times. `left` only ever increases, and never past `right`, so across the
**whole run** the inner loop body executes at most n times. Total: O(n) + O(n) = O(n).
This is the sliding-window argument (Ch 17), and `verify_snippets.py` measured exactly
2× work when n doubles.

The same reasoning covers two pointers moving toward each other (Ch 16): each step
moves one pointer inward, and they meet after at most n steps.

A doubling outer loop is another non-obvious case:

<!-- file: code/ch10_complexity/snippets.py -->

```python
def s07(n):
    total = 0
    i = 1
    while i <= n:
        for _ in range(i):
            total += 1
        i *= 2
    return total
```

The inner loop runs 1 + 2 + 4 + ... + 2^⌊log₂ n⌋ times, which is less than 2n. So it's
O(n), not O(n log n).

### 10.2.5 Amortized analysis

**`list.append`.** Most appends write to a spare slot in O(1). Occasionally the list is
full: CPython allocates about 12.5% more room and copies everything (Ch 6). The copies
happen at geometrically spaced sizes, so the total copying over n appends is a
constant multiple of n. Measured in Ch 6: building a list of N items doubled in time
when N doubled.

**Sliding window.** As 10.2.4 showed, one iteration of the outer loop may run the inner
`while` many times, but the total over the whole run is bounded by n. That's an
amortized argument: "expensive iterations are paid for by earlier cheap ones".

The difference from average case: amortized bounds hold for **every** input sequence. No
luck is involved.

### 10.2.6 Logarithms

| n | log₂ n |
|---|---|
| 10³ | ~10 |
| 10⁶ | ~20 |
| 10⁹ | ~30 |
| 10¹⁸ | ~60 |

Anything that halves the remaining problem each step takes O(log n) steps. Binary search
over 10⁹ candidate answers takes about 30 iterations (Ch 19). In Big-O the base doesn't
matter, because log₂ n and log₁₀ n differ only by a constant factor.

### 10.2.7 Recursion

Write the cost as a **recurrence**: the cost of one call in terms of the cost of its
recursive calls.

| Recurrence | Solution | Example |
|---|---|---|
| T(n) = T(n − 1) + O(1) | O(n) | Recursive sum, `s13` |
| T(n) = T(n / 2) + O(1) | O(log n) | Binary search |
| T(n) = 2T(n / 2) + O(n) | O(n log n) | Merge sort, `s14` |
| T(n) = 2T(n − 1) + O(1) | O(2ⁿ) | Two branches that each shrink n by 1, `s15` |
| T(n) = T(n − 1) + O(n) | O(n²) | Selection-sort-like recursion |

**Recursion uses stack space**: one frame per active call. `s13` (depth n) needs O(n)
auxiliary space even though it allocates nothing, and it hits CPython's default limit
of 1000 frames (Ch 1) at n ≈ 1000. Merge sort's depth is O(log n).

### 10.2.8 Space complexity

| Kind | Counts | Example |
|---|---|---|
| Input space | The arguments | `nums` itself |
| Auxiliary space | Extra memory your algorithm uses | A set of seen values: O(n); two pointers: O(1) |
| Output space | The returned structure | The result list of 238 Product of Array Except Self |

This book always states **time and auxiliary space separately**. Problems that ask for
"O(1) extra space" usually exclude the output (238's follow-up says so explicitly; Ch 9).

**Slicing in recursion is a hidden cost.** Compare two recursive binary searches:

```python
def contains_slicing(a: list[int], x: int) -> bool:
    if not a:
        return False
    mid = len(a) // 2
    if a[mid] == x:
        return True
    if a[mid] < x:
        return contains_slicing(a[mid + 1:], x)    # copies half the list
    return contains_slicing(a[:mid], x)


def contains_indices(a: list[int], x: int, lo: int = 0, hi: int | None = None) -> bool:
    if hi is None:
        hi = len(a)
    if lo >= hi:
        return False
    mid = (lo + hi) // 2
    if a[mid] == x:
        return True
    if a[mid] < x:
        return contains_indices(a, x, mid + 1, hi)
    return contains_indices(a, x, lo, mid)


data = list(range(0, 100, 2))
(contains_slicing(data, 42), contains_indices(data, 42))  # -> (True, True)
(contains_slicing(data, 43), contains_indices(data, 43))  # -> (False, False)
```

Same answers, different costs:

| Version | Time | Auxiliary space |
|---|---|---|
| Slicing | O(n): copies of n/2 + n/4 + ... ≈ n elements | O(n) in total allocations; O(log n) frames |
| Indices | O(log n) | O(log n) frames |

The slicing version silently turns O(log n) into O(n). Pass indices, not slices.

### 10.2.9 The Python budget

Measured with `bench/ch10_python_budget.py` (N = 10⁶ iterations, best of 5, CPython
3.13.2, Windows 11 laptop, ranges over runs):

| Loop body | Million iterations per second |
|---|---|
| empty (`pass`) | 78–92 |
| `total += i` | 25–31 |
| list index read | 25–26 |
| set membership `i in s` | 29–32 |
| dict lookup | 18–19 |
| `list.append` | 25–26 |
| **typical body: one read, one compare, two updates** | **5–11** |

The typical-body row is the one to plan with. Real solutions do several things per
iteration, and each Python-level operation costs tens of nanoseconds. Hence the
working budget of **~10⁷ simple steps per second**, and the constraints table in 10.1.

Two consequences students find uncomfortable:

1. **O(n²) at n = 5,000 is 2.5 × 10⁷ steps: several seconds of Python.** Problem setters
   often intend O(n²) there, and C++ finishes in well under a second. In Python, look
   for the O(n log n) idea, or make the inner loop a built-in.
2. **O(n³) at n = 500 doesn't pass in pure Python**: 1.25 × 10⁸ steps is 12–25 s at the
   measured speed.

## 10.3 Gotchas & traps

**1. Counting loops instead of work.** One line can be O(n): `in` on a list, slicing,
`pop(0)`, `+` on lists or strings (10.2.3).

**2. Calling every nested loop O(n²).** Count total pointer movement (10.2.4).

**3. Calling every doubling loop O(n log n).** `s07` is O(n) (10.2.4).

**4. Forgetting recursion stack space.** Depth n means O(n) auxiliary space, and a
`RecursionError` near depth 1000 (10.2.7).

**5. Slicing in recursion.** Turns O(log n) into O(n) (10.2.8).

**6. Reporting only time.** State auxiliary space too, separately (10.2.8).

**7. Trusting C++ rules of thumb in Python.** Python's budget is roughly 10⁷ simple steps
per second. Measured: 5–11 M/s for a realistic loop body (10.2.9).

**8. Average-case hash costs stated as worst case.** Say "O(1) average" (10.2.2).

## 10.4 Benchmarks

| File | Measures | Result (CPython 3.13.2) |
|---|---|---|
| `bench/ch10_python_budget.py` | Loop iterations per second for 8 loop bodies | 5–11 M/s typical body; 18–32 M/s single operation; 78–92 M/s empty |
| `code/ch10_complexity/verify_snippets.py` | Work growth as n doubles, for the 20 exercise snippets | All 20 match their classes (lines, comparisons or time) |

## 10.5 Exercises: give the complexity of 20 snippets

For each function, give the **time** and **auxiliary space**, each with its reason.
Answers are in Appendix D. `code/ch10_complexity/verify_snippets.py` confirms every time
class empirically:
- by counting executed Python lines as n doubles, for Python-level loops
- by counting comparisons on a wrapper type, for `in` on a list and `sorted`
- by timing, for `pop(0)` and string concatenation

<!-- file: code/ch10_complexity/snippets.py -->

```python
def s01(nums):
    total = 0
    for x in nums:
        total += x
    return total


def s02(nums):
    count = 0
    for x in nums:
        for y in nums:
            if x < y:
                count += 1
    return count


def s03(nums):
    n = len(nums)
    pairs = 0
    for i in range(n):
        for j in range(i + 1, n):
            pairs += nums[i] == nums[j]
    return pairs


def s04(n):
    steps = 0
    while n > 0:
        n //= 2
        steps += 1
    return steps


def s05(n):
    i = 1
    while i < n:
        i *= 2
    return i


def s06(nums):
    total = 0
    for x in nums:
        k = len(nums)
        while k > 1:
            k //= 2
            total += x
    return total
```

`s07` and `s09` appear in 10.2.4.

<!-- file: code/ch10_complexity/snippets.py -->

```python
def s08(nums):                      # nums sorted; pair with sum 0?
    left, right = 0, len(nums) - 1
    while left < right:
        s = nums[left] + nums[right]
        if s == 0:
            return True
        if s < 0:
            left += 1
        else:
            right -= 1
    return False
```

<!-- file: code/ch10_complexity/snippets.py -->

```python
def s10(nums):
    seen = []
    for x in nums:
        if x not in seen:           # list membership
            seen.append(x)
    return len(seen)


def s11(nums):
    seen = set()
    for x in nums:
        if x not in seen:           # set membership
            seen.add(x)
    return len(seen)


def s12(nums):
    nums = sorted(nums)
    best = 0
    for i in range(1, len(nums)):
        best = max(best, nums[i] - nums[i - 1])
    return best


def s13(n):
    if n == 0:
        return 0
    return 1 + s13(n - 1)


def s14(nums):
    if len(nums) <= 1:
        return nums
    mid = len(nums) // 2
    left, right = s14(nums[:mid]), s14(nums[mid:])
    merged, i, j = [], 0, 0
    while i < len(left) and j < len(right):
        if left[i] <= right[j]:
            merged.append(left[i])
            i += 1
        else:
            merged.append(right[j])
            j += 1
    return merged + left[i:] + right[j:]


def s15(n):
    if n <= 1:
        return 1
    return s15(n - 1) + s15(n - 1)


def s16(nums):
    n = len(nums)
    count = 0
    for i in range(n):
        for j in range(n):
            for k in range(n):
                count += nums[i] + nums[j] == nums[k]
    return count


def s17(n):
    total = 0
    for i in range(1, n + 1):
        for _ in range(i, n + 1, i):
            total += 1
    return total


def s18(nums):
    nums = list(nums)
    while nums:
        nums.pop(0)


def s19(n):
    i = 1
    while i * i <= n:
        i += 1
    return i - 1


def s20(s):
    result = ""
    for i in range(len(s)):
        result = result + s[i] if i % 2 == 0 else s[i] + result
    return result
```

## 10.6 Chapter cheat sheet

| Rule | Detail |
|---|---|
| Budget | ~10⁷ simple Python steps/s (measured 5–11 M/s typical) |
| n ≤ 10 / 20 / 100 | n! / 2ⁿ / n³ |
| n ≤ 3,000 | O(n²); at 5,000 O(n²) is several seconds in Python |
| n ≤ 10⁵ – 10⁶ | O(n log n) / O(n) |
| Value ≥ 10⁹ | O(log n) or O(√n) |
| Sequential code | Add the costs; keep the largest |
| Nested loops | Multiply, or sum inner counts when they depend on the outer variable |
| Pointer loops | Count total moves: two pointers and windows are O(n) |
| Recurrences | T(n−1)+1 → n; T(n/2)+1 → log n; 2T(n/2)+n → n log n; 2T(n−1)+1 → 2ⁿ |
| Space | State auxiliary space separately; recursion depth counts; slices copy |
| Hidden O(n) | `in` list, slice, `pop(0)`, `insert(0)`, `+` on sequences, `sorted` |
