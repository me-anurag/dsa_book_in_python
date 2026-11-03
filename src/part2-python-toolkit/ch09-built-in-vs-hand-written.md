# Chapter 9: Built-in vs Hand-Written: The Tier Map

## 9.0 At a glance

LeetCode's judge accepts any correct code that runs in time. An interviewer grades
something different: whether **you** can produce the algorithm. This chapter is the map
between the two, using the book's three tiers:

| Tier | Judge | Interviewer |
|---|---|---|
| **[USE-FREELY]** | Fine | Fine. Nobody wants `len` reimplemented |
| **[KNOW-HOW]** | Fine | Usually fine, **until** they say "without built-ins", or the problem forbids it. Then you must write it, and state its complexity either way |
| **[HAND-WRITE]** | No built-in exists, or the built-in skips the point | This *is* the question |

The chapter gives:
- the master table of every common operation and its tier
- hand-written versions of each **[KNOW-HOW]** tool, side by side, tested against the
  built-ins on random inputs in `code/ch09_tier_map/hand_written.py`
- the catalog of **[HAND-WRITE]** techniques, and where each is taught
- the phrases in a problem statement that take built-ins away

## 9.1 Complete reference: the tier map

| Operation | Built-in | Tier | Hand-written equivalent (where) |
|---|---|---|---|
| Length, indexing, slicing, iteration | `len`, `a[i]`, `a[i:j]`, `for` | **[USE-FREELY]** | n/a |
| Index + value, pairs, ranges | `enumerate`, `zip`, `range` | **[USE-FREELY]** | n/a |
| Hash map / hash set | `dict`, `set` | **[USE-FREELY]** | n/a (hash table internals are out of scope) |
| Grouping, queue | `defaultdict`, `deque` | **[USE-FREELY]** | n/a |
| Sum, any, all | `sum`, `any`, `all` | **[USE-FREELY]** | n/a |
| Build a string | `"".join(parts)` | **[USE-FREELY]** | n/a |
| Character codes | `ord`, `chr` | **[USE-FREELY]** | n/a |
| Reverse | `a.reverse()`, `a[::-1]`, `reversed` | **[KNOW-HOW]** | Two-pointer swap (9.2.1, Ch 16) |
| Sort | `sorted`, `list.sort` | **[KNOW-HOW]** | Merge sort, quicksort, counting sort (9.2.2, Ch 18) |
| Custom order | `cmp_to_key` | **[KNOW-HOW]** | Comparator inside a merge sort (Ch 18) |
| Count items | `Counter` | **[KNOW-HOW]** | Dict loop, or a 26-slot list (9.2.3, Ch 14) |
| Binary search | `bisect_left`, `bisect_right` | **[KNOW-HOW]** | Lower/upper bound loops (9.2.4, Ch 19) |
| Prefix sums | `itertools.accumulate` | **[KNOW-HOW]** | Running-total loop (9.2.5, Ch 15) |
| Substring search | `str.find`, `in`, `index`, `count` | **[KNOW-HOW]** | Brute force (9.2.6); KMP, Z, Rabin–Karp (Ch 29) |
| Tokenize / split | `split`, `partition` | **[KNOW-HOW]** | Character scan (9.2.7, Ch 24) |
| Min / max | `min`, `max` | **[KNOW-HOW]** | One-pass scan (9.2.8) |
| Top k, priority queue | `heapq` | **[KNOW-HOW]** | Describe the heap (9.2.9); full implementation out of scope |
| Parse an integer | `int(s)` | **[KNOW-HOW]** | `value = value * 10 + digit` (Ch 28) |
| Modular power | `pow(b, e, m)` | **[KNOW-HOW]** | Square-and-multiply |
| Two pointers | n/a | **[HAND-WRITE]** | Ch 16 |
| Sliding window | n/a | **[HAND-WRITE]** | Ch 17, Ch 27 |
| Kadane | n/a | **[HAND-WRITE]** | Ch 20 |
| Monotonic stack / deque | n/a | **[HAND-WRITE]** | Ch 21 |
| Dutch national flag (3-way partition) | n/a | **[HAND-WRITE]** | Ch 18 (75 Sort Colors) |
| Cyclic sort, index-as-hash | n/a | **[HAND-WRITE]** | Ch 23 |
| Boyer–Moore majority vote | n/a | **[HAND-WRITE]** | Ch 23 |
| In-place rotation (reversal trick) | n/a | **[HAND-WRITE]** | Ch 13 |
| Matrix spiral / rotate | n/a | **[HAND-WRITE]** | Ch 22 |
| Expand around center | n/a | **[HAND-WRITE]** | Ch 26 |
| KMP / Z-function / Rabin–Karp | n/a | **[HAND-WRITE]** | Ch 29 |
| String arithmetic (add, multiply) | `int()` forbidden | **[HAND-WRITE]** | Ch 28 |
| atoi-style parsing | `int()` too permissive (Ch 7) | **[HAND-WRITE]** | Ch 28 |

## 9.2 Explanations by group

Each block below is an excerpt of `code/ch09_tier_map/hand_written.py`. That file tests
every function against the built-in it replaces: 400 random trials, with duplicates,
negatives, empty inputs, and 912's maximum size of 5 × 10⁴ for the sorts.

### 9.2.1 Reverse

<!-- file: code/ch09_tier_map/hand_written.py -->

```python
def reverse_in_place(a: list) -> None:
    """Replaces a.reverse(). O(n) time, O(1) auxiliary: n // 2 swaps."""
    left, right = 0, len(a) - 1
    while left < right:
        a[left], a[right] = a[right], a[left]
        left += 1
        right -= 1
```

This is the two-pointer skeleton of Ch 16, and the whole answer to 344 Reverse String
(Ch 1).

### 9.2.2 Sorting

Three sorts cover everything an interviewer asks for. Ch 18 proves their bounds and
uses them on 912 Sort an Array, which forbids built-in sorting and asks for
O(n log n) time.

**Merge sort:** guaranteed O(n log n), stable, O(n) extra space.

<!-- file: code/ch09_tier_map/hand_written.py -->

```python
def merge_sort(a: list[int]) -> list[int]:
    """Replaces sorted(). O(n log n) time: log n levels, O(n) merge work per level.
    O(n) auxiliary: the merged lists. Stable: ties take from the left half first."""
    if len(a) <= 1:
        return a[:]
    mid = len(a) // 2
    left, right = merge_sort(a[:mid]), merge_sort(a[mid:])
    merged = []
    i = j = 0
    while i < len(left) and j < len(right):
        if left[i] <= right[j]:            # <= keeps equal items in order: stable
            merged.append(left[i])
            i += 1
        else:
            merged.append(right[j])
            j += 1
    merged.extend(left[i:])
    merged.extend(right[j:])
    return merged
```

**Quicksort:** in place, expected O(n log n) with a random pivot, O(n²) in the worst
case. The three-way partition puts all pivot-equal items in the middle at once, so an
array of identical values takes one pass instead of degrading to O(n²).

<!-- file: code/ch09_tier_map/hand_written.py -->

```python
def _quick_sort(a: list[int], lo: int, hi: int) -> None:
    while lo < hi:
        pivot = a[random.randint(lo, hi)]
        lt, i, gt = lo, lo, hi             # a[lo:lt] < pivot, a[gt+1:hi+1] > pivot
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
        if lt - lo < hi - gt:              # smaller side first, by recursion
            _quick_sort(a, lo, lt - 1)
            lo = gt + 1
        else:
            _quick_sort(a, gt + 1, hi)
            hi = lt - 1
```

Recursing only into the **smaller** part, and looping over the larger one, keeps the
recursion depth at O(log n). Each recursive call handles at most half the current
range. Ch 1's recursion limit of 1000 frames can't be reached that way.

**Counting sort:** O(n + k) for integers in a known range of size k.

<!-- file: code/ch09_tier_map/hand_written.py -->

```python
def counting_sort(a: list[int], lo: int, hi: int) -> list[int]:
    """Sorts ints known to lie in [lo, hi]. O(n + k) time and O(k) auxiliary space,
    k = hi - lo + 1. Only worth it when k is not much larger than n."""
    counts = [0] * (hi - lo + 1)
    for x in a:
        counts[x - lo] += 1
    out = []
    for offset, c in enumerate(counts):
        out.extend([offset + lo] * c)
    return out
```

912's values lie in [−5 × 10⁴, 5 × 10⁴], so k = 100,001 against n ≤ 5 × 10⁴. That's
O(n + k), well within limits, and the exercises file checks it at full size. With
values up to 10⁹ (most problems), k is far too large, which is why Ch 1 told you to read
the **value** range in the constraints.

| Sort | Time | Auxiliary space | Stable | Use when |
|---|---|---|---|---|
| Merge sort | O(n log n) always | O(n) | Yes | You need a guarantee, or stability |
| Quicksort (random pivot, 3-way) | O(n log n) expected, O(n²) worst | O(log n) stack | No | In-place is asked for |
| Counting sort | O(n + k) | O(k) | Yes, in the variant that places items by prefix counts | Small integer range |
| `sorted` / `list.sort` (Timsort) | O(n log n), O(n) on presorted runs | O(n) | Yes | Whenever built-ins are allowed |

### 9.2.3 Counting

<!-- file: code/ch09_tier_map/hand_written.py -->

```python
def count_items(items) -> dict:
    """Replaces Counter(items). O(n) time, O(k) space for k distinct items."""
    counts: dict = {}
    for x in items:
        counts[x] = counts.get(x, 0) + 1
    return counts


def count_letters(s: str) -> list[int]:
    """26-slot alternative for lowercase ASCII letters. O(n) time, O(1) space (26)."""
    counts = [0] * 26
    for c in s:
        counts[ord(c) - ord("a")] += 1
    return counts
```

The 26-slot list is O(1) space because its size doesn't depend on n, and two of them
compare in 26 steps. Chapters 14, 24 and 25 use it as the "fixed alphabet" alternative
to `Counter`. Ch 11 measures the three approaches against each other.

### 9.2.4 Binary search (lower and upper bound)

<!-- file: code/ch09_tier_map/hand_written.py -->

```python
def lower_bound(a: list[int], x: int) -> int:
    """Replaces bisect_left: first index with a[i] >= x. O(log n)."""
    lo, hi = 0, len(a)                     # answer lies in [lo, hi]
    while lo < hi:
        mid = (lo + hi) // 2
        if a[mid] < x:
            lo = mid + 1
        else:
            hi = mid
    return lo
```

`upper_bound` is identical except that the test is `a[mid] <= x`. The invariant "the
answer is in `[lo, hi]`" and the half-open loop `while lo < hi` are what prevent
infinite loops. Ch 19 builds every binary search from this template.

### 9.2.5 Prefix sums

<!-- file: code/ch09_tier_map/hand_written.py -->

```python
def prefix_sums(a: list[int]) -> list[int]:
    """Replaces list(accumulate(a, initial=0)). O(n)."""
    out = [0]
    for x in a:
        out.append(out[-1] + x)
    return out
```

With the leading 0, the sum of `a[i:j]` is `out[j] - out[i]` for every `0 <= i <= j <= n`
(Ch 15).

### 9.2.6 Substring search

<!-- file: code/ch09_tier_map/hand_written.py -->

```python
def find_sub(s: str, sub: str) -> int:
    """Replaces s.find(sub). Brute force: O((n - m + 1) * m) time, O(1) auxiliary."""
    n, m = len(s), len(sub)
    for start in range(n - m + 1):
        k = 0
        while k < m and s[start + k] == sub[k]:
            k += 1
        if k == m:
            return start
    return -1
```

It matches `str.find` exactly, including `sub == ""` (returns 0) and `m > n` (returns
-1). The O(n · m) worst case is why Ch 29 teaches linear-time matching.

### 9.2.7 Tokenizing

<!-- file: code/ch09_tier_map/hand_written.py -->

```python
def split_whitespace(s: str) -> list[str]:
    """Replaces s.split() for ASCII whitespace. O(n)."""
    words, current = [], []
    for c in s:
        if c in " \t\n\r\x0b\x0c":
            if current:
                words.append("".join(current))
                current = []
        else:
            current.append(c)
    if current:
        words.append("".join(current))
    return words
```

It builds each word from a list of characters, not with `+=` (Ch 5, 5.2.7). The final
`if current` handles a word that runs to the end of the string, the most common bug in
hand-written tokenizers. `"".join` stays **[USE-FREELY]**: nobody asks you to rebuild
string concatenation.

### 9.2.8 Min / max

<!-- file: code/ch09_tier_map/hand_written.py -->

```python
def max_scan(a: list[int]) -> int:
    """Replaces max(a) for a non-empty list. O(n), one pass."""
    best = a[0]
    for i in range(1, len(a)):
        if a[i] > best:
            best = a[i]
    return best
```

The interview variants extend the same scan: second largest in one pass (Exercise 4),
min and max together, the running minimum in 121 Best Time to Buy and Sell Stock
(Ch 17).

### 9.2.9 Heaps: what to know without writing one

You won't usually be asked to implement a heap in an array-and-string interview. You
**will** be asked how it works and what it costs:

```text
array:  [1, 3, 2, 7, 4]          index i has children 2i + 1 and 2i + 2,
                                  and parent (i - 1) // 2
tree:          1  (i=0)
             /   \
        3 (1)     2 (2)          heap property: every parent <= its children,
        /  \                      so the minimum is always at index 0
   7 (3)   4 (4)
```

| Operation | How | Cost (reason) |
|---|---|---|
| Peek | read `h[0]` | O(1) |
| Push | append at the end, then **sift up** (swap with the parent while smaller) | O(log n): the tree's height |
| Pop | move the last item to the root, then **sift down** (swap with the smaller child) | O(log n) |
| Heapify | sift down every non-leaf, from the last one up | O(n): most nodes are near the bottom and move only a little |

### 9.2.10 Problem-statement signals that take built-ins away

All rows were checked against the problem statements (book's paraphrase):

| Signal in the statement | What it rules out | Example (verified) |
|---|---|---|
| "in-place", "O(1) extra memory" | Building a new list; `s[:] = s[::-1]` (O(n) temporary); `sorted` | 344 Reverse String; 189 Rotate Array; 283 Move Zeroes |
| "without using the division operation" | `/`, `//` on the total product | 238 Product of Array Except Self |
| "don't convert the inputs to integers" / no big-integer library | `int(num1) * int(num2)` | 43 Multiply Strings |
| "without using any built-in sorting functions" + O(n log n) | `sorted`, `list.sort` | 912 Sort an Array |
| "O(n) time and O(1) auxiliary space" | Sorting (O(n log n)), sets and dicts (O(n) space) | 41 First Missing Positive |
| "must not modify the array" + constant extra space | Sorting in place, sign-marking, sets | 287 Find the Duplicate Number |
| "Follow-up: can you do it in O(n)?" | The O(n log n) sort-based solution, as the final answer | Common on sorting-shaped problems |

When a restriction appears, solve with **[HAND-WRITE]** techniques and say out loud which
built-in you're avoiding, and why.

### 9.2.11 What interviewers actually expect

- **State costs even when you use a built-in.** "I'll use `sorted`, which is O(n log n)
  time and O(n) space" earns credit. A silent `sorted` invites the follow-up "how does
  that work?".
- **The core idea is never a built-in.** If one call to a library function solves the
  problem, it isn't the intended solution. Expect "now do it without that".
- **Ask before using heavy tools.** "Is `Counter` OK, or would you like the dict
  version?" takes five seconds and removes the ambiguity.
- **On LeetCode, use whatever passes.** But practice the hand-written versions: the
  re-solve schedule (front matter, 0.4) is where they become automatic.

## 9.3 Gotchas & traps

**1. Using the forbidden tool anyway.** `sorted` in 912, `int()` in 43. The judge may even
accept it; the interviewer won't (9.2.10).

**2. "In-place" solved with a hidden copy.** `s[:] = s[::-1]` mutates correctly, but uses
O(n) extra space (Ch 1).

**3. Quicksort with a fixed pivot.** On sorted input, `a[hi]` as the pivot gives O(n²)
time and O(n) recursion depth, which hits the recursion limit. Randomize the pivot, and
recurse into the smaller side (9.2.2).

**4. Two-way partition with many duplicates.** Without the equal band, an array of
identical values is quadratic. Use three-way partitioning (9.2.2).

**5. Counting sort over a huge value range.** k = 2 × 10⁹ slots is not O(n) in any useful
sense (9.2.2).

**6. Hand-written tokenizer dropping the last word.** Flush `current` after the loop
(9.2.7).

**7. Saying "binary search" without the invariant.** Interviewers ask for the loop
condition and the update rules (9.2.4, Ch 19).

## 9.4 Benchmarks

None in this chapter. Hand-written merge sort vs `sorted`, and `Counter` vs a dict vs a
26-slot list, are measured in Ch 11.

## 9.5 Exercises

Answers are in Appendix D (code answers verified in `code/ch09_tier_map/exercises.py`).

**Warm-up**

1. Tag each with its tier: `len`, `sorted`, two pointers, `Counter`, `set`,
   `bisect_left`, Kadane, `str.find`, `enumerate`, monotonic stack.
2. For each phrase, name what's off limits: "in-place with O(1) extra memory"; "without
   division"; "don't convert the inputs to integers"; "no built-in sort, O(n log n)";
   "O(n) time and O(1) space"; "don't modify the array, constant extra space".

**Core**

3. Convert a string of decimal digits to an int without `int()`.
4. Return the largest value and the largest value strictly smaller than it, in one
   pass (`None` if there isn't one).
5. Count the letters of a lowercase string in a 26-slot list, and check the result
   against `Counter`.
6. Why does the quicksort above recurse into the smaller part and loop over the larger
   one?

**Tricky**

7. 912 forbids built-in sorting. Is counting sort a valid answer for its constraints?
   Give its time and space.
8. An interviewer lets you use `sorted`. What do you say about its cost?

## 9.6 Chapter cheat sheet

| Tier | Rule | Examples |
|---|---|---|
| **[USE-FREELY]** | Use anywhere | `len`, `enumerate`, `zip`, `dict`, `set`, `deque`, `sum`, `"".join` |
| **[KNOW-HOW]** | Use, state the cost, be able to write it | `sorted`, `reversed`, `Counter`, `bisect`, `accumulate`, `heapq`, `str.find`, `split`, `min`/`max`, `int()` |
| **[HAND-WRITE]** | It's the question | Two pointers, sliding window, Kadane, monotonic stack, Dutch flag, cyclic sort, Boyer–Moore, rotation, spiral, expand-around-center, KMP/Z/Rabin–Karp, string arithmetic, atoi |
| Signals | "in-place", "O(1) extra", "without division", "don't convert to integer", "no built-in sort", "don't modify" | 344, 189, 283, 238, 43, 912, 41, 287 |
| Sorts | Merge (stable, O(n) space), quick (random pivot, 3-way, smaller side first), counting (small range) | Ch 18 |
