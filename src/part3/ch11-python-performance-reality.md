# Chapter 11: Python Performance Reality

## 11.0 At a glance

CPython is slow at executing your loops and fast at executing its own. Nearly every
performance decision in this book follows from that one fact. This chapter measures it
on this book's machine (CPython 3.13.2, Windows 11 laptop, numbers in
`bench/RESULTS.md`):

| Comparison | Measured gap |
|---|---|
| Same loop in CPython vs Java (warmed JIT) | ~70–90× |
| `x in list` vs `x in set`, n = 10⁵ | ~15,000–19,000× |
| Draining 40,000 items: `list.pop(0)` vs `deque.popleft()` | ~800–1,000× |
| `sorted` vs a hand-written merge sort, 10⁵ ints | ~15× |
| Counting letters: 26 × `str.count` vs a Python loop | ~4× |
| `a[i:]` in a loop vs `a[i]`, n = 10⁴ | ~550× |

Two lessons come out of the table. **Complexity class decides whether you pass**: rows
2, 3 and 6 are O(n) vs O(1) or O(n²) vs O(n) in disguise. **"Push the loop into C"
decides by how much**: rows 4 and 5.

## 11.1 Complete reference

### Why CPython is slow

| Cause | What it means for a loop like `total += a[i]` |
|---|---|
| Interpreted bytecode | Each operation is several bytecode instructions, each dispatched by the interpreter loop |
| Dynamic typing | `+` must check the operand types at run time, every time |
| Boxed integers | Every int is a heap object (28+ bytes); arithmetic creates new objects |
| Per-operation overhead | Index checks, reference counting, attribute lookups |

These costs are **per Python-level operation**. Built-ins written in C (`sum`, `sorted`,
`str.count`, `set` operations, slicing, `"".join`) pay them once per call, not once per
element. Hence the golden rule:

> **Built-ins and C-implemented library functions are fast; Python-level loops are
> slow. Push the loop into C.**

### Measured costs on this machine

| Operation | Cost (CPython 3.13.2) | Source |
|---|---|---|
| One "typical" loop iteration (read, compare, two updates) | ~90–200 ns (5–11 M/s) | Ch 10, `bench/ch10_python_budget.py` |
| `x in set` | ~0.04 µs, independent of n | `bench/ch11_core.py` |
| `x in list` of 10⁵, value absent | ~0.7 ms | `bench/ch11_core.py` |
| `sorted` of 10⁵ random ints | 20–26 ms | `bench/ch11_core.py` |
| Python merge sort of 10⁵ ints | 344–394 ms | `bench/ch11_core.py` |
| A Python function call (recursive sum, per level) | ~0.2 µs | `bench/ch11_core.py` |

### Memory (`sys.getsizeof`, CPython 3.13.2, 64-bit)

| Object | Bytes |
|---|---|
| `int` 0 or 1 / 2³⁰ / 2⁶⁰ / 2¹⁰⁰ | 28 / 32 / 36 / 40 |
| `str` `""` / `"a"` / 100 ASCII chars | 41 / 42 / 141 |
| `str` `"é"` / one emoji | 61 / 64 (non-ASCII strings use a wider internal representation) |
| `list` of 1000 items (the list object only) | 8,056 |
| 1000 distinct large ints, list + the int objects | 36,056 |
| `set` of 1000 ints / `dict` of 1000 ints (container only) | 32,984 / 36,952 |

`getsizeof` reports only the object itself, not what it refers to. A list of n ints
costs 8 bytes per slot **plus** 28+ bytes per distinct int object, which is why Ch 6
measured 34.3 MiB for 10⁶ ints against 7.6 MiB for `array('q')`. Small ints (−5 to 256
in CPython) are cached and shared, which is an implementation detail.

## 11.2 Explanations by group

### 11.2.1 How CPython compares to other languages

Same workload in three languages: `total += (i * i) % 7` for i < 10⁷
(`bench/ch11_languages/`, each program times only its loop):

| Language | Time | Relative to CPython |
|---|---|---|
| Java 27, JIT warmed up | 9–10 ms | ~70–90× faster |
| C++, MinGW.org g++ 6.3 `-O2` | 46–47 ms | ~16× faster |
| CPython 3.13.2 | 736–804 ms | 1× |
| PyPy | not installed, not measured | |

**Read this table carefully.** It's one workload on one machine, and the C++ number is
distorted: this MinGW build targets **32-bit** x86 (`g++ -dumpmachine` prints
`mingw32`), so 64-bit `long long` arithmetic is emulated with library calls. A 64-bit
C++ compiler would very likely beat Java here (general knowledge, not measured on this
machine). Commonly quoted ranges are "C++ and Java are 10–100× faster than CPython on
loop-heavy code, and PyPy is often several times faster than CPython". Those are
general knowledge, not benchmarked here, and the ratio swings widely with the workload.
What the measurement does establish: **a CPython loop is one to two orders of
magnitude slower than compiled code**, so constraints tuned for C++ can be tight in
Python (Ch 10, 10.2.9).

### 11.2.2 Hash lookups vs scans

| n | `-1 in list` | `-1 in set` |
|---|---|---|
| 1,000 | 5.9–6.2 µs | 0.037–0.038 µs |
| 10,000 | 66–74 µs | 0.040–0.044 µs |
| 100,000 | 689–701 µs | 0.036–0.045 µs |

The list cost grows tenfold with n (O(n) scan). The set cost doesn't move (O(1)
average). Inside a loop of 10⁵ iterations, list membership at n = 10⁵ costs up to about 70
seconds; set membership costs about 4 milliseconds. **This is the most common
accidental O(n²) in Python solutions.**

### 11.2.3 Queues: `pop(0)` vs `deque`

| n | `list.pop(0)` until empty | `deque.popleft()` until empty |
|---|---|---|
| 20,000 | 388–429 ms | 0.6–1.0 ms |
| 40,000 | 1,595–2,343 ms | 1.9–2.3 ms |

Doubling n roughly quadrupled `pop(0)` (O(n²) in total) and doubled `popleft` (O(n)).
Ch 6 measured the same effect at 10⁵: 15.6 s. Every BFS and every queue simulation uses
`deque`.

### 11.2.4 Built-in sort vs hand-written sorts

Sorting 10⁵ random ints:

| Sort | Time (measured) | Relative |
|---|---|---|
| `sorted()` (Timsort, in C) | 20–26 ms | 1× |
| Merge sort in Python (Ch 9) | 344–394 ms | ~15× slower |
| Quicksort in Python, random pivot (Ch 9) | 429–490 ms | ~20× slower |

All three are O(n log n). The gap is pure constant factor: Python-level comparisons,
appends and index updates against C. Interview consequence: write merge sort when asked
(912), and use `sorted` everywhere else. It's never the slow part of a solution.

### 11.2.5 Counting: four ways

Counting letters in a 1,000,000-character lowercase string:

| Method | Time | Why |
|---|---|---|
| `[s.count(c) for c in ascii_lowercase]` | 30–35 ms | 26 passes, each entirely in C |
| `Counter(s)` | 76–96 ms | One pass; the counting loop is implemented in C |
| 26-slot list, Python loop | 110–130 ms | One Python-level iteration per character |
| dict, Python loop | 123–156 ms | Same, plus a hash lookup per character |

Twenty-six C passes beat one Python pass. That's "push the loop into C" in its purest
form. Practical reading:

- For a fixed small alphabet, `str.count` per letter measured fastest. It's O(26 · n) = O(n)
  but rarely what interviewers expect to see.
- `Counter` is the clean default when built-ins are allowed (**[KNOW-HOW]**).
- The 26-slot list is the interview-standard hand-written version (Ch 24). It's slower
  than `Counter` in CPython, but it's what "O(1) extra space" answers look like.

### 11.2.6 Recursion vs iteration

Summing 1..900 (depth 900, just under the default limit):

| Version | Time per call |
|---|---|
| Recursive | 166–201 µs |
| Loop | 47–56 µs |

The recursive version measured 3–4× slower: one function call per level, at roughly 0.2 µs
each. The bigger problem is the **depth limit**. `sys.getrecursionlimit()` is 1000 by
default. You can raise it:

```python
import sys

sys.getrecursionlimit()  # -> 1000


def depth(n: int) -> int:
    return 0 if n == 0 else 1 + depth(n - 1)


depth(5000)  # -> RecursionError: maximum recursion depth exceeded
```

On this machine, `sys.setrecursionlimit(200_000)` followed by `depth(100_000)`
succeeded on CPython 3.13.2. On older versions, or with a different platform's stack
size, very deep recursion can crash the interpreter outright instead of raising an
exception (general knowledge, not reproduced here). Rules:

1. For linear structures (arrays, strings), **use loops**. Recursion depth n is a
   liability.
2. Divide-and-conquer with depth O(log n) (merge sort, quicksort on the smaller side)
   is safe.
3. If you must recurse deeply, raise the limit inside the method and say why. Don't
   assume LeetCode's default is different from CPython's.

### 11.2.7 Slicing copies

| n | `rest = a[i:]` in a loop | `x = a[i]` in a loop |
|---|---|---|
| 5,000 | 47–56 ms | 0.17–0.21 ms |
| 10,000 | 187–226 ms | 0.34–0.39 ms |

Each `a[i:]` copies n − i references, so the loop is O(n²) in total, and it quadrupled
when n doubled. Indexing is O(1) per step. Typical disguises: `sum(a[i:j])` inside a
loop (use prefix sums, Ch 15), `s[i:] == t[i:]` (compare by index), `nums[1:]` in
recursion (pass an index, Ch 10).

### 11.2.8 LeetCode specifics

- **Runtime percentiles ("beats X%") are noisy.** The same code submitted twice can land
  far apart (general knowledge from common experience; not measured here). Server load
  varies, and small inputs are dominated by fixed overhead. Don't optimize for the
  percentile.
- **Correct complexity matters far more than micro-optimizations.** Every large gap in
  this chapter was a complexity mistake (O(n) vs O(1) membership, O(n²) vs O(n)
  draining or slicing). The micro-optimizations Ch 4 measured, such as caching
  `out.append` or comprehension vs loop, moved times by 20% at most.
- **Time limits apply per problem, and they aren't published.** Assume roughly 10⁷
  simple Python steps per second of budget (Ch 10).
- **When a correct-complexity solution is still too slow:**
  1. Look for a Python-level inner loop that a built-in can replace (`sum`, `set`
     operations, `str` methods, `sorted`, slicing done once).
  2. Hoist repeated work out of the loop (`len(nums)`, attribute lookups,
     `ord("a")`).
  3. Prefer flat lists to dicts keyed by small ints. Index reads ran faster than dict
     lookups in Ch 10's benchmark (25–26 vs 18–19 M/s).
  4. Then reconsider the algorithm: an "O(n log n)" with a heavy Python-level inner
     loop loses to an O(n) that uses built-ins.

### 11.2.9 Contrast: Codeforces-style judges

This book targets LeetCode, but one contrast is instructive. On judges that read stdin
and print to stdout (Codeforces, AtCoder):

- **I/O speed matters.** Reading 10⁵+ lines with `input()` is notoriously slow; solvers
  use `sys.stdin.readline` or read everything with `sys.stdin.buffer.read()` (general
  knowledge, not benchmarked here).
- **Time limits are strict and published** (often 1–2 seconds), and they're set with C++
  in mind.
- **PyPy is offered** as a separate language choice and is commonly much faster for
  loop-heavy code (general knowledge, not measured: PyPy isn't installed on this
  book's machine).

None of that applies to LeetCode's function-completion format: the judge calls your
method with ready-made arguments.

## 11.3 Gotchas & traps

**1. `x in list` inside a loop.** Measured 15,000–19,000× slower than a set at 10⁵ (11.2.2).

**2. `pop(0)` / `insert(0, x)` as a queue.** Use `deque` (11.2.3).

**3. Slicing inside loops or recursion.** O(n) per slice (11.2.7).

**4. Deep recursion on linear data.** Limit 1000; use loops (11.2.6).

**5. Hand-written sorts in production code.** Measured 15–20× slower than `sorted` (11.2.4).

**6. Chasing the runtime percentile.** Noise. Fix complexity first (11.2.8).

**7. Assuming C++ constraints are safe for Python.** One to two orders of magnitude
separate them (11.2.1).

**8. Misreading `sys.getsizeof`.** It excludes referenced objects (11.1).

## 11.4 Benchmarks

| File | Measures | Key result (CPython 3.13.2) |
|---|---|---|
| `bench/ch11_languages/run.py` | Same loop in C++ (32-bit MinGW), Java, CPython | CPython 736–804 ms; Java 9–10 ms; C++ 46–47 ms (32-bit, emulated 64-bit math) |
| `bench/ch11_core.py` | Membership, queues, sorting, counting, recursion, slicing | See 11.2.2–11.2.7 |
| Ch 4 / Ch 5 / Ch 6 / Ch 7 / Ch 8 / Ch 10 benchmarks | Comprehensions, string building, list ends, `range` membership, `nlargest`, loop budget | `bench/RESULTS.md` |

## 11.5 Exercises

Answers are in Appendix D (facts verified in `code/ch11_performance/exercises.py`;
timings in `bench/RESULTS.md`).

**Warm-up**

1. A loop of 10⁵ iterations does `if x in seen_list:` with `seen_list` growing to 10⁵
   items. Estimate the time using this chapter's numbers. What's the fix?
2. Why do 26 calls to `str.count` beat one Python loop over a 10⁶-character string?
3. Rank `sorted`, a Python merge sort and a Python quicksort on 10⁵ random ints.

**Core**

4. Rewrite `while q: q.pop(0)` for O(1) per pop.
5. `sys.getsizeof([5] * 1000)` equals `sys.getsizeof(list(range(10**6, 10**6 + 1000)))`.
   Why, and why is the second list much heavier in reality?
6. A recursive helper recurses once per element of a 5,000-element list. What happens
   by default? Give two fixes, and say which you prefer.

**Tricky**

7. Two solutions are both O(n). One uses a Python loop with a dict; the other calls
   `Counter`. Which is faster in CPython, and does it matter for passing?
8. A solution does `total = sum(nums[i:i + k])` for every i, with n = 10⁵ and
   k = 5 × 10⁴. What's its complexity, and what's the fix?

## 11.6 Chapter cheat sheet

| Rule | Measured basis |
|---|---|
| Complexity first | 15,000–19,000× (list vs set), 800–1,000× (`pop(0)` vs deque), 550× (slicing) |
| Push loops into C | 26 × `str.count` 4× faster than one Python loop; `sorted` 15× faster than merge sort |
| Budget | ~10⁷ simple steps/s (Ch 10) |
| CPython vs compiled | ~16–18× (32-bit C++) and ~70–90× (Java) slower on one loop workload |
| Recursion | Limit 1000; ~3–4× slower than loops; use loops for linear data |
| Memory | int ≥ 28 B; list slot 8 B + objects; `getsizeof` is shallow |
| LeetCode | Percentiles are noise; limits unpublished; fix the class, not the constant |
