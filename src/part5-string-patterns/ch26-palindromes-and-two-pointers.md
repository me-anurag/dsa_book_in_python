# Chapter 26: Palindromes & Two Pointers on Strings

## 26.0 At a glance

- **What this pattern solves:** checking, repairing, finding and counting palindromes.
  Two pointers move inward to validate; centers expand outward to discover.
- **Prerequisites:** Ch 16 (two pointers), Ch 24 (character handling), Ch 5
  (`isalnum`, `lower`).
- **Typical complexity:** O(n) to validate; O(n²) worst case to find or count by
  expanding around centers; O(n) with Manacher's algorithm (advanced, optional).
- **Tools used in this chapter:**
  - slicing, `[::-1]` **[USE-FREELY]**
  - `str.isalnum`, `str.lower`, `Counter` **[KNOW-HOW]**
  - inward two pointers with skipping, the single-skip retry, expand-around-center,
    palindrome DP and Manacher **[HAND-WRITE]**

## 26.1 Recognition signals

- "**palindrome**", "reads the same forward and backward"
- "**ignoring** non-alphanumeric characters / case" → skip while scanning
- "**at most one deletion**" → retry once at the first mismatch
- "**longest** palindromic substring" or "**count** palindromic substrings" → centers
- "**build** the longest palindrome from these letters" (order free) → counting, not
  scanning

**Look-alikes:**

| Looks like | Actually | Tell |
|---|---|---|
| Palindromic substring (contiguous) | Palindromic **subsequence** (not contiguous) | "Subsequence" → DP over i, j (out of this book's scope) |
| Validate a palindrome | Build a palindrome from a multiset | Order is free → counts (409) |
| Find the longest palindrome | Shortest palindrome by prepending (214) | Prefix that's a palindrome → string matching (Ch 29) |

## 26.2 Core idea

A palindrome is **symmetric around its center**. That gives two complementary moves:

- **Validate** by walking two pointers inward from the ends, comparing as they go. Any
  mismatch is final, unless you're allowed to skip.
- **Discover** by starting at a center and expanding outward while both sides match.
  Every palindrome has a center: a character (odd length) or a gap between two
  characters (even length). So there are 2n − 1 centers.

```text
expand around center between the two b's in "cabbad":

        c a b b a d
            ↑ ↑          s[2] == s[3]  -> "bb"
          ↑     ↑        s[1] == s[4]  -> "abba"
        ↑         ↑      s[0] != s[5]  -> stop. Longest at this center: "abba"
```

Brute force checks all O(n²) substrings at O(n) each: O(n³). Expanding reuses each
comparison: a center grows only while it still matches. That's O(n²) worst case (all
equal letters) and much less on typical text (26.7).

## 26.3 The template

**Inward validation with skipping:**

```python
def is_palindrome_alnum(s: str) -> bool:
    left, right = 0, len(s) - 1
    while left < right:
        if not s[left].isalnum():
            left += 1
        elif not s[right].isalnum():
            right -= 1
        elif s[left].lower() != s[right].lower():
            return False
        else:
            left += 1
            right -= 1
    return True


is_palindrome_alnum("No 'x' in Nixon")  # -> True
```

**Expand around a center:**

```python
def expand(s: str, left: int, right: int) -> int:
    """Length of the longest palindrome centered at (left, right)."""
    while left >= 0 and right < len(s) and s[left] == s[right]:
        left -= 1
        right += 1
    return right - left - 1


s = "cabbad"
max(max(expand(s, i, i), expand(s, i, i + 1)) for i in range(len(s)))  # -> 4
```

- **Invariants:** inward, everything outside `[left, right]` has been matched in mirror
  pairs. Outward, `s[left + 1 : right]` is a palindrome.
- **Complexity:** validation O(n) time, O(1) space; expanding over all 2n − 1 centers
  O(n²) time worst case, O(1) space.

## 26.4 Dry run

680 Valid Palindrome II on `"abca"` (one deletion allowed):

| left | right | `s[left]`, `s[right]` | Action |
|---|---|---|---|
| 0 | 3 | a, a | match → move both |
| 1 | 2 | b, c | **mismatch**: try skipping left (check `s[2..2]` = "c") or right (check `s[1..1]` = "b") |

Both sub-checks are single characters, so both are palindromes: **true** (delete "c" or
"b"). The key fact: if a single deletion can fix the string, the deleted character must
be one of the two at the **first** mismatch. Everything outside them already matches in
pairs.

## 26.5 Variations

| Variant | What changes | Example |
|---|---|---|
| Ignore some characters, case-insensitive | Skip non-alphanumerics; compare `.lower()` | 125 Valid Palindrome |
| At most one deletion | At the first mismatch, check both skip options once | 680 Valid Palindrome II |
| Longest palindromic substring | Expand around 2n − 1 centers (or DP, or Manacher) | 5 Longest Palindromic Substring |
| Count palindromic substrings | Every successful expansion step is one palindrome | 647 Palindromic Substrings |
| Build from letters | Pairs + at most one odd middle letter (counting) | 409 Longest Palindrome |

## 26.6 Worked problems

### 26.6.1 125 Valid Palindrome (Easy): <https://leetcode.com/problems/valid-palindrome/>

**Problem.** Is the string a palindrome once you keep only letters and digits and ignore
case? (An all-punctuation string counts as an empty palindrome.)

**Signatures.** LeetCode: `def isPalindrome(self, s: str) -> bool:`. Book: identical.

**Constraints → target.** n ≤ 2 × 10⁵, printable ASCII. O(n). O(1) space is the interview
bonus.

**Approach 1: filter, then compare with the reverse.**

<!-- file: code/ch26_palindromes/valid_palindrome.py -->

```python
class SolutionFilterReverse125:
    def isPalindrome(self, s: str) -> bool:
        # O(n) time, O(n) auxiliary: the cleaned copy and its reverse.
        cleaned = [c.lower() for c in s if c.isalnum()]
        return cleaned == cleaned[::-1]
```

**Approach 2: two pointers that skip (O(1) space).**

<!-- file: code/ch26_palindromes/valid_palindrome.py -->

```python
class Solution125:
    def isPalindrome(self, s: str) -> bool:
        # O(n) time, O(1) auxiliary: two pointers that skip non-alphanumerics.
        left, right = 0, len(s) - 1
        while left < right:
            if not s[left].isalnum():
                left += 1
            elif not s[right].isalnum():
                right -= 1
            elif s[left].lower() != s[right].lower():
                return False
            else:
                left += 1
                right -= 1
        return True
```

**Edge cases tested:**
- `" "` (empty after filtering)
- `"0P"` (digits count, and `0` ≠ `p`)
- a single alphanumeric with punctuation
- 1,000 random strings mixing letters, digits, spaces and commas

**Common bugs:**
- `isalpha()` instead of `isalnum()`, which drops digits.
- Skipping on only one side per iteration but advancing both pointers.
- Comparing without lowercasing.

**Built-in shortcut:** approach 1 (`isalnum`, `lower` are **[KNOW-HOW]**). It's clear and
O(n); mention its O(n) space.

### 26.6.2 680 Valid Palindrome II (Easy): <https://leetcode.com/problems/valid-palindrome-ii/>

**Problem.** Can the string become a palindrome by deleting **at most one** character?

**Signatures.** LeetCode: `def validPalindrome(self, s: str) -> bool:`. Book: identical.

**Constraints → target.** n ≤ 10⁵. Trying every deletion is O(n²) = 10¹⁰: hopeless. O(n).

**Approach 1: try every deletion.**

<!-- file: code/ch26_palindromes/valid_palindrome.py -->

```python
class SolutionBruteForce680:
    def validPalindrome(self, s: str) -> bool:
        # O(n^2) time: try every single deletion (each check O(n)). O(n) auxiliary.
        if s == s[::-1]:
            return True
        for i in range(len(s)):
            candidate = s[:i] + s[i + 1:]
            if candidate == candidate[::-1]:
                return True
        return False
```

**Approach 2: one retry at the first mismatch (optimal).**

<!-- file: code/ch26_palindromes/valid_palindrome.py -->

```python
class Solution680:
    def validPalindrome(self, s: str) -> bool:
        # O(n) time, O(1) auxiliary. Walk inward; at the FIRST mismatch the deleted
        # character must be one of the two mismatched ones, so try skipping each.
        def is_pal(left: int, right: int) -> bool:
            while left < right:
                if s[left] != s[right]:
                    return False
                left += 1
                right -= 1
            return True

        left, right = 0, len(s) - 1
        while left < right:
            if s[left] != s[right]:
                return is_pal(left + 1, right) or is_pal(left, right - 1)
            left += 1
            right -= 1
        return True
```

Why only two options: before the first mismatch, every outer pair matches, so deleting
one of them can't help (it would misalign everything inside). Therefore the deletion
must be `s[left]` or `s[right]`, and each option is a single O(n) check. Traced in 26.4.

**Edge cases tested:**
- `"deeee"` (delete the first character)
- `"cbbcc"`, where only the right-skip branch works. It catches solutions that try only
  one side.
- a single character
- 1,000 random a/b strings checked against brute force

**Common bugs:**
- Trying only one skip direction.
- Allowing more than one deletion by recursing again after a skip.

**Built-in shortcut:** none.

### 26.6.3 5 Longest Palindromic Substring (Medium): <https://leetcode.com/problems/longest-palindromic-substring/>

**Problem.** Return a longest contiguous palindromic substring (any one if there's a
tie).

**Signatures.** LeetCode: `def longestPalindrome(self, s: str) -> str:`. Book: identical.

**Constraints → target.** n ≤ 1000. O(n³) is 10⁹: too slow. O(n²) = 10⁶ is fine.
Manacher's O(n) is an optional advanced extra.

**Approach 1: check every substring.**

<!-- file: code/ch26_palindromes/longest_palindromic_substring.py -->

```python
class SolutionBruteForce:
    def longestPalindrome(self, s: str) -> str:
        # O(n^3) time: O(n^2) substrings, each checked in O(n). O(n) auxiliary.
        best = s[0]
        for i in range(len(s)):
            for j in range(i + len(best) + 1, len(s) + 1):   # only longer candidates
                sub = s[i:j]
                if sub == sub[::-1]:
                    best = sub
        return best
```

**Approach 2: DP table.**

<!-- file: code/ch26_palindromes/longest_palindromic_substring.py -->

```python
class SolutionDP:
    def longestPalindrome(self, s: str) -> str:
        # O(n^2) time and O(n^2) space: pal[i][j] = s[i..j] is a palindrome
        # = s[i] == s[j] and (length <= 3 or pal[i+1][j-1]).
        n = len(s)
        pal = [[False] * n for _ in range(n)]
        start, length = 0, 1
        for i in range(n - 1, -1, -1):            # i descending: pal[i+1] is ready
            for j in range(i, n):
                if s[i] == s[j] and (j - i < 3 or pal[i + 1][j - 1]):
                    pal[i][j] = True
                    if j - i + 1 > length:
                        start, length = i, j - i + 1
        return s[start:start + length]
```

**Approach 3: expand around center (optimal for interviews).**

<!-- file: code/ch26_palindromes/longest_palindromic_substring.py -->

```python
class Solution:
    def longestPalindrome(self, s: str) -> str:
        # Expand around center. O(n^2) time worst case ("aaaa...": each of the
        # 2n - 1 centers expands O(n)). O(1) auxiliary besides the answer slice.
        def expand(left: int, right: int) -> tuple[int, int]:
            while left >= 0 and right < len(s) and s[left] == s[right]:
                left -= 1
                right += 1
            return left + 1, right - left - 1     # start, length

        start, length = 0, 1
        for center in range(len(s)):
            for left, right in ((center, center), (center, center + 1)):  # odd, even
                st, ln = expand(left, right)
                if ln > length:
                    start, length = st, ln
        return s[start:start + length]
```

**Approach 4 (advanced, optional): Manacher's algorithm, O(n).**

<!-- file: code/ch26_palindromes/longest_palindromic_substring.py -->

```python
class SolutionManacher:
    def longestPalindrome(self, s: str) -> str:
        # ADVANCED. Manacher's algorithm: O(n) time, O(n) space.
        # Insert '#' between characters so every palindrome has odd length.
        # radius[i] = how far the palindrome centered at i extends. Reuse the
        # mirror's radius inside the rightmost known palindrome [center, right].
        t = "#" + "#".join(s) + "#"
        radius = [0] * len(t)
        center = right = 0
        for i in range(len(t)):
            if i < right:
                radius[i] = min(right - i, radius[2 * center - i])   # mirror
            while (i - radius[i] - 1 >= 0 and i + radius[i] + 1 < len(t)
                   and t[i - radius[i] - 1] == t[i + radius[i] + 1]):
                radius[i] += 1                     # expand beyond what's known
            if i + radius[i] > right:
                center, right = i, i + radius[i]
        best = max(range(len(t)), key=lambda i: radius[i])
        start = (best - radius[best]) // 2         # map back to s
        return s[start:start + radius[best]]
```

Why Manacher is linear: inside the rightmost palindrome found so far, position i mirrors
position `2·center − i`, whose radius is already known, so i starts from that radius
instead of 0. Explicit expansion only happens beyond `right`, and every successful
expansion pushes `right` further. `right` only increases, up to len(t), so all the
expansions together cost O(n). The `#` separators make even- and odd-length palindromes
look the same. Interviewers rarely expect Manacher; know that it exists and why it's
linear.

**Edge cases tested:**
- `"babad"` (two valid answers), `"cbbd"` (even length)
- a single character, all equal, no palindrome longer than 1
- n = 1000 of `"a"` (the worst case for expanding)
- 1,000 random cases: lengths equal brute force's, and each result is a real palindrome
  inside `s`

**Common bugs:**
- Forgetting even-length centers (`"bb"`).
- Off-by-one when converting `(left, right)` after the loop back to a substring.
- DP filled in the wrong order (`pal[i+1][j-1]` must already be computed).

**Built-in shortcut:** none.

### 26.6.4 647 Palindromic Substrings (Medium): <https://leetcode.com/problems/palindromic-substrings/>

**Problem.** Count the palindromic substrings of `s`. Substrings at different positions
count separately, even if they're equal.

**Signatures.** LeetCode: `def countSubstrings(self, s: str) -> int:`. Book: identical.

**Constraints → target.** n ≤ 1000: O(n²).

**Approach 1: check every substring.**

<!-- file: code/ch26_palindromes/count_palindromes.py -->

```python
class SolutionBruteForce:
    def countSubstrings(self, s: str) -> int:
        # O(n^3) time: every substring, each checked in O(n).
        n = len(s)
        return sum(1 for i in range(n) for j in range(i + 1, n + 1)
                   if s[i:j] == s[i:j][::-1])
```

**Approach 2: count during expansion (optimal for interviews).**

<!-- file: code/ch26_palindromes/count_palindromes.py -->

```python
class Solution:
    def countSubstrings(self, s: str) -> int:
        # Expand around each of the 2n - 1 centers; every successful expansion step
        # is exactly one palindrome. O(n^2) time worst case, O(1) auxiliary.
        count = 0
        for center in range(len(s)):
            for left, right in ((center, center), (center, center + 1)):
                while left >= 0 and right < len(s) and s[left] == s[right]:
                    count += 1
                    left -= 1
                    right += 1
        return count
```

**Approach 3 (advanced): Manacher, O(n).** Same radii as 26.6.3; a center with radius r
in the `#`-separated string accounts for ⌈r/2⌉ palindromes:

<!-- file: code/ch26_palindromes/count_palindromes.py -->

```python
class SolutionManacher:
    def countSubstrings(self, s: str) -> int:
        # ADVANCED: O(n). With '#' separators, a center with radius r in t covers
        # (r + 1) // 2 palindromes of s.
        t = "#" + "#".join(s) + "#"
        radius = [0] * len(t)
        center = right = 0
        for i in range(len(t)):
            if i < right:
                radius[i] = min(right - i, radius[2 * center - i])
            while (i - radius[i] - 1 >= 0 and i + radius[i] + 1 < len(t)
                   and t[i - radius[i] - 1] == t[i + radius[i] + 1]):
                radius[i] += 1
            if i + radius[i] > right:
                center, right = i, i + radius[i]
        return sum((r + 1) // 2 for r in radius)
```

Why counting during expansion is exact: each palindrome has exactly one center and one
radius, and the expansion from that center passes through it exactly once.

**Edge cases tested:**
- `"abc"` (3), `"aaa"` (6), `"abba"` (6)
- a single character
- 1,000 random a/b strings across all three
- n = 1000 of `"a"` (n(n + 1)/2)

**Common bugs:**
- Counting distinct strings instead of positions.
- Missing even centers.

**Built-in shortcut:** none.

## 26.7 Python-specific notes

**Worst case vs typical case**, measured (`bench/ch26_palindromes.py`, problem 5, n =
1000, best of 3, two runs):

| Input | DP O(n²) | Expand, O(n²) worst | Manacher O(n) |
|---|---|---|---|
| `"a" * 1000` | 68–73 ms | 46–51 ms | 0.9–1.2 ms |
| Random a–z | 25–30 ms | 0.5–0.7 ms | 0.8–1.0 ms |

Expansion is quadratic only when long palindromes exist. On random text each center
stops after one or two comparisons, and it measured slightly faster than Manacher.
Manacher wins by 40× or more on the worst case. The DP table measured slowest in both:
it fills an n × n table no matter what the input is.

**Slicing and `[::-1]`** make the simple check `s == s[::-1]` fast (both in C), but each
call costs O(n) time and O(n) memory. Fine once; avoid it inside loops over substrings
(that's approach 1's O(n³)).

## 26.8 Practice set

Hints are in Appendix D, under "Chapter 26 practice". Reference solutions:
`code/ch26_palindromes/practice_solutions.py`.

| # | Title | Difficulty | Link | Hint ref |
|---|---|---|---|---|
| 409 | Longest Palindrome | Easy | <https://leetcode.com/problems/longest-palindrome/> | D.26.1 |

## 26.9 Chapter cheat sheet

| | |
|---|---|
| **Signals** | Palindrome; ignore non-alnum / case; at most one deletion; longest/count palindromic substrings; build from letters |
| **Validate** | Two pointers inward; skip what doesn't count |
| **One deletion** | At the first mismatch, check `s[l+1..r]` or `s[l..r-1]` |
| **Discover** | Expand around 2n − 1 centers (odd and even) |
| **Advanced** | Manacher: O(n) via mirrored radii |
| **Complexity** | Validate O(n); expand O(n²) worst, near O(n) typical (measured); DP O(n²) time and space |
| **Top 3 bugs** | Missing even centers; trying only one skip direction; `isalpha` instead of `isalnum` |
