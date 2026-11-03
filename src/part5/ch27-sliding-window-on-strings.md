# Chapter 27: Sliding Window on Strings

## 27.0 At a glance

- **What this pattern solves:** the longest or shortest **substring** satisfying a
  constraint on its characters (no repeats, at most k changes, contains all of t), and
  fixed-length scans. It's Ch 17's window with character counts as the summary.
- **Prerequisites:** Ch 17 (variable windows, shrink-while-invalid), Ch 25 (count
  signatures), Ch 8 (`Counter`).
- **Typical complexity:** O(n) time (each index enters and leaves once), O(k) space for
  the counts.
- **Tools used in this chapter:**
  - `dict`, `set` **[USE-FREELY]**
  - `Counter` **[KNOW-HOW]**
  - last-index jumping, the max-frequency window, the need/have (`formed`) counter and
    word-level windows **[HAND-WRITE]**

## 27.1 Recognition signals

- "**longest substring** without repeating / with at most k distinct / after k
  replacements"
- "**minimum window** containing all characters of t"
- "substring of **fixed length** k" (count vowels, compare signatures)
- "concatenation of **all words** of equal length" → a window measured in **words**

**Look-alikes:**

| Looks like | Actually | Tell |
|---|---|---|
| Window over characters | Prefix sums (Ch 15) | Counting substrings whose **sum** equals k with negative contributions |
| Fixed window of size len(p) | Signature hashing (Ch 25) | Anagram windows: same technique, framed as signatures |
| Shortest window containing t | Subsequence matching (Ch 24) | "In order, not necessarily contiguous" → subsequence |

## 27.2 Core idea

The window `[left, right]` is valid or invalid according to its **character counts**.
Grow `right` one character at a time; while the window is invalid (or, for "shortest"
problems, while it's valid), move `left`. Each character enters once and leaves once, so
the work is O(n).

The hard part is deciding **validity in O(1)** instead of re-scanning the counts:

| Constraint | O(1) summary |
|---|---|
| No repeated characters | Set membership, or the last index of each character |
| At most k distinct | `len(counts)` (delete keys that drop to 0) |
| Fixable with ≤ k replacements | length − (max count) ≤ k |
| Contains every char of t with multiplicity | `formed` = number of distinct letters whose count is satisfied |

```text
need/have on s = "ADOBECODEBANC", t = "ABC"  (formed counts letters satisfied, out of 3)

[ADOBEC]ODEBANC     formed 3 -> record 6, shrink: drop A -> formed 2
 A[DOBECODEBA]NC    grows until A returns -> formed 3 -> record? 10 > 6, shrink...
 ...
 ADOBECODE[BANC]    formed 3 -> record 4  <- answer "BANC"
```

## 27.3 The template

**Shortest window that satisfies a counted requirement** (need/have):

```python
from collections import Counter


def shortest_covering(s: str, t: str) -> str:
    need = Counter(t)
    window: dict[str, int] = {}
    formed, left = 0, 0
    best = (float("inf"), 0)
    for right, c in enumerate(s):
        window[c] = window.get(c, 0) + 1
        if c in need and window[c] == need[c]:
            formed += 1
        while formed == len(need):
            best = min(best, (right - left + 1, left))
            d = s[left]
            window[d] -= 1
            if d in need and window[d] < need[d]:
                formed -= 1
            left += 1
    length, start = best
    return "" if length == float("inf") else s[start:start + length]


shortest_covering("ADOBECODEBANC", "ABC")  # -> 'BANC'
```

- **Invariant:** `formed` equals the number of distinct letters x in t with
  `window[x] >= need[x]`. It changes only when a count **crosses** the needed value.
- **Complexity:** O(m + n) time (each index of s enters and leaves once; building
  `need` is O(n)); O(k) space for k distinct letters.

**Longest window under a constraint:** the Ch 17 template. Add `s[right]`, shrink while
invalid, record `right - left + 1`.

## 27.4 Dry run

3 Longest Substring Without Repeating Characters on `"abba"`, last-index version
(27.6.1). The interesting step is the **stale** index.

| right | c | `last[c]` before | `last[c] >= left`? | left after | best |
|---|---|---|---|---|---|
| 0 | a | — | — | 0 | 1 |
| 1 | b | — | — | 0 | 2 |
| 2 | b | 1 | yes (1 ≥ 0) | 2 | 2 |
| 3 | a | 0 | **no** (0 < 2): stale, ignore | 2 | 2 |

Without the `>= left` check, step 4 would move `left` **back** to 1, making the window
`"bba"`, which contains a repeat.

## 27.5 Variations

| Variant | Window summary | Example |
|---|---|---|
| No repeats | Last index of each character; jump `left` | 3 Longest Substring Without Repeating Characters |
| ≤ k replacements | Counts + the max count (the non-shrinking trick) | 424 Longest Repeating Character Replacement |
| Fixed length k | Running vowel count | 1456 Maximum Number of Vowels in a Substring of Given Length |
| Cover t with multiplicity | need/have with `formed` | 76 Minimum Window Substring |
| Concatenation of equal-length words | A window of **words**, run once per offset 0..w−1 | 30 Substring with Concatenation of All Words |

## 27.6 Worked problems

### 27.6.1 3 Longest Substring Without Repeating Characters (Medium): <https://leetcode.com/problems/longest-substring-without-repeating-characters/>

**Problem.** Return the length of the longest substring with all distinct characters.
The characters are letters, digits, symbols and spaces.

**Signatures.** LeetCode: `def lengthOfLongestSubstring(self, s: str) -> int:`. Book:
identical.

**Constraints → target.** n ≤ 10⁵ (possibly 0). O(n).

**Approach 1: extend from every start.**

<!-- file: code/ch27_string_windows/longest_unique.py -->

```python
class SolutionBruteForce:
    def lengthOfLongestSubstring(self, s: str) -> int:
        # O(n * k) time with k = alphabet size bound on the inner run (the inner loop
        # stops at the first repeat, which happens within k + 1 characters).
        # O(k) auxiliary for the set.
        best = 0
        for i in range(len(s)):
            seen: set[str] = set()
            for j in range(i, len(s)):
                if s[j] in seen:
                    break
                seen.add(s[j])
            best = max(best, len(seen))
        return best
```

**Approach 2: window with a set.**

<!-- file: code/ch27_string_windows/longest_unique.py -->

```python
class SolutionWindowSet:
    def lengthOfLongestSubstring(self, s: str) -> int:
        # O(n) time: each character enters and leaves the set once. O(k) space.
        seen: set[str] = set()
        best = left = 0
        for right, c in enumerate(s):
            while c in seen:                    # shrink until c is unique again
                seen.remove(s[left])
                left += 1
            seen.add(c)
            best = max(best, right - left + 1)
        return best
```

**Approach 3: last-index jump (optimal).**

<!-- file: code/ch27_string_windows/longest_unique.py -->

```python
class Solution:
    def lengthOfLongestSubstring(self, s: str) -> int:
        # O(n) time, O(k) space: remember each character's last index and JUMP
        # left past the previous occurrence instead of stepping one by one.
        last: dict[str, int] = {}
        best = left = 0
        for right, c in enumerate(s):
            if c in last and last[c] >= left:   # repeat inside the current window
                left = last[c] + 1
            last[c] = right
            best = max(best, right - left + 1)
        return best
```

Approaches 2 and 3 are both O(n). Approach 3 moves `left` in one jump instead of one
step at a time, but needs the stale-index check (27.4).

**Edge cases tested:**
- an empty string, a single space
- `"abba"` (stale index)
- `"pwwkew"` (substring, not subsequence)
- 1,000 random strings with letters, a digit and a space

**Common bugs:**
- Missing `last[c] >= left`, which moves `left` backwards.
- Treating the answer as a subsequence.
- Assuming lowercase only. Spaces and symbols occur, so use a dict, not a 26-slot list.

**Built-in shortcut:** none (**[HAND-WRITE]**).

### 27.6.2 424 Longest Repeating Character Replacement (Medium): <https://leetcode.com/problems/longest-repeating-character-replacement/>

**Problem.** Uppercase string s and an integer k. You may change at most k characters.
Return the length of the longest substring of one repeated letter you can obtain.

**Signatures.** LeetCode: `def characterReplacement(self, s: str, k: int) -> int:`. Book:
identical.

**Constraints → target.** n ≤ 10⁵, so O(n) or O(26 · n).

**Key fact.** A window can be made uniform with ≤ k changes **iff** length − (count of its
most frequent letter) ≤ k: change everything except the most common letter.

**Approach 1: every start, extend while fixable.**

<!-- file: code/ch27_string_windows/char_replacement.py -->

```python
class SolutionBruteForce:
    def characterReplacement(self, s: str, k: int) -> int:
        # O(n^2 * 26) time: every start, extend while fixable. O(1) space.
        best = 0
        for i in range(len(s)):
            counts = [0] * 26
            for j in range(i, len(s)):
                counts[ord(s[j]) - 65] += 1
                if (j - i + 1) - max(counts) <= k:
                    best = max(best, j - i + 1)
        return best
```

**Approach 2: window with the exact max (O(26 · n)).**

<!-- file: code/ch27_string_windows/char_replacement.py -->

```python
class SolutionExactMax:
    def characterReplacement(self, s: str, k: int) -> int:
        # O(26 * n) time: shrink while the window is unfixable, recomputing the
        # exact max count (26 slots) each check. O(1) space.
        counts = [0] * 26
        best = left = 0
        for right, c in enumerate(s):
            counts[ord(c) - 65] += 1
            while (right - left + 1) - max(counts) > k:
                counts[ord(s[left]) - 65] -= 1
                left += 1
            best = max(best, right - left + 1)
        return best
```

**Approach 3: the non-shrinking window with a historical max (O(n)).**

<!-- file: code/ch27_string_windows/char_replacement.py -->

```python
class Solution:
    def characterReplacement(self, s: str, k: int) -> int:
        # O(n) time, O(1) space. `max_freq` is the largest count EVER seen in a
        # window; it is never decreased. The window only grows when a new record
        # max_freq allows it, and otherwise slides (shrinks by one as it grows by
        # one), so its length never drops below the best answer found so far.
        counts = [0] * 26
        max_freq = left = 0
        for right, c in enumerate(s):
            counts[ord(c) - 65] += 1
            max_freq = max(max_freq, counts[ord(c) - 65])
            if (right - left + 1) - max_freq > k:      # `if`, not `while`
                counts[ord(s[left]) - 65] -= 1
                left += 1
        return len(s) - left
```

Why the stale `max_freq` is safe: we only care about windows **longer** than the best so
far. A longer valid window needs a letter count larger than any seen before, which
raises `max_freq`. Until then, the window just slides at its current size. It's the
"non-shrinking window" idea Ch 17 verified for 1004. Here it's checked against brute
force on 1,000 random cases.

**Edge cases tested:**
- k = 0, k ≥ n
- a single letter, all equal
- 1,000 random strings over A–C, across all three approaches

**Common bugs:**
- Decreasing `max_freq` when shrinking (not needed, and it costs an O(26) recount).
- Using `while` with the stale max: it can over-shrink.
- Returning `best` but never updating it in the `if` version (this version returns
  `len(s) - left`).

**Built-in shortcut:** none. Approach 2 is the honest middle ground in interviews.

### 27.6.3 76 Minimum Window Substring (Hard): <https://leetcode.com/problems/minimum-window-substring/>

**Problem.** Return the shortest substring of s that contains every character of t,
**counting duplicates**, or "" if none exists. The answer is unique. The follow-up asks
for O(m + n).

**Signatures.** LeetCode: `def minWindow(self, s: str, t: str) -> str:`. Book: identical.

**Constraints → target.** m, n ≤ 10⁵; upper- and lowercase letters. O(m + n).

**Approach 1: every start, extend until covered.**

<!-- file: code/ch27_string_windows/min_window.py -->

```python
class SolutionBruteForce:
    def minWindow(self, s: str, t: str) -> str:
        # O(m^2 * k) time: every start, extend until it covers t (Counter
        # comparison over k distinct letters). Test-only.
        need = Counter(t)
        best = ""
        for i in range(len(s)):
            have: Counter = Counter()
            for j in range(i, len(s)):
                have[s[j]] += 1
                if have >= need:                         # multiset containment (3.10+)
                    if not best or j - i + 1 < len(best):
                        best = s[i:j + 1]
                    break
        return best
```

**Approach 2: need/have with a `formed` counter (optimal).**

<!-- file: code/ch27_string_windows/min_window.py -->

```python
class Solution:
    def minWindow(self, s: str, t: str) -> str:
        # The need/have technique. O(m + n) time: each index enters and leaves the
        # window once, and `formed` is updated in O(1). O(k) space.
        need = Counter(t)
        required = len(need)                     # distinct letters to satisfy
        window: dict[str, int] = {}
        formed = 0                               # letters currently satisfied
        best_len, best_start = float("inf"), 0
        left = 0
        for right, c in enumerate(s):
            window[c] = window.get(c, 0) + 1
            if c in need and window[c] == need[c]:
                formed += 1                      # c just became satisfied
            while formed == required:            # valid: record, then shrink
                if right - left + 1 < best_len:
                    best_len, best_start = right - left + 1, left
                d = s[left]
                window[d] -= 1
                if d in need and window[d] < need[d]:
                    formed -= 1                  # d just became unsatisfied
                left += 1
        return "" if best_len == float("inf") else s[best_start:best_start + best_len]
```

Why `==` and `<` are the right crossings: `formed` should count letters at or above their
requirement. A letter becomes satisfied exactly when its count rises **to** `need[x]`
(`==`), and unsatisfied exactly when it falls **below** (`<`). Extra copies don't change
`formed`, which keeps every update O(1) instead of comparing whole count tables.

**Edge cases tested:**
- `t = "aa"` with a single `a` in s
- a missing letter
- mixed case (`A` ≠ `a`)
- 1,000 random cases: lengths match brute force, and each result really covers t (ties
  are possible in random tests even though LeetCode's tests guarantee uniqueness)

**Common bugs:**
- Counting distinct letters only, which ignores multiplicity.
- Updating `formed` on every increment instead of only at the crossing.
- Recording outside the shrink loop.
- `Counter(window) >= need` per step, which is correct but O(k) per check.

**Built-in shortcut:** `Counter >= Counter` (3.10+, Ch 8) **[KNOW-HOW]** for the
brute-force check.

### 27.6.4 30 Substring with Concatenation of All Words (Hard, stretch): <https://leetcode.com/problems/substring-with-concatenation-of-all-words/>

**Problem.** Given s and k words of **equal length** w, return every start index in s of
a substring formed by concatenating all the words exactly once, in any order. The result
may be in any order.

**Signatures.** LeetCode:
`def findSubstring(self, s: str, words: List[str]) -> List[int]:`. Book: `list[str]`,
`list[int]`.

**Constraints → target.** n ≤ 10⁴, k ≤ 5000, w ≤ 30. Checking every start costs
O(n · k · w), up to 1.5 × 10⁹. The word-level window costs O(n · w).

**Approach 1: check every start.**

<!-- file: code/ch27_string_windows/concatenation.py -->

```python
class SolutionCheckEveryStart:
    def findSubstring(self, s: str, words: list[str]) -> list[int]:
        # O(n * k * w) time: at each of n starts, cut k words of length w and
        # compare counts. O(k) space.
        w, k = len(words[0]), len(words)
        need = Counter(words)
        total = w * k
        out = []
        for i in range(len(s) - total + 1):
            seen = Counter(s[i + j * w:i + (j + 1) * w] for j in range(k))
            if seen == need:
                out.append(i)
        return out
```

**Approach 2: a window over words, once per offset (optimal).**

<!-- file: code/ch27_string_windows/concatenation.py -->

```python
class Solution:
    def findSubstring(self, s: str, words: list[str]) -> list[int]:
        # Word-level sliding window, run once per offset 0..w-1. Each offset walks
        # the string in steps of w, adding/removing whole words: O(n / w) steps,
        # each O(w) to slice and hash a word -> O(n) per offset, O(n * w) total.
        # O(k) space.
        w, k = len(words[0]), len(words)
        need = Counter(words)
        out = []
        for offset in range(w):
            window: Counter = Counter()
            left = offset
            used = 0                             # words currently in the window
            for right in range(offset, len(s) - w + 1, w):
                word = s[right:right + w]
                if word not in need:             # breaks every window: restart after it
                    window.clear()
                    used = 0
                    left = right + w
                    continue
                window[word] += 1
                used += 1
                while window[word] > need[word]:  # too many copies: shrink by words
                    window[s[left:left + w]] -= 1
                    used -= 1
                    left += w
                if used == k:
                    out.append(left)
                    window[s[left:left + w]] -= 1  # slide one word to keep searching
                    used -= 1
                    left += w
        return sorted(out)
```

Why offsets: any valid concatenation starts at some index whose remainder mod w is
between 0 and w − 1. Within one offset, the string splits into consecutive w-length
words, and the problem becomes Ch 17's "window with counts ≤ need", with words in place
of characters.

**Edge cases tested:**
- overlapping answers (`"aaaaaa"`, `["aa", "aa"]`)
- duplicate words in the list
- a single one-letter word
- 1,000 random cases checked against approach 1

**Common bugs:**
- Running only offset 0, which misses answers that start at other remainders.
- Not resetting after a word that's not in the list.
- Forgetting duplicate words (compare counts, not sets).

**Built-in shortcut:** `Counter` **[KNOW-HOW]**.

## 27.7 Python-specific notes

- **Use a dict, not a 26-slot list, when the alphabet isn't fixed.** Problem 3 allows
  spaces, digits and symbols, and 76 mixes cases. Check the constraints (Ch 24).
- **`Counter` comparisons** (`>=`, `==`) are clean for brute-force checks, but each is
  O(k). Inside the window loop, prefer the O(1) crossing counter (`formed`).
- **Slicing words** in 30 costs O(w) per slice. That's part of the O(n · w) bound, and
  slicing runs in C.
- **`float("inf")` as "no answer yet"** is fine for lengths. Return `""` when it was
  never replaced.

## 27.8 Practice set

Hints are in Appendix D, under "Chapter 27 practice". Reference solutions:
`code/ch27_string_windows/practice_solutions.py`.

| # | Title | Difficulty | Link | Hint ref |
|---|---|---|---|---|
| 1456 | Maximum Number of Vowels in a Substring of Given Length | Medium | <https://leetcode.com/problems/maximum-number-of-vowels-in-a-substring-of-given-length/> | D.27.1 |

## 27.9 Chapter cheat sheet

| | |
|---|---|
| **Signals** | Longest/shortest substring with a character constraint; minimum covering window; fixed length; equal-length word concatenation |
| **Summaries** | Last index (no repeats); counts + max (replacements); `formed` crossings (cover t) |
| **need/have** | `formed += 1` when `window[x] == need[x]`; `formed -= 1` when it drops `< need[x]`; record inside the shrink loop |
| **Complexity** | O(n) time (O(m + n) for 76; O(n · w) for 30); O(k) space |
| **Top 3 bugs** | Stale last-index moving `left` backwards; ignoring multiplicity in t; one offset only in 30 |
