# Chapter 8: Standard Library for DSA

## 8.0 At a glance

Six standard-library modules do most of the heavy lifting in array and string
solutions: `collections`, `heapq`, `bisect`, `itertools`, `functools` and `math`. This
chapter lists the parts that matter, with signatures, costs, tiers and minimum
versions. Every version note was checked against the module's documentation. Every
example runs on Python 3.13.2.

Version notes you'll actually hit:

| Feature | Minimum |
|---|---|
| `bisect` functions' `key=` | 3.10 |
| `itertools.pairwise` | 3.10 |
| `Counter.total()`, Counter `<=`/`>=` comparisons, zero-count equality | 3.10 |
| `math.lcm`, `math.gcd` with more than two arguments, `functools.cache` | 3.9 |
| `itertools.accumulate(initial=)`, `math.isqrt`, `math.comb`, `math.perm`, `math.prod` | 3.8 |
| `itertools.batched` | 3.12 (`strict=` 3.13) |
| `heapq.heappush_max` and the other public max-heap functions | **3.14** |

Notation: n = number of items, k = a parameter (top-k, window size).

## 8.1 Complete reference

### `collections`

| Tool | Key operations | Time (reason) | Tier | Min version |
|---|---|---|---|---|
| `Counter(it)` | `c[x]` (0 if missing), `most_common(k)`, `elements()`, `total()`, `+ - & \|`, `==`, `<=` | Build O(n); `c[x]` O(1) average; `most_common(k)` O(n log k), or O(n log n) with no k | **[KNOW-HOW]** | `total`, comparisons 3.10 |
| `defaultdict(factory)` | `d[k]` creates `factory()` if missing | O(1) average per access | **[USE-FREELY]** | |
| `deque(it, maxlen=None)` | `append`, `appendleft`, `pop`, `popleft`, `rotate(k)`, `maxlen` | O(1) at both ends; O(n) indexing in the middle (per the docs); `rotate(k)` O(k) | **[USE-FREELY]** | `index`/`insert` 3.5 |
| `OrderedDict` | `move_to_end(key, last=True)`, `popitem(last=True)` | O(1) each | **[USE-FREELY]** | |
| `namedtuple(name, fields)` | `p.x`, `p[0]`, unpacking | O(1) field access | **[USE-FREELY]** | |

### `heapq` (min-heap on a plain list)

| Function | Effect | Time (reason) | Tier | Min version |
|---|---|---|---|---|
| `heapify(a)` | Rearranges `a` into a heap, in place | O(n): the docs say "in linear time" | **[KNOW-HOW]** | |
| `heappush(h, x)` | Adds `x` | O(log n): sift up | **[KNOW-HOW]** | |
| `heappop(h)` | Removes and returns the smallest | O(log n): sift down | **[KNOW-HOW]** | |
| `heappushpop(h, x)` | Push, then pop, in one step | O(log n) | **[KNOW-HOW]** | |
| `heapreplace(h, x)` | Pop, then push | O(log n) | **[KNOW-HOW]** | |
| `h[0]` | Peek at the smallest | O(1) | **[KNOW-HOW]** | |
| `nlargest(k, it, key=None)` / `nsmallest` | Top k | O(n log k) | **[KNOW-HOW]** | |
| `merge(*sorted_its, key=None, reverse=False)` | Lazy k-way merge | O(n log k) for k inputs | **[KNOW-HOW]** | `key`/`reverse` 3.5 |
| `heapify_max`, `heappush_max`, `heappop_max`, `heappushpop_max`, `heapreplace_max` | Max-heap versions | Same as the min versions | **[KNOW-HOW]** | **3.14** |

### `bisect` (binary search on a sorted list)

| Function | Returns | Time (reason) | Tier | Min version |
|---|---|---|---|---|
| `bisect_left(a, x, lo=0, hi=len(a), *, key=None)` | First index `i` with `a[i] >= x` | O(log n): halves the range each step | **[KNOW-HOW]** | `key` 3.10 |
| `bisect_right(a, x, ...)` (alias `bisect`) | First index `i` with `a[i] > x` | O(log n) | **[KNOW-HOW]** | `key` 3.10 |
| `insort_left(a, x, ...)` / `insort_right` (alias `insort`) | `None`; inserts `x` keeping order | **O(n)**: O(log n) search, then an O(n) `list.insert` (per the docs) | **[KNOW-HOW]** | `key` 3.10 |

### `itertools`

All return lazy iterators.

| Function | Produces | Time / count | Tier | Min version |
|---|---|---|---|---|
| `accumulate(it, func=operator.add, *, initial=None)` | Running totals (prefix sums) | O(n) | **[KNOW-HOW]** | `initial` 3.8 |
| `pairwise(it)` | `(a0, a1), (a1, a2), ...` | O(n) | **[USE-FREELY]** | 3.10 |
| `batched(it, n, *, strict=False)` | Tuples of n items (last may be short) | O(len) | **[USE-FREELY]** | 3.12; `strict` 3.13 |
| `groupby(it, key=None)` | `(key, group_iterator)` for each **run** of equal keys | O(n) | **[KNOW-HOW]** | |
| `chain(*its)`, `chain.from_iterable(its)` | Concatenation | O(total) | **[USE-FREELY]** | |
| `islice(it, stop)` / `islice(it, start, stop[, step])` | Lazy slice | O(stop) | **[USE-FREELY]** | |
| `zip_longest(*its, fillvalue=None)` | Like `zip`, padded to the longest | O(longest) | **[USE-FREELY]** | |
| `product(*its, repeat=1)` | Cartesian product | Count = product of the lengths | **[KNOW-HOW]** | |
| `permutations(it, r=None)` | r-length orderings | Count = n! / (n − r)! | **[KNOW-HOW]** | |
| `combinations(it, r)` | r-length subsets, in input order | Count = C(n, r) | **[KNOW-HOW]** | |
| `combinations_with_replacement(it, r)` | Multisets of size r | Count = C(n + r − 1, r) | **[KNOW-HOW]** | |
| `count(start=0, step=1)`, `cycle(it)`, `repeat(x, times)` | Infinite or repeated streams | O(1) per item | **[USE-FREELY]** | |

### `functools`

| Tool | Purpose | Tier | Min version |
|---|---|---|---|
| `@cache` | Unbounded memoization (same as `lru_cache(maxsize=None)`, lighter) | **[KNOW-HOW]** | 3.9 |
| `@lru_cache(maxsize=128)` | Bounded memoization; usable bare as `@lru_cache` | **[KNOW-HOW]** | bare form 3.8 |
| `cmp_to_key(cmp)` | Turns a comparator into a sort key (Ch 6) | **[KNOW-HOW]** | |
| `reduce(f, it[, initial])` | Folds a sequence into one value | **[USE-FREELY]** | |

### `math`

| Function | Purpose | Time | Tier | Min version |
|---|---|---|---|---|
| `gcd(*ints)` | Greatest common divisor | O(log min) per pair (Euclid) | **[KNOW-HOW]** | 3.5; >2 args 3.9 |
| `lcm(*ints)` | Least common multiple | Same | **[KNOW-HOW]** | 3.9 |
| `isqrt(n)` | Exact ⌊√n⌋ for any int | Exact for big ints | **[KNOW-HOW]** | 3.8 |
| `comb(n, k)`, `perm(n, k)` | C(n, k), P(n, k) | | **[KNOW-HOW]** | 3.8 |
| `prod(it, start=1)` | Product | O(n) | **[USE-FREELY]** | 3.8 |
| `ceil`, `floor` | Round toward ±∞ (return `int`) | O(1) | **[USE-FREELY]** | |
| `sqrt`, `log2`, `log` | Float results | O(1) | **[USE-FREELY]** | `log2` 3.3 |
| `inf`, `nan` | Float constants | | **[USE-FREELY]** | `inf` 3.5 |

### Third-party: `sortedcontainers.SortedList`

`SortedList` is a sorted list with O(log n) add and remove and indexable positions, in
practice (it isn't a balanced tree; its costs are amortized over an internal list of
lists). **It is not in the standard library.** LeetCode's Python environment has been
widely reported to include it, but this book couldn't verify that against LeetCode's
documentation (the page refused automated access). **Never assume it in an interview:**
have a `heapq`- or `bisect`-based plan.

## 8.2 Explanations by group

### 8.2.1 `Counter`

```python
from collections import Counter

c = Counter("abracadabra")
c.most_common(2)  # -> [('a', 5), ('b', 2)]
c["z"]  # -> 0
"z" in c  # -> False
c.total()  # -> 11
```

Reading a missing key returns 0 **without inserting it**. `most_common` breaks count ties
by first-insertion order: `'b'` (count 2) appears before `'r'` (count 2) in the input.

**Equality is the anagram test.** Since 3.10, missing and zero counts compare equal:

```python
from collections import Counter

Counter("listen") == Counter("silent")  # -> True
Counter(a=1) == Counter(a=1, b=0)  # -> True
Counter("ab") <= Counter("aabb")  # -> True
```

`<=` (3.10+) means "every count on the left is ≤ the count on the right": the multiset
subset test, which is exactly 383 Ransom Note (Ch 14).

**Arithmetic drops non-positive counts; `subtract` keeps them:**

```python
from collections import Counter

Counter(a=3, b=1) - Counter(a=1, b=2)  # -> Counter({'a': 2})
d = Counter(a=3, b=1)
d.subtract(Counter(a=1, b=2))
d  # -> Counter({'a': 2, 'b': -1})
Counter("aab") & Counter("abb")  # -> Counter({'a': 1, 'b': 1})
Counter("aab") | Counter("abb")  # -> Counter({'a': 2, 'b': 2})
```

`&` takes the minimum of each count, `|` the maximum.

**Tier: [KNOW-HOW].** The hand-written version is a dict loop, or a 26-int list for
lowercase letters (Ch 14, Ch 24). Interviewers often accept `Counter`, but expect you to
know the cost: building one is O(n) time and O(k) space for k distinct keys.

### 8.2.2 `defaultdict`

```python
from collections import defaultdict

groups = defaultdict(list)
for word in ["ab", "c", "de"]:
    groups[len(word)].append(word)
dict(groups)  # -> {2: ['ab', 'de'], 1: ['c']}
```

**Reading a missing key inserts it.** That's the point when you're appending, and a bug
when you're only checking:

```python
from collections import defaultdict

seen = defaultdict(int)
seen["x"]  # -> 0
len(seen)  # -> 1
```

Check membership with `in`, or read with `.get(key)`. Neither inserts.

### 8.2.3 `deque`

A double-ended queue with O(1) operations at **both** ends. It's the right structure for
BFS queues and sliding windows. Ch 6 measured `list.pop(0)` at 15.6 s for 10⁵ items;
`popleft` is O(1).

```python
from collections import deque

q = deque([1, 2, 3])
q.append(4)
q.popleft()  # -> 1
q.rotate(1)
q  # -> deque([4, 2, 3])
window = deque(maxlen=3)
for i in range(5):
    window.append(i)
window  # -> deque([2, 3, 4], maxlen=3)
```

With `maxlen`, appending to a full deque silently drops the item at the other end.
Indexing the **middle** of a deque is O(n) (per the docs), so use a list for random access.

### 8.2.4 `OrderedDict` and `namedtuple`

Plain dicts preserve insertion order (guaranteed since 3.7, Ch 7). `OrderedDict` adds
two O(1) operations that dicts lack, and together they make an LRU cache:

```python
from collections import OrderedDict

od = OrderedDict.fromkeys("abc")
od.move_to_end("a")
list(od)  # -> ['b', 'c', 'a']
od.popitem(last=False)  # -> ('b', None)
```

`move_to_end(key)` marks a key as most recently used; `popitem(last=False)` evicts the
least recently used. `code/ch08_stdlib/exercises.py` has a complete LRU class (Exercise
19).

`namedtuple` gives tuple fields names: readable, immutable and hashable.

```python
from collections import namedtuple

Point = namedtuple("Point", "x y")
p = Point(1, 2)
(p.x, p[1], p == (1, 2))  # -> (1, 2, True)
```

### 8.2.5 `heapq`

A heap is a list arranged so that `h[0]` is always the smallest item. `heapq` provides
**only a min-heap** before 3.14.

```python
import heapq

h = [5, 1, 9, 3]
heapq.heapify(h)
h[0]  # -> 1
heapq.heappush(h, 0)
heapq.heappop(h)  # -> 0
heapq.heappop(h)  # -> 1
```

**Max-heap by negation** (works on every version): push `-x`, and negate what you pop.

```python
import heapq

heap = []
for x in [5, 1, 9, 3, 7]:
    heapq.heappush(heap, -x)
[-heapq.heappop(heap) for _ in range(3)]  # -> [9, 7, 5]
```

Python 3.14 added public `heapify_max`, `heappush_max`, `heappop_max`,
`heappushpop_max` and `heapreplace_max`. Use them only if the judge's Python is 3.14+.
Negation works everywhere and is what interviewers expect to see.

**Tuples as entries** order by the first field, then the second, and so on. When two
priorities tie, Python compares the next field, and that fails for types that don't
support `<`:

```python
import heapq

h = []
heapq.heappush(h, (1, {"a": 1}))
heapq.heappush(h, (1, {"b": 2}))  # -> TypeError: '<' not supported between...
```

Fix: insert a unique, comparable tie-breaker, such as a counter, as the second field:
`(priority, order, item)`.

**`nlargest` vs sorting.** The docs: `nlargest`/`nsmallest` "perform best for smaller
values of n. For larger values, it is more efficient to use the sorted() function", and
for one item, use `min`/`max`. Measured (`bench/ch08_nlargest_vs_sorted.py`, 10⁶ random
floats, CPython 3.13.2, two runs):

| k | `heapq.nlargest(k, data)` | `sorted(data, reverse=True)[:k]` |
|---|---|---|
| 1 | 9–20 ms (`max`: 9–18 ms) | 230–403 ms |
| 10 | 17–33 ms | 222–384 ms |
| 1,000 | 20–39 ms | 228–380 ms |
| 100,000 | 573–840 ms | 364–717 ms (runs disagreed: crossover) |
| 500,000 | 2,955–3,370 ms | 393–402 ms |

In that benchmark, for small k the heap was about 10× faster; at k = n/2, sorting was
about 8× faster.
The complexities explain it: O(n log k) vs O(n log n), but the heap's per-item work is
heavier, so it only wins while log k is much smaller than log n.

**[KNOW-HOW]:** know that a heap is a complete binary tree stored in an array (children
of index i at 2i + 1 and 2i + 2), and why push and pop are O(log n). Writing a full heap
by hand is beyond this book (Ch 9).

### 8.2.6 `bisect`

```python
from bisect import bisect_left, bisect_right

a = [1, 2, 2, 2, 3]
bisect_left(a, 2)  # -> 1
bisect_right(a, 2)  # -> 4
bisect_right(a, 2) - bisect_left(a, 2)  # -> 3
bisect_left(a, 5)  # -> 5
```

| Question on a sorted list | Expression |
|---|---|
| Insert position keeping order (before equal items) | `bisect_left(a, x)` |
| How many items are < x | `bisect_left(a, x)` |
| How many items are ≤ x | `bisect_right(a, x)` |
| Count of x | `bisect_right(a, x) - bisect_left(a, x)` |
| Is x present? | `i = bisect_left(a, x)`, then `i < len(a) and a[i] == x` |

`key=` (3.10+) applies to the list's items, **not** to `x`:

```python
from bisect import bisect_left

pairs = [("x", 1), ("y", 4), ("z", 9)]
bisect_left(pairs, 4, key=lambda p: p[1])  # -> 1
```

**`insort` is O(n).** The search is O(log n), but `list.insert` shifts elements. A loop of
n `insort` calls is O(n²) in the worst case. Ch 19 writes `bisect_left` and `bisect_right`
by hand: they are the "lower bound" and "upper bound" templates.

### 8.2.7 `itertools`

```python
import itertools

runs = [(k, len(list(g))) for k, g in itertools.groupby("aaabbcaa")]
runs  # -> [('a', 3), ('b', 2), ('c', 1), ('a', 2)]
```

**`groupby` groups consecutive runs**, not all equal items. The docs: "Generally, the
iterable needs to already be sorted on the same key function." Here `a` appears twice,
because the input wasn't sorted. That's a feature for run-length problems (38 Count and
Say, Ch 28) and a bug for "group all equal items" (sort first, or use a dict).

```python
import itertools

list(itertools.pairwise([1, 4, 9]))  # -> [(1, 4), (4, 9)]
list(itertools.accumulate([1, 2, 3], initial=0))  # -> [0, 1, 3, 6]
list(itertools.batched("abcdefg", 3))  # -> [('a', 'b', 'c'), ('d', 'e', 'f'), ('g',)]
list(itertools.islice(itertools.count(10, 5), 3))  # -> [10, 15, 20]
list(itertools.zip_longest("ab", "x", fillvalue="-"))  # -> [('a', 'x'), ('b', '-')]
```

`accumulate(..., initial=0)` is the prefix-sum array **with** the leading 0 that makes
range sums `prefix[j] - prefix[i]` work without special cases (Ch 15).

**Combinatorics.** The outputs grow fast, so check the count against the constraints
before generating anything:

```python
import itertools
import math

pairs = list(itertools.product("ab", repeat=2))
pairs  # -> [('a', 'a'), ('a', 'b'), ('b', 'a'), ('b', 'b')]
list(itertools.combinations("abc", 2))  # -> [('a', 'b'), ('a', 'c'), ('b', 'c')]
multisets = list(itertools.combinations_with_replacement("ab", 2))
multisets  # -> [('a', 'a'), ('a', 'b'), ('b', 'b')]
(math.comb(5, 2), math.perm(5, 2))  # -> (10, 20)
```

`permutations` of 10 items is 3,628,800 tuples, and of 12 items 479,001,600. That's why
n ≤ 10 is the permutation limit in Ch 1's constraint table.

### 8.2.8 `functools`

**Memoization** stores results by argument, turning exponential recursion into
polynomial:

```python
from functools import cache


@cache
def fib(n: int) -> int:
    return n if n < 2 else fib(n - 1) + fib(n - 2)


fib(100)  # -> 354224848179261915075
```

Arguments must be hashable (no lists; pass tuples). Don't decorate a `Solution` method
itself: the cache lives on the class for the whole process, keyed by `self` and the
arguments. If the judge reuses one instance and the method reads per-test data from
`self` (say `self.nums`), it returns stale answers from an earlier test case; in any
case the cache grows across test cases. Define the cached function **inside** the
method, so each call gets a fresh cache.
DP is beyond this book; Ch 20 only touches the idea.

`reduce` folds a sequence: `reduce(lambda a, b: a * b, [1, 2, 3, 4])` is 24, but
`math.prod` says it more clearly.

### 8.2.9 `math`

```python
import math

x = (10**16 + 1) ** 2
math.isqrt(x) == 10**16 + 1  # -> True
int(math.sqrt(x)) == 10**16 + 1  # -> False
```

`math.sqrt` converts to a float (53 bits of precision) and loses the last digits.
`math.isqrt` works on exact integers. Use `isqrt` for perfect-square checks and for
loops up to √n.

```python
import math

(math.gcd(12, 18, 24), math.lcm(4, 6), math.prod([1, 2, 3, 4]))  # -> (6, 12, 24)
(math.ceil(7 / 2), -(-7 // 2))  # -> (4, 4)
```

`-(-a // b)` is ceiling division in pure integers. It avoids float conversion, which
matters for huge values.

## 8.3 Gotchas & traps

**1. `defaultdict` read inserts the key** (8.2.2).

**2. `Counter` `-` drops zero and negative counts**; `subtract` keeps them (8.2.1).

**3. Counter equality with zero counts differed before 3.10** (8.2.1).

**4. `heapq` is a min-heap.** Negate for a max-heap; the max functions need 3.14 (8.2.5).

**5. Heap entries that tie on priority compare the next field** (8.2.5).

**6. `insort` is O(n), not O(log n)** (8.2.6).

**7. `bisect` `key=` applies to list items, not the searched value** (8.2.6).

**8. `groupby` groups runs, not equal keys overall** (8.2.7).

**9. Exploding combinatorics.** Check the count first (8.2.7).

**10. `@cache` on a `Solution` method** outlives the test case: stale results and memory growth (8.2.8).

**11. `int(math.sqrt(n))` for big n.** Use `math.isqrt` (8.2.9).

**12. Assuming `sortedcontainers` exists.** Not stdlib; verify per platform (8.1).

## 8.4 Benchmarks

| File | Measures | Result (CPython 3.13.2) |
|---|---|---|
| `bench/ch08_nlargest_vs_sorted.py` | `nlargest(k)` vs `sorted()[:k]`, N = 10⁶ | Heap ~10× faster for k ≤ 1,000; sorting ~8× faster at k = 500,000 |

`deque.popleft` vs `list.pop(0)` is measured in Ch 6 (front of a list) and again in
Ch 11.

## 8.5 Exercises

Answers are in Appendix D (verified in `code/ch08_stdlib/exercises.py`).

**Warm-up**

1. `Counter("abracadabra").most_common(2)`?
2. For that Counter, what are `c["z"]` and `"z" in c` afterwards?
3. Test whether `"listen"` and `"silent"` are anagrams in one expression.
4. What do `Counter(a=3, b=1) - Counter(a=1, b=2)` and the `subtract` version give?
5. Group `["ab", "c", "de"]` by length. Then explain what `groups[99]` does to the dict.

**Core**

6. Use a deque as a FIFO queue, then rotate it right by one.
7. Keep only the last 3 items of a stream automatically.
8. Return the 3 largest values with `heapq` and negation.
9. Push `(1, {"a": 1})` and `(1, {"b": 2})` onto a heap. What happens? Fix it.
10. Count the occurrences of `x` in a sorted list in O(log n).
11. In `[("x", 1), ("y", 4), ("z", 9)]` (sorted by the number), find the insert position
    for 4 using `bisect` with `key=`.
12. Run-length encode `"aaabbcaa"` with `groupby`.
13. Adjacent differences of `[1, 4, 9]` with `pairwise`.
14. Prefix sums of `[1, 2, 3]` with a leading 0.
15. Check that `combinations(range(5), 2)` yields `math.comb(5, 2)` tuples.
16. Compute `fib(100)` with memoization.

**Tricky**

17. For `x = (10**16 + 1)**2`, compare `math.isqrt(x)` with `int(math.sqrt(x))`.
18. `gcd` of 12, 18 and 24 in one call; `lcm` of 4 and 6.
19. Build an LRU cache with `get` and `put` using `OrderedDict`.
20. Create a `Point` namedtuple and show three ways to read `x`.

## 8.6 Chapter cheat sheet

| Need | Tool | Cost / note |
|---|---|---|
| Frequencies | `Counter(it)` | O(n); `c[x]` 0 if missing; `==` anagram; `<=` subset (3.10+) |
| Grouping | `defaultdict(list)` | Reads insert keys |
| Queue / window | `deque`, `popleft`, `maxlen` | O(1) ends; O(n) middle |
| LRU | `OrderedDict.move_to_end`, `popitem(last=False)` | O(1) |
| Min-heap | `heapify` O(n), push/pop O(log n), `h[0]` peek | Max-heap: negate (3.14 adds `_max` functions) |
| Top-k | `nlargest(k, ...)` for small k; `sorted` for large k; `max` for k = 1 | Measured crossover near k = 10⁵ for n = 10⁶ |
| Sorted search | `bisect_left` (< x count), `bisect_right` (≤ x count) | O(log n); `insort` O(n); `key=` 3.10+ |
| Runs | `groupby` | Consecutive runs only |
| Prefix sums | `accumulate(a, initial=0)` | `initial` 3.8+ |
| Neighbors | `pairwise` | 3.10+ |
| Memoize | `@cache` (3.9+) inside the method | Hashable arguments |
| Exact int math | `isqrt`, `gcd`, `lcm` (3.9+), `comb`, `prod` | `isqrt` exact; `sqrt` is a float |
