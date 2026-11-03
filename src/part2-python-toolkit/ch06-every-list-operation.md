# Chapter 6: Every `list` Operation

## 6.0 At a glance

The list is the array of LeetCode array problems. It has **11 public methods** (checked
against `dir(list)` by `code/ch06_list/reference_check.py`) and a dozen operators. The
complexities differ wildly by **position**: `append` and `pop()` at the end are O(1);
`insert(0, x)` and `pop(0)` at the front are O(n). This chapter shows why, using
CPython's dynamic-array layout, and measures it.

Also covered: `sort` vs `sorted`, sort keys, stability and multi-key sorting,
`functools.cmp_to_key` with 179 Largest Number, slice assignment, and what "array"
means in an interview.

Notation: n = `len(a)`, k = number of items added, removed or sliced.

## 6.1 Complete reference

### Methods

| Method | Signature → returns | Mutates? | Time (reason) | Tier |
|---|---|---|---|---|
| `append` | `a.append(x) → None` | Yes | O(1) amortized: write to a spare slot; occasional resize (6.2.6) | **[USE-FREELY]** |
| `extend` | `a.extend(iterable) → None` | Yes | O(k) amortized: k appends | **[USE-FREELY]** |
| `insert` | `a.insert(i, x) → None` | Yes | O(n − i): shifts everything after `i` one slot right | **[USE-FREELY]** (know the cost) |
| `remove` | `a.remove(x) → None` | Yes | O(n): finds the first `x`, then shifts the rest left | **[USE-FREELY]** (know the cost) |
| `pop` | `a.pop(i=-1) → item` | Yes | O(1) for the last item; O(n − i) otherwise (shift left) | **[USE-FREELY]** |
| `clear` | `a.clear() → None` | Yes | O(n): releases each reference | **[USE-FREELY]** |
| `index` | `a.index(x[, start[, stop]]) → int` | No | O(n): linear scan; raises `ValueError` | **[USE-FREELY]** (know the cost) |
| `count` | `a.count(x) → int` | No | O(n): full scan | **[USE-FREELY]** |
| `sort` | `a.sort(*, key=None, reverse=False) → None` | Yes | O(n log n) worst, O(n) on already-sorted data (Timsort); O(n) auxiliary | **[KNOW-HOW]** |
| `reverse` | `a.reverse() → None` | Yes | O(n): n // 2 swaps; O(1) auxiliary | **[KNOW-HOW]** |
| `copy` | `a.copy() → list` | No | O(n): shallow copy of n references | **[USE-FREELY]** |

All of them are available in every Python version this book supports (3.10+).

### Operators and built-ins on lists

| Operation | Example | Mutates? | Time (reason) | Tier |
|---|---|---|---|---|
| Index | `a[i]`, `a[-1]` | No | O(1): pointer arithmetic | **[USE-FREELY]** |
| Assign item | `a[i] = x` | Yes | O(1) | **[USE-FREELY]** |
| Slice | `a[i:j]`, `a[::-1]`, `a[::2]` | No | O(k): copies k references | **[USE-FREELY]** |
| Slice assignment | `a[i:j] = it`, `a[:] = it` | Yes | O(n + k): may shift the tail | **[USE-FREELY]** |
| Extended slice assignment | `a[::2] = it` | Yes | O(k); lengths must match | **[USE-FREELY]** |
| Delete | `del a[i]`, `del a[i:j]`, `del a[::2]` | Yes | O(n): shifts the tail | **[USE-FREELY]** |
| Concatenate | `a + b` | No (new list) | O(n + m) | **[USE-FREELY]** |
| In-place concatenate | `a += it` | **Yes** (same as `extend`) | O(k) amortized | **[USE-FREELY]** |
| Repeat | `a * r` | No | O(n · r); repeats **references** (Ch 3) | **[USE-FREELY]** |
| Membership | `x in a` | No | O(n): linear scan with `==` | **[USE-FREELY]** (know the cost) |
| Equality | `a == b` | No | O(min(n, m)): stops at the first difference | **[USE-FREELY]** |
| Ordering | `a < b` | No | O(min(n, m)): lexicographic | **[USE-FREELY]** |
| Length | `len(a)` | No | O(1): stored | **[USE-FREELY]** |
| Unpacking | `x, y = a`, `first, *rest = a` | No | O(n) | **[USE-FREELY]** |
| `sorted(it, key=None, reverse=False)` | `sorted(a)` | No (new list) | O(n log n); O(n) auxiliary | **[KNOW-HOW]** |
| `reversed(a)` | `for x in reversed(a)` | No | O(1) to create; O(n) to consume | **[KNOW-HOW]** |
| `min`, `max`, `sum` | `max(a)` | No | O(n) | `min`/`max` **[KNOW-HOW]**, `sum` **[USE-FREELY]** |
| `functools.cmp_to_key(cmp)` | `sorted(a, key=cmp_to_key(f))` | No | O(n log n) comparisons, each calling `f` | **[KNOW-HOW]** |

## 6.2 Explanations by group

### 6.2.1 Adding: `append`, `extend`, `insert`, `+=`

```python
a = [1, 2, 3]
a.append([4, 5])               # one item, which is a list
a  # -> [1, 2, 3, [4, 5]]
b = [1, 2, 3]
b.extend([4, 5])               # each item of the iterable
b  # -> [1, 2, 3, 4, 5]
c = [1]
c.extend("ab")                 # a string is an iterable of characters
c  # -> [1, 'a', 'b']
```

`+=` behaves like `extend`, so it accepts any iterable. `+` requires another list:

```python
d = [1]
d += (2,)
d  # -> [1, 2]
d + (3,)  # -> TypeError: can only concatenate list (not "tuple") to list
```

`insert` clamps out-of-range positions instead of raising:

```python
e = [1, 2, 3]
e.insert(100, 9)
e.insert(-100, 0)
e  # -> [0, 1, 2, 3, 9]
```

### 6.2.2 Removing: `pop`, `remove`, `del`, `clear`

| You know | Use | Missing / empty |
|---|---|---|
| The position | `a.pop(i)` (returns it) or `del a[i]` | `IndexError` |
| The value | `a.remove(x)`: first occurrence only | `ValueError` |
| Nothing: remove everything | `a.clear()` or `del a[:]` | |

```python
a = [1, 2, 1, 3]
a.remove(1)
a  # -> [2, 1, 3]
[].pop()  # -> IndexError: pop from empty list
[1, 2].remove(5)  # -> ValueError: list.remove(x): x not in list
```

Removing **all** occurrences with a loop of `remove` calls is O(n²). Rebuild once
instead (O(n)), in place if the caller needs it (Ch 1):

```python
a = [2, 1, 3, 1]
a[:] = [x for x in a if x != 1]
a  # -> [2, 3]
```

### 6.2.3 Searching: `index`, `count`, `in`

All three are **linear scans**. Use them on lists only when n is small, or when you
call them once. For repeated membership tests, build a `set` (O(1) average per test,
Ch 12):

```python
a = [5, 6, 7]
a.index(7)  # -> 2
a.index(9)  # -> ValueError: 9 is not in list
6 in a  # -> True
```

### 6.2.4 Slices and slice assignment

A slice read **copies** k references:

```python
a = [0, 1, 2, 3, 4, 5]
a[1:4]  # -> [1, 2, 3]
a[::-1]  # -> [5, 4, 3, 2, 1, 0]
a[::2]  # -> [0, 2, 4]
a[10:]  # -> []
```

Slices never raise for out-of-range bounds; they clamp. Inside a loop, a slice like
`a[i:]` costs O(n − i) every time. That turns an O(n) loop into an O(n²) one, and in
recursion it costs memory too (Ch 10).

A slice **assignment** replaces a range, and the replacement may have a different
length:

```python
a = [1, 2, 3]
a[1:2] = [9, 9, 9]
a  # -> [1, 9, 9, 9, 3]
a[::2] = [0, 0, 0]
a  # -> [0, 9, 0, 9, 0]
```

An extended slice (with a step) must receive exactly as many items as it selects:

```python
a = [1, 2, 3, 4]
a[::2] = [0]  # -> ValueError: attempt to assign sequence of size 1 to extended slice...
```

`del` works with slices too: `del a[::2]` removes every other element.

### 6.2.5 Sorting

| | `a.sort()` | `sorted(it)` |
|---|---|---|
| Works on | lists only | any iterable (strings, sets, dict keys, generators) |
| Returns | `None` | a new list |
| Original | changed | unchanged |
| Auxiliary space | O(n) worst case (merge buffer) | O(n) for the new list, plus the same buffer |

```python
c = [3, 1, 2]
result = c.sort()
result is None  # -> True
c  # -> [1, 2, 3]
```

`x = x.sort()` is the classic bug: it sets `x` to `None`.

**`key=`** maps each item to the value that gets compared. The key function is called
once per item (n calls in total), not once per comparison:

```python
words = ["kiwi", "fig", "banana"]
sorted(words, key=len)  # -> ['fig', 'kiwi', 'banana']
sorted(words, key=len, reverse=True)  # -> ['banana', 'kiwi', 'fig']
```

**Stability.** The docs guarantee that `sort` and `sorted` are **stable**: items with
equal keys keep their original relative order. Stability is what makes multi-key
sorting by multiple passes work: sort by the **secondary** key first, then by the
primary key.

```python
people = [("bob", 25), ("amy", 30), ("cat", 25), ("dan", 30)]
by_name = sorted(people, key=lambda p: p[0])
by_name.sort(key=lambda p: p[1], reverse=True)
by_name  # -> [('amy', 30), ('dan', 30), ('bob', 25), ('cat', 25)]
```

Usually a **tuple key** does it in one pass. Negate numeric fields to reverse just
those fields:

<!-- cont -->

```python
ranked = sorted(people, key=lambda p: (-p[1], p[0]))
ranked == by_name  # -> True
```

Negation works only for numbers. To sort strings descending with numbers ascending,
use two stable passes, as above.

**Mixed types don't compare:**

```python
[1, "a"].sort()  # -> TypeError: '<' not supported between instances of 'str' and 'int'
```

### 6.2.6 Custom comparisons: `cmp_to_key` and 179 Largest Number

Some orders can't be expressed as a key on a single item, only as a comparison
between **two** items. 179 Largest Number (<https://leetcode.com/problems/largest-number/>)
asks you to arrange non-negative integers so that their concatenation is as large as
possible, and to return it as a string. `a` should come before `b` exactly when
`a + b > b + a` as strings.

LeetCode's signature: `def largestNumber(self, nums: List[int]) -> str:`. Book signature:
`def largestNumber(self, nums: list[int]) -> str:`.

Constraints: at most 100 numbers, each up to 10⁹. Brute force over all orderings is
O(n! · n · L) (L ≤ 10 digits), hopeless for n = 100. Sorting with the pairwise rule is
O(n log n) comparisons:

<!-- file: code/ch06_list/largest_number.py -->

```python
class Solution:
    def largestNumber(self, nums: list[int]) -> str:
        # O(n log n * L) time: Timsort makes O(n log n) comparisons, and each one
        # builds two strings of length <= 2L (L <= 10 digits here).
        # O(n * L) auxiliary space: the string forms of the numbers.
        def compare(a: str, b: str) -> int:
            if a + b > b + a:
                return -1          # a should come first
            if a + b < b + a:
                return 1
            return 0

        parts = sorted(map(str, nums), key=cmp_to_key(compare))
        result = "".join(parts)
        return "0" if result[0] == "0" else result     # [0, 0] -> "0", not "00"
```

A comparator returns a negative number if `a` comes first, a positive number if `b`
comes first, and 0 for a tie. `cmp_to_key` wraps it into a key object that `sorted`
can use.

Why a plain reverse string sort fails: it puts `"30"` before `"3"`, but `"330"` beats
`"303"`:

<!-- file: code/ch06_list/largest_number.py -->

```python
class SolutionWrongReverseSort:
    """Plain reverse string sort: right for many inputs, wrong for [3, 30, 34, 5, 9]."""

    def largestNumber(self, nums: list[int]) -> str:
        return "".join(sorted(map(str, nums), reverse=True))
```

It returns `"9534303"` instead of `"9534330"`. The test file checks the comparator
version against a brute-force search over all orderings, on 300 random small inputs.
Edge cases tested: `[0, 0, 0]` → `"0"`, a single element, all-equal values, 10⁹, a
shared prefix (`[824, 8247]`), and n = 100.

**Tier.** `sorted` + `cmp_to_key` is **[KNOW-HOW]**. In an interview, be ready to explain
why the pairwise rule gives a consistent order, and to write a merge sort that uses
`compare` (Ch 18).

### 6.2.7 How a Python list is built

CPython stores a list as a **dynamic array of pointers**:

```text
list object                       heap
┌──────────────┐
│ ob_size = 3  │   number of items in use
│ allocated = 4│   slots reserved
│ ob_item ─────┼──► [ • │ • │ • │   ]       one 8-byte pointer per slot
└──────────────┘      │   │   │   spare
                      ▼   ▼   ▼
                     int int int          the items live elsewhere
```

Consequences:

- **`a[i]` is O(1):** address = start + 8 · i.
- **`append` is O(1) amortized.** If a spare slot exists, append writes a pointer. If not,
  CPython allocates a larger block and copies the pointers. The capacities grow
  geometrically, so the total copying over n appends is O(n), which averages to O(1)
  per append.
- **`insert(0, x)` and `pop(0)` are O(n):** every pointer after the position moves by
  one slot.

You can watch the over-allocation through `sys.getsizeof`, which reports the list's own
size: a fixed header plus 8 bytes per **allocated** slot.

```python
import sys

a = []
sizes = []
for i in range(17):
    a.append(i)
    sizes.append(sys.getsizeof(a))
sizes[0], sizes[3], sizes[4], sizes[8], sizes[16]  # -> (88, 88, 120, 184, 248)
```

On CPython 3.13.2, the size changes only when the length reaches 1, 5, 9, 17, ...,
giving capacities of 4, 8, 16, 24, 32, 40. That matches CPython's growth rule,
`new_allocated = (n + (n >> 3) + 6) & ~3`: roughly 12.5% extra, plus a constant. The
exact numbers are an implementation detail and differ across versions. The
geometric growth is the part to remember.

**Measured** (`bench/ch06_list_ends.py`, CPython 3.13.2, best of 5, two runs). If an
operation is O(1), doubling N doubles the total time; if it's O(n), the total
quadruples.

| Total time to... | N = 25,000 | 50,000 | 100,000 | Per doubling |
|---|---|---|---|---|
| build with `append` | 0.6–0.9 ms | 1.4–2.8 ms | 3.0–5.2 ms | ~2× |
| build with `insert(0, x)` | 87–156 ms | 351–651 ms | 1.5–2.3 s | ~4× |
| empty with `pop()` | 1.3–2.3 ms | 2.6–5.0 ms | 6–11 ms | ~2× |
| empty with `pop(0)` | 0.96–1.0 s | 3.9 s | 15.6 s | ~4× |

At N = 100,000, emptying with `pop(0)` took **15.6 seconds** against **11 milliseconds**
with `pop()`. That's a TLE in any judge. For a queue, use `collections.deque`, whose
`popleft()` is O(1) (Ch 8). Or keep a head index into a list and never delete.

### 6.2.8 Arrays vs Python lists

When an interviewer says "array", they mean a **contiguous, fixed-type sequence with
O(1) indexing**. A Python list provides the same O(1) indexing, which is all array
algorithms need. The differences are in memory:

| | `list` | `array.array("q")` | NumPy array |
|---|---|---|---|
| Stores | pointers to int objects | raw 8-byte ints | raw typed values |
| Mixed types | Yes | No | No |
| 10⁶ distinct ints (measured, 3.13.2) | 34.3 MiB (list + int objects) | 7.6 MiB | not measured |
| Available on LeetCode | Yes | Yes (stdlib) | Not part of the stdlib; don't assume it |
| Used in this book | Always | Rarely | Never |

The `array` module (stdlib) stores values in a C array. It saves memory, but it doesn't
make Python-level loops faster (general knowledge, not benchmarked here). Use plain lists for LeetCode. Mention the memory
difference only if an interviewer asks about memory.

## 6.3 Gotchas & traps

**1. `x = x.sort()`** sets `x` to `None` (6.2.5).

**2. `append` vs `extend`** with a list argument (6.2.1).

**3. `pop(0)` / `insert(0, x)` in a loop.** O(n) each, so O(n²) in total. Measured:
15.6 s vs 11 ms at N = 100,000 (6.2.7).

**4. `remove` in a loop to delete all copies.** O(n²); rebuild with a comprehension
(6.2.2).

**5. `x in a_list` inside a loop.** O(n) each; use a `set` (6.2.3).

**6. Slicing inside a loop.** `a[i:]` copies (6.2.4).

**7. `a + (x,)`.** `TypeError`; `+` needs a list. `a += (x,)` works (6.2.1).

**8. `[[]] * 3`.** Three references to one list (Ch 3, Ch 4).

**9. Extended-slice assignment with the wrong length.** `ValueError` (6.2.4).

**10. Comparing lists compares lexicographically:**

```python
[1, 2] < [1, 2, 0]  # -> True
[2] > [1, 9, 9]  # -> True
```

**11. Sorting mixed types** raises `TypeError` (6.2.5).

**12. Reverse string sort for "largest concatenation"** is wrong (6.2.6).

## 6.4 Benchmarks

| File | Measures | Result (CPython 3.13.2) |
|---|---|---|
| `bench/ch06_list_ends.py` | `append` vs `insert(0)`; `pop()` vs `pop(0)`; list vs `array` memory | Front operations quadruple per doubling; 15.6 s vs 11 ms at N = 10⁵; 34.3 vs 7.6 MiB (6.2.7, 6.2.8) |

## 6.5 Exercises

Answers are in Appendix D (verified in `code/ch06_list/exercises.py`).

**Warm-up**

1. Starting from `[1, 2, 3]` each time, what do `a.append([4, 5])` and `a.extend([4, 5])`
   produce?
2. `c = [3, 1, 2]; result = c.sort()`. What are `result` and `c`?
3. `d = [1]`. Which of `d = d + (2,)` and `d += (2,)` works, and why?

**Core**

4. Sort `people = [("bob", 25), ("amy", 30), ("cat", 25), ("dan", 30)]` by age
   descending, then name ascending, in two ways: with one key, and with two stable
   sorts.
5. Remove the first `1` from `[1, 2, 1, 3]`. Then remove all remaining `1`s, in
   place, in O(n).
6. What does `insert` do with positions 100 and -100 on a 3-item list?
7. What is `g` after `g = [1, 2, 3]; del g[::2]`?
8. What happens on `h = [1, 2, 3, 4]; h[::2] = [0]`?
9. Why is `pop(0)` O(n)? What should you use for a FIFO queue?
10. For `[3, 30, 34, 5, 9]`, why does `"".join(sorted(map(str, nums), reverse=True))`
    give the wrong answer?

**Tricky**

11. Evaluate `[1, 2] < [1, 2, 0]` and `[2] > [1, 9, 9]`.
12. What does `[1, "a"].sort()` do?
13. `x = [[]] * 3; x[0].append(1)`. What is `x`?

## 6.6 Chapter cheat sheet

| Operation | Cost | Note |
|---|---|---|
| `a[i]`, `a[i] = x`, `len(a)` | O(1) | |
| `append`, `pop()` | O(1) amortized | Use the end of the list as a stack |
| `insert(0, x)`, `pop(0)`, `del a[0]` | O(n) | 1,400× slower at 10⁵ (`bench/ch06_list_ends.py`); use `deque` |
| `x in a`, `index`, `count`, `remove` | O(n) | Use a `set` for repeated lookups |
| `a[i:j]` | O(j − i) copy | Not free inside loops |
| `a[:] = ...` | O(n + k) | In-place replacement (Ch 1) |
| `a.sort()` / `sorted(a)` | O(n log n), stable | `sort` returns `None` |
| Multi-key | `key=lambda p: (-p[1], p[0])` | Or two stable passes, secondary first |
| Pairwise order | `cmp_to_key(cmp)` | 179 Largest Number |
| `a += it` | Mutates (extend) | `a + it` needs a list and creates a new one |
