# Chapter 3: Core Python Semantics That Decide DSA Correctness

## 3.0 At a glance

The bugs in this chapter don't crash. They produce a plausible wrong answer: a grid
where every row changes at once, a "last digit" of 7 for -123, a division that is
off by one for negative numbers, a swap that silently does nothing. Interviewers know
these traps, and the judge's hidden tests are full of negative numbers and edge
sizes. Learn the rules once, here, and later chapters assume them.

Tools in this chapter: `enumerate`, `zip`, `range`, `divmod`, `abs`, `copy.deepcopy`,
`math.inf` (all **[USE-FREELY]**); `reversed` **[KNOW-HOW]**.

## 3.1 Complete reference

### Semantics that change answers

| Topic | Python's rule | DSA consequence | Section |
|---|---|---|---|
| Assignment | `b = a` binds a second name to the **same object** | Mutating through `b` changes `a` | 3.2.1 |
| Mutability | `list`, `dict`, `set` mutable; `int`, `float`, `str`, `tuple`, `frozenset` immutable | Strings can't be edited in place; tuples can be dict keys | 3.2.2 |
| `[x] * n` | Repeats **references** to the same `x` | `[[0]*m]*n` is n references to one row | 3.2.3 |
| Copying | `a[:]`, `list(a)`, `a.copy()` are **shallow** | Inner lists stay shared | 3.2.4 |
| Integers | Unbounded; no overflow | Problems that "assume 32-bit" need manual checks | 3.2.5 |
| `//`, `%` | Floor division; `%` takes the sign of the divisor | `-7 // 2 == -4`, `-123 % 10 == 7` | 3.2.6 |
| Floats | IEEE-754 doubles; `inf`, `nan` exist | `0.1 + 0.2 != 0.3`; big ints lose precision as floats | 3.2.7 |
| Truthiness | `0`, `""`, `[]`, `{}`, `set()`, `None` are falsy | `if idx:` is wrong when `idx` can be 0 | 3.2.8 |
| Tuple assignment | Right side evaluated first; targets assigned **left to right** | Some index swaps silently do nothing | 3.2.9 |

### Mutability and hashability of built-in types

| Type | Mutable? | Hashable (usable as dict key / set element)? |
|---|---|---|
| `int`, `float`, `bool`, `str` | No | Yes |
| `tuple` | No | Yes, if all its elements are hashable |
| `frozenset` | No | Yes |
| `list`, `dict`, `set`, `bytearray` | Yes | No |

### Copy operations

n = number of top-level elements; N = total number of objects reachable.

| Operation | Depth | Time (reason) | Auxiliary space |
|---|---|---|---|
| `b = a` | None: same object | O(1): binds a name | O(1) |
| `a[:]`, `list(a)`, `a.copy()`, `copy.copy(a)` | Shallow | O(n): copies n references | O(n) |
| `[row[:] for row in grid]` | Two levels | O(total cells): copies every row | O(total cells) |
| `copy.deepcopy(a)` | Full | O(N): visits every reachable object and records it in a memo dict | O(N) |

## 3.2 Explanations by group

### 3.2.1 Names, objects, references

A variable is a **name** attached to an **object**. Assignment attaches a name; it
never copies:

```python
a = [1, 2]
b = a                  # second name, same list
b.append(3)
a  # -> [1, 2, 3]
a is b  # -> True
```

`==` compares values; `is` compares identity (same object). Use `is` only for
`None` (`if node is None`). Whether two equal ints or strings are the same object is
a CPython implementation detail; never rely on it.

### 3.2.2 Mutability: list vs str vs tuple

A mutable object can change after creation; an immutable one can't. Every "change"
to an immutable object creates a new object:

```python
s = "abc"
s[0] = "x"  # -> TypeError: 'str' object does not support item assignment
```

To build a modified string, collect characters in a list and `"".join` them (Ch 5,
Ch 24), or slice: `"x" + s[1:]` (O(n), because it builds a new string).

A tuple can't be reassigned, but **objects inside it** can still change:

```python
pair = (1, [2])
pair[1].append(3)
pair  # -> (1, [2, 3])
pair[0] = 5  # -> TypeError: 'tuple' object does not support item assignment
```

Immutability is what makes tuples, strings and frozensets **hashable**, so they can be
dict keys and set elements. `(row, col)` tuples as keys, sorted-string keys for
anagrams (Ch 14), `tuple(counts)` as a signature (Ch 25): all rely on this.

### 3.2.3 Aliasing bugs

**The grid trap.** `[x] * n` repeats a reference n times. For an immutable `x`, like
`0`, that's harmless, because you can only replace an element, never mutate it. For a
list it's a bug:

```python
grid = [[0] * 3] * 2
grid[0][0] = 1
grid  # -> [[1, 0, 0], [1, 0, 0]]
```

```text
grid = [[0] * 3] * 2                 grid = [[0] * 3 for _ in range(2)]

grid ──► [ • , • ]                   grid ──► [ • , • ]
           │   │                                │   │
           └─┬─┘                                ▼   ▼
             ▼                          [0, 0, 0]   [0, 0, 0]
         [0, 0, 0]                       two separate row lists
     one row list, two references
```

The fix creates a new row per iteration:

```python
grid = [[0] * 3 for _ in range(2)]
grid[0][0] = 1
grid  # -> [[1, 0, 0], [0, 0, 0]]
```

`[0] * 3` inside the comprehension is fine: its elements are ints.

**Appending a list you keep mutating.** In backtracking you build `path` step by step
and record finished paths. Appending `path` itself records a reference that keeps
changing:

```python
result = []
path = []
for x in [1, 2]:
    path.append(x)
    result.append(path)        # same object every time
result  # -> [[1, 2], [1, 2]]
```

Record a snapshot instead: `result.append(path[:])`, which gives `[[1], [1, 2]]`.

**Mutable default arguments** follow the same rule: the default object is created
once (Ch 1, Gotcha 5).

### 3.2.4 Copies

All shallow copies make a new outer list holding the **same** inner objects:

```python
import copy

rows = [[1, 2], [3, 4]]
shallow = rows[:]
deep = copy.deepcopy(rows)
rows[0].append(99)
shallow  # -> [[1, 2, 99], [3, 4]]
deep  # -> [[1, 2], [3, 4]]
```

For a list of immutable values (ints, strings), a shallow copy is a complete copy.
For a grid, `[row[:] for row in grid]` is enough, because the cells are ints.
`copy.deepcopy` handles any nesting, but it does more work per object (recursion,
plus a memo dict to handle shared references). It's slower than a row-by-row copy
(general knowledge, not benchmarked here), and rarely needed in array problems.

### 3.2.5 Integers: no overflow, and problems that pretend otherwise

Python ints grow as needed:

```python
2**64 + 1  # -> 18446744073709551617
```

Many LeetCode problems were written for languages with fixed-width ints, and they
encode that limit in the statement:

| Problem | What the statement requires |
|---|---|
| 7 Reverse Integer | Return 0 if the reversed value leaves the signed 32-bit range; assume you can't store 64-bit values |
| 8 String to Integer (atoi) | Clamp the parsed value to the signed 32-bit range (solved in Ch 28) |

In Python you must enforce these limits yourself: nothing overflows for you.

Problem 7 (<https://leetcode.com/problems/reverse-integer/>): reverse the digits of
a 32-bit signed integer, and return 0 if the result doesn't fit in 32 bits. LeetCode's
signature is `def reverse(self, x: int) -> int:`, and the book's is identical.

The string shortcut is correct in Python, but it builds the oversized value before
checking it, which is exactly what the statement says you can't do:

<!-- file: code/ch03_semantics/reverse_integer.py -->

```python
class SolutionString:
    def reverse(self, x: int) -> int:
        # O(d) time and O(d) auxiliary space, d = number of digits (<= 10 here):
        # str() builds d characters, slicing copies them, int() parses them.
        sign = -1 if x < 0 else 1
        result = sign * int(str(abs(x))[::-1])
        return result if INT_MIN <= result <= INT_MAX else 0
```

The interview version pops digits and checks **before** the multiply:

<!-- file: code/ch03_semantics/reverse_integer.py -->

```python
class Solution:
    def reverse(self, x: int) -> int:
        # O(d) time: one iteration per digit. O(1) auxiliary space.
        # Work on abs(x) so % and // behave like "last digit" and "drop last digit";
        # with a negative x they don't (see Ch 3, 3.2.5).
        sign = -1 if x < 0 else 1
        limit = INT_MAX if sign == 1 else -INT_MIN     # 2147483647 or 2147483648
        rest = abs(x)
        result = 0
        while rest:
            digit = rest % 10
            rest //= 10
            # result * 10 + digit > limit, rearranged so nothing exceeds limit
            if result > (limit - digit) // 10:
                return 0
            result = result * 10 + digit
        return sign * result
```

`d` is at most 10 here, so both are effectively O(1). The test file covers 0, single
digits, trailing zeros, both 32-bit extremes, and values just inside and just
outside the range.

**Why `abs(x)` first.** Digit extraction with `%` and `//` assumes a non-negative
number. With a negative number, both operators round toward negative infinity
(3.2.6):

```python
-123 % 10  # -> 7
-123 // 10  # -> -13
```

`7` isn't a digit of -123. Worse, `x //= 10` on a negative `x` never reaches 0:

```python
x = -123
steps = []
for _ in range(5):
    x //= 10
    steps.append(x)
steps  # -> [-13, -2, -1, -1, -1]
```

So `while x: x //= 10` is an **infinite loop** for any negative `x`.

**Int ↔ str limit (3.11+).** Converting a decimal string of more than 4300 digits to
an `int`, or the reverse, raises `ValueError`. The limit exists because decimal
conversion uses a non-linear algorithm, and huge inputs could stall a server (per the
`stdtypes` docs; security releases of some earlier versions have it too). Bases 2, 4,
8, 16 and 32 aren't limited:

```python
str(10**5000)  # -> ValueError: Exceeds the limit (4300 digits) for integer string...
```

This matters for "string arithmetic" problems (Ch 28): even where a problem allows
`int()`, inputs can be long enough to hit the limit. Those problems forbid `int()`
anyway, and that's the real reason to hand-write the arithmetic.

### 3.2.6 `//` and `%` with negatives

Python's `//` rounds toward **negative infinity** (floor), and `a % b` takes the sign
of `b`, so that `a == (a // b) * b + a % b` always holds. C, C++, Java, and every
LeetCode statement that says "truncate toward zero" round toward **zero** instead.

| `a`, `b` | `a // b` (floor) | `a % b` | Truncated toward zero | Agrees? |
|---|---|---|---|---|
| 7, 2 | 3 | 1 | 3 | Yes |
| -7, 2 | -4 | 1 | -3 | **No** |
| 7, -2 | -4 | -1 | -3 | **No** |
| -7, -2 | 3 | -1 | 3 | Yes |
| 6, -3 | -2 | 0 | -2 | Yes (exact division) |

They disagree exactly when the signs differ and the division isn't exact. Checked:

```python
(-7 // 2, -7 % 2)  # -> (-4, 1)
divmod(-7, 2)  # -> (-4, 1)
(7 // -2, 7 % -2)  # -> (-4, -1)
```

**Truncating division in Python.** Problems like 150 Evaluate Reverse Polish
Notation (<https://leetcode.com/problems/evaluate-reverse-polish-notation/>) say
division truncates toward zero. Two ways:

<!-- file: code/ch03_semantics/trunc_div.py -->

```python
def trunc_div(a: int, b: int) -> int:
    """a / b rounded toward zero, exact for any size of int. O(1) for word-size ints."""
    q = abs(a) // abs(b)
    return q if (a >= 0) == (b > 0) else -q


def trunc_div_float(a: int, b: int) -> int:
    """Common shortcut. Exact only while |a| and |b| stay below 2**53."""
    return int(a / b)
```

`int(a / b)` is common and fine for LeetCode-sized values (up to about 2⁵³ ≈ 9 × 10¹⁵).
Beyond that, `a / b` rounds to a float first:

```python
big = 10**18 + 1
int(big / 1)  # -> 1000000000000000000
```

`trunc_div` stays exact at any size. Use it when values can exceed 2⁵³, or when you'd
rather not think about it.

**`%` is useful because of the sign rule.** `i % n` is always in `[0, n)` for positive
`n`, even when `i` is negative. That's what makes circular indexing work: `(i - 1) % n`
from `i = 0` gives `n - 1` (used in Ch 21's circular next-greater element).

### 3.2.7 Floats, infinity, and mixed comparisons

Floats are IEEE-754 doubles: 53 bits of precision.

```python
0.1 + 0.2 == 0.3  # -> False
0.1 + 0.2  # -> 0.30000000000000004
```

| Need | Use | Why |
|---|---|---|
| A "larger than anything" sentinel | `math.inf` or `float('inf')` | Compares correctly with any int: `10**400 < math.inf` is `True` |
| Exact arithmetic | Keep everything in `int` | Ints are exact at any size; floats round |
| Compare two computed floats | `math.isclose(a, b)` | Rounding error makes `==` unreliable |
| An averaging problem (643) | Compare sums, divide once at the end | Fewer roundings |

**int vs float comparisons are exact.** Python compares the true mathematical values;
it doesn't convert the int to a float first:

```python
big = 10**20 + 1
big == float(big)  # -> False
float(big) == 10**20  # -> True
```

`float(big)` rounded the `+ 1` away. Mixing ints above 2⁵³ with floats loses
information in the conversion, not in the comparison.

**Infinity arithmetic has holes:**

```python
import math

math.inf - math.inf  # -> nan
math.nan == math.nan  # -> False
float(10**400)  # -> OverflowError: int too large to convert to float
```

`nan` is never equal to anything, itself included. If a `min`/`max` result turns into
`nan`, look for `inf - inf` or `0 * inf`.

### 3.2.8 Truthiness, chained comparisons, short-circuiting

**Falsy values:** `False`, `None`, `0`, `0.0`, `""`, `[]`, `()`, `{}`, `set()`,
`range(0)`. Everything else is truthy.

```python
nums = []
"empty" if not nums else "has items"  # -> 'empty'
```

**The `0` trap.** When a variable can legitimately be 0 (an index, a count, a value),
`if x:` treats 0 like "missing":

```python
def find(nums: list[int], target: int) -> int | None:
    for i, num in enumerate(nums):
        if num == target:
            return i
    return None


idx = find([5, 6], 5)
"found" if idx else "missing"  # -> 'missing'
"found" if idx is not None else "missing"  # -> 'found'
```

**Chained comparisons.** `a < b < c` means `a < b and b < c`, with `b` evaluated once.
It's the clean bounds check for grids:

```python
rows, cols = 3, 4
r, c = 2, 4
0 <= r < rows and 0 <= c < cols  # -> False
```

It also allows odd chains, like `1 < 3 > 2` (True). Don't write those; they confuse
readers.

**Short-circuiting.** `and` stops at the first falsy operand and `or` at the first
truthy one. Each returns that **operand**, not a bool:

```python
[] or "default"  # -> 'default'
0 and 1 / 0  # -> 0
```

The second line never divides. Put the bounds check first, so the index access only
runs when it's safe: `if i < len(nums) and nums[i] > 0:`.

### 3.2.9 Unpacking, swapping, starred assignment, walrus

```python
a, b = 1, 2
a, b = b, a                    # swap: right side becomes (2, 1) first
(a, b)  # -> (2, 1)
first, *rest = [10, 20, 30]
(first, rest)  # -> (10, [20, 30])
*init, last = "abc"
(init, last)  # -> (['a', 'b'], 'c')
```

The starred name always gets a **list** (even from a string), and it can be empty.

**The swap trap.** In `x, y = expr1, expr2`, Python evaluates the right side
completely, then assigns the targets **left to right**. If a target's index depends
on a value that an earlier target just changed, the swap breaks:

```python
nums = [1, 0]
i = 0
nums[i], nums[nums[i]] = nums[nums[i]], nums[i]
nums  # -> [1, 0]
```

Expected `[0, 1]`. Here's what happened:
1. The right side became `(0, 1)`.
2. `nums[0] = 0` ran.
3. `nums[nums[0]]` was now `nums[0]`, so `nums[0] = 1`.

This exact bug appears in cyclic sort (Ch 23). Fix it by computing the index first:

```python
nums = [1, 0]
i = 0
j = nums[i]
nums[i], nums[j] = nums[j], nums[i]
nums  # -> [0, 1]
```

**Walrus `:=` (3.8+, PEP 572)** assigns inside an expression. It's handy in `while`
conditions and comprehensions:

```python
data = [3, 1, 4, 1, 5]
it = iter(data)
total = 0
while (x := next(it, None)) is not None:
    total += x
total  # -> 14
```

Readability first: use it when it removes a duplicated computation, not to save a
line.

### 3.2.10 `enumerate`, `zip`, `range`, iterating backwards

| Tool | Gives you | Note |
|---|---|---|
| `enumerate(seq, start=0)` | `(index, value)` pairs | Never write `for i in range(len(seq)): x = seq[i]` when you need both |
| `zip(a, b)` | Pairs, stopping at the **shorter** input | Silent truncation hides bugs |
| `zip(a, b, strict=True)` (3.10+) | Same, but raises if lengths differ | Use when lengths must match |
| `range(start, stop, step)` | Lazy integers, stop excluded | O(1) memory, regardless of length |
| `reversed(seq)` **[KNOW-HOW]** | Lazy reverse iterator | No copy; needs a sequence, not a generator |
| `range(n - 1, -1, -1)` | n-1 down to 0 | The stop is -1, not 0, because stop is excluded |

```python
list(zip([1, 2, 3], "ab"))  # -> [(1, 'a'), (2, 'b')]
list(zip([1, 2], [1], strict=True))  # -> ValueError: zip() argument 2 is shorter...
list(range(4, -1, -2))  # -> [4, 2, 0]
list(reversed(range(4)))  # -> [3, 2, 1, 0]
list(range(5, 0))  # -> []
```

The last line is a common bug: a backwards range needs a negative step.

### 3.2.11 `match` (3.10+), briefly

Structural pattern matching (PEP 634) dispatches on the shape of a value. In array and
string problems it occasionally tidies up a parser (Ch 28). An `if`/`elif` chain is
always an acceptable alternative.

```python
def apply(op: str, a: int, b: int) -> int:
    match op:
        case "+":
            return a + b
        case "-":
            return a - b
        case "*":
            return a * b
        case _:
            raise ValueError(op)


apply("*", 6, 7)  # -> 42
```

## 3.3 Gotchas & traps

**1. `[[0] * m] * n`.** One row object, n references (3.2.3). Use a comprehension.

**2. `result.append(path)` while `path` keeps changing.** Append `path[:]` (3.2.3).

**3. Shallow copy of a grid.** `grid[:]` shares the rows. Use `[row[:] for row in grid]`.

**4. `x % 10` and `x // 10` on negative `x`.** The last digit of -123 comes out as 7, and
`while x: x //= 10` never ends. Work on `abs(x)` (3.2.5).

**5. `//` where the statement says "truncate toward zero".** Wrong for mixed signs.
Use `trunc_div` or `int(a / b)` (3.2.6).

**6. `int(a / b)` with huge values.** Exact only up to about 2⁵³ (3.2.6).

**7. Assuming overflow happens.** Nothing wraps around in Python. If the statement
says "return 0 on 32-bit overflow", check the range yourself (3.2.5).

**8. `if idx:` when `idx` can be 0.** Use `is not None` (3.2.8).

**9. Float equality.** `0.1 + 0.2 != 0.3`. Stay in integers, or use `math.isclose`
(3.2.7).

**10. Self-referencing swap** `nums[i], nums[nums[i]] = ...`. Save the index first
(3.2.9).

**11. `zip` truncating silently.** Use `strict=True` on 3.10+ (3.2.10).

**12. Backward `range` without a negative step.** `range(5, 0)` is empty (3.2.10).

**13. Using `is` for value comparison.** `is` checks identity. Use it only with
`None` (3.2.1).

## 3.4 Benchmarks

None run in this chapter. One relative-speed remark (deepcopy versus row copies,
3.2.4) is labeled as general knowledge. Copy costs are measured in Ch 11.

## 3.5 Exercises

Answers are in Appendix D (verified in `code/ch03_semantics/exercises.py`).

**Warm-up**

1. `a = [1, 2]; b = a; b += [3]`. What is `a`?
2. `grid = [[0] * 3] * 2; grid[0][0] = 1`. What is `grid`? Fix the construction.
3. Evaluate: `-7 // 2`, `-7 % 2`, `int(-7 / 2)`, `divmod(-7, 2)`.

**Core**

4. Write `trunc_div(a, b)` that rounds toward zero using only integer operations.
5. `rows = [[1, 2], [3, 4]]`, `shallow = rows[:]`, `deep = copy.deepcopy(rows)`, then
   `rows[0].append(99)`. What are `shallow` and `deep`?
6. Write `last_digit(x)` that returns 3 for both 123 and -123. What does `-123 % 10`
   return, and why?
7. Is `10**20 + 1 == float(10**20 + 1)` true? Explain where the information is lost.
8. What does `list(zip([1, 2], [1], strict=True))` do on 3.10+? And without
   `strict`?

**Tricky**

9. Write three ways to iterate `i` from `n - 1` down to 0. Which one copies?
10. Explain why `nums[i], nums[nums[i]] = nums[nums[i]], nums[i]` leaves
    `nums = [1, 0]` unchanged for `i = 0`, and write a correct swap.

## 3.6 Chapter cheat sheet

| Rule | Detail |
|---|---|
| Assignment never copies | `b = a` → same object; `is` checks identity |
| Grids | `[[0] * m for _ in range(n)]`, never `[[0] * m] * n` |
| Copies | `a[:]` shallow; grid → `[r[:] for r in grid]`; `deepcopy` for arbitrary nesting |
| Snapshots | `result.append(path[:])` |
| Ints | No overflow. Enforce "32-bit" limits by hand. Int ↔ str limited to 4300 digits (3.11+) |
| `//` / `%` | Floor; `%` has the divisor's sign. Digits: use `abs(x)` |
| Truncation | `abs(a) // abs(b)`, then fix the sign; `int(a / b)` only below 2⁵³ |
| Floats | `math.inf` as a sentinel; never `==` on computed floats; int/float comparisons are exact |
| Truthiness | `if not nums` for empty; `is None` when 0 is valid |
| Swaps | Right side first, targets left to right; save computed indices first |
| Iteration | `enumerate`; `zip(strict=True)` (3.10+); `range(n - 1, -1, -1)` |
