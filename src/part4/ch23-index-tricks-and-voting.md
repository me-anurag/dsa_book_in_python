# Chapter 23: Index Tricks & Voting

## 23.0 At a glance

- **What this pattern solves:** O(n)-time, **O(1)-extra-space** answers to "missing /
  duplicate / majority" questions that would normally need a set or a dict. They work
  by exploiting **structure in the values**: values in 1..n can serve as indices, XOR
  cancels pairs, and a majority survives pairwise cancellation.
- **Prerequisites:** Ch 3 (the self-referencing swap trap, 3.2.9), Ch 13 (in-place
  writes), Ch 14 (the hash-set versions these replace), Ch 19 (binary search on values).
- **Typical complexity:** O(n) time, O(1) auxiliary space.
- **Tools used in this chapter:**
  - `set`, `Counter` **[USE-FREELY]** / **[KNOW-HOW]** (the baselines)
  - `^` (XOR) **[USE-FREELY]**
  - cyclic sort, sign marking, Floyd's cycle detection, Boyer–Moore voting and binary
    search on value counts **[HAND-WRITE]**

## 23.1 Recognition signals

- Values are guaranteed to lie in **1..n** (or 0..n) for an array of length about n
- "**missing**", "**duplicate**", "**disappeared**", "**first missing positive**"
- "**O(1) extra space**" or "**without modifying the array**" on a problem a set would
  solve
- "every element appears **twice except one**" → XOR
- "appears **more than n/2 (or n/3) times**" → Boyer–Moore voting

**Look-alikes:**

| Looks like | Actually | Tell |
|---|---|---|
| Index-as-hash | Hash set (Ch 14) | No 1..n value range, or O(n) space is allowed |
| Cyclic sort (moves values) | Sign marking (448) | Sign marking keeps positions; both mutate the array |
| Either of those | Floyd / binary search on value (287) | "**Must not modify** the array" |
| Majority vote | Counter | No O(1)-space requirement |

## 23.2 Core idea

A set answers "have I seen v?" with O(n) extra memory. When every value is in 1..n, the
array **already has** a slot for each value: index v − 1. You can use that slot to
record information:

| Trick | What the slot stores | Cost |
|---|---|---|
| **Cyclic sort** | Move value v into slot v − 1 | Mutates order; O(n) total swaps |
| **Sign marking** | Make `nums[v - 1]` negative to mean "v seen" | Mutates signs; `abs()` recovers values |
| **Functional graph** | Treat `i → nums[i]` as a linked list | Read-only |

Two value-algebra tricks need no slots at all:

- **XOR:** `x ^ x == 0` and `x ^ 0 == x`, in any order. XOR everything, and the pairs
  vanish.
- **Voting:** cancel pairs of **different** values. A value with more than half the
  copies can't be fully cancelled, so it's the survivor.

```text
cyclic sort on [3, 4, -1, 1]   (value v belongs at index v - 1)

i=0: 3 -> swap with index 2   [-1, 4, 3, 1]
i=0: -1 out of range, move on
i=1: 4 -> swap with index 3   [-1, 1, 3, 4]
i=1: 1 -> swap with index 0   [1, -1, 3, 4]
i=1: -1 out of range, move on
...  final                    [1, -1, 3, 4]
first i with nums[i] != i + 1 is i = 1  ->  answer 2
```

## 23.3 The template

**Cyclic sort** (values 1..n go home; anything else is ignored):

```python
def cyclic_sort(nums: list[int]) -> None:
    n = len(nums)
    for i in range(n):
        while 1 <= nums[i] <= n and nums[nums[i] - 1] != nums[i]:
            j = nums[i] - 1                     # target index, computed first
            nums[i], nums[j] = nums[j], nums[i]


a = [3, 4, -1, 1]
cyclic_sort(a)
a  # -> [1, -1, 3, 4]
```

- **Invariant:** once a value v is at index v − 1, it never moves again.
- **Complexity:** O(n) time. Each swap places at least one value permanently, so there
  are at most n swaps across the whole run, plus O(n) loop steps. O(1) auxiliary space.
- **The swap must compute `j` first.** `nums[i], nums[nums[i] - 1] = ...` assigns
  `nums[i]` before evaluating the second target's index, which silently breaks
  (Ch 3, 3.2.9).
- **The `nums[j] != nums[i]` check** stops infinite swapping when the target already holds
  a duplicate of the same value.

**Boyer–Moore voting** (strict majority > n/2):

```python
def majority(nums: list[int]) -> int:
    candidate, count = nums[0], 0
    for x in nums:
        if count == 0:
            candidate = x
        count += 1 if x == candidate else -1
    return candidate


majority([2, 2, 1, 1, 1, 2, 2])  # -> 2
```

O(n) time, O(1) space. The result is guaranteed only if a majority exists; otherwise,
verify it with a second pass.

## 23.4 Dry run

Boyer–Moore on `[2, 2, 1, 1, 1, 2, 2]`:

| x | count before | Action | candidate | count after |
|---|---|---|---|---|
| 2 | 0 | adopt 2 | 2 | 1 |
| 2 | 1 | match | 2 | 2 |
| 1 | 2 | mismatch | 2 | 1 |
| 1 | 1 | mismatch | 2 | 0 |
| 1 | 0 | adopt 1 | 1 | 1 |
| 2 | 1 | mismatch | 1 | 0 |
| 2 | 0 | adopt 2 | 2 | 1 |

Survivor: 2. Every decrement pairs one 2 with one non-2 (or one 1 with one non-1). There
are four 2s and three others, so at least one 2 is left unpaired.

## 23.5 Variations

| Variant | What changes | Example |
|---|---|---|
| Missing from 0..n | XOR of indices and values, or the arithmetic-sum formula | 268 Missing Number |
| Single among pairs | XOR everything | 136 Single Number |
| All missing from 1..n | Sign marking, then collect the positive slots | 448 Find All Numbers Disappeared in an Array |
| Duplicate, read-only | Binary search on value counts, or Floyd's cycle detection | 287 Find the Duplicate Number |
| Majority > n/2 | One-candidate voting | 169 Majority Element |
| All > n/3 | Two candidates, then verify | 229 Majority Element II |
| First missing positive | Cyclic sort, then the first mismatch | 41 First Missing Positive |

## 23.6 Worked problems

### 23.6.1 41 First Missing Positive (Hard): <https://leetcode.com/problems/first-missing-positive/>

**Problem.** Return the smallest positive integer that doesn't appear in the array.
Required: O(n) time and O(1) auxiliary space.

**Signatures.** LeetCode: `def firstMissingPositive(self, nums: List[int]) -> int:`. Book:
`list[int]`.

**Constraints → target.** n ≤ 10⁵; values span the full 32-bit range. **Key fact:** the
answer is in 1..n + 1, since n values can occupy at most the slots 1..n. Only values in
1..n matter.

**Approach 1: sort.**

<!-- file: code/ch23_index_tricks/first_missing_positive.py -->

```python
class SolutionSort:
    def firstMissingPositive(self, nums: list[int]) -> int:
        # O(n log n) time (sort). O(n) auxiliary. Violates both requirements.
        want = 1
        for x in sorted(nums):
            if x == want:
                want += 1
        return want
```

**Approach 2: set.**

<!-- file: code/ch23_index_tricks/first_missing_positive.py -->

```python
class SolutionSet:
    def firstMissingPositive(self, nums: list[int]) -> int:
        # O(n) average time. O(n) auxiliary: the set. Violates the space limit.
        present = set(nums)
        want = 1
        while want in present:
            want += 1                           # at most n + 1 iterations
        return want
```

**Approach 3: cyclic sort (optimal).**

<!-- file: code/ch23_index_tricks/first_missing_positive.py -->

```python
class Solution:
    def firstMissingPositive(self, nums: list[int]) -> int:
        # Cyclic sort. O(n) time: every swap puts one value into its final slot
        # (value v belongs at index v - 1), so there are at most n swaps in total.
        # O(1) auxiliary space. Mutates nums (allowed here).
        n = len(nums)
        for i in range(n):
            while 1 <= nums[i] <= n and nums[nums[i] - 1] != nums[i]:
                j = nums[i] - 1                 # compute the target FIRST (Ch 3, 3.2.9)
                nums[i], nums[j] = nums[j], nums[i]
        for i in range(n):
            if nums[i] != i + 1:
                return i + 1
        return n + 1                            # 1..n all present
```

Why it works: after the placement pass, every value v in 1..n that occurs somewhere sits
at index v − 1. The first index i holding something else means i + 1 is missing. If all
match, the answer is n + 1. Traced in 23.2.

**Edge cases tested:**
- the three statement examples
- a single element, duplicates
- ±2³¹
- a shuffled 1..10⁵ (answer n + 1)
- 1,000 random cases checked against the set version

**Common bugs:**
- `nums[i], nums[nums[i] - 1] = nums[nums[i] - 1], nums[i]`: the Ch 3 swap trap. It
  loops forever or corrupts data.
- Missing the `nums[j] != nums[i]` duplicate guard, which causes an infinite loop on
  `[1, 1]`.
- Returning n instead of n + 1 when 1..n are all present.

**Built-in shortcut:** the set (approach 2) is fine on LeetCode's judge; it fails the
stated requirement.

### 23.6.2 287 Find the Duplicate Number (Medium): <https://leetcode.com/problems/find-the-duplicate-number/>

**Problem.** An array of n + 1 integers, each in 1..n, has exactly one value that
repeats (possibly more than twice). Return it **without modifying** the array, using
constant extra space. The follow-up asks for linear time.

**Signatures.** LeetCode: `def findDuplicate(self, nums: List[int]) -> int:`. Book:
`list[int]`.

**Constraints → target.** n ≤ 10⁵. Read-only and O(1) space rules out sorting, cyclic
sort, sign marking and sets.

**Approach 1: set (violates the space limit).**

<!-- file: code/ch23_index_tricks/find_duplicate.py -->

```python
class SolutionSet:
    def findDuplicate(self, nums: list[int]) -> int:
        # O(n) average time, O(n) auxiliary space. Violates the space limit.
        seen: set[int] = set()
        for x in nums:
            if x in seen:
                return x
            seen.add(x)
        raise ValueError("no duplicate")
```

**Approach 2: binary search on the value (O(n log n), meets the constraints).**

<!-- file: code/ch23_index_tricks/find_duplicate.py -->

```python
class SolutionCountBinarySearch:
    def findDuplicate(self, nums: list[int]) -> int:
        # Binary search on the VALUE: count(x) = how many entries are <= x.
        # Without the duplicate, count(x) == x; the first x with count(x) > x is
        # the duplicate (pigeonhole). O(n log n) time, O(1) space, read-only.
        lo, hi = 1, len(nums) - 1
        while lo < hi:
            mid = (lo + hi) // 2
            if sum(1 for x in nums if x <= mid) > mid:
                hi = mid
            else:
                lo = mid + 1
        return lo
```

This is Ch 19's "first True" over values 1..n, with the predicate `count(≤ x) > x`, which
is monotone.

**Approach 3: Floyd's cycle detection (O(n), the follow-up).**

<!-- file: code/ch23_index_tricks/find_duplicate.py -->

```python
class Solution:
    def findDuplicate(self, nums: list[int]) -> int:
        # Floyd's cycle detection on the "linked list" i -> nums[i]. O(n) time
        # (the follow-up), O(1) space, read-only. Index 0 has no incoming edge
        # (values are 1..n), so it's the list head; the duplicate value has two
        # incoming edges, so it's where the cycle begins.
        slow = fast = nums[0]
        while True:
            slow = nums[slow]
            fast = nums[nums[fast]]
            if slow == fast:
                break                            # met somewhere inside the cycle
        slow = nums[0]
        while slow != fast:                      # distance math: they meet at the entry
            slow = nums[slow]
            fast = nums[fast]
        return slow
```

Why it works:
1. **The structure.** Follow `i → nums[i]` from index 0. Every value is a valid index, so
   the walk never stops. With finitely many indices, it must enter a cycle.
2. **The duplicate is the cycle's entry.** The duplicate value d has two different
   indices pointing to it, and an entry point is exactly a node with two predecessors
   (one from the tail, one from inside the cycle).
3. **Finding the entry.** The tortoise (1 step) and hare (2 steps) meet inside the cycle.
   Restarting one pointer from the head and moving both 1 step makes them meet at the
   entry: if the tail has length T and the cycle has length C, the meeting point sits T
   steps (mod C) before the entry. That's the standard Floyd argument.

**Edge cases tested:**
- the statement examples
- one value repeated n + 1 times (`[3, 3, 3, 3, 3]`)
- the minimum size `[1, 1]`
- 1,000 random valid inputs, with an assertion that the array is unchanged afterwards

**Common bugs:**
- Starting `slow`/`fast` at index 0's value in one phase and at 0 itself in the other.
  Be consistent; the code above starts both phases from `nums[0]`.
- Using cyclic sort or sign marking, which modify the array (forbidden here).
- Binary searching on **indices** instead of **values**.

**Built-in shortcut:** none that meets the constraints.

### 23.6.3 169 Majority Element (Easy): <https://leetcode.com/problems/majority-element/>

**Problem.** Return the value that appears more than ⌊n/2⌋ times; one is guaranteed to
exist. The follow-up asks for O(n) time and O(1) space.

**Signatures.** LeetCode: `def majorityElement(self, nums: List[int]) -> int:`. Book:
`list[int]`.

**Constraints → target.** n ≤ 5 × 10⁴. O(n), and O(1) for the follow-up.

**Approach 1: count.**

<!-- file: code/ch23_index_tricks/majority_element.py -->

```python
class SolutionCounter:
    def majorityElement(self, nums: list[int]) -> int:
        # O(n) time, O(k) space for k distinct values.
        return Counter(nums).most_common(1)[0][0]
```

**Approach 2: sort and take the middle.**

<!-- file: code/ch23_index_tricks/majority_element.py -->

```python
class SolutionSort:
    def majorityElement(self, nums: list[int]) -> int:
        # O(n log n) time, O(n) auxiliary. A majority occupies more than half of the
        # sorted array, so it must cover the middle index.
        return sorted(nums)[len(nums) // 2]
```

**Approach 3: Boyer–Moore voting (optimal).**

<!-- file: code/ch23_index_tricks/majority_element.py -->

```python
class Solution:
    def majorityElement(self, nums: list[int]) -> int:
        # Boyer-Moore voting. O(n) time, O(1) space.
        # Pair off different values and cancel them; the majority has more copies
        # than everything else combined, so it survives the cancellations.
        candidate, count = nums[0], 0
        for x in nums:
            if count == 0:
                candidate = x
            count += 1 if x == candidate else -1
        return candidate                       # valid because a majority is guaranteed
```

Why it works: each decrement cancels one copy of the current candidate against one
different value. The majority has more than n/2 copies, and everything else combined has
fewer than n/2, so cancellations can't exhaust it. Traced in 23.4.

**Edge cases tested:**
- a single element, negatives
- 1,000 random arrays with a guaranteed strict majority, across all three approaches

**Common bugs:**
- Returning the candidate when no majority is guaranteed (that needs a verification
  pass; see 229).
- Initializing `count = 1` and also counting `nums[0]` again in the loop.

**Built-in shortcut:** `Counter(nums).most_common(1)` **[KNOW-HOW]**: O(n) time, O(k)
space.

### 23.6.4 229 Majority Element II (Medium): <https://leetcode.com/problems/majority-element-ii/>

**Problem.** Return every value that appears more than ⌊n/3⌋ times. There can be 0, 1 or
2 of them, and none is guaranteed. The follow-up asks for O(n) time and O(1) space.

**Signatures.** LeetCode: `def majorityElement(self, nums: List[int]) -> List[int]:`. Book:
`list[int]`.

**Constraints → target.** n ≤ 5 × 10⁴. O(n).

**Approach 1: count.**

<!-- file: code/ch23_index_tricks/majority_element_ii.py -->

```python
class SolutionCounter:
    def majorityElement(self, nums: list[int]) -> list[int]:
        # O(n) time, O(k) space.
        limit = len(nums) // 3
        return [v for v, c in Counter(nums).items() if c > limit]
```

**Approach 2: two-candidate voting, then verify (optimal).**

<!-- file: code/ch23_index_tricks/majority_element_ii.py -->

```python
class Solution:
    def majorityElement(self, nums: list[int]) -> list[int]:
        # Boyer-Moore with two candidates. O(n) time, O(1) space.
        # Pass 1: cancel triples of distinct values; anything with > n/3 copies must
        # survive as a candidate. Pass 2: VERIFY, since survivors may be false
        # positives (no majority is guaranteed here).
        cand1, cand2, count1, count2 = None, None, 0, 0
        for x in nums:
            if x == cand1:
                count1 += 1
            elif x == cand2:
                count2 += 1
            elif count1 == 0:
                cand1, count1 = x, 1
            elif count2 == 0:
                cand2, count2 = x, 1
            else:
                count1 -= 1                  # x cancels one copy of each candidate
                count2 -= 1
        limit = len(nums) // 3
        return [c for c in (cand1, cand2)
                if c is not None and nums.count(c) > limit]
```

Why it generalizes: each "else" cancels three **distinct** values at once (x and one copy
of each candidate). A value with more than n/3 copies can't be cancelled completely,
because each cancellation removes 3 elements and there are fewer than n/3 cancellations
available against it. The **order of the `if`/`elif` tests matters**: check for matches
with the existing candidates **before** adopting a new one, or the same value can occupy
both slots.

**Edge cases tested:**
- no qualifying value (`[1, 2, 3]`)
- two qualifying values (`[1, 2]`)
- all equal, a single element
- 2,000 random arrays checked against the Counter version

**Common bugs:**
- Skipping the verification pass, which returns false positives.
- Checking `count1 == 0` before `x == cand2`, which lets both candidates become the same
  value.
- `>=` instead of `>` against n // 3.

**Built-in shortcut:** `Counter` **[KNOW-HOW]** (approach 1). `nums.count(c)` in the
verification is two O(n) scans: fine.

## 23.7 Python-specific notes

- **The self-referencing swap** (Ch 3, 3.2.9) is the number-one cyclic-sort bug in Python.
  Always compute the target index into a variable first.
- **XOR works on negative ints** in Python (two's-complement semantics on unbounded ints):
  `-5 ^ -5 == 0`. 136's negatives are safe.
- **Sum formulas don't overflow** in Python, so 268 can use
  `n * (n + 1) // 2 - sum(nums)` safely. In C++ and Java, XOR is preferred partly to
  avoid overflow.
- **Sign marking needs `abs()`** when reading a value that may already have been flipped.
  In Python, `-0` is just `0`, which is why sign marking is used only when 0 can't
  appear (448's values are 1..n).

```python
-5 ^ -5  # -> 0
3 ^ 5 ^ 3  # -> 5
abs(-7) - 1  # -> 6
```

## 23.8 Practice set

Hints are in Appendix D, under "Chapter 23 practice". Reference solutions:
`code/ch23_index_tricks/practice_solutions.py`.

| # | Title | Difficulty | Link | Hint ref |
|---|---|---|---|---|
| 268 | Missing Number | Easy | <https://leetcode.com/problems/missing-number/> | D.23.1 |
| 136 | Single Number | Easy | <https://leetcode.com/problems/single-number/> | D.23.2 |
| 448 | Find All Numbers Disappeared in an Array | Easy | <https://leetcode.com/problems/find-all-numbers-disappeared-in-an-array/> | D.23.3 |

## 23.9 Chapter cheat sheet

| | |
|---|---|
| **Signals** | Values in 1..n; missing/duplicate/majority; O(1) space or read-only |
| **Cyclic sort** | `while 1 <= v <= n and nums[v-1] != v`: swap into place (compute `j` first) |
| **Sign marking** | `nums[abs(x) - 1] = -abs(...)`; positive slots = missing values |
| **XOR** | Pairs cancel: `x ^ x == 0`; works on negatives in Python |
| **Read-only duplicate** | Binary search on value counts O(n log n); Floyd O(n) |
| **Voting** | > n/2: one candidate. > n/3: two candidates + verification pass |
| **Complexity** | O(n) time, O(1) space |
| **Top 3 bugs** | Self-referencing swap; missing the duplicate guard (infinite loop); skipping verification in 229 |
