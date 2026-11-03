# Chapter 13: Array Fundamentals & In-Place Manipulation

## 13.0 At a glance

- **What this pattern solves:** rearranging, filtering or combining an array **inside the
  array itself**, with O(1) extra space, by scanning with one index and writing with
  another.
- **Prerequisites:** Ch 1 (the `-> None` contract, slice assignment), Ch 3 (aliasing),
  Ch 10 (complexity).
- **Typical complexity:** O(n) time, O(1) auxiliary space.
- **Tools used in this chapter:**
  - `len`, `range`, `enumerate`, slicing **[USE-FREELY]**
  - `list.reverse`, `sorted` **[KNOW-HOW]**
  - the write pointer, filling from the back, the reversal trick and carry propagation
    **[HAND-WRITE]**

## 13.1 Recognition signals

- "**in-place**", "**do not allocate extra space**", "**O(1) extra memory**"
- The return type is `-> None` (Ch 1), or "return k, the number of elements kept"
- "**Remove** / **move** / **keep** elements satisfying a condition, preserving order"
- "**Rotate** by k positions"
- "**Merge** into the first array, which has spare room at the end"
- Digits stored in an array, plus arithmetic ("add one", "add two numbers as arrays")

**Look-alikes:**

| Looks like | Actually | Tell |
|---|---|---|
| Write pointer (this chapter) | Two pointers from opposite ends (Ch 16) | Opposite ends: a sorted array and a pair condition, or a symmetric operation (reverse, palindrome) |
| Filter in place | Sliding window (Ch 17) | A window has a **contiguous range** with a constraint on its contents |
| Rotate | Cyclic sort (Ch 23) | Cyclic sort uses each value as the target index |

## 13.2 Core idea

The naive way to filter, rearrange or merge is to **build a new list** and copy it
back. That's O(n) extra space, which the problem forbids. The waste is that the new
list's prefix is exactly where the input's prefix used to be.

**The write pointer.** Read every element with one index (`read`). Keep a second index
(`write`) marking where the next kept element goes. Because `write <= read` at all
times, writing to `nums[write]` never destroys an element you haven't read yet.

```text
keep non-zeros of [0, 1, 0, 3, 12]:

read →  0  1  0  3  12
        ↑
write   (stays at 0 while read passes zeros; jumps forward on each kept value)

after:  [1, 3, 12 | 3, 12]     nums[:write] = the kept values, in order
                     └── leftovers: overwrite or ignore, as the problem requires
```

Two more tricks finish the toolbox:

- **Fill from the back** when the free space is at the end (88). Writing from the right
  end can't overwrite unread values, for the mirror-image reason.
- **Reverse to rotate** (189). Rotating right by k is: reverse everything, reverse the
  first k, reverse the rest. Three in-place passes, no copy.

## 13.3 The template

The write-pointer filter. It keeps every element for which `keep(x)` is true, in order:

```python
def compact(nums: list[int], keep) -> int:
    """Moves kept elements to the front, in order; returns how many were kept."""
    write = 0
    for read in range(len(nums)):
        if keep(nums[read]):
            nums[write] = nums[read]
            write += 1
    return write


a = [3, 2, 2, 3]
k = compact(a, lambda x: x != 3)
(k, a[:k])  # -> (2, [2, 2])
```

- **Invariant:** before each iteration, `nums[:write]` holds exactly the kept elements of
  `nums[:read]`, in their original order, and `write <= read`.
- **Complexity:** O(n) time, because each element is read once and written at most
  once. O(1) auxiliary space: two indices.

**The reversal helper** (used three times by 189):

```python
def reverse_range(nums: list[int], left: int, right: int) -> None:
    while left < right:
        nums[left], nums[right] = nums[right], nums[left]
        left += 1
        right -= 1


a = [1, 2, 3, 4, 5]
reverse_range(a, 1, 3)
a  # -> [1, 4, 3, 2, 5]
```

O(right − left) time, O(1) auxiliary space.

## 13.4 Dry run

283 Move Zeroes on `[0, 1, 0, 3, 12]`, using the swap version from 13.6.2. It swaps
`nums[read]` into `nums[write]` instead of copying, so the zeros drift to the back.

| Step | `read` | `nums[read]` | Action | `nums` after | `write` after |
|---|---|---|---|---|---|
| 1 | 0 | 0 | skip | `[0, 1, 0, 3, 12]` | 0 |
| 2 | 1 | 1 | swap positions 0, 1 | `[1, 0, 0, 3, 12]` | 1 |
| 3 | 2 | 0 | skip | `[1, 0, 0, 3, 12]` | 1 |
| 4 | 3 | 3 | swap positions 1, 3 | `[1, 3, 0, 0, 12]` | 2 |
| 5 | 4 | 12 | swap positions 2, 4 | `[1, 3, 12, 0, 0]` | 3 |

Invariant check after step 4: `nums[:2] = [1, 3]` are the non-zeros of `nums[:4]`, in
order, and `nums[2:4]` holds only zeros.

## 13.5 Variations

| Variant | What changes in the template | Example |
|---|---|---|
| Remove a value | `keep(x) = x != val`; return `write` | 27 Remove Element (Ch 1) |
| Keep the first copy of each value (sorted input) | Compare with the last **kept** value: `nums[read] != nums[write - 1]`; start `write = 1` | 26 Remove Duplicates from Sorted Array (Ch 1) |
| Move some values to the end, preserving order | Swap instead of copy, or copy then fill the tail | 283 Move Zeroes |
| Merge into an array with spare room at the end | Write pointer starts at the **end** and moves left | 88 Merge Sorted Array |
| Rotate | Three reversals | 189 Rotate Array |
| Digit arithmetic | Scan right to left with a carry | 66 Plus One |
| Partition by a predicate, any order | Write pointer with swaps (or two pointers from both ends) | 905 Sort Array By Parity |
| Build each row from the previous one | New row from adjacent pairs of the previous row | 118 Pascal's Triangle |

## 13.6 Worked problems

### 13.6.1 88 Merge Sorted Array (Easy): <https://leetcode.com/problems/merge-sorted-array/>

**Problem.** Two sorted arrays: `nums1` holds m real values followed by n placeholder
zeros, and `nums2` holds n values. Merge `nums2` into `nums1` so that `nums1` ends up
sorted. Return nothing.

**Signatures.** LeetCode:
`def merge(self, nums1: List[int], m: int, nums2: List[int], n: int) -> None:`.
Book: the same with `list[int]` (Ch 2).

**Constraints → target.** m + n ≤ 200, so anything works for the judge. The follow-up
asks for O(m + n) time; the interview answer also uses O(1) extra space.

**Approach 1: concatenate and sort.**

<!-- file: code/ch13_array_basics/merge_sorted_array.py -->

```python
class SolutionSort:
    def merge(self, nums1: list[int], m: int, nums2: list[int], n: int) -> None:
        # O((m + n) log(m + n)) time: sorts m + n items.
        # O(m + n) auxiliary space: the concatenation and the sorted copy.
        nums1[:] = sorted(nums1[:m] + nums2)
```

It ignores the fact that both inputs are already sorted, and so it pays for a sort.

**Approach 2: forward merge with a copy.** A standard two-way merge, but writing
forward into `nums1` would overwrite its unread values, so first copy them aside:

<!-- file: code/ch13_array_basics/merge_sorted_array.py -->

```python
class SolutionCopyFront:
    def merge(self, nums1: list[int], m: int, nums2: list[int], n: int) -> None:
        # O(m + n) time: one forward merge.
        # O(m) auxiliary space: a copy of nums1's real elements, so the forward
        # writes can't overwrite values we still need.
        front = nums1[:m]
        i = j = 0
        for write in range(m + n):
            if j >= n or (i < m and front[i] <= nums2[j]):
                nums1[write] = front[i]
                i += 1
            else:
                nums1[write] = nums2[j]
                j += 1
```

**Approach 3: merge from the back (optimal).** The free space is at the **end** of
`nums1`, so place the largest remaining element there and move left:

<!-- file: code/ch13_array_basics/merge_sorted_array.py -->

```python
class Solution:
    def merge(self, nums1: list[int], m: int, nums2: list[int], n: int) -> None:
        # O(m + n) time: each step places one element.
        # O(1) auxiliary space: three indices. Filling from the back is safe because
        # write >= i at every step, so we never overwrite an unread nums1 value.
        i, j, write = m - 1, n - 1, m + n - 1
        while j >= 0:
            if i >= 0 and nums1[i] > nums2[j]:
                nums1[write] = nums1[i]
                i -= 1
            else:
                nums1[write] = nums2[j]
                j -= 1
            write -= 1
```

Why the improvement works: `write = i + j + 1` before every step. Since `j >= 0` while
the loop runs, `write > i`, so the slot being written is never one still waiting to be
read. The loop stops when `nums2` is exhausted: whatever remains of `nums1` is already
in place.

**Edge cases tested:**
- m = 0 and n = 0
- all of `nums2` smaller than `nums1`
- all-equal values
- ±10⁹
- 500 random cases checked against approach 1

**Common bugs:**
- Looping `while i >= 0 and j >= 0` and then forgetting to copy the rest of `nums2`
  (the loop above runs on `j` alone, which avoids this).
- Merging forward without a copy, which overwrites unread values.
- `nums1 = ...` instead of mutating (Ch 1).

**Built-in shortcut:** approach 1 uses `sorted` **[KNOW-HOW]**. It's acceptable on
LeetCode, but an interviewer will ask for approach 3.

### 13.6.2 283 Move Zeroes (Easy): <https://leetcode.com/problems/move-zeroes/>

**Problem.** Move every 0 to the end of the array, keeping the non-zero values in their
original order. Do it in place.

**Signatures.** LeetCode: `def moveZeroes(self, nums: List[int]) -> None:`. Book:
`list[int]`.

**Constraints → target.** n ≤ 10⁴ and in-place, without copying: O(n) time, O(1) space.

**Approach 1: build, then copy back.**

<!-- file: code/ch13_array_basics/move_zeroes.py -->

```python
class SolutionNewList:
    def moveZeroes(self, nums: list[int]) -> None:
        # O(n) time, but O(n) auxiliary space: builds a new list. The problem asks
        # for in place without making a copy.
        non_zero = [x for x in nums if x != 0]
        nums[:] = non_zero + [0] * (len(nums) - len(non_zero))
```

**Approach 2: write pointer, then fill.** The 13.3 template with `keep(x) = x != 0`,
then zero out the tail:

<!-- file: code/ch13_array_basics/move_zeroes.py -->

```python
class SolutionWriteThenFill:
    def moveZeroes(self, nums: list[int]) -> None:
        # O(n) time: one pass to compact, one to fill. O(1) auxiliary space.
        write = 0
        for x in nums:
            if x != 0:
                nums[write] = x
                write += 1
        for i in range(write, len(nums)):
            nums[i] = 0
```

**Approach 3: swap in one pass.** Swapping instead of copying carries the zeros along,
so no fill pass is needed (traced in 13.4):

<!-- file: code/ch13_array_basics/move_zeroes.py -->

```python
class Solution:
    def moveZeroes(self, nums: list[int]) -> None:
        # O(n) time: one pass. O(1) auxiliary space.
        # Invariant: nums[:write] holds the non-zeros seen so far, in order;
        # nums[write:read] holds only zeros.
        write = 0
        for read in range(len(nums)):
            if nums[read] != 0:
                nums[write], nums[read] = nums[read], nums[write]
                write += 1
```

Approaches 2 and 3 are both optimal in Big-O. The problem's follow-up asks you to
minimize the total number of operations: approach 3 does one pass instead of two, and
writes nothing extra when zeros are rare. (When `write == read`, the swap is a harmless self-swap.)

**Edge cases tested:**
- a single zero, a single non-zero
- all zeros, no zeros
- negatives
- 500 random arrays
- n = 10⁴

**Common bugs:**
- `nums.remove(0)` in a loop, then appending zeros: O(n²) (Ch 6), and mutating while
  iterating (Ch 1).
- Sorting with `key=lambda x: x == 0`. It's actually correct, because the sort is
  stable, but it's O(n log n) and not in-place in spirit. Say so if you use it.

**Built-in shortcut:** `nums.sort(key=lambda x: x == 0)` **[KNOW-HOW]**: O(n log n) time,
O(n) auxiliary. Correct thanks to stability (Ch 6), but not what's being tested.

### 13.6.3 189 Rotate Array (Medium): <https://leetcode.com/problems/rotate-array/>

**Problem.** Rotate the array to the right by k steps: the last k elements move to the
front. k may exceed n.

**Signatures.** LeetCode: `def rotate(self, nums: List[int], k: int) -> None:`. Book:
`list[int]`.

**Constraints → target.** n ≤ 10⁵ and k ≤ 10⁵. O(n · k) is 10¹⁰: hopeless. The target
is O(n) time. The follow-up asks for at least three approaches, one of them in place
with O(1) extra space: exactly the three below.

**Approach 1: rotate by one, k times.**

<!-- file: code/ch13_array_basics/rotate_array.py -->

```python
class SolutionOneStepAtATime:
    def rotate(self, nums: list[int], k: int) -> None:
        # O(n * k) time: k rotations by one, each shifting n items (insert at 0).
        # O(1) auxiliary space. With n = k = 10^5 that's 10^10 steps: TLE.
        for _ in range(k % len(nums)):
            nums.insert(0, nums.pop())
```

**Approach 2: extra array.** Element i belongs at `(i + k) % n`:

<!-- file: code/ch13_array_basics/rotate_array.py -->

```python
class SolutionExtraArray:
    def rotate(self, nums: list[int], k: int) -> None:
        # O(n) time. O(n) auxiliary space: the rotated copy.
        n = len(nums)
        rotated = [0] * n
        for i, x in enumerate(nums):
            rotated[(i + k) % n] = x
        nums[:] = rotated
```

**Approach 3: three reversals (optimal).**

<!-- file: code/ch13_array_basics/rotate_array.py -->

```python
class Solution:
    def rotate(self, nums: list[int], k: int) -> None:
        # O(n) time: three reversals touch each element at most twice.
        # O(1) auxiliary space: in-place swaps.
        n = len(nums)
        k %= n

        def reverse(left: int, right: int) -> None:
            while left < right:
                nums[left], nums[right] = nums[right], nums[left]
                left += 1
                right -= 1

        reverse(0, n - 1)        # [1..7], k=3 -> [7 6 5 4 3 2 1]
        reverse(0, k - 1)        #              -> [5 6 7 4 3 2 1]
        reverse(k, n - 1)        #              -> [5 6 7 1 2 3 4]
```

Why it works: write the array as A + B, where B is the last k elements. The answer is
B + A. Reversing everything gives rev(B) + rev(A), with B's elements already in the
first k slots, and reversing each block separately restores each block's internal
order. `k %= n` handles k ≥ n (Ch 1, Gotcha 8). When k becomes 0, `reverse(0, -1)` does
nothing, and the two full reversals cancel out.

**Edge cases tested:**
- k = 0, k = n, k > n
- a single element, all-equal values
- 300 random cases against approach 1
- n = 10⁵ with k = 10⁵ − 1

**Common bugs:**
- Forgetting `k %= n`.
- An off-by-one in `reverse(0, k - 1)`: the bounds are inclusive.
- `nums = nums[-k:] + nums[:-k]`, which rebinds instead of mutating.

**Built-in shortcut:** `nums[:] = nums[-k:] + nums[:-k]` after `k %= n` (Ch 1). O(n) time
and O(n) auxiliary space, so it fails the follow-up. It's the fastest version in
CPython, though (13.7).

### 13.6.4 66 Plus One (Easy): <https://leetcode.com/problems/plus-one/>

**Problem.** A non-negative integer is given as a list of its decimal digits, most
significant first, with no leading zeros. Return the digits of that integer plus one.

**Signatures.** LeetCode: `def plusOne(self, digits: List[int]) -> List[int]:`. Book:
`list[int]`.

**Constraints → target.** At most 100 digits. O(d) is trivial; the point is the carry
logic.

**Approach 1: convert to an int.** Legal in Python, where ints are unbounded and 100
digits is far below the 4300-digit conversion limit (Ch 3):

<!-- file: code/ch13_array_basics/plus_one.py -->

```python
class SolutionViaInt:
    def plusOne(self, digits: list[int]) -> list[int]:
        # O(d) time and space for d digits (d <= 100 is far below the 4300-digit
        # int/str limit, Ch 3). Correct in Python, but it skips the point: the
        # interviewer wants the carry logic.
        value = int("".join(map(str, digits))) + 1
        return [int(c) for c in str(value)]
```

**Approach 2: carry from the right (optimal).**

<!-- file: code/ch13_array_basics/plus_one.py -->

```python
class Solution:
    def plusOne(self, digits: list[int]) -> list[int]:
        # O(d) time worst case (all 9s), O(1) typical: stops at the first non-9.
        # O(1) auxiliary space, except the one case that needs a new leading 1.
        for i in range(len(digits) - 1, -1, -1):
            if digits[i] < 9:
                digits[i] += 1
                return digits
            digits[i] = 0                     # 9 + 1 = 10: write 0, carry 1
        return [1] + digits                   # every digit was 9
```

The carry only continues through 9s. As soon as one digit is below 9, adding one
there absorbs the carry, and we're done. If every digit was 9, the result is a 1
followed by zeros, which is the one case that needs a new list.

**Edge cases tested:**
- `[0]`, `[9]`, all 9s
- a carry that stops midway
- 100 nines
- 500 random cases against approach 1

**Common bugs:**
- Building the number as an int in a language without big integers. In Python it
  "works", but it's not what's being tested.
- Forgetting the all-9s case.
- `range(len(digits) - 1, 0, -1)`, which misses index 0 (Ch 3).

**Built-in shortcut:** approach 1, `int()` **[KNOW-HOW]**. For problems that forbid
conversion (43 Multiply Strings, Ch 9), only the carry pattern is allowed.

## 13.7 Python-specific notes

**O(1) space vs speed in CPython.** Measured (`bench/ch13_rotate.py`): rotating 10⁵ ints
by 33,333, best of 5, two runs:

| Approach | Auxiliary space | Time |
|---|---|---|
| Three reversals (Python swaps) | O(1) | 8.0–9.0 ms |
| Extra array (Python loop) | O(n) | 7.9–8.5 ms |
| `nums[:] = nums[-k:] + nums[:-k]` (C copies) | O(n) | 1.6–1.7 ms |

The O(1)-space answer measured about 5× slower than slice assignment, because its loop runs
in Python while the slices copy in C (Ch 11's golden rule). Both are O(n) and both pass.
**In an interview, give the reversal answer when O(1) space is asked for, and mention
that slicing is faster in CPython at the cost of O(n) memory.** That trade-off is
exactly the kind of judgment interviewers want to hear.

**Idioms:**
- Mutate, don't rebind: `nums[i] = ...`, `nums[:] = ...` (Ch 1).
- A nested helper (`reverse` inside `rotate`) reads `nums` from the enclosing scope. It
  mutates the list without rebinding the name, so no `nonlocal` is needed (Ch 1).
- `range(len(a) - 1, -1, -1)` for right-to-left scans (Ch 3).

## 13.8 Practice set

Hints are in Appendix D, under "Chapter 13 practice".

| # | Title | Difficulty | Link | Hint ref |
|---|---|---|---|---|
| 1929 | Concatenation of Array | Easy | <https://leetcode.com/problems/concatenation-of-array/> | D.13.1 |
| 1480 | Running Sum of 1d Array | Easy | <https://leetcode.com/problems/running-sum-of-1d-array/> | D.13.2 |
| 27 | Remove Element | Easy | <https://leetcode.com/problems/remove-element/> | D.13.3 |
| 26 | Remove Duplicates from Sorted Array | Easy | <https://leetcode.com/problems/remove-duplicates-from-sorted-array/> | D.13.4 |
| 905 | Sort Array By Parity | Easy | <https://leetcode.com/problems/sort-array-by-parity/> | D.13.5 |
| 118 | Pascal's Triangle | Easy | <https://leetcode.com/problems/pascals-triangle/> | D.13.6 |

(27 and 26 are solved in Ch 1. Re-solve them from a blank file as part of this chapter's
practice.)

## 13.9 Chapter cheat sheet

| | |
|---|---|
| **Signals** | in-place, O(1) extra, `-> None`, "return k", remove/move/keep, rotate, merge into the first array, digit arrays |
| **Template** | `write = 0; for read ...: if keep(nums[read]): nums[write] = nums[read]; write += 1` |
| **Invariant** | `nums[:write]` = kept elements of `nums[:read]`, in order; `write <= read` |
| **Variants** | Fill from the back (88); three reversals (189); carry from the right (66); swap to carry the rejects along (283) |
| **Complexity** | O(n) time, O(1) auxiliary |
| **Top 3 bugs** | Rebinding instead of mutating; forgetting `k %= n`; forward-merging over unread values |
| **CPython note** | Slicing is ~5× faster than O(1)-space loops (measured); state the trade-off |
