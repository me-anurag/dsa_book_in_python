# Chapter 14: Hashing & Frequency Counting

## 14.0 At a glance

- **What this pattern solves:** questions of the form "have I seen this before?", "how
  many times?", "which items share a property?" and "where is its partner?". It answers
  them in O(1) average time per element, by trading memory for speed.
- **Prerequisites:** Ch 7 (hashability), Ch 8 (`Counter`, `defaultdict`), Ch 12 (dict
  and set costs).
- **Typical complexity:** O(n) average time, O(n) auxiliary space. With a fixed alphabet
  of size Σ, counting needs only O(Σ) = O(1) space.
- **Tools used in this chapter:**
  - `dict`, `set`, `defaultdict`, `tuple` keys **[USE-FREELY]**
  - `Counter`, `heapq.nlargest`, `sorted` **[KNOW-HOW]**
  - complement lookup, signature keys, bucket sort and run-start detection
    **[HAND-WRITE]**

## 14.1 Recognition signals

- "**exists / contains / duplicate / seen before**" → set
- "**count / frequency / most common / at least k times**" → dict counting
- "find **two elements** that combine to a target", input **not sorted** → complement map
- "**anagram / permutation / same characters**" → frequency signature
- "**group** items that share X" → dict from signature to list
- "**one-to-one mapping / pattern / isomorphic**" → two dicts (both directions)
- "**O(n)** required, and sorting would be O(n log n)" → hashing replaces sorting
- "**top k**" → count, then heap or bucket

**Look-alikes:**

| Looks like | Actually | Tell |
|---|---|---|
| Two Sum with a hash map | Two pointers (Ch 16) | Input **sorted**, or O(1) extra space required (167 Two Sum II) |
| Counting over the whole array | Sliding window with counts (Ch 17, 27) | "**substring / subarray** with at most / exactly k ..." |
| Counting characters | Fixed 26-slot list (Ch 24) | Alphabet known and small; O(1) space asked for |
| "Sum equals k" over subarrays | Prefix sum + hash map (Ch 15) | **Contiguous** subarray |

## 14.2 Core idea

Brute force compares every element with every other one: O(n²). The waste is that each
comparison asks a question whose answer we could have **remembered**. A hash table
stores what you've seen so far, so "have I seen v?" costs O(1) on average instead of a
scan.

```text
Two Sum, target 9, nums = [2, 7, 11, 15]

i=0  num=2   need 7   seen = {}          -> no;  remember 2 -> 0
i=1  num=7   need 2   seen = {2: 0}      -> yes: answer [0, 1]
```

Three families of keys cover almost everything:

| Key | Answers | Example |
|---|---|---|
| The value itself | Seen? Where? How often? | 1, 217, 387 |
| A **signature** (sorted string, count tuple) | Which items are equivalent? | 49, 242 |
| A **derived** value (complement, x − 1, (row, col, box)) | Does the partner, predecessor or conflict exist? | 1, 128, 36 |

## 14.3 The template

**Count, then query:**

```python
def frequencies(items) -> dict:
    counts: dict = {}
    for x in items:
        counts[x] = counts.get(x, 0) + 1
    return counts


frequencies("banana")  # -> {'b': 1, 'a': 3, 'n': 2}
```

**Seen-before in one pass** (the complement variant of this is Two Sum):

```python
def first_repeat(nums: list[int]) -> int | None:
    seen: set[int] = set()
    for x in nums:
        if x in seen:
            return x
        seen.add(x)
    return None


first_repeat([3, 1, 4, 1, 5])  # -> 1
first_repeat([1, 2, 3]) is None  # -> True
```

- **Invariant:** at the start of each iteration, `seen` (or `counts`) describes exactly
  the elements before the current index.
- **Complexity:** O(n) average time, because each element costs O(1) average hash
  operations. O(k) auxiliary space for k distinct keys (≤ n).

**Group by signature:**

```python
from collections import defaultdict


def group_by(items, signature) -> list[list]:
    groups = defaultdict(list)
    for x in items:
        groups[signature(x)].append(x)
    return list(groups.values())


group_by(["ab", "c", "de"], len)  # -> [['ab', 'de'], ['c']]
```

## 14.4 Dry run

128 Longest Consecutive Sequence on `[100, 4, 200, 1, 3, 2]`, set version (14.6.4).
`values = {1, 2, 3, 4, 100, 200}`. A walk starts only at x where x − 1 is absent.

| x | x − 1 in set? | Action | Walk | `best` |
|---|---|---|---|---|
| 1 | no | start a walk | 1 → 2 → 3 → 4 (5 absent): length 4 | 4 |
| 2 | yes | skip | | 4 |
| 3 | yes | skip | | 4 |
| 4 | yes | skip | | 4 |
| 100 | no | start | 100 (101 absent): length 1 | 4 |
| 200 | no | start | 200: length 1 | 4 |

(A set's iteration order isn't specified; the table uses sorted order for readability.
The answer doesn't depend on the order.) Every value is visited by exactly one walk, so
the walks total n steps.

## 14.5 Variations

| Variant | What changes | Example |
|---|---|---|
| Existence | `set`, return on first hit | 217 Contains Duplicate |
| Complement lookup | Key = value, look up `target - x` **before** inserting x | 1 Two Sum |
| Frequency equality | Compare two count maps (or 26-slot lists) | 242 Valid Anagram |
| Frequency ≥ | Each needed count ≤ the available count | 383 Ransom Note |
| First with count 1 | Count first, then scan again in input order | 387 First Unique Character in a String |
| Bijection | Two maps, a→b and b→a, both must stay consistent | 205 Isomorphic Strings, 290 Word Pattern |
| Group by signature | `defaultdict(list)` keyed by sorted string or count tuple | 49 Group Anagrams |
| Top k by count | Count, then a heap (O(n log k)) or buckets (O(n)) | 347 Top K Frequent Elements |
| Run starts | Set + "start only where x − 1 is absent" | 128 Longest Consecutive Sequence |
| Composite keys | `(row, digit)`, `(col, digit)`, `(box, digit)` in sets | 36 Valid Sudoku |

## 14.6 Worked problems

### 14.6.1 1 Two Sum (Easy): <https://leetcode.com/problems/two-sum/>

**Problem.** Given an integer array and a target, return the indices of the two distinct
positions whose values add up to the target. Exactly one answer exists.

**Signatures.** LeetCode: `def twoSum(self, nums: List[int], target: int) -> List[int]:`.
Book: `list[int]`.

**Constraints → target.** n ≤ 10⁴, values up to ±10⁹. All pairs is n²/2 ≈ 5 × 10⁷ checks:
several seconds at the Python budget (Ch 10). The follow-up asks for better than O(n²): O(n) with a
hash map.

**Approach 1: all pairs.**

<!-- file: code/ch14_hashing/two_sum.py -->

```python
class SolutionBruteForce:
    def twoSum(self, nums: list[int], target: int) -> list[int]:
        # O(n^2) time: every pair. O(1) auxiliary space.
        n = len(nums)
        for i in range(n):
            for j in range(i + 1, n):
                if nums[i] + nums[j] == target:
                    return [i, j]
        return []
```

**Approach 2: sort, then two pointers.** Sorting enables the Ch 16 technique, but the
answer needs the **original** indices, so sort the indices by value instead:

<!-- file: code/ch14_hashing/two_sum.py -->

```python
class SolutionSortTwoPointers:
    def twoSum(self, nums: list[int], target: int) -> list[int]:
        # O(n log n) time: sorting the indices dominates; the scan is O(n).
        # O(n) auxiliary space: the sorted index list.
        order = sorted(range(len(nums)), key=lambda i: nums[i])
        left, right = 0, len(order) - 1
        while left < right:
            total = nums[order[left]] + nums[order[right]]
            if total == target:
                return sorted([order[left], order[right]])
            if total < target:
                left += 1
            else:
                right -= 1
        return []
```

**Approach 3: complement map (optimal).**

<!-- file: code/ch14_hashing/two_sum.py -->

```python
class Solution:
    def twoSum(self, nums: list[int], target: int) -> list[int]:
        # O(n) average time: one pass, O(1) average dict operations.
        # O(n) auxiliary space: the dict holds up to n values.
        index_of: dict[int, int] = {}
        for i, num in enumerate(nums):
            complement = target - num
            if complement in index_of:          # check BEFORE inserting num:
                return [index_of[complement], i]   # an element can't pair with itself
            index_of[num] = i
        return []
```

Why it works: when the second element of the answer pair is reached, the first one is
already in `index_of`. One pass is enough, so there's no need to build the whole map
first.

**Edge cases tested:**
- equal values (`[3, 3]`)
- a value that is exactly half the target but appears once (`[3, 2, 4]`, target 6:
  must not return `[0, 0]`)
- negatives, ±10⁹
- n = 10⁴
- 500 random cases checked against brute force

**Common bugs:**
- Inserting before checking, which pairs an element with itself.
- Returning values instead of indices.
- Sorting `nums` itself, which loses the original indices.

**Built-in shortcut:** none needed. The dict *is* the solution (**[USE-FREELY]**); the
complement idea is **[HAND-WRITE]**.

### 14.6.2 49 Group Anagrams (Medium): <https://leetcode.com/problems/group-anagrams/>

**Problem.** Given a list of lowercase strings, group together those that are
rearrangements of each other. Groups and their members may come in any order.

**Signatures.** LeetCode:
`def groupAnagrams(self, strs: List[str]) -> List[List[str]]:`. Book: `list[str]`,
`list[list[str]]`.

**Constraints → target.** N ≤ 10⁴ strings, each of length L ≤ 100. Comparing all pairs is
O(N² · L) = 10¹⁰: hopeless. Computing a key per string and grouping is O(N · L log L)
or O(N · L).

**Approach 1: compare with every group.**

<!-- file: code/ch14_hashing/group_anagrams.py -->

```python
class SolutionPairwise:
    def groupAnagrams(self, strs: list[str]) -> list[list[str]]:
        # O(N^2 * L) time: each string is compared (Counter equality, O(L)) with
        # one representative per existing group, up to N groups.
        # O(N * L) auxiliary space: one Counter per group.
        groups: list[list[str]] = []
        signatures: list[Counter] = []
        for s in strs:
            sig = Counter(s)
            for idx, existing in enumerate(signatures):
                if existing == sig:
                    groups[idx].append(s)
                    break
            else:
                signatures.append(sig)
                groups.append([s])
        return groups
```

**Approach 2: sorted string as the key.** Anagrams sort to the same string:

<!-- file: code/ch14_hashing/group_anagrams.py -->

```python
class SolutionSortedKey:
    def groupAnagrams(self, strs: list[str]) -> list[list[str]]:
        # O(N * L log L) time: sorting each string builds its key.
        # O(N * L) auxiliary space: the keys and the groups.
        groups: defaultdict[str, list[str]] = defaultdict(list)
        for s in strs:
            groups["".join(sorted(s))].append(s)
        return list(groups.values())
```

**Approach 3: count tuple as the key.** Anagrams have identical letter counts. A list
can't be a dict key, but a tuple can (Ch 7):

<!-- file: code/ch14_hashing/group_anagrams.py -->

```python
class Solution:
    def groupAnagrams(self, strs: list[str]) -> list[list[str]]:
        # O(N * (L + 26)) time: one counting pass per string plus a 26-tuple key.
        # O(N * 26) auxiliary space for the keys, plus the groups.
        groups: defaultdict[tuple[int, ...], list[str]] = defaultdict(list)
        base = ord("a")
        for s in strs:
            counts = [0] * 26
            for c in s:
                counts[ord(c) - base] += 1
            groups[tuple(counts)].append(s)      # lists aren't hashable; tuples are
        return list(groups.values())
```

Approach 3 has the better complexity, but approach 2 sorts in C. Which is faster depends
on L, as measured in 14.7.

**Edge cases tested:**
- the empty string, and several empty strings
- a single string
- duplicates of the same word
- 300 random cases checked against approach 1, comparing order-insensitively

**Common bugs:**
- Using `sorted(s)` (a list) as the key, which raises `TypeError: unhashable type`.
- Using `Counter(s)` as the key, which is also unhashable (`frozenset(Counter(s).items())`
  works).
- Comparing outputs in a fixed order in your own tests.

**Built-in shortcut:** `defaultdict(list)` **[USE-FREELY]**, `sorted` **[KNOW-HOW]**.

### 14.6.3 347 Top K Frequent Elements (Medium): <https://leetcode.com/problems/top-k-frequent-elements/>

**Problem.** Return the k values that occur most often. The answer is guaranteed to be
unique, and any order is accepted.

**Signatures.** LeetCode: `def topKFrequent(self, nums: List[int], k: int) -> List[int]:`.
Book: `list[int]`.

**Constraints → target.** n ≤ 10⁵. The follow-up demands **better than O(n log n)**: a
heap gives O(n + u log k), and buckets give O(n) (u = number of distinct values).

**Approach 1: count, sort the distinct values by count.**

<!-- file: code/ch14_hashing/top_k_frequent.py -->

```python
class SolutionSortCounts:
    def topKFrequent(self, nums: list[int], k: int) -> list[int]:
        # O(n + u log u) time: counting is O(n), sorting the u distinct values is
        # O(u log u), which is O(n log n) when most values are distinct.
        # O(u) auxiliary space.
        counts = Counter(nums)
        return sorted(counts, key=counts.get, reverse=True)[:k]
```

**Approach 2: count, then a size-k heap.**

<!-- file: code/ch14_hashing/top_k_frequent.py -->

```python
class SolutionHeap:
    def topKFrequent(self, nums: list[int], k: int) -> list[int]:
        # O(n + u log k) time: a size-k heap over the u distinct values.
        # O(u + k) auxiliary space.
        counts = Counter(nums)
        return heapq.nlargest(k, counts, key=counts.get)
```

**Approach 3: bucket by frequency (optimal).** A frequency is an integer between 1 and
n, so it can be used as a list index. No comparison sort is needed:

<!-- file: code/ch14_hashing/top_k_frequent.py -->

```python
class Solution:
    def topKFrequent(self, nums: list[int], k: int) -> list[int]:
        # O(n) time: counting O(n), bucketing O(u), collecting O(n) at most.
        # O(n) auxiliary space: the count dict and n + 1 buckets.
        # Bucket sort works because a frequency is an integer between 1 and n.
        counts: dict[int, int] = {}
        for x in nums:
            counts[x] = counts.get(x, 0) + 1
        buckets: list[list[int]] = [[] for _ in range(len(nums) + 1)]
        for value, freq in counts.items():
            buckets[freq].append(value)
        result: list[int] = []
        for freq in range(len(buckets) - 1, 0, -1):
            for value in buckets[freq]:
                result.append(value)
                if len(result) == k:
                    return result
        return result
```

Why it beats O(n log n): sorting compares counts, but the counts come from a small
integer range (1..n), so counting sort's idea applies (Ch 9). That's the same reason
counting sort beats comparison sorts.

**Edge cases tested:**
- a single element, all-equal values, negatives
- k = u at maximum size (n = 10⁵)
- 300 random inputs with distinct frequencies (so the top k is unique), checked across
  all three approaches

**Common bugs:**
- `[[]] * (n + 1)` for the buckets, which aliases one list (Ch 3).
- Sizing the buckets by u instead of n + 1 (a frequency can be n).
- Returning counts instead of values.

**Built-in shortcut:** `Counter(nums).most_common(k)` **[KNOW-HOW]**:
O(n + u log k), using a heap internally. Fine on LeetCode; state its cost.

### 14.6.4 128 Longest Consecutive Sequence (Medium): <https://leetcode.com/problems/longest-consecutive-sequence/>

**Problem.** Return the length of the longest run of consecutive integers (x, x + 1,
x + 2, ...) that can be formed from the array's values. Order in the array doesn't
matter, and duplicates count once.

**Signatures.** LeetCode: `def longestConsecutive(self, nums: List[int]) -> int:`. Book:
`list[int]`.

**Constraints → target.** n ≤ 10⁵, values ±10⁹. The statement **requires O(n) time**, so
sorting (O(n log n)) is out, and counting sort is impossible with a value range of
2 × 10⁹.

**Approach 1: extend from every value (brute force).**

<!-- file: code/ch14_hashing/longest_consecutive.py -->

```python
class SolutionBruteForce:
    def longestConsecutive(self, nums: list[int]) -> int:
        # O(n^3) worst case: for each x, extend x+1, x+2, ... and each `in` on a
        # list is O(n). O(1) auxiliary space. Test-only.
        best = 0
        for x in nums:
            length = 1
            while x + length in nums:
                length += 1
            best = max(best, length)
        return best
```

**Approach 2: sort.**

<!-- file: code/ch14_hashing/longest_consecutive.py -->

```python
class SolutionSort:
    def longestConsecutive(self, nums: list[int]) -> int:
        # O(n log n) time: the sort dominates. O(n) auxiliary: the sorted copy.
        # Correct, but violates the stated O(n) requirement.
        if not nums:
            return 0
        values = sorted(set(nums))
        best = run = 1
        for i in range(1, len(values)):
            run = run + 1 if values[i] == values[i - 1] + 1 else 1
            best = max(best, run)
        return best
```

**Approach 3: set, and walk only from run starts (optimal).**

<!-- file: code/ch14_hashing/longest_consecutive.py -->

```python
class Solution:
    def longestConsecutive(self, nums: list[int]) -> int:
        # O(n) average time: each value starts at most one walk (only when x - 1 is
        # absent), and each value is visited by at most one walk, so all the walks
        # together take O(n) steps. O(n) auxiliary space: the set.
        values = set(nums)
        best = 0
        for x in values:                      # iterate the set: no duplicate starts
            if x - 1 in values:
                continue                      # not the start of a run
            length = 1
            while x + length in values:
                length += 1
            best = max(best, length)
        return best
```

Why the improvement works: approach 1 rewalks the same run from every member, which is
quadratic for one long run. Walking only from the **start** of each run (x − 1 absent)
visits each value at most once in total. That's the amortized argument from Ch 10: a
`while` inside a `for`, still O(n). The trace is in 14.4.

**Edge cases tested:**
- an empty array, a single value
- all-equal values, duplicates in a run
- negatives with 10⁹
- one run of 10⁵ shuffled values (worst case for approach 1)
- 500 random cases checked against brute force

**Common bugs:**
- Iterating over `nums` instead of `values`: with many duplicates of a run start, the
  same walk runs repeatedly, which is quadratic on `[1] * n + [2, ..., m]`-style inputs.
- Forgetting the `x - 1` check, which makes it quadratic.
- Returning 1 for an empty input.

**Built-in shortcut:** none; the start-only walk is **[HAND-WRITE]**.

## 14.7 Python-specific notes

**Which anagram key is faster?** Measured (`bench/ch14_anagram_keys.py`, N = 10,000
strings, best of 5, two runs):

| Key | L = 5 | L = 100 |
|---|---|---|
| `"".join(sorted(s))` (C sort, O(L log L)) | 2.9 ms | 71–77 ms |
| 26-slot count tuple (Python loop, O(L + 26)) | 5.1–5.5 ms | 49–53 ms |
| `frozenset(Counter(s).items())` | 18–19 ms | 77–83 ms |

For short strings the C-level sort won, because the count key pays a fixed cost to
build 26 slots. At 49's maximum length (100) the count key won by about 1.4×. Either is
acceptable on LeetCode. State the trade-off: sorted keys are simpler and win for short
strings, while count keys have the better complexity class and win as L grows. A
`Counter`-based key was the slowest in this measurement.

**Hashable keys.** Dict keys must be hashable (Ch 7): use `str`, `tuple`, `int`,
`frozenset`, never `list`, `dict`, `set` or `Counter`.

**Iteration order.** A dict iterates in insertion order (guaranteed since 3.7). A set's
order is arbitrary and, for strings, can change between runs (Ch 7). If your output
order matters (it rarely does here), sort it.

**`defaultdict` vs `dict.get`.** `counts[x] = counts.get(x, 0) + 1` works with a plain
dict. `defaultdict(int)` reads more cleanly, but it inserts keys on read (Ch 8).

## 14.8 Practice set

Hints are in Appendix D, under "Chapter 14 practice".

| # | Title | Difficulty | Link | Hint ref |
|---|---|---|---|---|
| 217 | Contains Duplicate | Easy | <https://leetcode.com/problems/contains-duplicate/> | D.14.1 |
| 242 | Valid Anagram | Easy | <https://leetcode.com/problems/valid-anagram/> | D.14.2 |
| 383 | Ransom Note | Easy | <https://leetcode.com/problems/ransom-note/> | D.14.3 |
| 387 | First Unique Character in a String | Easy | <https://leetcode.com/problems/first-unique-character-in-a-string/> | D.14.4 |
| 205 | Isomorphic Strings | Easy | <https://leetcode.com/problems/isomorphic-strings/> | D.14.5 |
| 290 | Word Pattern | Easy | <https://leetcode.com/problems/word-pattern/> | D.14.6 |
| 36 | Valid Sudoku | Medium | <https://leetcode.com/problems/valid-sudoku/> | D.14.7 |

## 14.9 Chapter cheat sheet

| | |
|---|---|
| **Signals** | seen/duplicate, count/frequency, pair to a target (unsorted), anagram, group by, bijection/pattern, top k, O(n) where sorting is O(n log n) |
| **Templates** | Count: `d[x] = d.get(x, 0) + 1`. Seen: check, then `add`. Group: `defaultdict(list)[signature(x)].append(x)` |
| **Keys** | Value; signature (sorted string or 26-tuple); derived (complement, x − 1, `(box, digit)`) |
| **Complexity** | O(n) average time; O(k) space for k distinct keys (O(1) for a fixed alphabet) |
| **Top 3 bugs** | Insert-before-check in Two Sum; unhashable keys (list, Counter); `[[]] * n` buckets |
| **Optimal tricks** | Bucket by frequency (347, O(n)); walk only from run starts (128, O(n)) |
| **CPython** | Sorted key faster for short strings; count tuple faster at L = 100 (measured) |
