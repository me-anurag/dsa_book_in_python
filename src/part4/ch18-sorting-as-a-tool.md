# Chapter 18: Sorting as a Tool (+ Writing Sorts by Hand)

## 18.0 At a glance

- **What this pattern solves:** problems that become simple once the data is in order:
  intervals, greedy choices, "k-th" questions, partitioning by category. Plus the
  hand-written sorts interviewers ask for when built-ins are banned.
- **Prerequisites:** Ch 6 (`sort`, `key=`, stability, `cmp_to_key`), Ch 9 (the
  hand-written sorts at a glance), Ch 10 (recurrences).
- **Typical complexity:** O(n log n) time (comparison sorting); O(n + k) for counting
  sort; expected O(n) for quickselect.
- **Tools used in this chapter:**
  - `sorted`, `list.sort` with `key=`, `heapq` **[KNOW-HOW]**
  - merge sort, quicksort (random pivot, 3-way), counting sort, Dutch national flag,
    interval sweeps, earliest-end greedy and quickselect **[HAND-WRITE]**

## 18.1 Recognition signals

- "**intervals**", "meetings", "overlap", "merge ranges" → sort by start (or by end)
- "**k-th largest/smallest**", "**median**" → sort, heap or quickselect
- "**no built-in sort**", "O(n log n)" → write merge sort or quicksort (912)
- "**only values 0, 1, 2**" (few categories), "**one pass**", "in place" → Dutch flag (75)
- "**minimum number to remove** so that none overlap" → greedy after sorting by end
  (435)
- "**arrange** to form the largest/smallest" → custom comparator (179, Ch 6)

**Look-alikes:**

| Looks like | Actually | Tell |
|---|---|---|
| Sort then scan | Hashing (Ch 14) | You only need equality or counts, not order; O(n) required (128) |
| k-th largest by sorting | Heap / quickselect / counting | k small, or the value range small, or the follow-up says "without sorting" |
| Greedy after sorting | DP | The greedy exchange argument fails (weighted intervals) |

## 18.2 Core idea

Unsorted data hides relationships. After sorting, related items sit next to each other:
overlapping intervals become adjacent, equal values form runs, and the k-th element sits
at a fixed index. The sort costs O(n log n) once, and the remaining work is often a
single O(n) scan.

When the sort itself is the question (912), three ideas cover it:

| Sort | Idea | Why O(n log n) (or better) |
|---|---|---|
| Merge sort | Split in half, sort each half, merge | T(n) = 2T(n/2) + O(n) → O(n log n) in every case (Ch 10) |
| Quicksort | Partition around a pivot, recurse on the parts | Random pivot: expected O(n log n); a bad pivot every time: O(n²) |
| Counting sort | Count occurrences of each value | O(n + k) for k possible values; no comparisons, so the Ω(n log n) comparison bound doesn't apply |

## 18.3 The template

**Sort, then sweep** (intervals):

```python
def merge_overlapping(intervals: list[list[int]]) -> list[list[int]]:
    result: list[list[int]] = []
    for start, end in sorted(intervals):
        if result and start <= result[-1][1]:
            result[-1][1] = max(result[-1][1], end)
        else:
            result.append([start, end])
    return result


merge_overlapping([[8, 10], [1, 3], [2, 6]])  # -> [[1, 6], [8, 10]]
```

- **Invariant:** `result` is sorted, its intervals are disjoint, and together they cover
  exactly the intervals processed so far.
- **Complexity:** O(n log n) time (the sort), O(n) auxiliary space for the sorted copy.

**Three-way partition** (the engine of quicksort, quickselect and the Dutch flag). The
invariant, with `pivot` the dividing value:

```text
 lo        lt         i          gt         hi
  [ < pivot | == pivot | unknown  | > pivot ]
```

Each step classifies `a[i]` and shrinks the unknown region by one. O(n) time, O(1) space.

## 18.4 Dry run

75 Sort Colors, Dutch flag (18.6.2), on `[2, 0, 2, 1, 1, 0]`. Regions: `[:low]` = 0s,
`[low:mid]` = 1s, `[high+1:]` = 2s.

| Step | low | mid | high | `nums[mid]` | Action | `nums` after |
|---|---|---|---|---|---|---|
| 1 | 0 | 0 | 5 | 2 | swap mid, high; high−1 | `[0, 0, 2, 1, 1, 2]` |
| 2 | 0 | 0 | 4 | 0 | swap low, mid; low+1, mid+1 | `[0, 0, 2, 1, 1, 2]` |
| 3 | 1 | 1 | 4 | 0 | swap low, mid; low+1, mid+1 | `[0, 0, 2, 1, 1, 2]` |
| 4 | 2 | 2 | 4 | 2 | swap mid, high; high−1 | `[0, 0, 1, 1, 2, 2]` |
| 5 | 2 | 2 | 3 | 1 | mid+1 | unchanged |
| 6 | 2 | 3 | 3 | 1 | mid+1 | unchanged |

`mid = 4 > high = 3`: done, in one pass. At step 1, `mid` did **not** advance, because the
value swapped in from `high` (a 0) still had to be examined.

## 18.5 Variations

| Variant | What changes | Example |
|---|---|---|
| Write the sort | Merge / quick / counting | 912 Sort an Array |
| Few distinct values | Three-way partition, one pass | 75 Sort Colors |
| Merge overlaps | Sort by start; extend the last interval while `start <= last_end` | 56 Merge Intervals |
| Insert into sorted intervals | No sort needed: three phases (before, overlap, after) | 57 Insert Interval |
| Max non-overlapping | Sort by **end**; keep an interval whenever `start >= last_end` | 435 Non-overlapping Intervals |
| Pairwise order | `cmp_to_key` comparator | 179 Largest Number (Ch 6) |
| k-th element | Sort / size-k heap / quickselect / counting | 215 Kth Largest Element in an Array |

## 18.6 Worked problems

### 18.6.1 912 Sort an Array (Medium): <https://leetcode.com/problems/sort-an-array/>

**Problem.** Sort an integer array ascending without any built-in sort, in O(n log n)
time, using as little extra space as possible.

**Signatures.** LeetCode: `def sortArray(self, nums: List[int]) -> List[int]:`. Book:
`list[int]`.

**Constraints → target.** n ≤ 5 × 10⁴; values in [−5 × 10⁴, 5 × 10⁴]. O(n²) is 2.5 × 10⁹
steps: TLE. Note the **small value range**: counting sort is legal and O(n + k).

**Approach 1: insertion sort (O(n²), shown for contrast).**

<!-- file: code/ch18_sorting/sort_array.py -->

```python
class SolutionInsertion:
    def sortArray(self, nums: list[int]) -> list[int]:
        # O(n^2) worst/average time (each element may shift past all earlier ones);
        # O(n) on already-sorted input. O(1) auxiliary. Too slow for n = 5 * 10^4.
        a = nums[:]
        for i in range(1, len(a)):
            x = a[i]
            j = i - 1
            while j >= 0 and a[j] > x:
                a[j + 1] = a[j]
                j -= 1
            a[j + 1] = x
        return a
```

**Approach 2: bottom-up merge sort.** Ch 9's version was recursive and allocated new
lists at every level. This one merges runs of width 1, 2, 4, ... and swaps between two
buffers, so it needs one extra array and no recursion:

<!-- file: code/ch18_sorting/sort_array.py -->

```python
class SolutionMergeSort:
    def sortArray(self, nums: list[int]) -> list[int]:
        # O(n log n) time in every case: log n levels, O(n) merging per level.
        # O(n) auxiliary: one buffer reused at every level (bottom-up, no recursion).
        a = nums[:]
        n = len(a)
        buffer = [0] * n
        width = 1
        while width < n:
            for lo in range(0, n, 2 * width):
                mid = min(lo + width, n)
                hi = min(lo + 2 * width, n)
                i, j, k = lo, mid, lo
                while i < mid and j < hi:
                    if a[i] <= a[j]:           # <= keeps the sort stable
                        buffer[k] = a[i]
                        i += 1
                    else:
                        buffer[k] = a[j]
                        j += 1
                    k += 1
                while i < mid:
                    buffer[k] = a[i]
                    i += 1
                    k += 1
                while j < hi:
                    buffer[k] = a[j]
                    j += 1
                    k += 1
            a, buffer = buffer, a              # swap roles; no copying back
            width *= 2
        return a
```

**Complexity proof (merge sort).** The `width` doubles each round, so there are
⌈log₂ n⌉ rounds. In each round every element is written to `buffer` exactly once: O(n)
per round. Total O(n log n) in the **worst** case too: the number of rounds and the work per round
don't depend on the input's order.
Space: `buffer` plus the working copy, O(n).

**Approach 3: quicksort (random pivot, three-way partition).**

<!-- file: code/ch18_sorting/sort_array.py -->

```python
class SolutionQuickSort:
    def sortArray(self, nums: list[int]) -> list[int]:
        # Expected O(n log n) time (random pivot); O(n^2) worst case with
        # vanishing probability. O(log n) stack: recurse into the smaller side.
        a = nums[:]
        self._sort(a, 0, len(a) - 1)
        return a

    def _sort(self, a: list[int], lo: int, hi: int) -> None:
        while lo < hi:
            pivot = a[random.randint(lo, hi)]
            lt, i, gt = lo, lo, hi             # 3-way: < pivot | == pivot | > pivot
            while i <= gt:
                if a[i] < pivot:
                    a[lt], a[i] = a[i], a[lt]
                    lt += 1
                    i += 1
                elif a[i] > pivot:
                    a[i], a[gt] = a[gt], a[i]
                    gt -= 1
                else:
                    i += 1
            if lt - lo < hi - gt:
                self._sort(a, lo, lt - 1)
                lo = gt + 1
            else:
                self._sort(a, gt + 1, hi)
                hi = lt - 1
```

**Complexity argument (quicksort).** Each partition is O(size). A random pivot lands in
the middle half of the range with probability 1/2, and then both parts are at most 3/4
of the size. So along any path the size shrinks by a constant factor every O(1)
expected levels: O(log n) expected depth, with O(n) work per level. That gives
**expected O(n log n)**. The worst case (always the extreme pivot) is O(n²), but with a
random pivot its probability vanishes as n grows. The three-way partition makes
all-equal inputs O(n): the whole range lands in the `==` band at once. The stack stays
O(log n) because recursion only enters the smaller part.

**Approach 4: counting sort (O(n + k), uses the value range).**

<!-- file: code/ch18_sorting/sort_array.py -->

```python
class SolutionCountingSort:
    def sortArray(self, nums: list[int]) -> list[int]:
        # O(n + k) time, k = 10^5 + 1 possible values; O(k) auxiliary.
        # Not a comparison sort, so the O(n log n) lower bound doesn't apply.
        lo, hi = -5 * 10**4, 5 * 10**4
        counts = [0] * (hi - lo + 1)
        for x in nums:
            counts[x - lo] += 1
        out = []
        for offset, c in enumerate(counts):
            if c:
                out.extend([offset + lo] * c)
        return out
```

**Which to present?** In an interview: merge sort (it's guaranteed) or quicksort (it's in
place), with the complexity arguments above. Mention counting sort when the constraints
show a small value range: it's the fastest of the three here (18.7).

**Edge cases tested:**
- a single element, all equal
- the extreme values ±5 × 10⁴
- 300 random arrays checked against `sorted` for all four approaches
- n = 5 × 10⁴ random, all-equal and already-sorted inputs for quicksort

**Common bugs:**
- Quicksort with a fixed first/last pivot on sorted input: O(n²) and a
  `RecursionError`.
- A two-way partition, which degrades on duplicates.
- A merge using `<` instead of `<=`, which loses stability.
- Forgetting the leftover runs after the merge loop.

**Built-in shortcut:** `sorted(nums)`, which the problem forbids. It also measured
fastest by far (18.7).

### 18.6.2 75 Sort Colors (Medium): <https://leetcode.com/problems/sort-colors/>

**Problem.** An array holds only 0s, 1s and 2s. Sort it in place without the library sort.
The follow-up asks for one pass with constant extra space.

**Signatures.** LeetCode: `def sortColors(self, nums: List[int]) -> None:`. Book:
`list[int]`.

**Constraints → target.** n ≤ 300, so speed is irrelevant. The point is the one-pass,
O(1)-space algorithm.

**Approach 1: count, then overwrite (two passes).**

<!-- file: code/ch18_sorting/sort_colors.py -->

```python
class SolutionCounting:
    def sortColors(self, nums: list[int]) -> None:
        # O(n) time but TWO passes: count, then overwrite. O(1) auxiliary (3 counts).
        counts = [0, 0, 0]
        for x in nums:
            counts[x] += 1
        i = 0
        for color in range(3):
            for _ in range(counts[color]):
                nums[i] = color
                i += 1
```

**Approach 2: Dutch national flag (one pass, optimal).**

<!-- file: code/ch18_sorting/sort_colors.py -->

```python
class Solution:
    def sortColors(self, nums: list[int]) -> None:
        # Dutch national flag: ONE pass, O(n) time, O(1) auxiliary.
        # Invariant: nums[:low] == 0s, nums[low:mid] == 1s, nums[high + 1:] == 2s,
        # and nums[mid:high + 1] is still unexamined.
        low, mid, high = 0, 0, len(nums) - 1
        while mid <= high:
            if nums[mid] == 0:
                nums[low], nums[mid] = nums[mid], nums[low]
                low += 1
                mid += 1
            elif nums[mid] == 1:
                mid += 1
            else:
                nums[mid], nums[high] = nums[high], nums[mid]
                high -= 1                  # don't advance mid: the swapped-in value
                                           # hasn't been examined yet
```

Why it works: each iteration shrinks the unexamined region `[mid, high]` by one, while
the invariant holds. When it's empty, the array is 0s, then 1s, then 2s. Swapping a 0 to
`low` can advance `mid` too, because `nums[low]` was a 1 (or `low == mid`) and is now in
place. Traced in 18.4.

**Edge cases tested:**
- a single element, all equal, reversed order
- 500 random arrays checked against `sorted`

**Common bugs:**
- Advancing `mid` after swapping with `high`.
- `while mid < high`, which leaves the last element unexamined.

**Built-in shortcut:** `nums.sort()`, which is forbidden here.

### 18.6.3 56 Merge Intervals (Medium): <https://leetcode.com/problems/merge-intervals/>

**Problem.** Given intervals `[start, end]`, merge every group of overlapping intervals and
return the resulting disjoint intervals. **Intervals that touch (`[1, 4]`, `[4, 5]`)
count as overlapping.**

**Signatures.** LeetCode: `def merge(self, intervals: List[List[int]]) -> List[List[int]]:`.
Book: `list[list[int]]`.

**Constraints → target.** n ≤ 10⁴: O(n log n).

**Approach 1: merge any overlapping pair until none remains.**

<!-- file: code/ch18_sorting/merge_intervals.py -->

```python
class SolutionRepeatedMerge:
    def merge(self, intervals: list[list[int]]) -> list[list[int]]:
        # O(n^3) worst case: up to n merges, each found by an O(n^2) pair scan.
        # O(n) auxiliary. Test-only.
        result = [iv[:] for iv in intervals]
        merged = True
        while merged:
            merged = False
            for i in range(len(result)):
                for j in range(i + 1, len(result)):
                    a, b = result[i], result[j]
                    if a[0] <= b[1] and b[0] <= a[1]:          # overlap or touch
                        result[i] = [min(a[0], b[0]), max(a[1], b[1])]
                        del result[j]
                        merged = True
                        break
                if merged:
                    break
        return sorted(result)
```

**Approach 2: sort by start, sweep (optimal).**

<!-- file: code/ch18_sorting/merge_intervals.py -->

```python
class Solution:
    def merge(self, intervals: list[list[int]]) -> list[list[int]]:
        # O(n log n) time: the sort dominates; the scan is O(n).
        # O(n) auxiliary: the sorted copy (the output is extra, not auxiliary).
        result: list[list[int]] = []
        for start, end in sorted(intervals):          # by start (then end)
            if result and start <= result[-1][1]:      # overlaps or touches last
                result[-1][1] = max(result[-1][1], end)
            else:
                result.append([start, end])
        return result
```

Why sorting makes one pass enough: after sorting by start, an interval can only overlap
the **last** merged block. Every earlier block ends before the last block starts, which
is no later than the current start. `max(...)` handles nested intervals (`[1, 10]`
swallowing `[2, 3]`).

**Edge cases tested:**
- touching intervals, unsorted input, nested intervals
- zero-length `[5, 5]`, duplicates
- a chain of 10⁴ touching intervals
- 400 random cases checked against approach 1

**Common bugs:**
- `start < last_end`, which fails to merge touching intervals (this problem says they
  merge).
- `result[-1][1] = end` instead of `max`, which breaks nested intervals.
- Mutating the input lists while sorting a shallow copy of them (`sorted(intervals)`
  copies the outer list only).

**Built-in shortcut:** `sorted` **[KNOW-HOW]** is expected here. The sweep is the
**[HAND-WRITE]** part.

### 18.6.4 215 Kth Largest Element in an Array (Medium): <https://leetcode.com/problems/kth-largest-element-in-an-array/>

**Problem.** Return the k-th largest value, counting duplicates (the k-th position in
descending order, not the k-th distinct value). The statement asks whether you can do it
without sorting.

**Signatures.** LeetCode: `def findKthLargest(self, nums: List[int], k: int) -> int:`. Book:
`list[int]`.

**Constraints → target.** n ≤ 10⁵; values in [−10⁴, 10⁴]. Any O(n log n) passes; the
interest is in the alternatives.

**Approach 1: sort.**

<!-- file: code/ch18_sorting/kth_largest.py -->

```python
class SolutionSort:
    def findKthLargest(self, nums: list[int], k: int) -> int:
        # O(n log n) time: full sort. O(n) auxiliary: the sorted copy.
        return sorted(nums)[-k]
```

**Approach 2: a size-k min-heap.** It keeps the k largest values seen so far; its root is
the k-th largest:

<!-- file: code/ch18_sorting/kth_largest.py -->

```python
class SolutionHeap:
    def findKthLargest(self, nums: list[int], k: int) -> int:
        # O(n log k) time: a min-heap holding the k largest seen so far; its root
        # is the k-th largest. O(k) auxiliary space.
        heap: list[int] = []
        for x in nums:
            if len(heap) < k:
                heapq.heappush(heap, x)
            elif x > heap[0]:
                heapq.heapreplace(heap, x)
        return heap[0]
```

**Approach 3: quickselect (expected O(n)).** Partition as in quicksort, but continue
**only into the side that contains the target index**:

<!-- file: code/ch18_sorting/kth_largest.py -->

```python
class SolutionQuickselect:
    def findKthLargest(self, nums: list[int], k: int) -> int:
        # Expected O(n) time: each partition is O(size) and, with a random pivot,
        # the part we keep shrinks geometrically on average (n + n/2 + ... ~ 2n).
        # Worst case O(n^2). O(1) auxiliary besides the working copy (iterative).
        a = nums[:]
        target = len(a) - k                     # index in ascending order
        lo, hi = 0, len(a) - 1
        while True:
            pivot = a[random.randint(lo, hi)]
            lt, i, gt = lo, lo, hi              # 3-way partition (Ch 9)
            while i <= gt:
                if a[i] < pivot:
                    a[lt], a[i] = a[i], a[lt]
                    lt += 1
                    i += 1
                elif a[i] > pivot:
                    a[i], a[gt] = a[gt], a[i]
                    gt -= 1
                else:
                    i += 1
            if target < lt:
                hi = lt - 1
            elif target > gt:
                lo = gt + 1
            else:
                return pivot                    # target falls in the == band
```

**Approach 4: counting (the value range is only 2 × 10⁴ + 1).**

<!-- file: code/ch18_sorting/kth_largest.py -->

```python
class SolutionCounting:
    def findKthLargest(self, nums: list[int], k: int) -> int:
        # O(n + R) time, R = 2 * 10^4 + 1 possible values: walk counts from the
        # top. O(R) auxiliary. Works only because the value range is small.
        offset = 10**4
        counts = [0] * (2 * 10**4 + 1)
        for x in nums:
            counts[x + offset] += 1
        remaining = k
        for v in range(len(counts) - 1, -1, -1):
            remaining -= counts[v]
            if remaining <= 0:
                return v - offset
        raise ValueError("k out of range")
```

Why quickselect is linear on average: quicksort recurses into **both** parts (n log n);
quickselect into **one**. With a random pivot the kept part is, on average, a constant
fraction of the range, so the work is n + n/2 + n/4 + ... ≤ 2n in expectation.

**Edge cases tested:**
- duplicates (k counts positions), a single element, all equal
- the extreme values ±10⁴
- n = 10⁵ with k = 500
- 500 random cases across all four approaches

**Common bugs:**
- Returning the k-th **distinct** value (`sorted(set(nums))`).
- An off-by-one between "k-th largest" and the ascending index `n - k`.
- A max-heap of all n elements popped k times: O(n + k log n), correct but heavier than
  the size-k heap.

**Built-in shortcut:** `heapq.nlargest(k, nums)[-1]` **[KNOW-HOW]**: O(n log k).

## 18.7 Python-specific notes

**Theory vs CPython**, measured (`bench/ch18_sorting.py`, best of 3, two runs):

| 912, n = 50,000 | ms |
|---|---|
| `sorted()` (forbidden by the problem; shown for scale) | 5.5–5.9 |
| Counting sort (Python) | 12.2–12.5 |
| Bottom-up merge sort (Python) | 85.8–86.2 |
| Quicksort, random 3-way (Python) | 107–131 |

| 215, n = 100,000 | k = 10 | k = 50,000 |
|---|---|---|
| Sort | 13–15 ms | 14.7–15.3 ms |
| Size-k heap | 3.1–3.4 ms | 18.9–19.2 ms |
| Quickselect (Python, expected O(n)) | 14–25 ms | 22–31 ms |
| Counting (value range 2 × 10⁴) | 4.2–5.3 ms | 4.7–4.9 ms |

Uncomfortable but true:
1. **Quickselect, the "optimal" O(n) answer, measured slower than sorting** in CPython. Its
   O(n) Python-level partition loop costs more than `sorted`'s O(n log n) work in C. In an
   interview, present quickselect for the complexity; on LeetCode, `sorted` or a heap is
   faster in practice.
2. **The heap wins for small k**: one cheap comparison against `heap[0]` per element, and
   only rare pushes.
3. **Counting wins whenever the constraints give a small value range.** Read the value
   bounds (Ch 1).
4. **No hand-written comparison sort comes within 10× of `sorted`.** That's expected
   (Ch 11). Write them when asked, and use `sorted` otherwise.

**Sorting lists of lists.** `sorted(intervals)` compares `[start, end]` lists
lexicographically: by start, then end. `key=lambda iv: iv[0]` is equivalent here, and
stability keeps equal starts in input order.

## 18.8 Practice set

Hints are in Appendix D, under "Chapter 18 practice". Reference solutions:
`code/ch18_sorting/practice_solutions.py`, and `code/ch06_list/largest_number.py` for 179.

| # | Title | Difficulty | Link | Hint ref |
|---|---|---|---|---|
| 57 | Insert Interval | Medium | <https://leetcode.com/problems/insert-interval/> | D.18.1 |
| 435 | Non-overlapping Intervals | Medium | <https://leetcode.com/problems/non-overlapping-intervals/> | D.18.2 |
| 179 | Largest Number | Medium | <https://leetcode.com/problems/largest-number/> | D.18.3 |

## 18.9 Chapter cheat sheet

| | |
|---|---|
| **Signals** | Intervals, k-th, "no built-in sort", few categories, greedy selection, custom order |
| **Sorts** | Merge: O(n log n) always, stable, O(n) space. Quick: expected O(n log n), random pivot + 3-way + smaller side first. Counting: O(n + k) for a small range |
| **Partition** | `< pivot | == pivot | unknown | > pivot`; don't advance `mid` after swapping with `high` |
| **Intervals** | Merge: sort by start, `start <= last_end` (56: touching merges). Max kept: sort by **end**, `start >= last_end` (435: touching is fine) |
| **k-th** | Heap O(n log k); quickselect expected O(n); counting O(n + R) |
| **Top 3 bugs** | Fixed pivot on sorted input; `<` vs `<=` at interval boundaries; k-th distinct vs k-th position |
| **CPython** | `sorted` beats every Python-level sort; quickselect measured slower than sorting |
