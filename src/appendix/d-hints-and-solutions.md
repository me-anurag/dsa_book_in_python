# Appendix D: Hints and Solutions

Answers to every chapter's exercises, and hints for every practice problem. They're
kept here, away from the problems, so you don't see them by accident. Every answer
that involves code is verified in that chapter's `code/chNN_*/exercises.py`.

## Chapter 1: Anatomy of a LeetCode Solution

1. **(a), (c), (d), (f).** (b) and (e) bind the local name `nums` to a new list; the
   caller's list is untouched. (d) works because `+=` on a list is `extend`.
2. Two-pointer swap: O(1) auxiliary space, because it uses only two indices.
   `s[:] = s[::-1]`: O(n) auxiliary space, because `s[::-1]` is a temporary copy of
   n items. (A third option, `s.reverse()`, is O(1) but **[KNOW-HOW]**.)
3. **27: accepted.** Its judge sorts `nums[:3] = [2, 3, 1]` to `[1, 2, 3]`.
   **26: rejected.** Its judge compares `[2, 3, 1]` with `[1, 2, 3]` in order.
4. `None` (shown as `null`). A function that falls off its end returns `None`.
5. The first test case receives 1, the second receives 2. `calls` lives on the class,
   so it survives across instances. Use a local variable.
6. `UnboundLocalError`. `count += 1` is an assignment, so Python treats `count` as
   local to the nested function, and that local has no value yet. Fix: put
   `nonlocal count` at the top of the nested function, or keep the count in a
   mutable object (`count = [0]`, then `count[0] += 1`).
7. (a) O(n log n) or better. (b) Exponential is fine: 2¹⁸ ≈ 2.6 × 10⁵ subsets.
   (c) O(n²) comfortably (9 × 10⁴ steps). O(n³) = 2.7 × 10⁷ steps is roughly 2.5–5 s
   at the 5–11 M steps/s measured in Ch 10: likely too slow.
   (d) O(log x) or O(√x): x itself is up to ~2.1 × 10⁹, too large to loop up to.
8. Without the modulo, `k = 4 > n = 3` makes `nums[-4:]` the whole list and
   `nums[:-4]` empty, so the result is `[1, 2, 3]` unchanged, instead of
   `[3, 1, 2]`. Rotating by k is the same as rotating by k mod n.
9. One version:

   ```python
   import copy

   def check_in_place(method, args: tuple, expected, arg_index: int = 0) -> bool:
       args = copy.deepcopy(args)        # the test's own data stays untouched
       method(*args)
       return args[arg_index] == expected
   ```

10. Yes: the result is `[3, 2, 1]`. CPython converts the right-hand side to a
    complete sequence before replacing the slice, so `reversed(nums)` finishes
    reading the old contents before any write happens.

## Chapter 2: Type Hints, Completely

1. `def f(a: list[int], m: dict[str, list[int]], o: tuple[int, int] | None) -> set[str]:`
2. (a) 3.9 (built-in generics). (b) 3.10 (`X | Y`). (c) 3.7: the future import
   stores annotations as strings, so nothing in them is evaluated.
3. `'xy'`. Hints are not checked; `+` on two strings concatenates them.
4. `tuple[int]`: exactly one int. `tuple[int, int]`: exactly two ints, such as
   (row, col). `tuple[int, ...]`: any number of ints.
5. Add `from typing import List` at the top, or rewrite the hints as `list[int]`.
   (On 3.14+ the `def` itself no longer fails, because annotations are lazy.)
6. `list[int]` is a generic alias, and `isinstance` refuses those because it can't
   check element types without scanning the list. Use
   `isinstance(x, list) and all(isinstance(v, int) for v in x)`.
7. (a) `dict[str, list[int]]` (b) `list[list[str]]`
   (c) `collections.deque[tuple[int, int]]` (d) `collections.abc.Callable[[str], int]`
8. `TypeError: g() missing 1 required positional argument: 'k'`. `Optional[int]` says
   the value may be `None`; it doesn't give the parameter a default. Write
   `k: int | None = None` for that.
9. On 3.8, `list` isn't subscriptable, and annotations are evaluated when `def` runs,
   so `list[int]` raises `TypeError`. The future import stores the annotation as the
   string `'list[int]'` and never evaluates it.
10. `collections.defaultdict[tuple[int, int], list[str]]`

## Chapter 3: Core Python Semantics

1. `[1, 2, 3]`. `b` and `a` name the same list, and `+=` on a list mutates it.
2. `[[1, 0, 0], [1, 0, 0]]`: both rows are the same object. Fix:
   `grid = [[0] * 3 for _ in range(2)]`.
3. `-4`, `1`, `-3`, `(-4, 1)`. `//` floors; `int(-7 / 2)` truncates the float -3.5.
4. One version:

   ```python
   def trunc_div(a: int, b: int) -> int:
       q = abs(a) // abs(b)
       return q if (a >= 0) == (b > 0) else -q
   ```
5. `shallow == [[1, 2, 99], [3, 4]]`, because it shares the inner lists.
   `deep == [[1, 2], [3, 4]]`.
6. `return abs(x) % 10`. `-123 % 10` is `7`: `%` takes the divisor's sign, so the
   result is in `[0, 10)`, and -123 = -13 · 10 + 7.
7. False. `float(10**20 + 1)` rounds to `1e20`, because a double has 53 bits of
   precision. The comparison itself is exact; the information was lost in the
   conversion.
8. `ValueError: zip() argument 2 is shorter than argument 1`. Without `strict`, the
   result is `[(1, 1)]`: silently truncated.
9. `reversed(range(n))`, `range(n - 1, -1, -1)`, `range(n)[::-1]`. None of these
   copies: reversing or slicing a `range` gives another lazy object. (By contrast,
   `list(range(n))[::-1]` builds two lists.)
10. The right side evaluates to `(0, 1)`. Then `nums[0] = 0` runs, and only then is
    the second target's index `nums[0]` evaluated, which is now 0, so `nums[0] = 1`.
    Fix: `j = nums[i]`, then `nums[i], nums[j] = nums[j], nums[i]`.

## Chapter 4: Comprehensions & Generators

| # | Answer | Result / note |
|---|---|---|
| 1 | `[x * x for x in range(10)]` | `[0, 1, 4, ..., 81]` |
| 2 | `[x for x in nums if x % 2 == 0]` | `[4, 2, 6]` |
| 3 | `[x if x > 0 else 0 for x in nums]` | `[3, 0, 4, 0, 5, 0, 2, 6]` |
| 4 | `[len(w) for w in words]` | `[5, 0, 4, 3, 6]` |
| 5 | `[w[0].upper() for w in words if w]` | The filter skips `""`, which has no `w[0]` |
| 6 | `{c: ord(c) for c in "abc"}` | `{'a': 97, 'b': 98, 'c': 99}` |
| 7 | `{x % 3 for x in nums}` | `{0, 1, 2}`; `-1 % 3 == 2`, `-9 % 3 == 0` |
| 8 | `{v: k for k, v in d.items()}` | `{1: 'a', 2: 'b'}` |
| 9 | `sum(1 for x in nums if x % 2 == 0)` | `3` |
| 10 | `any(x < 0 for x in nums)` | `True`, stops at `-1` |
| 11 | `[x for row in grid for x in row]` | `[1, 2, 3, 4, 5, 6]` |
| 12 | `[[grid[r][c] for r in range(R)] for c in range(C)]` | `[[1, 4], [2, 5], [3, 6]]` |
| 13 | `[[0] * 3 for _ in range(2)]` | Fresh row per iteration |
| 14 | `[[1 if r == c else 0 for c in range(3)] for r in range(3)]` | |
| 15 | `[(i, j) for i in range(3) for j in range(i + 1, 3)]` | `[(0, 1), (0, 2), (1, 2)]` |
| 16 | `{v: i for i, v in enumerate(vals)}` | `{7: 2, 8: 1}`: last wins |
| 17 | `{v: i for i, v in reversed(list(enumerate(vals)))}` | `{7: 0, 8: 1}` |
| 18 | `{c: s.count(c) for c in set(s)}` | O(k · n): each of k distinct characters rescans n characters. One pass with a dict or `Counter` is O(n) |
| 19 | `{n: [w for w in words if len(w) == n] for n in {len(w) for w in words}}` | O(k · n) for k distinct lengths; a loop appending to `defaultdict(list)` is O(n) |
| 20 | `list(accumulate(a))`, or the walrus comprehension | `[1, 3, 6, 10]`; prefer `accumulate` or a loop |
| 21 | `[(1 + dr, 0 + dc) for dr, dc in DIRS if 0 <= 1 + dr < 2 and 0 <= 0 + dc < 3]` | `[(0, 0), (1, 1)]` |
| 22 | `all(row == sorted(row) for row in grid)` | `True` |
| 23 | `"".join(c.lower() for c in s if c.isalnum())` | `'amanaplan'` |
| 24 | `sum(c in "aeiou" for c in "education")` | `5` |
| 25 | `[square[i][i] for i in range(len(square))]` | `[1, 4]` |
| 26 | `[[7, 0], [7, 0]]` | Both rows are one object |
| 27 | `(3, 0)` | The second `sum` sees an exhausted generator |
| 28 | `[2, 2, 2]`; fix: `lambda i=i: i` gives `[0, 1, 2]` | Late binding |
| 29 | `[('x', '1'), ('x', '2'), ('y', '1'), ('y', '2')]` | Outer loop first |
| 30 | `[0, 1, -2, 3]` | Even values are negated (0 stays 0) |
| 31 | `(False, True)` | Vacuous truth for `all` |
| 32 | `[list(r) for r in zip(*square[::-1])]` | `[[3, 1], [4, 2]]` |
| 33 | `NameError: name 'leak' is not defined` | Loop variables don't leak from comprehensions |

## Chapter 5: Every `str` Method

1. `['hello', 'world']` and `['', '', 'hello', '', '', 'world', '', '']`.
2. `'c'`. `find` returned -1, and -1 indexes the last character, so the "not found"
   case silently reads real data.
3. None of the three methods. `isdigit` accepts `'²'`; `isdecimal` accepts `'٣'`. Use
   `'0' <= c <= '9'`.
4. `"".join(c.lower() for c in s if c.isalnum())` gives `'amanaplanacanalpanama'`.
5. Advance by one after each match:

   ```python
   def count_overlapping(s: str, sub: str) -> int:
       count = 0
       start = s.find(sub)
       while start != -1:
           count += 1
           start = s.find(sub, start + 1)
       return count
   ```

6. `" ".join(reversed(s.split()))`: O(n) time, O(n) auxiliary space.
7. `"7".zfill(3)` and `f"{7:03d}"`.
8. `table = str.maketrans(lower, lower[1:] + lower[0])` with
   `lower = string.ascii_lowercase`, then `s.translate(table)`.
9. `('k', '=', 'v=w')` and `('k=v', '=', 'w')`.
10. Append characters to a list and `"".join` once at the end: O(n). `s += c`
    copies the string unless CPython can resize it in place, and that optimization
    is an implementation detail that a second reference disables. Measured: about 4×
    time per doubling of n in that case (Ch 5, 5.2.7).
11. `"They'Re Bill'S"`. Use `string.capwords(s)`.
12. `'c'` (strip removes any of the characters `a`/`b` from both ends), and `'cba'`.
13. A `list[str]`, `['a', 'b', 'c']`. Use `"".join(sorted("bca"))`.
14. True. Strings compare character by character, and `'1' < '9'`. Compare `int`s for
    numeric order.

## Chapter 6: Every `list` Operation

1. `append` gives `[1, 2, 3, [4, 5]]` (one new item); `extend` gives `[1, 2, 3, 4, 5]`.
2. `result is None`; `c == [1, 2, 3]`.
3. Only `d += (2,)`. `+=` is `extend`, which accepts any iterable; `+` needs a list
   on both sides and raises `TypeError`.
4. `sorted(people, key=lambda p: (-p[1], p[0]))`, or sort by name first and then
   `sort(key=lambda p: p[1], reverse=True)`. Stability keeps the name order within
   equal ages. Both give `[('amy', 30), ('dan', 30), ('bob', 25), ('cat', 25)]`.
5. `a.remove(1)` gives `[2, 1, 3]`. Then `a[:] = [x for x in a if x != 1]` gives
   `[2, 3]` in O(n).
6. It clamps: 100 appends at the end, -100 inserts at the front. No error.
7. `[2]`.
8. `ValueError: attempt to assign sequence of size 1 to extended slice of size 2`.
9. Every element after index 0 shifts one slot left: n − 1 pointer moves. Use
   `collections.deque` and `popleft()`, which is O(1).
10. It orders `"30"` before `"3"` (a longer string with an equal prefix sorts later,
    so it comes first in reverse order), producing `"...303"`. But `"3" + "30" =
    "330"` beats `"303"`. Compare `a + b` with `b + a` instead.
11. `True` (a proper prefix is smaller) and `True` (`2 > 1` decides it immediately).
12. `TypeError: '<' not supported between instances of 'str' and 'int'`.
13. `[[1], [1], [1]]`: three references to one list.

## Chapter 7: Built-in Functions for DSA

1. Only `sorted`. `range` is a lazy sequence; the others are one-shot iterators.
2. `ValueError` (message: `max() iterable argument is empty` on 3.13). Use
   `max([], default=0)`.
3. `'b'`: the first key with the maximal value.
4. `2`, `4`, `2.67`. Ties round to even; `2.675` is stored slightly below 2.675.
5. `int` accepts `" 42 "`, `"+7"` and `"4_2"`, and rejects `"1e3"`, `"3.0"` and `""`.
   Its rules aren't the problem's rules, and the whole point of 8 is to parse character
   by character.
6. `format(5, "b")` or `bin(5)[2:]`; parse back with `int("101", 2)`.
7. `pow(3, -1, 7) == 5`, and `3 * 5 % 7 == 1`.
8. `sum(flags) == 2`. `bool` is a subclass of `int`, with `True == 1`.
9. `[x for part in lists for x in part]` or `list(chain.from_iterable(lists))`, both
   O(total). `sum(lists, [])` builds a new list at each `+`, copying everything so far:
   O(total²) in the worst case.
10. `next(it, "empty")`.
11. 3.11: `0.9999999999999999`. 3.13: `1.0`. Since 3.12, `sum` of floats uses a more
    accurate summation algorithm.
12. `(2, 'a')`: the first elements tie, so the second elements decide.
13. Yes: int membership in a range is O(1) arithmetic. No: a float falls back to a
    linear scan of all 10¹⁰ elements (measured at 156–174 ms for a 10⁶-element range).
14. `[2, 4]`, then `[]`: the iterator is exhausted.

## Chapter 8: Standard Library for DSA

1. `[('a', 5), ('b', 2)]`. `'b'` and `'r'` both have count 2, and `'b'` was seen first.
2. `0` and `False`: reading a missing key from a `Counter` doesn't insert it.
3. `Counter("listen") == Counter("silent")`.
4. `Counter({'a': 2})`: `-` drops counts ≤ 0. `subtract` keeps them:
   `Counter({'a': 2, 'b': -1})`.
5. `groups = defaultdict(list)`, then `groups[len(w)].append(w)` gives
   `{2: ['ab', 'de'], 1: ['c']}`. Reading `groups[99]` **inserts** `99: []`.
6. `q.append(x)` / `q.popleft()`; `q.rotate(1)` moves the last item to the front.
7. `deque(maxlen=3)`: appending to a full deque drops the oldest item.
8. Push `-x` for each x, pop three times, and negate: `[9, 7, 5]`.
9. `TypeError`: the priorities tie, so Python compares the dicts. Push
   `(priority, order, item)` with a unique, increasing `order`.
10. `bisect_right(a, x) - bisect_left(a, x)`.
11. `bisect_left(pairs, 4, key=lambda p: p[1]) == 1` (3.10+). The key applies to the
    list items; `4` is already a key value.
12. `[(k, len(list(g))) for k, g in groupby(s)]` gives
    `[('a', 3), ('b', 2), ('c', 1), ('a', 2)]`.
13. `[b - a for a, b in pairwise(nums)]` gives `[3, 5]`.
14. `list(accumulate([1, 2, 3], initial=0))` gives `[0, 1, 3, 6]`.
15. `len(list(combinations(range(5), 2))) == math.comb(5, 2) == 10`.
16. Decorate a recursive `fib` with `@cache`: `fib(100) == 354224848179261915075`.
17. `isqrt` gives exactly `10**16 + 1`. `int(math.sqrt(x))` gives `10000000000000002`,
    because the float conversion lost precision.
18. `math.gcd(12, 18, 24) == 6` (3.9+ for three arguments); `math.lcm(4, 6) == 12`.
19. On `get`: if the key is present, `move_to_end(key)` and return the value. On `put`:
    assign, `move_to_end(key)`, and if over capacity, `popitem(last=False)`. See
    `code/ch08_stdlib/exercises.py`.
20. `Point = namedtuple("Point", "x y")`; then `p.x`, `p[0]`, or `x, y = p`.

## Chapter 9: Built-in vs Hand-Written

1. `len`, `set`, `enumerate`: **[USE-FREELY]**. `sorted`, `Counter`, `bisect_left`,
   `str.find`: **[KNOW-HOW]**. Two pointers, Kadane, monotonic stack: **[HAND-WRITE]**.
2. In order:
   - "in-place, O(1) extra memory": new lists, slice copies, `sorted`.
   - "without division": `/` and `//` on the total product.
   - "don't convert the inputs to integers": `int(...)` on the inputs.
   - "no built-in sort, O(n log n)": `sorted` and `list.sort`.
   - "O(n) time, O(1) space": sorting, and sets or dicts that grow with n.
   - "don't modify the array, constant extra space": sorting in place, sign marking,
     sets.
3. `value = value * 10 + (ord(c) - ord("0"))` for each character, starting from 0.
4. One pass keeps `best` and `second`. On a new maximum, the old `best` moves to
   `second`. A value equal to `best` is skipped. See `max_and_second` in the exercises
   file.
5. `counts[ord(c) - ord("a")] += 1`; the non-zero slots equal `Counter(s)`.
6. To bound the recursion depth by O(log n). Each recursive call gets at most half of
   the current range; the larger half is handled by the loop, which uses no stack.
7. Yes. k = 100,001 possible values and n ≤ 5 × 10⁴: O(n + k) time and O(k)
   auxiliary space, about 1.5 × 10⁵ steps. It's within limits and not a comparison
   sort, so the O(n log n) requirement is met with room to spare.
8. "Timsort: O(n log n) time in the worst case, O(n) on already-sorted runs, O(n)
   auxiliary space, and stable."

## Chapter 10: Complexity Analysis

Time classes confirmed empirically by `code/ch10_complexity/verify_snippets.py`.

| # | Time (reason) | Auxiliary space (reason) |
|---|---|---|
| s01 | O(n): one pass | O(1): one accumulator |
| s02 | O(n²): n × n pairs | O(1) |
| s03 | O(n²): n(n − 1)/2 pairs | O(1) |
| s04 | O(log n): n halves each step | O(1) |
| s05 | O(log n): i doubles until it reaches n | O(1) |
| s06 | O(n log n): n outer iterations × log n halvings | O(1) |
| s07 | O(n): 1 + 2 + 4 + ... < 2n | O(1) |
| s08 | O(n): each step moves one pointer inward; at most n − 1 steps | O(1) |
| s09 | O(n): `right` moves n times; `left` moves at most n times in total | O(1) |
| s10 | O(n²): `x not in seen` scans a list that grows to n | O(n): `seen` |
| s11 | O(n) average: set operations are O(1) average | O(n): `seen` |
| s12 | O(n log n): `sorted` dominates the O(n) loop | O(n): the sorted copy |
| s13 | O(n): n calls of O(1) | O(n): n stack frames (and `RecursionError` near 1000) |
| s14 | O(n log n): T(n) = 2T(n/2) + O(n) | O(n): slices and merged lists at each level (O(log n) frames) |
| s15 | O(2ⁿ): two calls of size n − 1 | O(n): recursion depth n |
| s16 | O(n³): three nested full loops | O(1) |
| s17 | O(n log n): Σ n/i = n · H(n) ≈ n ln n (harmonic series) | O(1) |
| s18 | O(n²): each `pop(0)` shifts the remaining items | O(n): the copy `list(nums)` |
| s19 | O(√n): i runs up to √n | O(1) |
| s20 | O(n²): every step copies `result` (strings are immutable), and the prepend branch can't use CPython's in-place append optimization | O(n): `result` |

## Chapter 11: Python Performance Reality

1. Up to about 10⁵ × 0.7 ms ≈ 70 s (about half that on average, since the list grows).
   Use a `set`: about 10⁵ × 0.04 µs ≈ 4 ms.
2. `str.count` runs its whole scan in C. 26 C-level passes (about 30 ms measured) cost
   less than 10⁶ Python-level iterations (110–156 ms measured).
3. `sorted` (20–26 ms) ≪ merge sort (344–394 ms) < quicksort (429–490 ms), all
   O(n log n). The gaps are constant factors.
4. `q = deque(q)`, then `while q: q.popleft()`.
5. `getsizeof` counts only the list's own pointer array: 8 bytes per slot, either way.
   The first list's 1000 slots all point to one cached int `5`; the second points to
   1000 separate int objects of 28+ bytes each (36,056 bytes in total, measured).
6. `RecursionError` at the default limit of 1000. Fixes: rewrite as a loop (preferred,
   since linear data doesn't need recursion), or raise `sys.setrecursionlimit` inside
   the method (it works on 3.13 here, but deep recursion is fragile across versions).
7. `Counter` (76–96 ms vs 123–156 ms for a dict loop on 10⁶ characters). Both are
   O(n), so both pass whenever O(n) is the target. The constant factor rarely decides
   acceptance.
8. O(n · k) = 5 × 10⁹ element copies and additions: hopeless. Use prefix sums (Ch 15)
   or a sliding window (Ch 17): O(n).

## Chapter 12: Operation Complexity Cheat Sheet

1. O(n²): each iteration copies a prefix and scans it. Keep a `set` of values seen so
   far: O(n) average.
2. O(n²): each `pop(0)` shifts the list. Use `deque` and `popleft()`: O(n).
3. O(n²): every prepend copies `result`. Use `s[::-1]`, or append to a list, reverse it,
   and `"".join` it: O(n).
4. O(n · k): each window is copied and summed. Use a sliding window sum (Ch 17) or prefix
   sums (Ch 15): O(n).
5. O(n²) in the worst case (all distinct): list membership. Use a set; to keep first-seen
   order, `list(dict.fromkeys(nums))` gives `[3, 1, 2]` for `[3, 1, 3, 2, 1]`, in O(n).
6. O(n²): `min(nums[i:])` copies and scans the suffix every time. Precompute suffix
   minimums right to left: O(n).
7. Time is fine: O(len(a) + len(b)) to build both sets, then O(min) = O(10) to
   intersect. But `set(a)` stores 10⁶ items. `set(b).intersection(a)` scans `a` once
   with O(10) extra memory.
8. O(n²) in the worst case: each `insort` is O(n). If all values are inserted before any
   query, append them and sort once: O(n log n).
9. O(n log n), and it builds two lists. `Counter(s) == Counter(t)`, or two 26-slot counts:
   O(n).
10. O(N · L log L) for N words: sorting each word dominates. A 26-count tuple as the key
    is O(N · (L + 26)) (Ch 14).

## Chapter 13 practice: Array Fundamentals & In-Place

Each entry gives a hint first, then the solution idea. Read only the hint the first time.

**D.13.1: 1929 Concatenation of Array.**
*Hint:* The answer has length 2n, and index i and index i + n hold the same value.
*Solution:* `nums + nums`, or fill `ans[i] = ans[i + n] = nums[i]`. O(n) time; the
output is O(n) (no auxiliary space beyond it).

**D.13.2: 1480 Running Sum of 1d Array.**
*Hint:* Each answer is the previous answer plus one element.
*Solution:* A running total (see `code/ch00_front_matter/running_sum.py`), or in place:
`for i in range(1, n): nums[i] += nums[i - 1]`. O(n) time, O(1) auxiliary.
`itertools.accumulate` is the **[KNOW-HOW]** shortcut.

**D.13.3: 27 Remove Element.**
*Hint:* The 13.3 template with `keep(x) = x != val`.
*Solution:* `code/ch01_anatomy/remove_element.py`. O(n) time, O(1) auxiliary.

**D.13.4: 26 Remove Duplicates from Sorted Array.**
*Hint:* Compare each value with the last value you **kept**, not with its neighbor in
the input.
*Solution:* `code/ch01_anatomy/remove_duplicates.py`. O(n) time, O(1) auxiliary.

**D.13.5: 905 Sort Array By Parity.**
*Hint:* Any order is accepted, so stability doesn't matter.
*Solution:* A write pointer that swaps each even value forward (283's approach 3 with
`keep(x) = x % 2 == 0`), or two pointers from both ends that swap a misplaced odd value
on the left with a misplaced even value on the right (Ch 16). O(n) time, O(1) auxiliary.

**D.13.6: 118 Pascal's Triangle.**
*Hint:* Every row starts and ends with 1; each inner value is the sum of the two values
above it.
*Solution:* `row = [1] + [prev[j - 1] + prev[j] for j in range(1, len(prev))] + [1]`,
starting from `[1]`. O(numRows²) time, which is also the output size; O(1) auxiliary
beyond the output.

Reference solutions for these hints: `code/ch13_array_basics/practice_solutions.py`.

## Chapter 14 practice: Hashing & Frequency Counting

Reference solutions: `code/ch14_hashing/practice_solutions.py`.

**D.14.1: 217 Contains Duplicate.**
*Hint:* A set never holds two equal values.
*Solution:* `len(set(nums)) != len(nums)`, or stop early with a seen-set. O(n) average
time, O(n) space.

**D.14.2: 242 Valid Anagram.**
*Hint:* Equal lengths, equal letter counts.
*Solution:* One 26-slot list: +1 for each character of `s`, −1 for each of `t`, then
check that all slots are zero. O(n) time, O(1) space. Follow-up (any Unicode
characters): `Counter(s) == Counter(t)`, O(n) time and O(k) space. Ch 25 revisits this
problem three ways.

**D.14.3: 383 Ransom Note.**
*Hint:* Each letter of the note consumes one copy of that letter from the magazine.
*Solution:* Count the magazine's letters, decrement for each note letter, and fail if a
count goes negative. O(m + n) time, O(1) space. Shortcut:
`Counter(ransomNote) <= Counter(magazine)` (3.10+, Ch 8).

**D.14.4: 387 First Unique Character in a String.**
*Hint:* Two passes: count everything, then find the first character in **input order**
with count 1.
*Solution:* `Counter`, then `enumerate(s)`. O(n) time, O(1) space (26 letters).

**D.14.5: 205 Isomorphic Strings.**
*Hint:* The mapping must be consistent **both ways**: `"badc"`/`"baba"` is consistent
from s to t, but not from t to s.
*Solution:* Two dicts, updated with `setdefault`; fail on any conflict. O(n) time,
O(k) space.

**D.14.6: 290 Word Pattern.**
*Hint:* 205 with words instead of characters. Check the lengths first.
*Solution:* `words = s.split()`; if the counts differ, return False; then two dicts as
in 205. O(len(s)) time.

**D.14.7: 36 Valid Sudoku.**
*Hint:* Each filled digit belongs to exactly one row, one column and one 3 × 3 box.
Box index: `(r // 3, c // 3)`.
*Solution:* One set of tagged tuples: `("row", r, d)`, `("col", c, d)`,
`("box", r // 3, c // 3, d)`. Any repeat means invalid. Validity doesn't require the
board to be solvable. O(1) time and space for the fixed 9 × 9 board.

## Chapter 15 practice: Prefix Sums & Difference Arrays

Reference solutions: `code/ch15_prefix_sums/practice_solutions.py`.

**D.15.1: 303 Range Sum Query - Immutable.**
*Hint:* Do the work once in `__init__`. Every `sumRange` call should cost O(1).
*Solution:* Store `prefix` with a leading 0. `sumRange(left, right)` is inclusive, so
return `prefix[right + 1] - prefix[left]`. O(n) build, O(1) per query, O(n) space.

**D.15.2: 724 Find Pivot Index.**
*Hint:* If you know the total and the sum to the left of i, you know the sum to the
right.
*Solution:* Scan with a running `left`. At index i, check
`left == total - left - nums[i]`, then add `nums[i]`. Return the first match, else -1.
O(n) time, O(1) space.

**D.15.3: 523 Continuous Subarray Sum.**
*Hint:* Equal prefix remainders mod k mean a multiple of k in between. The length must
be at least 2, so you need **where** a remainder first appeared, not how often.
*Solution:* Map remainder → first index, seeded `{0: -1}`. On a repeat at index i,
return True if `i - first_index[r] >= 2`. Never overwrite an earlier index. O(n) time,
O(min(n, k)) space. `[0, 0]` is valid (0 is a multiple of k); `[0]` alone is too short.

## Chapter 16 practice: Two Pointers

**D.16.1: 344 Reverse String.**
*Hint:* Swap the two ends, then move both inward.
*Solution:* `code/ch01_anatomy/reverse_string.py`. O(n) time, O(1) space.

**D.16.2: 977 Squares of a Sorted Array.**
*Hint:* The largest square is at one of the two **ends**: big negatives on the left, big
positives on the right.
*Solution:* Two pointers at the ends; fill the output from the back with the larger
square, moving that pointer inward. O(n) time, which meets the follow-up (not "square
and sort", which is O(n log n)). See `code/ch16_two_pointers/practice_solutions.py`.

**D.16.3: 18 4Sum.**
*Hint:* 3Sum with one more fixed value: sort, two nested loops for the first two values,
two pointers for the rest, and skip duplicates at **every** level.
*Solution:* O(n³) time with n ≤ 200 (8 × 10⁶ steps at most); O(n) for the sorted copy.
Python ints make the 4 × 10⁹ sums safe. See the reference solution.

## Chapter 17 practice: Sliding Window

Reference solutions: `code/ch17_sliding_window/practice_solutions.py`.

**D.17.1: 121 Best Time to Buy and Sell Stock.**
*Hint:* For a sale on day i, the best buy is the cheapest day before i.
*Solution:* Track the running minimum and the best `price - minimum`. O(n) time, O(1)
space. (It's a window whose left edge jumps to each new minimum.)

**D.17.2: 904 Fruit Into Baskets.**
*Hint:* Two baskets means "at most 2 distinct values in the window".
*Solution:* The 992 helper `_at_most` with k = 2, returning the longest length instead of
a count: a dict of counts, shrink while more than 2 keys, delete zero-count keys. O(n)
time, O(1) space.

**D.17.3: 239 Sliding Window Maximum.**
*Hint:* A counter can't tell you the new maximum after the old one leaves. Keep the
candidates **with their indices**, and throw away the ones that have left the window.
*Solution here:* A max-heap of `(-value, index)`; pop from the top while its index is out
of the window. O(n log n) time, O(n) space. The optimal O(n) monotonic deque is in
Ch 21.

## Chapter 18 practice: Sorting as a Tool

Reference solutions: `code/ch18_sorting/practice_solutions.py`.

**D.18.1: 57 Insert Interval.**
*Hint:* The input is already sorted and disjoint, so you don't need to sort. Walk it in
three phases.
*Solution:* (1) Copy the intervals that end before the new one starts. (2) Absorb every
interval with `start <= new_end`, widening the new interval. (3) Append it, then the rest.
O(n) time. Touching intervals merge, as in 56.

**D.18.2: 435 Non-overlapping Intervals.**
*Hint:* Removing the fewest means keeping the most. Which interval should you keep
first?
*Solution:* Sort by **end**. Keep an interval when `start >= last_end`; touching is fine
here. Answer = n − kept. O(n log n). Exchange argument: the earliest-ending interval
leaves the most room, so some optimal solution includes it. The reference file checks
the greedy against brute force over all subsets.

**D.18.3: 179 Largest Number.**
*Hint:* Compare two numbers by which concatenation is larger: `a + b` vs `b + a`.
*Solution:* Ch 6, 6.2.6, with `cmp_to_key`; or a merge sort using that comparator when
the built-in is off limits. Watch the all-zeros case.

## Chapter 19 practice: Binary Search

Reference solutions: `code/ch19_binary_search/practice_solutions.py`.

**D.19.1: 704 Binary Search.**
*Hint:* Closed interval `[lo, hi]`, loop while `lo <= hi`, return on equality.
*Solution:* The classic loop. O(log n) time, O(1) space.

**D.19.2: 35 Search Insert Position.**
*Hint:* The insert position is the first index whose value is ≥ target.
*Solution:* Lower bound over `[0, len(nums)]` (the template in 19.3). O(log n).

**D.19.3: 74 Search a 2D Matrix.**
*Hint:* Each row continues the previous one, so the grid is one sorted list of
m × n cells.
*Solution:* Binary search over k in `[0, m·n − 1]`, reading `matrix[k // n][k % n]`.
O(log(m · n)).

**D.19.4: 153 Find Minimum in Rotated Sorted Array.**
*Hint:* Compare `nums[mid]` with `nums[hi]`, not `nums[lo]`.
*Solution:* If `nums[mid] > nums[hi]`, the minimum is right of `mid` (`lo = mid + 1`);
else it's at `mid` or left (`hi = mid`). O(log n).

**D.19.5: 1011 Capacity To Ship Packages Within D Days.**
*Hint:* Binary search on the capacity. What's the smallest capacity that could
possibly work, and what's one that surely works?
*Solution:* Search `[max(weights), sum(weights)]`. `days_needed(c)` is a greedy O(n)
pass (start a new day when the next package doesn't fit), and it never increases as c
grows. O(n log(sum)).

**D.19.6: 4 Median of Two Sorted Arrays (stretch).**
*Hint:* Don't merge. Choose how many elements i to take from the shorter array into the
"left half". Then j = half − i is forced. The partition is correct when both left
maxima are ≤ both right minima.
*Solution:* Binary search i over `[0, m]`, using ±∞ sentinels at the array edges.
O(log min(m, n)) time, O(1) space. Verified against merge-and-pick on 1,000 random
pairs.

## Chapter 20 practice: Kadane & Subarray Optimization

Reference solutions: `code/ch20_kadane/practice_solutions.py`.

**D.20.1: 122 Best Time to Buy and Sell Stock II.**
*Hint:* With unlimited transactions, a long rise from a to b earns the same as selling
and rebuying at every step in between.
*Solution:* Sum every positive `prices[i] - prices[i - 1]`. O(n) time, O(1) space. The
same answer comes from a two-state recurrence (best cash when holding nothing / when
holding one share), the DP view of this chapter. Both are checked against an exhaustive
search on 500 random price lists.

## Chapter 21 practice: Monotonic Stack & Deque

Reference solutions: `code/ch21_monotonic/practice_solutions.py`.

**D.21.1: 496 Next Greater Element I.**
*Hint:* Answer the question for **every** value of `nums2` once, then just look up the
`nums1` values.
*Solution:* One monotonic-stack pass over `nums2` builds `value → next greater` in a dict
(the values are distinct). Then `[d.get(x, -1) for x in nums1]`. O(m + n), which meets
the follow-up.

**D.21.2: 901 Online Stock Span.**
*Hint:* When today's price beats a previous day, it also beats every day that one
already covered.
*Solution:* A stack of `(price, span)` pairs, prices decreasing. Pop pairs with price ≤
today, adding their spans to today's (starting at 1). Push `(price, span)`. Amortized
O(1) per call.

**D.21.3: 735 Asteroid Collision.**
*Hint:* Only a left-mover (negative) arriving after a right-mover (positive) can
collide, and it hits the most recent survivor first.
*Solution:* A stack of survivors. For a negative asteroid, pop smaller positive tops;
on equal size both vanish; a bigger top destroys the incoming one. Push it if it
survives. O(n). Checked against a step-by-step simulation on 1,000 random inputs.

## Chapter 22 practice: Matrices

Reference solutions: `code/ch22_matrices/practice_solutions.py`.

**D.22.1: 867 Transpose Matrix.**
*Hint:* The matrix may not be square, so build a new n × m grid; don't swap in place.
*Solution:* `[[matrix[r][c] for r in range(m)] for c in range(n)]`, or
`[list(col) for col in zip(*matrix)]`. O(m · n) time; the output is the only extra
memory.

**D.22.2: 240 Search a 2D Matrix II.**
*Hint:* Start at a corner where one direction only increases and the other only
decreases.
*Solution:* Start at the top-right. If the value is too big, move left (the whole column
below is bigger still); if too small, move down (the whole row to the left is smaller
still). O(m + n) time, O(1) space. Checked on 500 random sorted grids.

## Chapter 23 practice: Index Tricks & Voting

Reference solutions: `code/ch23_index_tricks/practice_solutions.py`.

**D.23.1: 268 Missing Number.**
*Hint:* You know exactly which numbers **should** be there: 0..n.
*Solution:* XOR all indices 0..n with all values; the pairs cancel, leaving the missing
value. Or `n * (n + 1) // 2 - sum(nums)` (no overflow in Python). O(n) time, O(1)
space.

**D.23.2: 136 Single Number.**
*Hint:* What does `x ^ x` equal? Does order matter for XOR?
*Solution:* XOR every element; the pairs cancel. O(n) time, O(1) space. Works with
negatives.

**D.23.3: 448 Find All Numbers Disappeared in an Array.**
*Hint:* Values are 1..n, so value v can "mark" slot v − 1 without losing the slot's own
value, if you only change its sign.
*Solution:* For each x, make `nums[abs(x) - 1]` negative. Afterwards, every index i whose
value is still positive means i + 1 never appeared. O(n) time, O(1) auxiliary (output
excluded), which meets the follow-up.

## Chapter 24 practice: String Fundamentals

Reference solutions: `code/ch24_strings/practice_solutions.py`.

**D.24.1: 58 Length of Last Word.**
*Hint:* Start at the end. Skip trailing spaces first, then count.
*Solution:* Two backward loops: skip spaces, then count letters until a space or the
start. O(n) worst case, O(1) space. Shortcut: `len(s.split()[-1])`, which is O(n) space.

**D.24.2: 392 Is Subsequence.**
*Hint:* Match each character of `s` at the **earliest** possible position in `t`.
*Solution:* One pass over `t`, advancing an index into `s` on each match. O(len(t)) time,
O(1) space. Follow-up (huge numbers of `s` for one `t`): precompute, for each letter, the
sorted list of its positions in `t`; for each `s`, binary-search the next position after
the current one (`bisect_right`). O(len(s) · log len(t)) per query.

**D.24.3: 1768 Merge Strings Alternately.**
*Hint:* One index, two strings; append whichever still has a character at that index.
*Solution:* Collect the characters in a list and `"".join` them. O(m + n) time.

## Chapter 25 practice: Hashing on Strings

**D.25.1: 49 Group Anagrams (revisit).**
*Hint:* Pick the key consciously this time: a sorted string (any alphabet) or a 26-count
tuple (lowercase only). Which is faster depends on the string length (measured in Ch 14, 14.7).
*Solution:* `code/ch14_hashing/group_anagrams.py`. O(N · L log L) with sorted keys,
O(N · (L + 26)) with count keys.

**D.25.2: 567 Permutation in String.**
*Hint:* This is 438 asking "does at least one window exist?".
*Solution:* A sliding 26-count window of size `len(s1)` over `s2`; return True on the
first window whose counts equal `s1`'s. O(26 · n) time, O(1) space. See
`code/ch25_string_hashing/practice_solutions.py` (checked against all permutations on 500
random cases).

## Chapter 26 practice: Palindromes

**D.26.1: 409 Longest Palindrome.**
*Hint:* You choose the order. Each pair of equal letters can be mirrored; how many
unpaired letters can a palindrome hold?
*Solution:* Add `count // 2 * 2` for every letter (case-sensitive), then add 1 if any count
was odd (one middle letter). O(n) time, O(1) space. See
`code/ch26_palindromes/practice_solutions.py` (checked against brute force over all
arrangements).

## Chapter 27 practice: Sliding Window on Strings

**D.27.1: 1456 Maximum Number of Vowels in a Substring of Given Length.**
*Hint:* A fixed window of size k: one character enters, one leaves.
*Solution:* Count the vowels in the first k characters, then slide:
`count += (entering is a vowel) - (leaving is a vowel)`, tracking the maximum. O(n) time,
O(1) space. See `code/ch27_string_windows/practice_solutions.py`.

## Chapter 28 practice: Parsing, Simulation & Stacks

Reference solutions: `code/ch28_parsing/practice_solutions.py`.

**D.28.1: 20 Valid Parentheses.**
*Hint:* The most recently opened bracket must close first.
*Solution:* Push openers. A closer must match the popped top (fail if the stack is
empty). At the end, the stack must be empty. O(n).

**D.28.2: 1047 Remove All Adjacent Duplicates In String.**
*Hint:* After a removal, the characters on either side become adjacent. What structure
remembers "the character just before"?
*Solution:* A stack of survivors: pop on equality with the top, otherwise push. Then
`"".join(stack)`. O(n).

**D.28.3: 67 Add Binary.**
*Hint:* Column addition from the right with a carry, base 2.
*Solution:* `total = carry + bits`; append `total % 2`; `carry = total // 2`; reverse at
the end. O(max(m, n)).

**D.28.4: 415 Add Strings.**
*Hint:* 67 in base 10. No `int()` on the inputs.
*Solution:* The 28.3 template. O(max(m, n)).

**D.28.5: 71 Simplify Path.**
*Hint:* Split on `/`. Which pieces change the directory stack, and which are ignored?
*Solution:* Skip `""` and `"."`; `".."` pops (if possible); anything else, including
`"..."`, is pushed. Answer `"/" + "/".join(stack)`. O(n).

**D.28.6: 6 Zigzag Conversion.**
*Hint:* Simulate the pen: the row index goes 0, 1, …, numRows − 1, then back up.
*Solution:* One list per row; append each character to the current row and bounce
direction at the top and bottom rows. Handle `numRows == 1`. O(n).

**D.28.7: 38 Count and Say.**
*Hint:* Each term run-length encodes the previous term: "count, then digit" for each run.
*Solution:* Iterate n − 1 times with `itertools.groupby` (or a manual run scan). That's
the follow-up's iterative version. O(total length of all terms).

## Chapter 29 practice: String Matching

**D.29.1: 686 Repeated String Match.**
*Hint:* If b fits inside some number of copies of a, how many copies can that possibly
take? Think about where b can start inside the first copy.
*Solution:* Let q = ⌈len(b) / len(a)⌉. b occurs in `a * q` or in `a * (q + 1)`, or never:
starting partway into the first copy needs at most one extra copy. Two KMP searches
(or `in`): O(n + m). See `code/ch29_string_matching/practice_solutions.py` (checked
against trying up to 15 copies).

## Chapter 30: Pattern Recognition

| # | Pattern | Target | Book problem |
|---|---|---|---|
| 1 | Two pointers from both ends | O(n) time, O(1) space | 167 (Ch 16) |
| 2 | Prefix sum + hash map of counts | O(n) | 560 (Ch 15) |
| 3 | Sliding window + last index | O(n) | 3 (Ch 27) |
| 4 | Monotonic decreasing stack | O(n) | 739 (Ch 21) |
| 5 | Count + bucket sort (or heap) | O(n) (or O(n log k)) | 347 (Ch 14) |
| 6 | Floyd's cycle detection (or binary search on value) | O(n) (or O(n log n)), O(1) space | 287 (Ch 23) |
| 7 | Binary search on the answer | O(n log max) | 875 (Ch 19) |
| 8 | Transpose + reverse rows (or 4-way swaps) | O(n²), O(1) space | 48 (Ch 22) |
| 9 | Kadane | O(n), O(1) space | 53 (Ch 20) |
| 10 | Sort by start, sweep | O(n log n) | 56 (Ch 18) |
| 11 | Count signature | O(n) | 242 (Ch 14, 25) |
| 12 | Sliding window, need/have | O(m + n) | 76 (Ch 27) |
| 13 | Stack of signed terms | O(n) | 227 (Ch 28) |
| 14 | Monotonic deque | O(n) | 239 (Ch 21) |
| 15 | Cyclic sort | O(n), O(1) space | 41 (Ch 23) |
| 16 | Prefix function (KMP) | O(n) | 1392 (Ch 29) |
| 17 | Write pointer with swaps | O(n), O(1) space | 283 (Ch 13) |
| 18 | Sliding 26-count signature | O(n) | 567 (Ch 25) |
| 19 | Binary search on the answer + greedy check | O(n log sum) | 1011 (Ch 19) |
| 20 | Expand around 2n − 1 centers | O(n²) | 647 (Ch 26) |

## Chapter 32: Find the Bug

Fixes are verified in `code/ch32_debugging/exercises.py`.

| # | Exposing input | Bug | Fix |
|---|---|---|---|
| E1 | `[-3, -1]` → 0 | Initial 0 beats every negative value | `best = nums[0]` |
| E2 | Set `g[0][0]` → both rows change | `[[0] * n] * m` repeats one row | `[[0] * n for _ in range(m)]` |
| E3 | `(-3, 0)` → −2 | `//` floors | Truncate: `abs(total) // 2` with the sign restored |
| E4 | `[2, 2, 3]` → `[2, 3]` | Removing while iterating skips elements | `nums[:] = [x for x in nums if x != val]` |
| E5 | Any list: unchanged | Rebinds the local name | Mutate: `s.reverse()` or `s[:] = s[::-1]` |
| E6 | `"a  b"` → `['a', '', 'b']` | `split(" ")` keeps empty strings | `text.split()` |
| E7 | `[5, 6, 7]` → indices `[2, 1]` | Stop 0 excludes index 0 | `range(len(nums) - 1, -1, -1)` |
| E8 | `"abc"` → `'c'` | −1 indexes the last character | Check `i != -1` first |
| E9 | `[3]`, k = 6 → 1 | Inserting before looking pairs x with itself | Look up first, then insert |
| E10 | `[5, 6]`, target 5 → False | Index 0 is falsy | `idx is not None` |
