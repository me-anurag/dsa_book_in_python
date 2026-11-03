# Chapter 1: Anatomy of a LeetCode Solution

## 1.0 At a glance

LeetCode doesn't run your program. It runs **its own program**, which creates your
`Solution` object, calls one method, and inspects what comes back, or for some
problems, what your method did to the arguments. Most "my code works locally but
LeetCode says Wrong Answer" bugs in easy problems come from misunderstanding that
contract, not from the algorithm.

This chapter covers:

- what each part of a solution file is for, and the rules for each
- what the judge checks for each return type, including `-> None`
- why `nums = nums[::-1]` fails and `nums[:] = ...` works
- how "return `k` and modify `nums`" problems are judged
- how to test locally with the same contract
- how to read constraints before you write code (a preview of Ch 10)

Problems used: 344 Reverse String, 26 Remove Duplicates from Sorted Array,
27 Remove Element, 189 Rotate Array, 88 Merge Sorted Array. Their full pattern
treatment is in Ch 13 and Ch 16; here they only illustrate the contract.

## 1.1 Complete reference

### Parts of a solution file

| Element | What it is | Rule |
|---|---|---|
| `class Solution:` | The class the judge instantiates | Keep the name exactly as given. |
| `def methodName(self, ...)` | The entry point the judge calls | Keep the method name, parameter order and count exactly as given. You may rename parameters only if the judge passes them positionally, and that isn't documented, so don't. |
| `self` | The `Solution` instance | First parameter of every method. Needed to call helper methods. |
| Helper method `def _helper(self, ...)` | Another method on the class | Call it as `self._helper(...)`. Use it for logic you want to test or reuse. |
| Nested function `def helper(...)` inside the method | A closure | Reads the method's local variables directly. Needs `nonlocal` to *rebind* them. |
| `nonlocal name` | Declaration inside a nested function | Lets the nested function assign to `name` from the enclosing function. |
| Class attribute `x = ...` in the class body | One value shared by every instance | Never store per-test state here (Gotcha 4). |
| Imports at the top | Module imports | LeetCode pre-imports many modules; write them anyway so the file runs locally. |
| `if __name__ == "__main__":` block | Local tests | Not pasted into LeetCode. |

### Return contracts

| Annotation (book form) | What the judge compares | Typical mistake |
|---|---|---|
| `-> int` | The returned integer | Forgetting `return`, which returns `None` |
| `-> bool` | `True` / `False` | Returning `"true"` or `1` |
| `-> str` | The returned string | Returning a list of characters instead of `"".join(...)` |
| `-> list[int]`, `-> list[str]` | The returned list, **in order**, unless the statement says "in any order" | Returning a set or a generator |
| `-> list[list[int]]` | List of lists. Inner/outer order matters unless the statement says otherwise | Appending the same inner list object repeatedly (Ch 3) |
| `-> None` | **The argument you were given**, after your method returns | Rebinding the parameter name instead of mutating the object |
| `-> int` + "modify `nums`" (26, 27) | The returned `k` **and** the first `k` elements of `nums` | Building a new list and returning its length |
| `-> int \| None` / `Optional[...]` | A value, or `None` meaning "no answer" | Returning `-1` when the problem wants `None`, or the reverse |

### Which operations change the caller's list

Inside a method, `nums` is a **name** bound to the caller's list object. Some
operations change that object (the judge sees it); others only point the local name
at a new object (the judge doesn't).

| Code inside the method | Caller's list changed? | Auxiliary space (reason) |
|---|---|---|
| `nums[i] = x`, swaps `nums[i], nums[j] = nums[j], nums[i]` | Yes | O(1): one element |
| `nums.reverse()` | Yes | O(1): swaps in place |
| `nums.sort()` | Yes | O(n) worst case: Timsort's merge step uses a temporary buffer |
| `nums.append`, `extend`, `insert`, `pop`, `remove`, `clear`, `del nums[i]` | Yes | O(1) for the call itself (`extend` stores its k new items, as any mutation must) |
| `nums += [x]` | **Yes**: for lists, `+=` is `extend` | O(1) beyond the new items |
| `nums[:] = <any iterable>` | Yes: replaces the contents of the same object | O(n): the new contents are built first |
| `nums = nums[::-1]` | **No**: rebinds the local name | O(n) for a copy nobody sees |
| `nums = sorted(nums)` | **No** | O(n) |
| `nums = nums + [x]` | **No**: `+` builds a new list | O(n) |
| `return nums[::-1]` from a `-> None` method | **No**: the return value is ignored | O(n) |

Tools in this chapter: slicing **[USE-FREELY]**, `len` **[USE-FREELY]**, `set`
**[USE-FREELY]**, `list.reverse` **[KNOW-HOW]**, `sorted` / `list.sort`
**[KNOW-HOW]**, the two-pointer swap **[HAND-WRITE]**, the write-pointer pass
**[HAND-WRITE]**.

## 1.2 Explanations by group

### 1.2.1 Why the class exists

You write a class; the judge writes the rest. LeetCode's driver code isn't public,
but its observable behavior is simple: for each test case it builds the inputs,
calls your method on a `Solution` instance, and checks the result. This book's
`code/ch01_anatomy/mini_judge.py` models that contract:

<!-- file: code/ch01_anatomy/mini_judge.py -->

```python
def judge_return(solution_cls: type, method: str, args: tuple, expected: Any) -> bool:
    """Judge a problem where the answer is the return value."""
    result = getattr(solution_cls(), method)(*copy.deepcopy(args))
    return result == expected


def judge_in_place(solution_cls: type, method: str, args: tuple, expected: Any,
                   arg_index: int = 0) -> bool:
    """Judge a `-> None` problem: the answer is what the argument now holds."""
    args = copy.deepcopy(args)
    returned = getattr(solution_cls(), method)(*args)
    # Whatever the method returns is ignored; only the caller's object counts.
    del returned
    return args[arg_index] == expected
```

Three consequences:

1. **No `input()`, no `print()` for the answer.** The inputs arrive as arguments, and
   the answer leaves as a return value or a mutation. A `print` doesn't count as an
   answer; it only shows up in LeetCode's stdout panel.
2. **The method name must match exactly.** The judge looks the method up by name.
3. **Your class must not carry state between test cases.** Whether the judge creates
   a fresh `Solution()` per test case or reuses one isn't documented. Write code that's
   correct either way: keep all state in local variables.

### 1.2.2 `self`, helper methods, nested functions, `nonlocal`

There are three ways to split a solution into pieces. All three are fine on LeetCode.

| Style | Access to the method's locals | When to use |
|---|---|---|
| Helper method `self._helper(args)` | Only through arguments | Independent logic you want to test on its own, or recursion |
| Nested function | Reads them directly (a closure) | Short helpers that need several of the method's variables |
| Module-level function | Only through arguments | Utilities shared by several classes in one file |

```python
class Solution:
    def countEven(self, nums: list[int]) -> int:
        return sum(1 for x in nums if self._is_even(x))   # helper method

    def _is_even(self, x: int) -> bool:
        return x % 2 == 0
```

The same helper as a nested function, plus a counter it updates:

```python
def count_negatives(nums: list[int]) -> int:
    count = 0

    def visit(x: int) -> None:
        nonlocal count          # without this line: UnboundLocalError
        if x < 0:
            count += 1

    for x in nums:
        visit(x)
    return count


count_negatives([-1, 2, -3])  # -> 2
```

**Why `nonlocal` is needed.** `count += 1` is an assignment. Python decides at compile
time that any name assigned inside a function is local to that function, so without
`nonlocal`, `visit` has its own `count` that was never set:

```python
def broken() -> int:
    count = 0

    def visit() -> None:
        count += 1

    visit()
    return count


broken()  # -> UnboundLocalError: cannot access local variable 'count'...
```

(Message checked on Python 3.11.4 and 3.13.2. Older versions word it differently.)

`nonlocal` is only needed to **rebind** a name. Mutating an object through the name
needs nothing: `seen.add(x)` or `result.append(x)` inside a nested function works
without any declaration, because the name `seen` is never reassigned.

**Recursion has a depth limit.** CPython's default limit is 1000 frames
(`sys.getrecursionlimit()`; the value is a CPython default, not a language rule).
A recursive helper that recurses once per element fails on LeetCode-sized inputs:

```python
import sys

sys.getrecursionlimit()  # -> 1000


def reverse_recursive(s: list[str], left: int, right: int) -> None:
    if left < right:
        s[left], s[right] = s[right], s[left]
        reverse_recursive(s, left + 1, right - 1)


data = ["a"] * 10_000          # 344 allows up to 10^5 characters
last = len(data) - 1
reverse_recursive(data, 0, last)  # -> RecursionError: maximum recursion depth exceeded
```

Prefer loops for linear scans. Ch 11 covers raising the limit and when that's safe.

### 1.2.3 Return contracts

The judge compares the value you return with the expected value. Return exactly the
type in the signature; don't count on the judge converting anything for you. Two
mistakes that catch beginners:

```python
def max_value(nums: list[int]) -> int:
    best = max(nums)            # computed, never returned


print(max_value([1, 5, 2]))  # -> None
```

A function that reaches its end without `return` returns `None`. The judge shows it
as `null`.

```python
def squares(nums: list[int]) -> list[int]:
    return (x * x for x in nums)        # a generator, not a list


type(squares([1, 2]))  # -> <class 'generator'>
```

When the signature says `list[int]`, return a list: `[x * x for x in nums]`.

### 1.2.4 `-> None` means "modify in place"

When the signature returns `None`, the judge ignores your return value and inspects
the object it passed in. That's why this fails:

```python
def reverse_wrong(s: list[str]) -> None:
    s = s[::-1]                 # builds a new list; points the LOCAL name at it


letters = ["a", "b", "c"]
reverse_wrong(letters)
print(letters)  # -> ['a', 'b', 'c']
```

What happens in memory:

```text
Before the call:               Inside reverse_wrong, after s = s[::-1]:

letters ──► [a, b, c]          letters ──► [a, b, c]     <- judge inspects this
                               s ───────► [c, b, a]      <- new list, discarded
                                                            when the call returns
```

Assignment to a bare name (`s = ...`) never changes an object. It only changes
which object the name refers to. To change the caller's object, mutate it:

```python
def reverse_slice(s: list[str]) -> None:
    s[:] = s[::-1]              # replace the CONTENTS of the same list object


def reverse_swap(s: list[str]) -> None:
    left, right = 0, len(s) - 1
    while left < right:
        s[left], s[right] = s[right], s[left]
        left += 1
        right -= 1


letters = ["a", "b", "c"]
reverse_slice(letters)
print(letters)  # -> ['c', 'b', 'a']
reverse_swap(letters)
print(letters)  # -> ['a', 'b', 'c']
```

```text
s[:] = s[::-1]:

letters ──┐
          ├──► [c, b, a]     same object, new contents
s ────────┘
```

Both are accepted by the judge. They differ in auxiliary space:

| Version | Time (reason) | Auxiliary space (reason) |
|---|---|---|
| `s[:] = s[::-1]` | O(n): one copy, then one write of n items | O(n): `s[::-1]` is a temporary copy |
| Two-pointer swap | O(n): n // 2 swaps of O(1) each | O(1): two indices |
| `s.reverse()` **[KNOW-HOW]** | O(n): n // 2 swaps in C | O(1) |

Problem 344 Reverse String
(<https://leetcode.com/problems/reverse-string/>) asks you to reverse a list of
characters in place using O(1) extra memory. That rules out `s[:] = s[::-1]` even
though the judge would accept its output. The interview answer is the swap loop:

<!-- file: code/ch01_anatomy/reverse_string.py -->

```python
class Solution:
    def reverseString(self, s: list[str]) -> None:
        # O(n) time: n // 2 swaps, each O(1).
        # O(1) auxiliary space: two indices; the swap tuple has a fixed size.
        left, right = 0, len(s) - 1
        while left < right:
            s[left], s[right] = s[right], s[left]
            left += 1
            right -= 1
```

LeetCode's signature is `def reverseString(self, s: List[str]) -> None:`. The book's
`list[str]` is the same thing written in the modern style (3.9+). Ch 2 explains both.

**Slice assignment is also the quick fix for "I built a new list".** For 88 Merge
Sorted Array (<https://leetcode.com/problems/merge-sorted-array/>), where the result
must end up in `nums1`, this is accepted:

```python
def merge_simple(nums1: list[int], m: int, nums2: list[int], n: int) -> None:
    nums1[:] = sorted(nums1[:m] + nums2)


nums1 = [1, 2, 3, 0, 0, 0]
merge_simple(nums1, 3, [2, 5, 6], 3)
print(nums1)  # -> [1, 2, 2, 3, 5, 6]
```

It's O((m + n) log(m + n)) time because it sorts m + n items, and O(m + n) auxiliary
space for the slice, the concatenation and the sorted copy. Ch 13 does it in O(m + n)
time and O(1) space with three pointers, which is what an interviewer expects.

**`nums[:] = ...` accepts any iterable**, including an iterator over the same list.
CPython builds the complete new contents before it writes anything:

```python
nums = [1, 2, 3]
nums[:] = reversed(nums)
nums  # -> [3, 2, 1]
```

### 1.2.5 "Return `k` and modify `nums`"

Some problems want both: a count `k` returned, and the first `k` slots of `nums`
rewritten. The judge reads `nums[:k]` from **the same list it passed in**. Anything at
index `k` or later is ignored.

| Problem | What the judge does with `nums[:k]` |
|---|---|
| 27 Remove Element (<https://leetcode.com/problems/remove-element/>): remove every occurrence of `val` | **Sorts** it, then compares. Order doesn't matter. |
| 26 Remove Duplicates from Sorted Array (<https://leetcode.com/problems/remove-duplicates-from-sorted-array/>): keep one copy of each value | Compares it **in order**. |

Both are solved with a write pointer: scan with one index, and copy each element
you keep to position `write`, then advance `write`.

<!-- file: code/ch01_anatomy/remove_element.py -->

```python
class Solution:
    def removeElement(self, nums: list[int], val: int) -> int:
        # O(n) time: one pass. O(1) auxiliary space: one write index.
        write = 0
        for num in nums:
            if num != val:
                nums[write] = num
                write += 1
        return write
```

LeetCode's signature: `def removeElement(self, nums: List[int], val: int) -> int:`.

Trace on `nums = [3, 2, 2, 3]`, `val = 3`:

| Step | `num` | Kept? | `nums` after | `write` |
|---|---|---|---|---|
| 1 | 3 | no | `[3, 2, 2, 3]` | 0 |
| 2 | 2 | yes | `[2, 2, 2, 3]` | 1 |
| 3 | 2 | yes | `[2, 2, 2, 3]` | 2 |
| 4 | 3 | no | `[2, 2, 2, 3]` | 2 |

Returned `k = 2`; the judge reads `nums[:2] == [2, 2]`. The leftover `2, 3` at the end
is ignored.

The wrong version builds a new list:

<!-- file: code/ch01_anatomy/remove_duplicates.py -->

```python
class SolutionReturnsListLength:
    """Wrong: builds a new list. The judge reads the caller's nums, not this one."""

    def removeDuplicates(self, nums: list[int]) -> int:
        unique = sorted(set(nums))
        return len(unique)          # k is right, but nums[:k] is unchanged
```

On `[1, 1, 2]` this returns `k = 2`, but `nums[:2]` is still `[1, 1]`: Wrong Answer.

### 1.2.6 Running LeetCode code locally

Run your class with the judge's contract, not your own. For each kind of problem:

| Problem kind | Local test |
|---|---|
| Return value | `assert Solution().method(args) == expected` |
| `-> None` | Call the method on a list you keep a name for, then `assert that_list == expected` |
| Return `k` + modify | `k = Solution().method(nums)`, then check `k` and `nums[:k]` (sorted or not, as the problem's judge does) |

Every file in `code/` ends with such a block. Here is the one for 27, which mirrors
its judge (sort the prefix, then compare):

<!-- file: code/ch01_anatomy/remove_element.py -->

```python
def judge(nums: list[int], val: int, expected: list[int]) -> bool:
    k = Solution().removeElement(nums, val)
    return k == len(expected) and sorted(nums[:k]) == sorted(expected)


if __name__ == "__main__":
    assert judge([3, 2, 2, 3], 3, [2, 2])
    assert judge([0, 1, 2, 2, 3, 0, 4, 2], 2, [0, 0, 1, 3, 4])
    assert judge([], 1, [])                    # empty (allowed: length 0..100)
    assert judge([1], 1, [])                   # single element, removed
    assert judge([1], 2, [1])                  # single element, kept
    assert judge([4, 4, 4], 4, [])             # all equal to val
    assert judge([5, 6, 7], 0, [5, 6, 7])      # nothing to remove
    assert judge(list(range(51)) + [50] * 49, 50, list(range(50)))  # max length 100
    print("all tests passed")
```

Edge cases to test on every array problem: empty input (if the constraints allow
it), a single element, all-equal values, negatives (if allowed), and the maximum
size. The maximum-size test catches complexity problems before the judge does.

### 1.2.7 Reading constraints: a preview

The constraints block isn't decoration. It tells you how fast your solution must
be, before you've written anything. For example, `1 <= nums.length <= 10^5` means a
solution doing n² = 10¹⁰ steps is hopeless, and O(n log n) or O(n) is the target.

A rule-of-thumb table for Python (Ch 10 derives it):

| Largest n in the constraints | Target complexity | Typical technique |
|---|---|---|
| ≤ 10 | O(n!) | Try all permutations |
| ≤ 20 | O(2ⁿ) or O(n · 2ⁿ) | Try all subsets |
| ≤ 100 | O(n³) | Triple loop (at n = 500, O(n³) is far too slow in Python) |
| ≤ 3,000 | O(n²) | Double loop (at n = 5,000, O(n²) takes seconds in Python) |
| ≤ 10⁵ – 10⁶ | O(n log n) or O(n) | Sorting, hashing, two pointers, sliding window |
| ≥ 10⁹ (a value, not a length) | O(log n) or O(√n) | Binary search, math |

Ch 10 measures Python's speed (5–11 million simple loop iterations per second on this
book's machine), and that's where these limits come from.

Also read the **value** ranges: `-10^9 <= nums[i] <= 10^9` tells you values can be
negative (which breaks some sliding-window ideas, Ch 17) and too large to use as array
indices (so counting sort is out, Ch 18).

## 1.3 Gotchas & traps

**1. Rebinding the parameter in a `-> None` method.** Covered in 1.2.4. The judge sees
the old contents.

**2. Returning a value from a `-> None` method.** The judge ignores it:

```python
def reverse_returning(s: list[str]) -> None:
    return s[::-1]


letters = ["x", "y"]
reverse_returning(letters)  # -> ['y', 'x']
letters  # -> ['x', 'y']
```

**3. Missing `return`.** The method returns `None`; covered in 1.2.3.

**4. State stored on the class.** A class attribute is created once, when the class
body runs, and is shared by every instance:

```python
class Solution:
    seen = set()                     # created once, shared

    def containsDuplicate(self, nums: list[int]) -> bool:
        for num in nums:
            if num in self.seen:
                return True
            self.seen.add(num)
        return False


Solution().containsDuplicate([1, 2, 3])  # -> False
Solution().containsDuplicate([3, 4])  # -> True
```

The second test case has no duplicates, but `seen` still holds `{1, 2, 3}` from the
first. Even a brand-new instance shares it. Keep state in local variables.

**5. A mutable default argument in a helper.** Same mechanism: the default list is
created once, when `def` runs:

```python
def collect(x: int, acc: list[int] = []) -> list[int]:
    acc.append(x)
    return acc


collect(1)  # -> [1]
collect(2)  # -> [1, 2]
```

Use `acc: list[int] | None = None` (3.10+) and `if acc is None: acc = []` inside.

**6. `+=` on a list mutates the caller's list; `+` doesn't:**

```python
def add_plus_equals(nums: list[int]) -> None:
    nums += [0]                      # list.__iadd__ = extend: same object


def add_plus(nums: list[int]) -> None:
    nums = nums + [0]                # new object, local name only


a = [1]
add_plus_equals(a)
a  # -> [1, 0]
add_plus(a)
a  # -> [1, 0]
```

**7. Modifying a list while iterating over it skips elements:**

```python
nums = [2, 2, 3]
for x in nums:
    if x == 2:
        nums.remove(x)
nums  # -> [2, 3]
```

The loop's internal index moves to position 1 after the first removal, but the
second `2` has shifted into position 0. Use a write pointer (1.2.5) or build a new
list and assign it back with `nums[:] = ...`.

**8. A negative `k` in a slice is not the same as `k` from the end when `k > n`:**

```python
def rotate_no_mod(nums: list[int], k: int) -> None:
    nums[:] = nums[-k:] + nums[:-k]


nums = [1, 2, 3]
rotate_no_mod(nums, 4)               # rotating 3 items by 4 = rotating by 1
nums  # -> [1, 2, 3]
```

Expected `[3, 1, 2]`. When `k > n`, `nums[-k:]` is the whole list and `nums[:-k]` is
empty, so nothing moves. Reduce first: `k %= len(nums)` (see
`code/ch01_anatomy/rotate_slice_assign.py`; 189 allows `k` up to 10⁵ with `n` as
small as 1).

**9. Recursing once per element.** Fails above ~1000 levels; see 1.2.2.

**10. Changing the method name or parameter list.** The judge calls the method by
name, with the template's arguments. A renamed method, or an extra required
parameter, produces a runtime error on the judge, not a Wrong Answer.

## 1.4 Benchmarks

None. This chapter makes no performance claims.

## 1.5 Exercises

Answers are in Appendix D (verified in `code/ch01_anatomy/exercises.py`).

**Warm-up**

1. Inside a `-> None` method, which of these change the caller's list?
   (a) `nums.sort()` (b) `nums = sorted(nums)` (c) `nums[:] = sorted(nums)`
   (d) `nums += [0]` (e) `nums = nums + [0]` (f) `nums[0], nums[-1] = nums[-1], nums[0]`
2. Fix `def reverseString(self, s): s = s[::-1]` in two different ways. Give the
   auxiliary space of each, with the reason.
3. A method returns `k = 3`, and afterwards `nums == [2, 3, 1, 9, 9]`. The expected
   values are `[1, 2, 3]`. Is that accepted under 27's judge? Under 26's?
4. A `-> int` method computes the answer but has no `return` statement. What does the
   judge receive?

**Core**

5. Find the bug: a `Solution` class counts calls with a class attribute `calls = 0`,
   and each call does `Solution.calls += 1` and returns `Solution.calls`. What do two
   separate test cases receive?
6. A nested function does `count += 1` on a variable from the enclosing method. Name
   the exception, explain it, and fix it.
7. For each constraint, give the target complexity: (a) `n <= 2 * 10^5`
   (b) `n <= 18` (c) `n <= 300` (d) `1 <= x <= 2^31 - 1` where `x` is a single number.

**Tricky**

8. For 189, why is `k %= len(nums)` needed in the slice-assignment solution? What does
   the version without it produce for `nums = [1, 2, 3]`, `k = 4`?
9. Write a local helper `check_in_place(method, args, expected)` that tests any
   `-> None` method without changing the test's own copy of `args`.
10. Does `nums[:] = reversed(nums)` reverse correctly, given that `reversed(nums)`
    reads the list while it is being replaced? Explain.

## 1.6 Chapter cheat sheet

| Rule | Detail |
|---|---|
| The judge calls you | No `input()`; no `print()` for the answer; exact method name |
| No state between test cases | Locals only. No class attributes, no mutable defaults |
| `-> None` | Mutate the argument: index assignment, swaps, `.reverse()`, `.sort()`, `nums[:] = ...` |
| Never in `-> None` | `nums = ...` (rebinds a local), `return new_list` (ignored) |
| Return `k` + modify | Judge reads `nums[:k]` of *its* list; 27 sorts it, 26 doesn't |
| `+=` vs `+` on lists | `+=` mutates (extend); `+` builds a new list |
| Recursion | ~1000-frame default limit in CPython; loop over linear data |
| Constraints first | n ≤ 10⁵ → O(n log n) or better (Ch 10) |
| Local tests | Mirror the judge: check the return value, or the mutated argument |
