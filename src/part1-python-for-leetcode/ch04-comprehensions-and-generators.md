# Chapter 4: Comprehensions & Generators

## 4.0 At a glance

Comprehensions are the idiomatic way to build a list, dict or set from another
iterable in one readable expression. Generator expressions do the same thing lazily,
one item at a time, without building the whole collection. You'll use both in almost
every solution: building grids, counting, filtering, checking conditions with
`any`/`all`.

This chapter covers:

- the four forms and how to read them
- filters versus conditional expressions (the two different `if`s)
- nested comprehensions, flattening, and grids
- generators: laziness, memory, single use, short-circuiting
- when *not* to use a comprehension
- measured performance on this book's machine

Tools: list/dict/set comprehensions, generator expressions, `sum`, `any`, `all`,
`zip`, `map`, `filter` (all **[USE-FREELY]**); `itertools.accumulate` **[KNOW-HOW]**.

## 4.1 Complete reference

### The four forms

| Form | Syntax | Produces | Evaluated |
|---|---|---|---|
| List comprehension | `[expr for x in it]` | `list` | Eagerly: the whole list is built at once |
| Set comprehension | `{expr for x in it}` | `set` | Eagerly |
| Dict comprehension | `{key: value for x in it}` | `dict` | Eagerly |
| Generator expression | `(expr for x in it)` | generator (an iterator) | Lazily: one item per `next()` |

There is **no tuple comprehension**. `(x for x in it)` is a generator; write
`tuple(x for x in it)` for a tuple.

### Clauses

| Clause | Position | Meaning |
|---|---|---|
| `for x in it` | After the expression; can repeat | Loops, read **left to right = outer to inner** |
| `if cond` (filter) | After a `for` | Keep only items where `cond` is true |
| `a if cond else b` (conditional expression) | **Inside** the expression, before `for` | Choose a value per item; keeps every item |

### Complexity

For an input of n items, where the expression and condition cost O(1):

| Construct | Time (reason) | Auxiliary space (reason) |
|---|---|---|
| `[f(x) for x in it]` | O(n): one evaluation per item | O(n): the result list (it *is* the output, so for a returned list it's output space) |
| `{...}` set or dict | O(n) average: one hash insert per item | O(n): the result |
| `(f(x) for x in it)` consumed by `sum`/`max`/... | O(n): each item produced once | O(1): only the current item exists |
| `any(...)` / `all(...)` over a generator | O(k): stops at the first decisive item k | O(1) |
| `[x for row in grid for x in row]` | O(R · C): every cell once | O(R · C) |

## 4.2 Explanations by group

### 4.2.1 List, set and dict comprehensions

A comprehension is a `for` loop that produces a collection:

```python
squares = []
for x in range(5):
    squares.append(x * x)
squares  # -> [0, 1, 4, 9, 16]
[x * x for x in range(5)]  # -> [0, 1, 4, 9, 16]
```

Sets and dicts use braces:

```python
nums = [3, -1, 4, -1, 5]
{x % 3 for x in nums}  # -> {0, 1, 2}
{c: ord(c) for c in "abc"}  # -> {'a': 97, 'b': 98, 'c': 99}
{v: k for k, v in {"a": 1, "b": 2}.items()}  # -> {1: 'a', 2: 'b'}
```

`-1 % 3` is `2` (Ch 3, 3.2.6), so the set has all three remainders.

**Duplicate keys in a dict comprehension: the last one wins.** That's how to build a
"value → last index" map. For the first index, iterate in reverse:

```python
values = [7, 8, 7]
{v: i for i, v in enumerate(values)}  # -> {7: 2, 8: 1}
{v: i for i, v in reversed(list(enumerate(values)))}  # -> {7: 0, 8: 1}
```

### 4.2.2 Filter vs conditional expression

These look similar and do different things:

```python
nums = [3, -1, 4, -1, 5]
[x for x in nums if x > 0]  # -> [3, 4, 5]
[x if x > 0 else 0 for x in nums]  # -> [3, 0, 4, 0, 5]
```

| You want | Write | Where the `if` goes |
|---|---|---|
| Fewer items (drop some) | `[x for x in it if cond]` | After the `for` |
| Same number of items, transformed differently | `[a if cond else b for x in it]` | Before the `for`, with a required `else` |

`[x for x in it if cond else 0]` is a `SyntaxError`: a filter can't have an `else`.

### 4.2.3 Nested comprehensions and grids

**Read order = loop order.** Multiple `for` clauses nest left to right, exactly like
the equivalent indented loops:

```python
pairs = [(a, b) for a in "xy" for b in "12"]
pairs  # -> [('x', '1'), ('x', '2'), ('y', '1'), ('y', '2')]
```

```text
[(a, b) for a in "xy" for b in "12"]       is      for a in "xy":
          ─────────── ───────────                       for b in "12":
             outer       inner                               append((a, b))
```

**Flattening** a 2D list uses two `for` clauses (outer = rows):

```python
grid = [[1, 2, 3], [4, 5, 6]]
[x for row in grid for x in row]  # -> [1, 2, 3, 4, 5, 6]
```

**Building a 2D list** uses a comprehension *inside* a comprehension. The inner one is
the expression, so it runs once per outer item and produces a fresh row each time:

```python
rows, cols = 2, 3
[[0] * cols for _ in range(rows)]  # -> [[0, 0, 0], [0, 0, 0]]
identity = [[1 if r == c else 0 for c in range(3)] for r in range(3)]
identity  # -> [[1, 0, 0], [0, 1, 0], [0, 0, 1]]
```

| Shape | Meaning |
|---|---|
| `[x for row in grid for x in row]` | Flat: two loops, one output list |
| `[[f(x) for x in row] for row in grid]` | Nested: one output row per input row |

**Transposing** swaps rows and columns. Use a comprehension over column indices, or
`zip(*grid)`:

```python
grid = [[1, 2, 3], [4, 5, 6]]
rows, cols = len(grid), len(grid[0])
[[grid[r][c] for r in range(rows)] for c in range(cols)]  # -> [[1, 4], [2, 5], [3, 6]]
[list(col) for col in zip(*grid)]  # -> [[1, 4], [2, 5], [3, 6]]
```

`zip(*grid)` unpacks the rows as separate arguments, so `zip` pairs up their first
elements, then their second elements, and so on. It's O(R · C).

### 4.2.4 The `[[0] * m] * n` trap, generalized

Ch 3 (3.2.3) shows the diagram. The rule: **`[obj] * n` gives n references to one
`obj`**. That's harmless for immutable objects, which can only be replaced, and a bug
for mutable ones:

```python
buckets = [[]] * 3
buckets[0].append("a")
buckets  # -> [['a'], ['a'], ['a']]
```

The same trap hides in `dict.fromkeys`:

```python
groups = dict.fromkeys(["x", "y"], [])
groups["x"].append(1)
groups  # -> {'x': [1], 'y': [1]}
```

Fixes: `[[] for _ in range(3)]`, `{k: [] for k in keys}`, or
`collections.defaultdict(list)` (Ch 8).

### 4.2.5 Generator expressions

A generator expression produces items **on demand**. Nothing runs until something
asks for the next item:

```python
gen = (x * x for x in range(3))
type(gen)  # -> <class 'generator'>
next(gen)  # -> 0
list(gen)  # -> [1, 4]
```

**Memory.** `sum([...])` builds the whole list first. `sum(...)` over a generator
holds one item at a time. Measured with `tracemalloc` for N = 1,000,000
(`bench/ch04_generator_memory.py`, Python 3.13.2):

| Expression | Peak memory |
|---|---|
| `sum([x * x for x in range(N)])` | 39,500 KiB |
| `sum(x * x for x in range(N))` | 1 KiB |

When a generator is the only argument, its parentheses double as the call's:
`sum(x * x for x in range(N))`. With two arguments you need them both:
`sorted((x for x in it), reverse=True)`.

**Counting with `sum`.** `True` and `False` are the ints 1 and 0, so these both count
matches:

```python
nums = [3, -1, 4, -1, 5, 2]
sum(1 for x in nums if x % 2 == 0)  # -> 2
sum(c in "aeiou" for c in "education")  # -> 5
```

**`any` / `all` short-circuit.** They stop at the first item that decides the answer,
so only a generator saves work (a list comprehension would already be fully built):

```python
checked = []


def is_negative(x: int) -> bool:
    checked.append(x)
    return x < 0


any(is_negative(x) for x in [5, -1, 7, 8])  # -> True
checked  # -> [5, -1]
```

Edge cases: `any([])` is `False` (no item is true), and `all([])` is `True` (no item is
false). An empty grid is "all rows sorted".

**A generator can be consumed only once:**

```python
gen = (x for x in [1, 2])
sum(gen)  # -> 3
sum(gen)  # -> 0
```

The second `sum` gets an exhausted generator. No error is raised. If you need the
values twice, build a list.

### 4.2.6 When a comprehension is the wrong tool

| Situation | Why a comprehension is wrong | Write instead |
|---|---|---|
| You only want side effects | `[print(x) for x in it]` builds a useless list of `None`s | A `for` loop |
| Each item depends on the previous result (running sum, state machine) | Needs hidden state | A `for` loop, or `itertools.accumulate` **[KNOW-HOW]** |
| More than two `for`s, or nested conditions | Unreadable; interviewers will ask you to rewrite it | Loops with named intermediate variables |
| You need `break` | Comprehensions can't stop early | A loop, or `any` / `next` with a generator |
| The O(n) work is hidden per item | `{c: s.count(c) for c in set(s)}` is O(k · n) | One counting pass, or `Counter` (Ch 8) |

Running sum, the right way and a legal-but-dubious way:

```python
from itertools import accumulate

list(accumulate([1, 2, 3, 4]))  # -> [1, 3, 6, 10]
total = 0
[total := total + x for x in [1, 2, 3, 4]]  # -> [1, 3, 6, 10]
```

The walrus version (3.8+) works, but it relies on a side effect inside a
comprehension. In an interview, write the loop or `accumulate`.

### 4.2.7 Scope of the loop variable

In Python 3 a comprehension's loop variable doesn't leak out:

```python
[leak for leak in range(3)]  # -> [0, 1, 2]
leak  # -> NameError: name 'leak' is not defined
```

Lambdas created inside a comprehension capture the **variable**, not its value at
creation time. They all see its final value:

```python
funcs = [lambda: i for i in range(3)]
[f() for f in funcs]  # -> [2, 2, 2]
funcs = [lambda i=i: i for i in range(3)]
[f() for f in funcs]  # -> [0, 1, 2]
```

The default argument `i=i` is evaluated when each lambda is created, which freezes the
current value. This comes up with sort keys and callbacks built in a loop.

### 4.2.8 Performance: comprehension vs loop vs `map`

Measured in `bench/ch04_comprehension_vs_loop.py`: N = 100,000, CPython 3.13.2,
Windows 11 laptop. Five runs. This machine is noisy, and identical runs differed by up
to 2× in absolute time, so the table shows the **range** across runs and the text only
states conclusions that held in every run (raw numbers in `bench/RESULTS.md`).

| Task | Variant | ms per call (min–max of 5 runs) |
|---|---|---|
| Build `[x * x ...]` | list comprehension | 3.5 – 8.8 |
| | `for` + `append` | 3.5 – 10.0 |
| | `for` + cached `append` | 4.1 – 10.6 |
| | `list(map(lambda x: x * x, data))` | 6.3 – 16.1 |
| Keep evens | list comprehension | 3.7 – 7.9 |
| | `for` + `append` | 3.3 – 8.4 |
| Sum of squares | `sum(generator)` | 5.4 – 11.7 |
| | `sum([list comp])` | 5.1 – 10.8 |
| | plain loop with `total +=` | 4.8 – 8.7 |

What held in every run:

1. **`map` with a `lambda` was the slowest way to build the list**, at 1.3× to 3.5× the
   comprehension's time. The lambda is a Python function call per element. (`map`
   with a built-in function, such as `map(int, tokens)`, avoids that call overhead;
   that case is not benchmarked here.)
2. **Comprehension vs `for` + `append`: within about 20%, and neither won
   consistently.** On CPython 3.13 the difference is too small to matter. Older
   guides say comprehensions are clearly faster; that may hold on older versions
   (general knowledge, not benchmarked here).
3. **`sum(generator)` was not faster than `sum([list])`.** The generator's advantage is
   memory (4.2.5), not speed.

**The practical rule:** choose by readability, and generators when the data is large
or you might stop early. None of these differences changes a complexity class; a 20%
constant factor won't turn a TLE into an Accept. Ch 11 covers what does.

## 4.3 Gotchas & traps

**1. `[mutable] * n`, `dict.fromkeys(keys, [])`.** Shared object (4.2.4).

**2. A generator reused after it's exhausted.** It silently yields nothing (4.2.5).

**3. A filter with `else`.** `SyntaxError`. Move it in front of the `for` as
`a if c else b` (4.2.2).

**4. Reversed loop order in a nested comprehension.** `[x for x in row for row in grid]`
fails: `row` is used before the clause that defines it. The outer loop comes first
(4.2.3).

**5. Late-binding lambdas.** Use `lambda i=i: ...` (4.2.7).

**6. A comprehension used for side effects.** Builds a list of `None`s (4.2.6).

**7. Hidden O(n) inside the expression.** `[x for x in a if x in b]` is O(len(a) ·
len(b)) when `b` is a list, because each `in` scans `b`. Convert `b` to a set first:
O(len(a) + len(b)) on average (Ch 12).

**8. `any([...])` instead of `any(...)`.** The brackets build the whole list first,
so the short-circuit saves nothing.

**9. Expecting a tuple from `(x for x in it)`.** That's a generator; use `tuple(...)`.

## 4.4 Benchmarks

| File | Measures | Result (CPython 3.13.2) |
|---|---|---|
| `bench/ch04_comprehension_vs_loop.py` | Build/filter/sum: comprehension vs loop vs `map(lambda)` | `map(lambda)` slowest in every run; others within noise (4.2.8) |
| `bench/ch04_generator_memory.py` | Peak memory, list vs generator into `sum` | 39,500 KiB vs 1 KiB for N = 10⁶ |

## 4.5 Exercises

Answers are in Appendix D (verified in `code/ch04_comprehensions/exercises.py`).
Unless stated otherwise, use:
`nums = [3, -1, 4, -1, 5, -9, 2, 6]`, `words = ["apple", "", "kiwi", "fig", "banana"]`,
`grid = [[1, 2, 3], [4, 5, 6]]`, `square = [[1, 2], [3, 4]]`.

**Warm-up: transformations**

1. Squares of 0..9.
2. The even values of `nums`.
3. `nums` with every negative replaced by 0.
4. The length of each word.
5. The uppercased first letter of each non-empty word.
6. A dict mapping each of `"abc"` to its code point.
7. The set of `x % 3` for `x` in `nums`. (Careful with negatives.)
8. Invert `{"a": 1, "b": 2}`.
9. Count the even values of `nums` without building a list.
10. Is any value of `nums` negative? (Short-circuiting.)

**Core: grids, maps, strings**

11. Flatten `grid`.
12. Transpose `grid` with a nested comprehension (no `zip`).
13. A 2 × 3 grid of zeros (no aliasing).
14. The 3 × 3 identity matrix.
15. All index pairs `(i, j)` with `0 <= i < j < 3`.
16. Map each value of `[7, 8, 7]` to its **last** index.
17. Map each value of `[7, 8, 7]` to its **first** index.
18. Character frequencies of `"banana"` with a dict comprehension. What is its
    complexity, and why is it worse than one counting pass?
19. Group `words` by length into a dict of lists using comprehensions. What is the
    complexity?
20. Running sums of `[1, 2, 3, 4]` (two ways).
21. The in-bounds 4-directional neighbors of cell `(1, 0)` in a 2 × 3 grid, using
    `DIRS = [(-1, 0), (1, 0), (0, -1), (0, 1)]`.
22. Is every row of `grid` sorted? One expression.
23. `"A man, a plan!"` reduced to lowercase alphanumerics, as a string.
24. Count the vowels in `"education"` by summing booleans.
25. The main diagonal of `square`.

**Tricky: predict the output**

26. `g = [[0] * 2] * 2; g[1][0] = 7; g`
27. `gen = (x for x in [1, 2]); (sum(gen), sum(gen))`
28. `[f() for f in [lambda: i for i in range(3)]]`, and its fix.
29. `[(x, y) for x in "xy" for y in "12"]`
30. `[x if x % 2 else -x for x in range(4)]`
31. `(any([]), all([]))`
32. Rotate `square` 90° clockwise with `zip`. (Hint: reverse the rows first.)
33. What happens on `leak` after `[leak for leak in range(3)]`, if `leak` wasn't
    defined before?

## 4.6 Chapter cheat sheet

| Form | Use |
|---|---|
| `[f(x) for x in it if p(x)]` | Transform + filter → list |
| `[a if c else b for x in it]` | Per-item choice, same length |
| `{k: v for ...}` / `{x for ...}` | Dict / set; dict keeps the **last** duplicate key |
| `[x for row in grid for x in row]` | Flatten (outer loop first) |
| `[[0] * m for _ in range(n)]` | Grid without aliasing |
| `zip(*grid)` | Transpose, O(R · C) |
| `sum(1 for ...)`, `sum(bools)` | Count without a list |
| `any(gen)` / `all(gen)` | Short-circuit; `any([]) == False`, `all([]) == True` |
| Generator | One pass only; O(1) memory; 39,500 KiB → 1 KiB measured for 10⁶ items |
| Speed (3.13, measured) | Comprehension ≈ loop; `map(lambda)` slowest; choose for readability |
