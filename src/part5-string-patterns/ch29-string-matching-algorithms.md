# Chapter 29: String Matching Algorithms

## 29.0 At a glance

- **What this pattern solves:** finding a pattern inside a text, and the family of
  problems built on the same machinery: **borders** (prefixes that are also suffixes),
  periods, and palindromic prefixes. There are four methods: brute force, Rabin–Karp,
  KMP and the Z-function.
- **Prerequisites:** Ch 25 (rolling hashes), Ch 26 (palindromes), Ch 12 (what `in` costs).
- **Typical complexity:** brute force O(n · m) worst case; Rabin–Karp O(n + m) expected;
  KMP and Z O(n + m) worst case, with O(m) or O(n + m) space.
- **Tools used in this chapter:**
  - `str.find`, `in` **[KNOW-HOW]**
  - the prefix function (KMP), the Z-function and Rabin–Karp **[HAND-WRITE]**

## 29.1 Recognition signals

- "find the **first occurrence** / all occurrences of a pattern" **without** built-ins
- "longest **prefix that is also a suffix**" → prefix function (1392)
- "is the string a **repetition** of a substring?" → period from the prefix function (459)
- "shortest palindrome by **adding characters in front**" → border of
  `s + '#' + reversed(s)` (214)
- "how many copies of a so that b is a substring" → bounded repetition + matching (686)

**When is `str.find` / `in` acceptable?** On LeetCode, always: it's correct and fast (29.7).
In an interview, when matching isn't the point of the question. When the question **is**
matching (28 asked "without built-ins", or 1392/214, which need borders rather than
positions), write KMP or Z.

**What CPython uses.** The What's New in Python 3.10 notes say substring search
functions (`in`, `find`) "now sometimes use Crochemore & Perrin's 'Two-Way' string
searching algorithm to avoid quadratic behavior on long strings". The current CPython
time-complexity docs list forward search as linear in the worst case. Ch 12 measured it
flat on an adversarial input.

## 29.2 Core idea

**Brute force wastes information.** After matching k characters and failing, it shifts
the pattern by one and re-reads text it has already seen.

**KMP keeps it.** If `needle[:k]` matched and the next character fails, the only useful
next alignments are those where a **prefix of the needle lines up with a suffix of what
was just matched**: a *border* of `needle[:k]`. The prefix function precomputes, for
every k, the longest such border, so KMP never moves backward in the text.

```text
needle = "aabaaab"
pi     =  0 1 0 1 2 2 3      pi[i] = longest proper border of needle[:i+1]

needle[:6] = "aabaaa" has border "aa" (length 2): after a mismatch at position 6,
continue comparing at needle[2] instead of starting over.
```

**Z-function** computes, for every position i, how long `s[i:]` agrees with `s`'s prefix.
Run it on `needle + sep + text`: a value ≥ m means a match.

**Rabin–Karp** compares rolling hashes (Ch 25) and verifies on equality.

## 29.3 The template

**Prefix function:**

<!-- file: code/ch29_string_matching/str_str.py -->

```python
def prefix_function(p: str) -> list[int]:
    """pi[i] = length of the longest proper prefix of p[:i+1] that is also its
    suffix. O(m) time: k rises by at most 1 per step and every fallback lowers it."""
    pi = [0] * len(p)
    k = 0
    for i in range(1, len(p)):
        while k > 0 and p[i] != p[k]:
            k = pi[k - 1]                     # fall back to the next shorter border
        if p[i] == p[k]:
            k += 1
        pi[i] = k
    return pi
```

- **Invariant:** at the start of iteration i, k = `pi[i - 1]`, the length of the longest
  border of `p[:i]`.
- **Complexity:** O(m) time: k increases at most m times in total, and each `while`
  iteration decreases it, so the fallbacks total at most m. O(m) space.

**Z-function:**

<!-- file: code/ch29_string_matching/str_str.py -->

```python
def z_function(s: str) -> list[int]:
    """z[i] = length of the longest common prefix of s and s[i:] (z[0] = len(s)).
    O(n): the window [left, right) is the rightmost match found so far; positions
    inside it start from a known value, and right only moves forward."""
    n = len(s)
    z = [0] * n
    if n:
        z[0] = n
    left = right = 0
    for i in range(1, n):
        if i < right:
            z[i] = min(right - i, z[i - left])
        while i + z[i] < n and s[z[i]] == s[i + z[i]]:
            z[i] += 1
        if i + z[i] > right:
            left, right = i, i + z[i]
    return z
```

The test file asserts `prefix_function("aabaaab") == [0, 1, 0, 1, 2, 2, 3]` and
`z_function("aabxaab") == [7, 1, 0, 0, 3, 1, 0]`. Work both out by hand once: it's the
fastest way to internalize the fallback rule.

## 29.4 Dry run

KMP searching `needle = "aab"` (pi = [0, 1, 0]) in `haystack = "aaaab"`:

| i | c | k before | mismatch fallbacks | k after | match? |
|---|---|---|---|---|---|
| 0 | a | 0 | | 1 | |
| 1 | a | 1 | | 2 | |
| 2 | a | 2 | needle[2] = b ≠ a → k = pi[1] = 1; needle[1] = a = a | 2 | |
| 3 | a | 2 | same fallback | 2 | |
| 4 | b | 2 | | 3 | **k = 3 = m → start 4 − 3 + 1 = 2** |

The text pointer i never moves backward. Brute force would restart at index 1, then 2,
re-reading characters.

## 29.5 Variations

| Variant | What changes | Example |
|---|---|---|
| First occurrence | KMP / Z / Rabin–Karp / brute force | 28 Find the Index of the First Occurrence in a String |
| Longest border | Answer = `pi[-1]` | 1392 Longest Happy Prefix |
| Longest palindromic prefix | Border of `s + '#' + s[::-1]` | 214 Shortest Palindrome |
| Is s a repetition? | Period `p = n − pi[-1]`; p divides n and pi[-1] > 0; or `s in (s + s)[1:-1]` | 459 Repeated Substring Pattern |
| Bounded repetition + search | Try ⌈len(b)/len(a)⌉ and one more copy | 686 Repeated String Match |

## 29.6 Worked problems

### 29.6.1 28 Find the Index of the First Occurrence in a String (Easy): <https://leetcode.com/problems/find-the-index-of-the-first-occurrence-in-a-string/>

**Problem.** Return the index of the first occurrence of `needle` in `haystack`, or −1.

**Signatures.** LeetCode: `def strStr(self, haystack: str, needle: str) -> int:`. Book:
identical.

**Constraints → target.** n, m ≤ 10⁴, lowercase. Brute force is O(n · m) = 10⁸ in the worst
case. The linear methods are O(n + m).

**Method 1: brute force.**

<!-- file: code/ch29_string_matching/str_str.py -->

```python
class SolutionBruteForce:
    def strStr(self, haystack: str, needle: str) -> int:
        # O((n - m + 1) * m) time worst case, O(1) auxiliary.
        n, m = len(haystack), len(needle)
        for start in range(n - m + 1):
            k = 0
            while k < m and haystack[start + k] == needle[k]:
                k += 1
            if k == m:
                return start
        return -1
```

**Method 2: Rabin–Karp.**

<!-- file: code/ch29_string_matching/str_str.py -->

```python
class SolutionRabinKarp:
    def strStr(self, haystack: str, needle: str) -> int:
        # Expected O(n + m) time: rolling hash, verify on hash equality (worst case
        # O(n * m) if many collisions). O(1) auxiliary.
        n, m = len(haystack), len(needle)
        if m > n:
            return -1
        base, mod = 256, (1 << 61) - 1
        top = pow(base, m - 1, mod)
        target = window = 0
        for i in range(m):
            target = (target * base + ord(needle[i])) % mod
            window = (window * base + ord(haystack[i])) % mod
        for start in range(n - m + 1):
            if window == target and haystack[start:start + m] == needle:
                return start
            if start + m < n:
                window = ((window - ord(haystack[start]) * top) * base
                          + ord(haystack[start + m])) % mod
        return -1
```

**Method 3: KMP (worst-case linear, the interview standard).**

<!-- file: code/ch29_string_matching/str_str.py -->

```python
class Solution:
    def strStr(self, haystack: str, needle: str) -> int:
        # KMP. O(n + m) time worst case, O(m) auxiliary (the prefix function).
        # On a mismatch, the needle shifts so that its longest border lines up:
        # haystack characters are never re-read.
        pi = prefix_function(needle)
        k = 0                                  # characters of needle matched so far
        for i, c in enumerate(haystack):
            while k > 0 and c != needle[k]:
                k = pi[k - 1]
            if c == needle[k]:
                k += 1
            if k == len(needle):
                return i - k + 1
        return -1
```

**Method 4: Z-function.**

<!-- file: code/ch29_string_matching/str_str.py -->

```python
class SolutionZ:
    def strStr(self, haystack: str, needle: str) -> int:
        # Z-function on needle + separator + haystack. O(n + m) time and space.
        # The separator (not a lowercase letter) stops matches from crossing over.
        combined = needle + "\x00" + haystack
        z = z_function(combined)
        m = len(needle)
        for i in range(m + 1, len(combined)):
            if z[i] >= m:
                return i - m - 1
        return -1
```

| Method | Time | Space | Notes |
|---|---|---|---|
| Brute force | O(n · m) worst | O(1) | Fine on random text, terrible on adversarial input (29.7) |
| Rabin–Karp | O(n + m) expected | O(1) | Verify hits; extends to many patterns of the same length |
| KMP | O(n + m) worst | O(m) | Text pointer never moves back; the prefix function has other uses (29.6.2) |
| Z | O(n + m) worst | O(n + m) | Simple to reason about; needs a separator |

**Edge cases tested:**
- a needle longer than the haystack
- a single character
- `"aaaaab"` / `"aab"` (fallbacks), `"mississippi"` / `"issip"`
- 3,000 random a/b strings checked against `str.find`
- an adversarial needle of 5,001 characters

**Common bugs:**
- KMP's fallback written as `k = pi[k]` instead of `pi[k - 1]`.
- A Z-function separator that can occur in the input.
- Rabin–Karp without verification.

**Built-in shortcut:** `haystack.find(needle)` **[KNOW-HOW]**, which measured far faster
than any Python-level matcher (29.7).

### 29.6.2 1392 Longest Happy Prefix (Hard): <https://leetcode.com/problems/longest-happy-prefix/>

**Problem.** Return the longest non-empty proper prefix of s that is also a suffix (they
may overlap), or "".

**Signatures.** LeetCode: `def longestPrefix(self, s: str) -> str:`. Book: identical.

**Constraints → target.** n ≤ 10⁵. O(n²) is too slow; the answer is a prefix-function value.

**Approach 1: try lengths from longest down.**

<!-- file: code/ch29_string_matching/borders.py -->

```python
class SolutionBruteForce1392:
    def longestPrefix(self, s: str) -> str:
        # O(n^2) time: try every length from longest to shortest, compare O(n).
        for length in range(len(s) - 1, 0, -1):
            if s[:length] == s[-length:]:
                return s[:length]
        return ""
```

**Approach 2: prefix function (optimal).**

<!-- file: code/ch29_string_matching/borders.py -->

```python
class Solution1392:
    def longestPrefix(self, s: str) -> str:
        # O(n): the answer is exactly pi[-1], the longest proper border of s.
        return s[:prefix_function(s)[-1]]
```

The prefix function's last value is, by definition, the length of the longest proper
border of the whole string.

**Edge cases tested:**
- `"level"`, `"ababab"` (overlapping), `"aaaa"`
- a single character
- 2,000 random strings checked against brute force

**Common bugs:**
- Allowing the whole string (it must be **proper**).
- Hashing prefixes and suffixes without verification.

**Built-in shortcut:** none in the stdlib.

### 29.6.3 214 Shortest Palindrome (Hard): <https://leetcode.com/problems/shortest-palindrome/>

**Problem.** Add characters **in front of** s to make the shortest possible palindrome.

**Signatures.** LeetCode: `def shortestPalindrome(self, s: str) -> str:`. Book: identical.

**Constraints → target.** n ≤ 5 × 10⁴ (possibly 0). O(n²) is too slow in Python. Target O(n).

**Key fact.** Keep the longest **palindromic prefix** of s, and prepend the reverse of
the rest.

**Approach 1: test prefixes from longest down.**

<!-- file: code/ch29_string_matching/borders.py -->

```python
class SolutionBruteForce214:
    def shortestPalindrome(self, s: str) -> str:
        # O(n^2): find the longest palindromic PREFIX by checking from the longest
        # down; then prepend the reverse of what's left.
        for end in range(len(s), -1, -1):
            if s[:end] == s[:end][::-1]:
                return s[end:][::-1] + s
        return s
```

**Approach 2: prefix function on `s + '#' + reversed(s)` (optimal).**

<!-- file: code/ch29_string_matching/borders.py -->

```python
class Solution214:
    def shortestPalindrome(self, s: str) -> str:
        # O(n): the longest palindromic prefix of s is the longest border of
        # s + '#' + reverse(s) (a prefix of s that equals a suffix of reverse(s)
        # is a palindrome). The separator stops the border from crossing the middle.
        combined = s + "#" + s[::-1]
        keep = prefix_function(combined)[-1]
        return s[keep:][::-1] + s
```

Why: a suffix of `reverse(s)` of length L is the reverse of `s[:L]`. So "prefix of s equals
a suffix of reverse(s)" means `s[:L] == reverse(s[:L])`, which says exactly that `s[:L]` is
a palindrome. The `#` keeps the border inside the original length.

**Edge cases tested:**
- `"aacecaaa"`, `"abcd"`
- the empty string, an already-palindromic string
- 2,000 random strings checked against brute force

**Common bugs:**
- Omitting the separator (the border can then exceed n, as with `"aaa"`).
- Looking for the longest palindromic **substring** instead of **prefix**.

**Built-in shortcut:** none.

### 29.6.4 459 Repeated Substring Pattern (Easy): <https://leetcode.com/problems/repeated-substring-pattern/>

**Problem.** Can s be written as some substring repeated two or more times?

**Signatures.** LeetCode: `def repeatedSubstringPattern(self, s: str) -> bool:`. Book:
identical.

**Constraints → target.** n ≤ 10⁴. Checking every divisor of n is fine, but two O(n) ideas
are worth knowing.

**Approach 1: the doubled-string trick.**

<!-- file: code/ch29_string_matching/borders.py -->

```python
class SolutionConcat459:
    def repeatedSubstringPattern(self, s: str) -> bool:
        # The (s + s)[1:-1] trick: s is a repetition iff s occurs inside s + s with
        # the first and last characters removed. O(n) with a linear search.
        return s in (s + s)[1:-1]
```

**Approach 2: period from the prefix function.**

<!-- file: code/ch29_string_matching/borders.py -->

```python
class Solution459:
    def repeatedSubstringPattern(self, s: str) -> bool:
        # O(n): with p = n - pi[-1] (the smallest period), s is a repetition of its
        # first p characters iff pi[-1] > 0 and p divides n.
        n = len(s)
        border = prefix_function(s)[-1]
        return border > 0 and n % (n - border) == 0
```

Why the period: a border of length b means s has period n − b (shifting by n − b maps s
onto itself). The longest border gives the smallest period p. s is a whole number of
copies of `s[:p]` exactly when p divides n.

**Edge cases tested:**
- `"abab"`, `"aba"`, `"abcabcabcabc"`
- a single character (false), `"aa"` (true)
- 2,000 random strings, many deliberately periodic, checked against a divisor brute force

**Common bugs:**
- Forgetting the `pi[-1] > 0` check (n % n == 0 is true for any string).
- Testing only half-length repeats.

**Built-in shortcut:** approach 1 uses `in` **[KNOW-HOW]**.

## 29.7 Python-specific notes

**Built-in vs hand-written**, measured (`bench/ch29_matching.py`, n = 10⁴, best of 3, two
runs):

| Input | `str.find` | Brute force | Rabin–Karp | KMP | Z |
|---|---|---|---|---|---|
| Adversarial (`"a"*10⁴`, `"a"*5000 + "b"`) | 0.006 ms | 1,534–2,360 ms | 3.0–6.8 ms | 2.0–4.4 ms | 4.5–10.0 ms |
| Random letters, needle = last 50 characters | 0.001–0.002 ms | 0.9–1.5 ms | 2.7–6.5 ms | 0.8–1.7 ms | 1.6–3.9 ms |

1. **Brute force is quadratic on adversarial input:** 1.5–2.4 s against 2–10 ms for the
   linear methods. On random text it was competitive, because mismatches happen
   immediately.
2. **`str.find` was hundreds to thousands of times faster than any Python-level
   matcher.** That's partly the C loop, and partly the algorithm: on the adversarial case
   it can rule matches out quickly, since the needle's `'b'` never appears. Use it
   whenever matching isn't the point of the question.
3. Among the linear methods, KMP measured fastest here; Rabin–Karp pays for modular
   arithmetic per character.

**Separators** in Z/KMP concatenations must be characters that can't appear in the input
(`"\x00"`, `"#"` for lowercase-only problems).

## 29.8 Practice set

Hints are in Appendix D, under "Chapter 29 practice". Reference solutions:
`code/ch29_string_matching/practice_solutions.py`.

| # | Title | Difficulty | Link | Hint ref |
|---|---|---|---|---|
| 686 | Repeated String Match | Medium | <https://leetcode.com/problems/repeated-string-match/> | D.29.1 |

## 29.9 Chapter cheat sheet

| | |
|---|---|
| **Signals** | Find a pattern without built-ins; prefix = suffix; repetition/period; palindromic prefix |
| **Prefix function** | `pi[i]` = longest proper border of `p[:i+1]`; fallback `k = pi[k-1]`; O(m) |
| **KMP** | Text pointer never moves back; O(n + m) |
| **Z** | `z[i]` = LCP(s, s[i:]); search on `needle + sep + text`; O(n + m) |
| **Rabin–Karp** | Rolling hash + verification; O(n + m) expected |
| **Borders** | Longest = `pi[-1]`; period = n − `pi[-1]`; palindromic prefix via `s + '#' + s[::-1]` |
| **Top 3 bugs** | `pi[k]` instead of `pi[k-1]`; missing separators; unverified hash hits |
| **CPython** | `str.find` (Two-Way since 3.10, per the release notes) is far faster than any Python matcher (measured) |
