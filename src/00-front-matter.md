# Front Matter: How to Use This Book

## 0.1 What this book is

This book teaches **arrays and strings to the level LeetCode and coding interviews
demand**, in Python. It assumes you can write functions, loops, classes, lists and
dicts. It does not assume you know the conventions LeetCode solutions are written in,
how to reason about complexity, or which built-ins you are allowed to lean on.

| In scope | Out of scope |
|---|---|
| Arrays, strings, matrices, and the patterns built on them | Trees, graphs, DP beyond Kadane-level recurrences |
| LeetCode's function-completion format (`class Solution`) | stdin/stdout judges (Codeforces), except for one performance contrast in Ch 11 |
| Python 3.10+ | Python 2, and Python 3.9 or older |
| Interview-quality code | Code golf (one-liners appear only as labeled asides) |

The plain truth: **reading this book will not make you good at these problems.
Solving problems will.** The book gives you the language, the toolkit, the patterns
and a method. Section 0.4 is the method. Skip it and the rest is worth much less.

## 0.2 How the book is organized

| Part | Chapters | Kind | What you get |
|---|---|---|---|
| I. The LeetCode Python Dialect | 1–4 | Concept | `class Solution`, return contracts, in-place rules, type hints, the semantics that cause wrong answers |
| II. The Complete Toolkit | 5–9 | Reference | Every `str` and `list` method, the built-ins and stdlib modules that matter, and which of them you must be able to hand-write |
| III. Complexity & Performance | 10–12 | Concept + reference | How to derive complexity, what constraints tell you, why CPython is slow and what to do about it |
| IV. Array Patterns | 13–23 | Pattern | In-place writes, hashing, prefix sums, two pointers, sliding window, sorting, binary search, Kadane, monotonic stack, matrices, index tricks |
| V. String Patterns | 24–29 | Pattern | String fundamentals, hashing, palindromes, windows on strings, parsing, matching algorithms |
| VI. Mastery | 30–32 | Concept + practice | Pattern recognition, mixed sets with no chapter labels, the common bugs |
| Appendices | A–E | Reference | Cheat sheets, master problem list, re-solve tracker, hints and solutions index, glossary |

**Two chapter shapes.** Concept and reference chapters (Parts I–III, VI) open with an
exhaustive reference table and explain afterwards. Pattern chapters (Parts IV–V)
follow a fixed order:

| Section | Contents |
|---|---|
| NN.0 At a glance | What the pattern solves, prerequisites, typical complexity, tools with tier tags |
| NN.1 Recognition signals | Phrases in a problem statement that point to the pattern, and look-alikes |
| NN.2 Core idea | Why brute force wastes work, and the insight that removes the waste |
| NN.3 The template | A reusable skeleton, its invariant, its complexity |
| NN.4 Dry run | A trace table on a small input |
| NN.5 Variations | What changes in the template, per variant |
| NN.6 Worked problems | Brute force → improved → optimal, with edge cases and common bugs |
| NN.7 Python-specific notes | Idioms, pitfalls, benchmark results |
| NN.8 Practice set | Problems to solve yourself; hints are in Appendix D |
| NN.9 Cheat sheet | Half a page to review before an interview |

## 0.3 Reading order

The default is front to back. Parts IV and V assume Parts I–III.

| Your situation | Path |
|---|---|
| New to DSA in Python | Read everything in order. Do the practice sets before moving to the next chapter. |
| Comfortable with Python, weak on patterns | Ch 1, 2 (skim), 3, 9, 10, then Parts IV–VI in order. Use Part II as a lookup reference. |
| Interview in under a month | Ch 1, 3, 10, then Ch 13–17, 19, 24, 26–28, then Ch 30–32. Come back for the rest. |
| Looking something up | Part II, Ch 12, and Appendix A. |

**Don't skip Ch 1, 3 and 10**, whatever your path. Ch 1 explains the in-place contract
that silently fails submissions. Ch 3 covers `//` with negatives, aliasing and the
`[[0]*m]*n` grid. Ch 10 is how you turn constraints into a target complexity before
you write a line.

## 0.4 The practice method

### The loop, for every problem

1. **Read and restate.** Put the problem in your own words. Write down the input sizes
   from the constraints and the complexity they allow (Ch 10 gives the table).
2. **Struggle for 25–40 minutes.** Start with a brute force, even a slow one. If you
   have no idea at all after 25 minutes, go to step 3. If you are making progress,
   keep going up to 40.
3. **Take a hint before the solution.** Hints are in Appendix D, away from the
   problems so you don't see them by accident. Take one hint, then try again.
4. **Read the solution, close it, and re-code it from a blank file.** If you can't
   re-code it without looking, you did not understand it yet. Read it again and repeat.
5. **Log it** in your pattern notebook (below).
6. **Schedule the re-solves** (below).

### Re-solve schedule

Re-solve each problem from a blank file, with no notes and no hints, on this schedule
(day 1 is the day you first solved it or read its solution):

| Re-solve | When | Passing means |
|---|---|---|
| 1 | Day 1 (same day) | Correct and accepted, without looking at anything |
| 2 | Day 3 | Same, and you can state the complexity with its reason |
| 3 | Day 7 | Same, in noticeably less time than the first solve |
| 4 | Day 21 | Same. After this, the problem is "owned". |

**The book's rule: if you fail a re-solve, restart that problem's schedule at day 1.**
Appendix C has a tracker table for this.

Why: recognizing a solution when you read it is a different skill from producing it
under time pressure. Spaced re-solving trains the second one. Solving a problem once
mostly trains the first.

### The pattern notebook

Keep one table, in a file or on paper. The third column is the one that matters:

| Problem | Pattern | The clue that revealed it |
|---|---|---|
| 1 Two Sum | Hash map, complement lookup | Find a pair with a given sum; input is *not* sorted; O(n) wanted |
| 167 Two Sum II | Two pointers, opposite ends | Same task, but the input *is* sorted and extra space should be O(1) |
| 560 Subarray Sum Equals K | Prefix sum + hash map | "Contiguous subarray" + "sum equals k" + negatives allowed (so a sliding window doesn't work) |
| 3 Longest Substring Without Repeating Characters | Variable sliding window | "Longest contiguous substring" + a constraint you can maintain as the window moves |

When you meet an unseen problem in an interview, the third column is what you will
search your memory for. "Two Sum uses a hash map" doesn't help you there. "Unsorted
input, pair, target sum → hash map" does. Ch 30 is built from these clues.

### Habits that waste your time

- **Reading solutions on autopilot.** Without step 4 it feels like progress and
  isn't.
- **Counting solved problems.** 100 problems re-solved on schedule beat 300 seen once.
- **Chasing LeetCode's "beats X%" runtime number.** It's noisy, and correct complexity
  matters far more (Ch 11 explains both).
- **Skipping the brute force.** Interviewers ask for it, and it's usually the start of
  the optimal idea.
- **Submitting without running your own edge cases.** Every file in this book shows
  which cases to test (Section 0.5).

## 0.5 Environment

### Python version

**The book's code needs Python 3.10 or newer.** It uses two annotation features:

| Feature | Example | Minimum version | Source |
|---|---|---|---|
| Built-in generics | `list[int]`, `dict[str, int]` | 3.9 (PEP 585) | Python docs, "Generic Alias Type": *Added in version 3.9* |
| Union operator | `int \| None` | 3.10 (PEP 604) | Python docs, "Union Type": *Added in version 3.10* |

All code in this book was run on **Python 3.13.2** (Windows 11). Check your version:

```text
> python --version
Python 3.13.2
```

Then run the environment check that comes with the book:

```text
> python code/ch00_front_matter/check_env.py
OK: Python 3.13.2
```

It fails with a clear message if your Python is older than 3.10, or if asserts are
disabled (more on that below).

**LeetCode's Python version changes over time**, so this book doesn't hard-code it.
Look it up on LeetCode's help-center page about language environments. Ch 2 explains
why LeetCode's templates still show `List[int]` and why that doesn't matter.

### Anatomy of a file in `code/`

Every worked solution in the book has a file in `code/chNN_<topic>/`. They all look
like this one (problem 1480, solved properly in Ch 13):

<!-- file: code/ch00_front_matter/running_sum.py -->

```python
"""1480. Running Sum of 1d Array: https://leetcode.com/problems/running-sum-of-1d-array/

Front-matter example of the layout every file in code/ follows.
Solved properly in Chapter 13.

LeetCode signature: def runningSum(self, nums: List[int]) -> List[int]:
Book signature:     def runningSum(self, nums: list[int]) -> list[int]:
"""


class Solution:
    def runningSum(self, nums: list[int]) -> list[int]:
        # O(n) time: one pass, O(1) work per element.
        # O(1) auxiliary space: only `total`; the output list is not auxiliary.
        result = []
        total = 0
        for num in nums:
            total += num
            result.append(total)
        return result


if __name__ == "__main__":
    s = Solution()
    # Examples
    assert s.runningSum([1, 2, 3, 4]) == [1, 3, 6, 10]
    assert s.runningSum([1, 1, 1, 1, 1]) == [1, 2, 3, 4, 5]
    assert s.runningSum([3, 1, 2, 10, 1]) == [3, 4, 6, 16, 17]
    # Edge cases
    assert s.runningSum([7]) == [7]                      # single element
    assert s.runningSum([-1, -2, 3]) == [-1, -3, 0]      # negatives
    assert s.runningSum([0, 0, 0]) == [0, 0, 0]          # all equal
    assert s.runningSum([]) == []                        # empty (outside constraints)
    print("all tests passed")
```

| Part | Purpose |
|---|---|
| Docstring | Problem number, link, both signatures. Problem statements are never copied; the book summarizes them. |
| `class Solution` | Exactly what you paste into LeetCode. Same method name as on the site. |
| Complexity comments | Time and auxiliary space, each with its reason. |
| `if __name__ == "__main__":` | Runs only when you execute the file directly. It holds the tests. |
| `assert` lines | The problem's examples first, then edge cases: empty input, a single element, all-equal values, negatives, and maximum sizes where relevant. |

### Running the tests

Run one file:

```text
> python code/ch00_front_matter/running_sum.py
all tests passed
```

If an assert fails, Python stops at that line and prints `AssertionError` with the
line number. That is your failing test case.

Run every file in a chapter, or the whole book, with `run_code.py` in the project
root:

```text
> python run_code.py ch00_front_matter
PASS  code/ch00_front_matter/check_env.py
PASS  code/ch00_front_matter/running_sum.py

2/2 files passed (Python 3.13.2)
```

**Pitfall: never run the tests with `python -O`.** The `-O` flag removes every
`assert` statement, so broken code "passes" silently. `check_env.py` detects this:

```text
> python -O code/ch00_front_matter/check_env.py
asserts are disabled (python -O?); tests would not run
```

### Moving between your machine and LeetCode

- **To LeetCode:** paste the imports and the `class Solution` block. Leave out the
  `if __name__ == "__main__":` block; it's test code for your machine, not part of
  the answer.
- **From LeetCode:** when a submission fails, copy the failing input into a new
  `assert` in your file. Fix the code locally until every assert passes, then
  resubmit.
- **Imports:** LeetCode pre-imports many modules. Your machine doesn't, so the book
  always writes imports explicitly (`from collections import Counter`).

## 0.6 Notation

### Tier tags

Every built-in tool in the book carries one of three tags:

| Tag | Meaning | Examples |
|---|---|---|
| **[USE-FREELY]** | Use anywhere. Nobody expects you to reimplement it. | `len`, `enumerate`, `dict`, `set` |
| **[KNOW-HOW]** | Use it on LeetCode, but be able to write it by hand when an interviewer says "without built-ins" or the problem forbids it. | `sorted`, `reversed`, `Counter`, `bisect`, `heapq`, `str.find`, `accumulate` |
| **[HAND-WRITE]** | No built-in exists, or using one defeats the point of the problem. You write the logic. | Two pointers, sliding window, KMP, Kadane, in-place rotation, monotonic stack, string arithmetic without `int()` |

Ch 9 is the complete map of which tool gets which tag.

### Complexity

| Symbol | Meaning |
|---|---|
| *n* | Length of the main input (`len(nums)`, `len(s)`) |
| *m* | Length of a second input, or the number of columns in a matrix |
| *k* | A parameter from the problem (window size, "top k", etc.) |
| *Σ* | Alphabet size: 26 for lowercase English letters |
| O(·) | Upper bound on growth. Ch 10 defines Θ and Ω as well. |
| log | Base 2 unless stated. In Big-O the base doesn't change the class. |

Rules the book follows, and you should too:

- **Time and auxiliary space are stated separately.** Auxiliary space is the extra
  memory the algorithm uses beyond its input. The output isn't counted unless the
  text says so.
- **Every complexity comes with a reason.** Not "O(n log n)", but "O(n log n) because
  sorting n items dominates; the loop afterwards is O(n)."
- **Average and worst case are separated** where they differ. For example, dict
  lookup is O(1) on average and O(n) in the worst case (Ch 12 gives the reasons).
- **"Amortized"** means averaged over a sequence of operations, as with `list.append`
  (Ch 6 and Ch 10).

### Version tags

A feature that needs a minimum Python version is tagged inline, for example
`int | None` (3.10+). Every tag is checked against the official docs or by running
the code; unverified claims are flagged rather than guessed.

### Performance claims

A claim like "X is faster than Y" appears in one of two forms:

| Form | Meaning |
|---|---|
| **Measured:** numbers + `bench/<file>.py` + Python version | Run it yourself; your numbers will differ, but the ratio should be similar. |
| *General knowledge, not benchmarked here.* | Widely documented, but not measured in this book. Treat it as a claim, not a result. |

### Other conventions

- Printed output is shown as a comment: `print(sorted("cab"))  # -> ['a', 'b', 'c']`.
- Each worked problem shows LeetCode's signature (`List[int]`, `Optional[int]`) once,
  followed by the book's (`list[int]`, `int | None`). The two are equivalent; Ch 2
  explains why.
- Pointers are named `left`/`right`, and indices `i`/`j`. A solution never mixes the
  two styles.
- Problems are linked as `https://leetcode.com/problems/<slug>/` and summarized in
  the book's own words.

## 0.7 Before Chapter 1

- [ ] `python --version` prints 3.10 or newer
- [ ] `python code/ch00_front_matter/check_env.py` prints `OK`
- [ ] `python run_code.py ch00_front_matter` reports 2/2 passed
- [ ] You have a LeetCode account
- [ ] You have started a pattern notebook and a re-solve tracker (Appendix C)
