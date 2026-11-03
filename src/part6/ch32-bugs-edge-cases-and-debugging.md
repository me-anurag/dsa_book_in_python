# Chapter 32: Bugs, Edge Cases & Debugging

## 32.0 At a glance

Most failed submissions aren't wrong algorithms. They're right algorithms with a
**small, predictable bug** on an **edge case** the hidden tests always include. This
chapter collects:

- the edge-case checklist for array and string problems
- the 25 bugs that cost Python solvers the most submissions, each with a tiny checked
  demo and a pointer to the chapter that explains it
- a debugging method: dry-run tables, minimal failing inputs, and **stress testing
  against a brute force**, with a reusable tool (`code/ch32_debugging/stress.py`) that
  finds and *shrinks* counterexamples automatically

## 32.1 Complete reference: the edge-case checklist

Run your solution, in your head or with asserts, on every row that the constraints allow.

| Category | Inputs to try | Typical failure |
|---|---|---|
| Size | empty (`[]`, `""`), length 1, length 2 | Index errors, `max([])`, wrong `while` bounds |
| Values | all equal; all distinct; all negative; zeros | Duplicate handling; max initialized to 0; division by zero |
| Order | already sorted; reverse sorted; alternating | Worst cases of quicksort, stacks, windows |
| Extremes | ±2³¹ boundaries; the maximum n | Overflow rules (atoi, 7); TLE at full size |
| Strings | spaces only; repeated separators; mixed case; non-letters | `split(" ")`; case-sensitive comparisons; `isdigit` surprises |
| Structure | answer at index 0; answer at the last index; no answer | `if idx:`; off-by-one ranges; wrong sentinel |
| Duplicates | repeated values in the answer; duplicate triplets | 3Sum skipping; "first" vs "last" occurrence |
| Mutation | does the caller's list need to change? Must it **not** change? | Rebinding in `-> None`; modifying a read-only input (287) |

## 32.2 Explanations by group: the 25 most common bugs

Each demo prints the buggy behavior. Every one was run by the book's snippet checker.

### Indexing and loops

**1. Stopping one short.** `range(n - 1)` skips the last index.

```python
nums = [1, 2, 3]
[nums[i] for i in range(len(nums) - 1)]  # -> [1, 2]
```

**2. Backward ranges that miss index 0** (Ch 3, 3.2.10). The stop is exclusive, so it
must be −1.

```python
list(range(3 - 1, 0, -1))  # -> [2, 1]
list(range(3 - 1, -1, -1))  # -> [2, 1, 0]
```

**3. `max` initialized to 0** (Ch 20). It's wrong when every value is negative.

```python
best = 0
for x in [-3, -1]:
    best = max(best, x)
best  # -> 0
```

**4. Binary search that never terminates** (Ch 19). With `lo = mid` and
`mid = (lo + hi) // 2`, two remaining elements loop forever. Always shrink the range:
`lo = mid + 1` or `hi = mid`.

**5. A Python negative index silently wrapping** (Ch 22). `grid[r - 1]` with r = 0 reads
the last row.

```python
grid = [[1, 2], [3, 4]]
grid[0 - 1][0]  # -> 3
```

### Aliasing and mutation

**6. Two names, one list** (Ch 3).

```python
a = [1, 2]
b = a
b.append(3)
a  # -> [1, 2, 3]
```

**7. `[[0] * m] * n`** (Ch 3, Ch 4).

```python
grid = [[0] * 2] * 2
grid[0][0] = 9
grid  # -> [[9, 0], [9, 0]]
```

**8. Mutable default arguments** (Ch 1).

```python
def collect(x, acc=[]):
    acc.append(x)
    return acc


collect(1)
collect(2)  # -> [1, 2]
```

**9. Mutating a list while iterating over it** (Ch 1).

```python
nums = [2, 2, 3]
for x in nums:
    if x == 2:
        nums.remove(x)
nums  # -> [2, 3]
```

**10. Rebinding instead of mutating in a `-> None` method** (Ch 1).

```python
def reverse_in_place(s):
    s = s[::-1]


letters = ["a", "b"]
reverse_in_place(letters)
letters  # -> ['a', 'b']
```

**11. A self-referencing swap** (Ch 3, Ch 23).

```python
nums = [1, 0]
i = 0
nums[i], nums[nums[i]] = nums[nums[i]], nums[i]
nums  # -> [1, 0]
```

### Return values and truthiness

**12. Forgetting `return`.** The judge receives `None` (Ch 1).

**13. `x = x.sort()`** (Ch 6).

```python
x = [3, 1, 2]
x = x.sort()
x is None  # -> True
```

**14. `if idx:` when 0 is a valid index** (Ch 3).

```python
idx = [5, 6].index(5)
"found" if idx else "missing"  # -> 'missing'
```

**15. An exhausted generator or iterator** (Ch 4, Ch 7).

```python
squares = map(lambda v: v * v, [1, 2])
list(squares)  # -> [1, 4]
list(squares)  # -> []
```

### Arithmetic

**16. `//` where the problem truncates toward zero** (Ch 3, Ch 28).

```python
-7 // 2  # -> -4
int(-7 / 2)  # -> -3
```

**17. Digits of a negative number with `%`** (Ch 3).

```python
-123 % 10  # -> 7
abs(-123) % 10  # -> 3
```

**18. Assuming overflow will happen.** Python ints never wrap, so "return 0 on 32-bit
overflow" must be checked by hand (Ch 3, 7 and 8).

### Strings

**19. `split(" ")` on irregular spacing** (Ch 5, Ch 24).

```python
"a  b".split(" ")  # -> ['a', '', 'b']
"a  b".split()  # -> ['a', 'b']
```

**20. `find` returning −1, then indexing with it** (Ch 5).

```python
s = "abc"
s[s.find("z")]  # -> 'c'
```

**21. `s += c` in a loop.** It's quadratic once the string is aliased; measured in Ch 5
(5.2.7). Build a list and `"".join` it.

### Pattern-specific

**22. Inserting before checking** in complement lookups (Ch 14, 1 Two Sum). An element
pairs with itself.

```python
seen = {}
count = 0
for x in [3]:
    seen[x] = 1
    count += seen.get(6 - x, 0)
count  # -> 1
```

**23. A missing `{0: 1}` seed** in prefix-sum counting (Ch 15). Subarrays starting at
index 0 are missed.

**24. A sliding window over negative values** (Ch 17). The shrink rule assumes that
growing the window can't decrease the sum. Use prefix sums.

**25. A stale index moving `left` backwards** (Ch 27, problem 3). Check
`last[c] >= left`. The stress tester in 32.2.1 finds this bug on its own.

### 32.2.1 Stress testing and shrinking

Every test file in this book compares a fast solution with a slow, obviously correct one
on hundreds of random inputs. `code/ch32_debugging/stress.py` packages that, and adds
**shrinking**: when it finds a failing input, it deletes elements while the input still
fails, so you get a **minimal** counterexample to trace by hand.

<!-- file: code/ch32_debugging/stress.py -->

```python
def shrink(data: list | str, fails: Callable[[Any], bool]) -> list | str:
    """Greedily delete elements while the input still fails. O(n^2) calls worst case;
    inputs here are tiny. Returns a locally minimal failing input."""
    changed = True
    while changed:
        changed = False
        for i in range(len(data)):
            candidate = data[:i] + data[i + 1:]
            if fails(candidate):
                data = candidate
                changed = True
                break
    return data
```

<!-- file: code/ch32_debugging/stress.py -->

```python
def stress(fast: Callable, slow: Callable, generate: Callable[[random.Random], Any],
           trials: int = 2000, seed: int = 0) -> Any | None:
    """Return a minimal failing input, or None if all trials agree."""
    rng = random.Random(seed)

    def fails(x: Any) -> bool:
        try:
            return fast(x) != slow(x)
        except Exception:                      # a crash is also a failure
            return True

    for _ in range(trials):
        x = generate(rng)
        if fails(x):
            return shrink(x, fails)
    return None
```

Run against bug 25 (problem 3 without the stale-index check), it returns a 4-character
counterexample such as `"baab"`. Shrinking `"abbaab"` by hand gives the same length, and
`"abba"` fails too. **Minimal isn't unique**; any of them is small enough to dry-run. The
fixed version passes all 2,000 trials.

**How to use it in practice:**
1. Write the brute force first (step 3 of the attack method, Ch 30). It's your oracle.
2. Generate **small** inputs over a **tiny alphabet** (`"ab"`, values −3..3). Collisions,
   duplicates and edge cases then appear in almost every trial.
3. When it fails, dry-run the shrunk input with a table.

### 32.2.2 Dry-run tables

Write one column per variable and one row per iteration, as in every chapter's "Dry run"
section. Fill it in **from the code**, not from what you meant the code to do. The first
row where the table disagrees with your expectation is the bug. Ch 27's table for
`"abba"` (27.4) is exactly the dry run that exposes bug 25.

### 32.2.3 Assert-driven development

1. Write the asserts **before** the solution: the statement examples, then one line per
   checklist row (32.1).
2. Run the file after every small change. A failing assert gives the exact input.
3. Keep the brute force in the file as a reference class, as every chapter's code does.
4. Never run with `python -O`, which strips asserts (front matter, 0.5).

### 32.2.4 Debugging on LeetCode itself

- When a hidden test fails, LeetCode shows the input. Copy it into a local assert and
  shrink it if it's large.
- `print` output appears in the stdout panel and doesn't affect the answer. Remove it
  before the final submit, because printing large structures costs time.
- "Time Limit Exceeded" usually means the wrong complexity class, not a slow constant
  factor (Ch 11). Check the constraints table (Ch 10) before micro-optimizing.

## 32.3 Gotchas & traps

**1. Testing only the statement's examples.** They're chosen to explain the problem, not
to break your code. Use the checklist.

**2. A brute force with the same bug.** If the oracle shares logic with the fast version,
both can be wrong together. Make the oracle as simple and as different as possible:
all subarrays, `sorted`, `int()`, `str.find`.

**3. Large random tests.** They take long to run, and their failures are unreadable. Small
inputs and many trials find the same bugs, with failures you can trace by hand.

**4. Fixing the symptom.** Special-casing the failing input without understanding why it
failed. The next hidden test will fail too.

## 32.4 Benchmarks

One measurement, to show the tool is cheap enough to run on every change: 2,000 trials of
the fixed problem-3 solution against its brute force took 18 ms (best of 3, CPython
3.13.2, measured with `timeit` on the `stress` call; logged in `bench/RESULTS.md`).

## 32.5 Exercises: find the bug

Each function below has one bug. Name the input that exposes it, and fix it. Answers are
in Appendix D (verified in `code/ch32_debugging/exercises.py`).

<!-- file: code/ch32_debugging/exercises.py -->

```python
def max_value_buggy(nums):          # E1
    best = 0
    for x in nums:
        best = max(best, x)
    return best
```

<!-- file: code/ch32_debugging/exercises.py -->

```python
def grid_buggy(m, n):               # E2
    return [[0] * n] * m
```

<!-- file: code/ch32_debugging/exercises.py -->

```python
def mid_avg_buggy(a, b):            # E3: truncating average of possibly negative ints
    return (a + b) // 2
```

<!-- file: code/ch32_debugging/exercises.py -->

```python
def remove_val_buggy(nums, val):    # E4
    for x in nums:
        if x == val:
            nums.remove(x)
    return nums
```

<!-- file: code/ch32_debugging/exercises.py -->

```python
def reverse_buggy(s):               # E5: -> None contract
    s = s[::-1]
```

<!-- file: code/ch32_debugging/exercises.py -->

```python
def words_buggy(text):              # E6
    return text.split(" ")
```

<!-- file: code/ch32_debugging/exercises.py -->

```python
def last_index_buggy(nums):         # E7: iterate backwards
    return [i for i in range(len(nums) - 1, 0, -1)]
```

<!-- file: code/ch32_debugging/exercises.py -->

```python
def find_digit_buggy(s):            # E8: first digit's index, or -1
    i = s.find("7")
    return s[i]                     # returns the LAST char when absent
```

<!-- file: code/ch32_debugging/exercises.py -->

```python
def count_pairs_buggy(nums, k):     # E9: count i < j with nums[i] + nums[j] == k
    seen = {}
    count = 0
    for x in nums:
        seen[x] = seen.get(x, 0) + 1
        count += seen.get(k - x, 0)     # BUG: counts x with itself
    return count
```

<!-- file: code/ch32_debugging/exercises.py -->

```python
def has_index_buggy(nums, target):  # E10: index 0 treated as "not found"
    idx = next((i for i, x in enumerate(nums) if x == target), None)
    return bool(idx)
```

## 32.6 Chapter cheat sheet

| Before submitting | |
|---|---|
| Edge cases | Empty, 1, 2 elements; all equal; negatives; extremes; answer at index 0 / last / none |
| Contracts | `-> None` mutates; return the right type; don't modify read-only input |
| Arithmetic | `//` floors; `%` takes the divisor's sign; no overflow (check limits by hand) |
| Strings | `split()`, not `split(" ")`; check `find` against −1; build with `join` |
| Loops | Backward ranges stop at −1; binary search always shrinks; windows only with non-negative values |
| Method | Brute-force oracle → small random inputs → shrink → dry-run table |
