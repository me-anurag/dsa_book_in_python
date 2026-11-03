# Chapter 24: String Fundamentals for DSA

## 24.0 At a glance

- **What this pattern solves:** the everyday mechanics every string problem relies on:
  - mapping characters to indices (`ord(c) - ord("a")`)
  - counting with a 26-slot list
  - building results efficiently
  - scanning and tokenizing by hand
  - comparing strings
  - working around immutability

  Plus the classic warm-up problems that test exactly those mechanics.
- **Prerequisites:** Ch 5 (every `str` method, `split()` vs `split(" ")`, `+=` vs
  `join`), Ch 9 (hand-written tokenizing), Ch 13 (write pointers and the reversal trick).
- **Typical complexity:** O(n) time. O(1) extra space for a fixed alphabet, or O(n) when
  building a new string, which is unavoidable because strings are immutable.
- **Tools used in this chapter:**
  - `ord`, `chr`, `len`, slicing, `"".join` **[USE-FREELY]**
  - `split`, `startswith`, `sorted` on strings **[KNOW-HOW]**
  - character scanning, tokenizing, greedy table conversion and in-place reversal on a
    character list **[HAND-WRITE]**

## 24.1 Recognition signals

- Inputs restricted to "**lowercase English letters**" → a 26-slot list instead of a dict
- "**words separated by spaces**", "**multiple spaces**" → tokenizing (built-in or by hand)
- "**prefix**", "**subsequence**", "**merge alternately**" → index scanning with one or
  two pointers
- Symbol systems with a value table (Roman numerals) → table lookup + a greedy or
  look-ahead rule
- "**in place**" on a string → convert to a `list` first; Python strings can't change

**Look-alikes:**

| Looks like | Actually | Tell |
|---|---|---|
| Character counting | Sliding window on strings (Ch 27) | The counts are over a **moving substring** |
| Signature / anagram | Hashing on strings (Ch 25) | Grouping or comparing many strings by content |
| Scanning digits | Parsing (Ch 28) | Signs, overflow clamping, operators, nesting |

## 24.2 Core idea

**Characters are small integers.** `ord("a")` is 97 and `ord("z")` is 122, so
`ord(c) - ord("a")` maps lowercase letters to 0..25. That turns "count letters" into a
26-slot list (O(1) space, comparable in 26 steps) and makes Caesar shifts arithmetic.

```python
s = "banana"
counts = [0] * 26
for c in s:
    counts[ord(c) - ord("a")] += 1
counts[:3]  # -> [3, 1, 0]
chr(ord("a") + (ord("z") - ord("a") + 1) % 26)  # -> 'a'
```

**Strings are immutable, so build and edit with lists:**

| Need | Do | Don't |
|---|---|---|
| Build an output piece by piece | `parts.append(...)`, then `"".join(parts)` | `s += c` in a loop (Ch 5: quadratic once aliased) |
| Edit "in place" | `chars = list(s)`, edit, then `"".join(chars)` | `s[i] = c` (`TypeError`) |
| Read a piece | Index or slice: O(1) / O(k) | Slicing inside a loop when an index would do |

**Comparison is lexicographic by code point** (Ch 5): `"Z" < "a"`, and `"10" < "9"`.

## 24.3 The template

**Scan words from a string by hand** (the core of 58 and 151):

```python
def words_right_to_left(s: str) -> list[str]:
    words = []
    i = len(s) - 1
    while i >= 0:
        while i >= 0 and s[i] == " ":
            i -= 1
        end = i
        while i >= 0 and s[i] != " ":
            i -= 1
        if end > i:
            words.append(s[i + 1:end + 1])
    return words


words_right_to_left("  the sky  ")  # -> ['sky', 'the']
```

- **Invariant:** at the top of the outer loop, every word to the right of `i` has
  already been collected.
- **Complexity:** O(n) time (each index is visited once by one of the inner loops);
  O(n) space for the output.

**Two-pointer scan over two strings** (the core of 392 and 1768):

```python
def is_subsequence(s: str, t: str) -> bool:
    i = 0
    for c in t:
        if i < len(s) and s[i] == c:
            i += 1
    return i == len(s)


(is_subsequence("ace", "abcde"), is_subsequence("aec", "abcde"))  # -> (True, False)
```

O(len(t)) time, O(1) space.

## 24.4 Dry run

13 Roman to Integer on `"MCMXCIV"` with the look-ahead rule (subtract a symbol if the
next one is larger):

| i | c | value | next | rule | total |
|---|---|---|---|---|---|
| 0 | M | 1000 | C (100) | add | 1000 |
| 1 | C | 100 | M (1000) | **subtract** | 900 |
| 2 | M | 1000 | X (10) | add | 1900 |
| 3 | X | 10 | C (100) | **subtract** | 1890 |
| 4 | C | 100 | I (1) | add | 1990 |
| 5 | I | 1 | V (5) | **subtract** | 1989 |
| 6 | V | 5 | end | add | 1994 |

## 24.5 Variations

| Variant | What changes | Example |
|---|---|---|
| Common prefix of many strings | Vertical scan column by column; stop at the first mismatch | 14 Longest Common Prefix |
| Last token | Scan from the right, skipping trailing spaces | 58 Length of Last Word |
| Subsequence | Greedy two pointers; many queries → per-letter index lists + binary search | 392 Is Subsequence |
| Interleave | One index, two strings, `join` at the end | 1768 Merge Strings Alternately |
| Reverse word order | `split` + `join`; by hand; in place on a char list | 151 Reverse Words in a String |
| Symbols → number | Value table + look-ahead subtraction | 13 Roman to Integer |
| Number → symbols | Greedy over a descending table including subtractive pairs | 12 Integer to Roman |

## 24.6 Worked problems

### 24.6.1 151 Reverse Words in a String (Medium): <https://leetcode.com/problems/reverse-words-in-a-string/>

**Problem.** Reverse the order of the words in a string. Words are separated by one or
more spaces, and there may be leading or trailing spaces. The result must use single
spaces and no padding. The follow-up asks for O(1) extra space if strings were mutable.

**Signatures.** LeetCode: `def reverseWords(self, s: str) -> str:`. Book: identical.

**Constraints → target.** n ≤ 10⁴, so O(n). The point is the mechanics: built-in, by hand,
and in place.

**Approach 1: built-ins.**

<!-- file: code/ch24_strings/reverse_words.py -->

```python
class SolutionBuiltins:
    def reverseWords(self, s: str) -> str:
        # O(n) time: split, reverse, join are each one pass. O(n) auxiliary.
        # split() with no argument collapses runs of whitespace (Ch 5).
        return " ".join(reversed(s.split()))
```

This is the right answer on LeetCode. Note `split()`, **not** `split(" ")`, which would
produce empty strings for the repeated spaces (Ch 5, 5.2.4).

**Approach 2: hand tokenizer (no `split`).**

<!-- file: code/ch24_strings/reverse_words.py -->

```python
class SolutionHandTokenizer:
    def reverseWords(self, s: str) -> str:
        # O(n) time, O(n) auxiliary. No split/join of words: scan from the right,
        # collecting each word's characters, then emit words in that order.
        words: list[str] = []
        i = len(s) - 1
        while i >= 0:
            while i >= 0 and s[i] == " ":
                i -= 1                              # skip spaces
            end = i
            while i >= 0 and s[i] != " ":
                i -= 1                              # walk back over one word
            if end >= 0 and end > i:
                words.append(s[i + 1:end + 1])
        return " ".join(words)
```

**Approach 3: the follow-up's in-place algorithm on a character list.**

<!-- file: code/ch24_strings/reverse_words.py -->

```python
class Solution:
    def reverseWords(self, s: str) -> str:
        # The follow-up's in-place algorithm on a mutable char array (a list, since
        # Python strings are immutable). O(n) time; O(1) auxiliary beyond the array.
        chars = list(s)

        def reverse(left: int, right: int) -> None:
            while left < right:
                chars[left], chars[right] = chars[right], chars[left]
                left += 1
                right -= 1

        # 1. Compact: remove extra spaces with a write pointer (Ch 13).
        write = 0
        for read in range(len(chars)):
            if chars[read] != " ":
                if write > 0 and chars[read - 1] == " ":
                    chars[write] = " "              # one space before a new word
                    write += 1
                chars[write] = chars[read]
                write += 1
        del chars[write:]
        # 2. Reverse everything, then 3. reverse each word back (Ch 13's rotation).
        reverse(0, len(chars) - 1)
        start = 0
        for end in range(len(chars) + 1):
            if end == len(chars) or chars[end] == " ":
                reverse(start, end - 1)
                start = end + 1
        return "".join(chars)
```

Why reverse-then-reverse works: reversing the whole string puts the words in reverse
order, but spells each word backwards. Reversing each word again fixes the spelling. It's
the same idea as 189's rotation (Ch 13). In Python, `list(s)` and the final `join` cost
O(n) each, which is unavoidable with immutable strings. The **algorithm** itself uses
O(1) extra space, which is what the follow-up asks about.

**Edge cases tested:**
- leading/trailing/multiple spaces
- a single word, a padded single word
- mixed letters and digits
- 1,000 random strings checked across all three approaches

**Common bugs:**
- `split(" ")`, which leaves empty strings.
- Forgetting the last word in a hand tokenizer.
- A compaction that writes a leading space before the first word (the `write > 0` check
  prevents it).

**Built-in shortcut:** approach 1. `split` is **[KNOW-HOW]**; `join` and `reversed` are
standard.

### 24.6.2 13 Roman to Integer (Easy): <https://leetcode.com/problems/roman-to-integer/>

**Problem.** Convert a valid Roman numeral (1..3999) to an integer. A smaller symbol
placed before a larger one is subtracted (IV = 4, CM = 900).

**Signatures.** LeetCode: `def romanToInt(self, s: str) -> int:`. Book: identical.

**Constraints → target.** At most 15 characters. Correctness is the issue, not speed.

**Approach 1: rewrite the subtractive pairs, then add.**

<!-- file: code/ch24_strings/roman_numerals.py -->

```python
class SolutionRomanToIntReplace:
    def romanToInt(self, s: str) -> int:
        # O(n) time: six replace passes plus one sum, over n <= 15 characters.
        # Rewrite subtractive pairs as additive ones, then just add.
        for pair, expanded in (("CM", "DCCCC"), ("CD", "CCCC"), ("XC", "LXXXX"),
                               ("XL", "XXXX"), ("IX", "VIIII"), ("IV", "IIII")):
            s = s.replace(pair, expanded)
        return sum(VALUE[c] for c in s)
```

**Approach 2: look-ahead subtraction (optimal and general).**

<!-- file: code/ch24_strings/roman_numerals.py -->

```python
class SolutionRomanToInt:
    def romanToInt(self, s: str) -> int:
        # O(n) time, O(1) space. A symbol smaller than the one after it is
        # subtracted (IV = 5 - 1); otherwise it's added.
        total = 0
        for i, c in enumerate(s):
            v = VALUE[c]
            if i + 1 < len(s) and v < VALUE[s[i + 1]]:
                total -= v
            else:
                total += v
        return total
```

Both use the value table:

<!-- file: code/ch24_strings/roman_numerals.py -->

```python
VALUE = {"I": 1, "V": 5, "X": 10, "L": 50, "C": 100, "D": 500, "M": 1000}
```

Why look-ahead works: in a valid numeral, a symbol is subtracted exactly when a larger
symbol follows it. Traced in 24.4.

**Edge cases tested:**
- "I", "MMMCMXCIX" (the extremes)
- **all 3999 values**, round-tripped through the 12 solution below

**Common bugs:**
- Comparing with the **previous** symbol and forgetting to undo its addition (a
  `total += v - 2 * prev` style exists, but it's easy to get wrong).
- Indexing `s[i + 1]` without a bounds check.

**Built-in shortcut:** approach 1's `replace` chain **[KNOW-HOW]**. It's cute, but the
look-ahead rule is what interviewers expect.

### 24.6.3 12 Integer to Roman (Medium): <https://leetcode.com/problems/integer-to-roman/>

**Problem.** Convert an integer in 1..3999 to a Roman numeral, using the standard
subtractive forms (4 = IV, 9 = IX, 40 = XL, 90 = XC, 400 = CD, 900 = CM).

**Signatures.** LeetCode: `def intToRoman(self, num: int) -> str:`. Book: identical.

**Constraints → target.** num ≤ 3999. Bounded output (at most 15 characters), so O(1).

**Approach: greedy over a descending table that includes the subtractive pairs.**

<!-- file: code/ch24_strings/roman_numerals.py -->

```python
# Largest first, including the six subtractive pairs, so greedy works.
TABLE = [(1000, "M"), (900, "CM"), (500, "D"), (400, "CD"), (100, "C"), (90, "XC"),
         (50, "L"), (40, "XL"), (10, "X"), (9, "IX"), (5, "V"), (4, "IV"), (1, "I")]
```

<!-- file: code/ch24_strings/roman_numerals.py -->

```python
class SolutionIntToRoman:
    def intToRoman(self, num: int) -> str:
        # O(1) time for num <= 3999: at most 13 table entries and at most 15 output
        # characters. Greedy: take the largest value that still fits, repeatedly.
        parts: list[str] = []
        for value, symbol in TABLE:
            count, num = divmod(num, value)
            parts.append(symbol * count)
        return "".join(parts)
```

Why greedy is correct here: with the subtractive pairs in the table, each decimal digit's
symbols come from a fixed small set, and taking the largest fitting value never blocks a
later choice. The test file confirms it by round-tripping **every** value from 1 to 3999
through both conversions. (A naive "brute force", trying all symbol strings, isn't a
sensible baseline for this problem. The exhaustive round trip is the stronger check.)

**Edge cases tested:**
- the statement examples
- all 3999 values round-tripped

**Common bugs:**
- Leaving the subtractive pairs out of the table, which produces "IIII" for 4.
- An ascending table order, which breaks the greedy choice.

**Built-in shortcut:** none.

### 24.6.4 14 Longest Common Prefix (Easy): <https://leetcode.com/problems/longest-common-prefix/>

**Problem.** Return the longest string that is a prefix of every string in the list (""
if none).

**Signatures.** LeetCode: `def longestCommonPrefix(self, strs: List[str]) -> str:`. Book:
`list[str]`.

**Constraints → target.** At most 200 strings of length ≤ 200. Any linear-ish scan works.
The interest is in three clean ways to think about it.

**Approach 1: horizontal (shrink a candidate).**

<!-- file: code/ch24_strings/longest_common_prefix.py -->

```python
class SolutionHorizontal:
    def longestCommonPrefix(self, strs: list[str]) -> str:
        # O(k * L + L^2) time, L = len(strs[0]): one O(L) startswith per string,
        # plus at most L shrink steps overall (the prefix only shrinks), each O(L)
        # to check and copy. O(L) auxiliary for the candidate.
        prefix = strs[0]
        for s in strs[1:]:
            while not s.startswith(prefix):
                prefix = prefix[:-1]            # shrink until it matches
        return prefix
```

**Approach 2: compare the smallest and largest strings.**

<!-- file: code/ch24_strings/longest_common_prefix.py -->

```python
class SolutionMinMax:
    def longestCommonPrefix(self, strs: list[str]) -> str:
        # O(S) time: min and max each compare strings lexicographically (each
        # comparison O(m)), then O(m) to compare the two. The common prefix of ALL
        # strings equals that of the lexicographically smallest and largest
        # (they differ the most). O(1) auxiliary besides the output.
        first, last = min(strs), max(strs)      # O(S) instead of a full sort
        i = 0
        while i < len(first) and i < len(last) and first[i] == last[i]:
            i += 1
        return first[:i]
```

**Approach 3: vertical scan (optimal in the worst case).**

<!-- file: code/ch24_strings/longest_common_prefix.py -->

```python
class Solution:
    def longestCommonPrefix(self, strs: list[str]) -> str:
        # Vertical scan. O(k * m) time: compare column by column, stopping at the
        # first mismatch or the end of the shortest string. O(1) auxiliary besides
        # the output slice.
        first = strs[0]
        for i in range(len(first)):
            c = first[i]
            for s in strs[1:]:
                if i == len(s) or s[i] != c:
                    return first[:i]
        return first
```

Why approach 2 is valid: if the smallest and largest strings (in lexicographic order)
share a prefix p, every string between them in that order also starts with p. Strings
sort by their first differing character, so any string that deviated from p would fall
outside the range from smallest to largest.

**Edge cases tested:**
- a single string, an empty string in the list
- all equal
- one string being a prefix of another
- 1,000 random lists checked against the standard library's `os.path.commonprefix`

**Common bugs:**
- Indexing past the end of a shorter string (the `i == len(s)` check).
- Returning `None` instead of `""`.

**Built-in shortcut:** `os.path.commonprefix(strs)` **[KNOW-HOW]**. It's a character-wise
common prefix despite living in `os.path`, and the test file uses it as the reference.
Mention it as an aside.

## 24.7 Python-specific notes

**Unicode caveats (short).** LeetCode string problems are almost always ASCII, where one
character is one code point. Beyond ASCII:

```python
len("é")  # -> 1
len("é")  # -> 2
"é" == "é"  # -> False
"Straße".lower() == "strasse"  # -> False
"Straße".casefold() == "strasse"  # -> True
```

`"é"` can be one code point or two (an `e` plus a combining accent). The two forms look
identical but compare unequal unless normalized (`unicodedata.normalize`). `casefold()`
handles case-insensitive comparison beyond ASCII (Ch 5). The `ord(c) - ord("a")` trick is
valid **only** for the 26 ASCII lowercase letters: check the constraints before using a
26-slot list.

**Building strings:** always a list plus `join` (measured in Ch 5: `+=` turns quadratic as
soon as the string is aliased).

**`os.path.commonprefix` and `str.startswith` with a tuple** are handy built-ins.
`str.removeprefix` (3.9+) removes an exact prefix once (Ch 5).

## 24.8 Practice set

Hints are in Appendix D, under "Chapter 24 practice". Reference solutions:
`code/ch24_strings/practice_solutions.py`.

| # | Title | Difficulty | Link | Hint ref |
|---|---|---|---|---|
| 58 | Length of Last Word | Easy | <https://leetcode.com/problems/length-of-last-word/> | D.24.1 |
| 392 | Is Subsequence | Easy | <https://leetcode.com/problems/is-subsequence/> | D.24.2 |
| 1768 | Merge Strings Alternately | Easy | <https://leetcode.com/problems/merge-strings-alternately/> | D.24.3 |

## 24.9 Chapter cheat sheet

| | |
|---|---|
| **Signals** | Lowercase-only alphabet; words and spaces; prefix/subsequence; symbol tables; "in place" on a string |
| **Char ↔ index** | `ord(c) - ord("a")`, `chr(ord("a") + k)`; ASCII lowercase only |
| **Counting** | `counts = [0] * 26`; compare two counts in 26 steps |
| **Building** | `parts.append(...)`, then `"".join(parts)`; edit via `list(s)` |
| **Tokenizing** | `split()` (not `split(" ")`), or scan with skip-spaces / walk-word loops |
| **Complexity** | O(n) time; O(1) space for a fixed alphabet, O(n) to build output |
| **Top 3 bugs** | `split(" ")`; the forgotten last word; `ord` arithmetic on non-ASCII input |
