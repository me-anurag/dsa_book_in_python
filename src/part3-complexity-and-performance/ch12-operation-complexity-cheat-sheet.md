# Chapter 12: Operation Complexity Cheat Sheet

## 12.0 At a glance

One place for the cost of every common operation, with the **reason** for each. It's
designed to be printed: the tables are self-contained, and the text below them is
optional.

Sources: the CPython documentation page "Time complexity of built-in types" (for
`list`, `dict`, `set`, `str`, `tuple`, `range`); the `collections`, `heapq` and
`bisect` docs (Ch 8); and this book's measurements (`bench/RESULTS.md`). These are
**CPython** costs; other implementations may differ.

Conventions: n = current size of the container; k = a parameter (index, count, slice
length); "avg" = average case for hash-based types, assuming well-distributed hashes;
"amortized" = averaged over a sequence of operations (Ch 10).

## 12.1 Complete reference

### `list`

| Operation | Average | Worst | Reason |
|---|---|---|---|
| `a[i]`, `a[i] = x`, `len(a)` | O(1) | O(1) | Pointer arithmetic; length is stored |
| `a.append(x)` | O(1) amortized | O(n) for one resizing call | Spare capacity; geometric growth (Ch 6) |
| `a.pop()` | O(1) | O(1) | Last slot; nothing moves |
| `a.pop(i)`, `del a[i]`, `a.insert(i, x)` | O(n − i) | O(n) at i = 0 | Shifts everything after i |
| `x in a`, `a.index(x)`, `a.count(x)`, `a.remove(x)` | O(n) | O(n) | Linear scan with `==` |
| `a[i:j]` | O(j − i) | | Copies j − i references |
| `a[i:j] = t` | O(len(t)) when lengths match; else O(n − i + len(t)) | | Shifts the tail when the size changes |
| `a + b` | O(len(a) + len(b)) | | New list |
| `a += t`, `a.extend(t)` | O(len(t)) amortized | | Appends each item |
| `a * k` | O(n · k) | | Repeats references |
| `a.sort()`, `sorted(a)` | O(n log n) | O(n log n) | Timsort; O(n) on already-sorted runs |
| `a.reverse()` | O(n) | | n/2 swaps |
| `min`, `max`, `sum` | O(n) | | One pass |
| `a.copy()`, `list(a)`, `a[:]` | O(n) | | Shallow copy |
| `a == b`, `a < b` | O(min(n, m)) | | Stops at the first difference |

### `str` (immutable)

| Operation | Average | Worst | Reason |
|---|---|---|---|
| `s[i]`, `len(s)` | O(1) | O(1) | Fixed-width internal storage; length stored |
| `s[i:j]` | O(j − i) | | New string |
| `s + t` | O(n + m) | | New string; **in a loop this is quadratic** |
| `"".join(parts)` | O(total length) | | Sizes once, copies once |
| `sub in s`, `s.find(sub)`, `s.index(sub)`, `s.count(sub)` | O(n) | O(n + m) | Two-way forward search (docs; measured flat, 12.2.3) |
| `s.rfind(sub)`, `s.rindex(sub)` | O(n) | O(n · m) | Per the docs |
| `s == t`, `s < t` | O(min(n, m)) | | Character by character |
| `s.split()`, `s.strip()`, `s.lower()`, `s.replace(a, b)` | O(n) | | One pass, new string(s) |
| `s[::-1]`, `sorted(s)` | O(n), O(n log n) | | `sorted` returns a **list** |
| `hash(s)` | O(n) | | Hashes every character |

### `dict` (and `defaultdict`, `Counter`, `OrderedDict`)

| Operation | Average | Worst | Reason |
|---|---|---|---|
| `d[k]`, `d.get(k)`, `k in d` | O(1) | O(n) | Hash table; worst case when all keys collide |
| `d[k] = v` | O(1) amortized | O(n) | Occasional resize; collisions |
| `del d[k]`, `d.pop(k)` | O(1) | O(n) | Same |
| Iterate keys / values / items | O(n) | | Visits each entry |
| `d.copy()`, `dict(d)` | O(n) | | |
| `d.update(t)`, `d \|= t` (3.9+) | O(len(t)) | | One insert per item |
| `len(d)` | O(1) | | Stored |
| `Counter(it)` | O(len(it)) | | One counting pass |
| `c.most_common(k)` | O(n log k) | | Heap of size k (O(n log n) with no k) |
| `OrderedDict.move_to_end`, `popitem(last=False)` | O(1) | | Linked ordering |

### `set` (and `frozenset`)

| Operation | Average | Worst | Reason |
|---|---|---|---|
| `x in s`, `s.add(x)`, `s.discard(x)`, `s.remove(x)` | O(1) | O(n) | Hash table |
| `s \| t` (union) | O(len(s) + len(t)) | | Copies both |
| `s & t` (intersection) | O(min(len(s), len(t))) | O(len(s) · len(t)) | Iterates over the smaller one, looking up in the larger |
| `s - t` (difference) | O(len(s)) | | Checks each element of s |
| `s ^ t` (symmetric difference) | O(len(s) + len(t)) | | |
| `s \|= t`, `s.update(t)` | O(len(t)) | | |
| `s <= t` (subset) | O(len(s)) | | Looks up each element of s |
| `set(it)` | O(len(it)) | | |
| `len(s)` | O(1) | | Stored |

### `tuple` and `range`

| Operation | Cost | Reason |
|---|---|---|
| `t[i]`, `len(t)` | O(1) | |
| `x in t` | O(n) | Linear scan |
| `t[i:j]` | O(j − i) | New tuple |
| `hash(t)` | O(n) | Hashes each element |
| `r[i]`, `len(r)`, `r[i:j]` | O(1) | Arithmetic on start/stop/step |
| `x in r` with an int | O(1) | Arithmetic check |
| `x in r` with a non-int (e.g. a float) | O(n) | Falls back to a scan: measured 156–174 ms for 10⁶ (Ch 7) |

### `collections.deque`

| Operation | Cost | Reason |
|---|---|---|
| `append`, `appendleft`, `pop`, `popleft` | O(1) | Both ends are directly reachable |
| `q[0]`, `q[-1]` | O(1) | Ends |
| `q[i]` in the middle | O(n) | Per the docs: "slows to O(n) in the middle" |
| `rotate(k)` | O(k) | Moves k items between ends |
| `x in q`, `remove(x)` | O(n) | Scan |
| With `maxlen`: append to a full deque | O(1) | Drops one item from the other end |

### `heapq` (min-heap in a list)

| Operation | Cost | Reason |
|---|---|---|
| `heapify(a)` | O(n) | The docs: "in linear time" (Ch 9 explains why) |
| `heappush`, `heappop`, `heappushpop`, `heapreplace` | O(log n) | Sift up/down along one root-to-leaf path |
| `h[0]` | O(1) | The minimum is at the root |
| `nlargest(k, it)`, `nsmallest(k, it)` | O(n log k) | Heap of size k; measured faster than sorting only for small k (Ch 8) |
| Remove an arbitrary item | O(n) | Find it, then re-heapify |

### `bisect`

| Operation | Cost | Reason |
|---|---|---|
| `bisect_left`, `bisect_right` | O(log n) | Halves the range each step |
| `insort_left`, `insort_right` | O(n) | O(log n) search + O(n) `list.insert` (per the docs) |

## 12.2 Explanations by group

### 12.2.1 Hidden costs

The expensive line often doesn't look expensive:

| Looks like | Actually | Fix |
|---|---|---|
| `if x in seen:` with `seen` a list | O(n) scan | `seen` as a `set` |
| `queue.pop(0)` | O(n) shift | `deque.popleft()` |
| `sum(nums[i:j])` in a loop | O(j − i) copy + O(j − i) sum | Prefix sums (Ch 15) |
| `s = s + c` in a loop | O(len(s)) copy each time | List + `"".join` |
| `sorted(s)` to compare anagrams | O(n log n), returns a list | `Counter` or a 26-slot count, O(n) |
| `key = tuple(window)` per step | O(k) to build each key | Rolling update (Ch 25) |
| `min(nums)` inside a loop | O(n) per call | Track the running minimum |
| `len(s)` | O(1) | Nothing to fix |
| `s1 == s2` for long strings | O(min length) | Fine once; costly in a loop over pairs |
| `max(d, key=d.get)` | O(n) | Fine once |

### 12.2.2 Worst case vs average case for hashing

The docs' dict and set costs are **average-case**: they assume the hash function spreads
keys well. In the worst case, when every key hashes to the same slot, each O(1)
operation becomes O(n). For LeetCode's ints, strings and tuples, plan with the average
case (Ch 10, 10.2.2), and say "O(1) average" when you state it.

One more subtle cost from the docs: iterating over a dict or set scans its internal
table, **which isn't shrunk when you delete items**. After deleting most of a large set,
iterating it still takes time proportional to its former size, until a later insertion
triggers a resize.

### 12.2.3 Substring search really is linear (forward)

The CPython docs state that forward searches (`in`, `find`, `index`, `count`) use a
two-way algorithm with a linear worst case. Measured on an adversarial input
(`bench/ch12_substring_search.py`): searching for `"a" * m + "b"` in `"a" * 10⁶`, where
a naive algorithm does about m comparisons at each of 10⁶ positions:

| m | Time |
|---|---|
| 10 | 2.7–3.0 ms |
| 100 | 2.6–2.8 ms |
| 1,000 | 2.7–2.8 ms |
| 10,000 | 2.7–2.8 ms |

Flat as m grows 1000×: linear in practice. So `sub in s` is **[KNOW-HOW]** not because it's
slow, but because interviewers ask you to implement matching yourself (Ch 29).

### 12.2.4 Using the tables

Before writing a solution:

1. List the operations inside your main loop.
2. Look up each one's cost and multiply by the loop count.
3. Compare the total with the budget for the constraints (Ch 10, about 10⁷ simple steps
   per second).

For example: a loop over n = 10⁵ items that does `x in some_list` (n = 10⁵) costs
10¹⁰ operations, so change the data structure. The same loop doing `x in some_set`
costs about 10⁵.

## 12.3 Gotchas & traps

**1. Treating `in` the same for every container.** O(1) for set/dict (and int-in-range),
O(n) for list/tuple/str (12.1).

**2. Forgetting that `insort` and `pop(0)` are O(n)** (12.1).

**3. Slices in loops.** Every slice is a copy (12.2.1).

**4. `s & t` is not O(len(s) + len(t))**; it's O(min) on average. Iterate over the smaller
side when you write it by hand (12.1).

**5. `sorted` on a string returns a list** (12.1).

**6. Hash costs stated as worst case, or worst case ignored.** Say "O(1) average" (12.2.2).

## 12.4 Benchmarks

| File | Measures | Result (CPython 3.13.2) |
|---|---|---|
| `bench/ch12_substring_search.py` | Adversarial forward search, m = 10 to 10⁴ | Flat at 2.6–3.0 ms: linear |

Other tables in this chapter cite measurements from Ch 6, 7, 8, 10 and 11
(`bench/RESULTS.md`).

## 12.5 Exercises: spot the hidden cost

Answers are in Appendix D. For each snippet, n = `len(nums)` or `len(s)`. Give the time
complexity and a fix if it's worse than necessary.

1. `for x in nums: if x in nums[:i]: ...` (with `i` the loop index)
2. `while q: node = q.pop(0)`
3. `result = ""` then `for c in s: result = c + result`
4. `for i in range(n): window_sum = sum(nums[i:i + k])`
5. `seen = []` then `for x in nums: if x not in seen: seen.append(x)`
6. `for i in range(n): best = max(best, min(nums[i:]))`
7. `common = set(a) & set(b)` where `len(a) = 10⁶` and `len(b) = 10`
8. `for x in nums: bisect.insort(sorted_list, x)`
9. `if sorted(s) == sorted(t):`
10. `for word in words: key = "".join(sorted(word))` (L = maximum word length)

## 12.6 Chapter cheat sheet

| O(1) | O(log n) | O(n) | O(n log n) |
|---|---|---|---|
| `a[i]`, `len`, `append`, `pop()`, dict/set get/add/in (avg), deque ends, `h[0]`, int in `range` | `heappush`/`heappop`, `bisect_left`/`right` | `x in list/str/tuple`, `pop(0)`, `insert(0)`, `insort`, slices, `+`, `min`/`max`/`sum`, copies, `join`, `split`, `heapify`, `Counter(it)`, forward substring search | `sorted`, `list.sort` |
