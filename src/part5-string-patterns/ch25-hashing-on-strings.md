# Chapter 25: Hashing on Strings

## 25.0 At a glance

- **What this pattern solves:** comparing, grouping and searching strings **by
  content** in O(1) per comparison, after an O(length) step that turns each string (or
  each window of a string) into a **key**: a sorted string, a count tuple, an integer
  encoding, or a rolling hash.
- **Prerequisites:** Ch 14 (hash maps, signatures, 49 Group Anagrams), Ch 17 (fixed
  windows), Ch 24 (`ord` arithmetic, 26-slot counts).
- **Typical complexity:** O(n) or O(n · L) time to build keys; O(1) average per hash
  lookup.
- **Tools used in this chapter:**
  - `dict`, `set`, `tuple` keys **[USE-FREELY]**
  - `sorted` on strings, `Counter` **[KNOW-HOW]**
  - sliding count signatures, bit-packed keys, Rabin–Karp rolling hashes and
    length-prefixed encoding **[HAND-WRITE]**

This chapter revisits 242 and 49 from Ch 14 with a different focus: **which key to
choose, and what it costs**.

## 25.1 Recognition signals

- "**anagram**", "**permutation of**", "**same characters**" → a count or sorted signature
- "find all **windows / substrings** of s that are anagrams of p" → a sliding count
  signature
- "**repeated substrings** of fixed length", "**duplicate** sequences" → substring keys
  or rolling hashes
- "**serialize / encode** a list of strings into one string" → length-prefixed framing
- A small fixed alphabet (lowercase, DNA's ACGT) → counts or bit-packing instead of
  general hashing

**Look-alikes:**

| Looks like | Actually | Tell |
|---|---|---|
| Sliding signature (fixed size) | Variable sliding window (Ch 27) | "Longest/shortest substring with a constraint": the size isn't fixed |
| Rolling hash to find a repeat | String matching (Ch 29) | Find **one pattern** in a text: KMP/Z |
| Group by signature | Sorting (Ch 18) | You need an order, not equivalence classes |

## 25.2 Core idea

Comparing two strings costs O(L). Comparing each of n strings with every other one is
O(n² · L). A **key** that's equal exactly when the strings are "equivalent" turns
equivalence into hashing:

| Equivalence | Key | Cost to build | Notes |
|---|---|---|---|
| Same multiset of letters (anagrams) | `"".join(sorted(s))` | O(L log L) | Any alphabet |
| | `tuple(26 counts)` | O(L + 26) | Lowercase only; hashable |
| | `Counter(s)` (compare directly) | O(L) | Not hashable; use `frozenset(c.items())` as a key |
| Identical substring | The substring itself | O(L) slice + O(L) hash | Simple; measured fastest in CPython (25.7) |
| | Bit-packed integer (fixed small alphabet) | O(1) per step, rolling | Exact: no collisions |
| | Rabin–Karp hash mod M | O(1) per step, rolling | Collisions possible: **verify** |

**Rolling** means updating a window's key in O(1) as the window slides, instead of
rebuilding it in O(L): add the entering character, remove the leaving one.

```text
DNA window key, 2 bits per letter (A=00, C=01, G=10, T=11), window of 3:

"ACG" -> 00 01 10       slide in "T":  shift left 2, OR in 11, mask to 6 bits
"CGT" ->    01 10 11
```

## 25.3 The template

**Sliding count signature** (fixed window of size m over s):

```python
def windows_matching_counts(s: str, p: str) -> list[int]:
    m = len(p)
    need, window = [0] * 26, [0] * 26
    for c in p:
        need[ord(c) - 97] += 1
    out = []
    for i, c in enumerate(s):
        window[ord(c) - 97] += 1
        if i >= m:
            window[ord(s[i - m]) - 97] -= 1
        if i >= m - 1 and window == need:
            out.append(i - m + 1)
    return out


windows_matching_counts("cbaebabacd", "abc")  # -> [0, 6]
```

- **Invariant:** after processing index i, `window` holds the counts of `s[i-m+1 .. i]`
  (or of everything so far, while i < m − 1).
- **Complexity:** O(26 · n) = O(n) time (two updates and one 26-slot comparison per
  step); O(1) auxiliary space.

**Rabin–Karp rolling hash** (window of length L, base B, modulus M):

```text
h(window) = c0·B^(L-1) + c1·B^(L-2) + ... + c(L-1)        (mod M)
slide:    h = (h - c_leaving · B^(L-1)) · B + c_entering   (mod M)
```

With a large prime-like M (here 2⁶¹ − 1), collisions are rare. They're never impossible,
so **compare the actual substrings on a hash hit**.

## 25.4 Dry run

438 Find All Anagrams on `s = "cbaebabacd"`, `p = "abc"` (window size 3). Counts are
shown only for a, b, c, e, d.

| i | enters | leaves | window counts (a, b, c, d, e) | equals need (1,1,1,0,0)? |
|---|---|---|---|---|
| 0 | c | | 0 0 1 0 0 | |
| 1 | b | | 0 1 1 0 0 | |
| 2 | a | | 1 1 1 0 0 | **yes** → start 0 |
| 3 | e | c | 1 1 0 0 1 | no |
| 4 | b | b | 1 1 0 0 1 | no |
| 5 | a | a | 1 1 0 0 1 | no |
| 6 | b | e | 1 2 0 0 0 | no |
| 7 | a | b | 2 1 0 0 0 | no |
| 8 | c | a | 1 1 1 0 0 | **yes** → start 6 |
| 9 | d | b | 1 0 1 1 0 | no |

Answer: `[0, 6]`.

## 25.5 Variations

| Variant | What changes | Example |
|---|---|---|
| Two strings, same multiset? | Sorted / Counter / 26 counts | 242 Valid Anagram (three ways) |
| Group many strings | Dict keyed by signature (sorted string or count tuple) | 49 Group Anagrams (Ch 14) |
| All anagram windows | Sliding count signature | 438 Find All Anagrams in a String |
| Any anagram window? | Same, return on the first match | 567 Permutation in String |
| Repeated fixed-length substrings | Substring set; bit-packed rolling key; Rabin–Karp with verification | 187 Repeated DNA Sequences |
| One string ↔ many strings | Length-prefixed framing | 271 Encode and Decode Strings (Premium) |

## 25.6 Worked problems

### 25.6.1 242 Valid Anagram, revisited three ways (Easy): <https://leetcode.com/problems/valid-anagram/>

**Problem.** Are two lowercase strings rearrangements of each other? The follow-up asks
what to do if the input can contain any Unicode characters.

**Signatures.** LeetCode: `def isAnagram(self, s: str, t: str) -> bool:`. Book: identical.

**Constraints → target.** Lengths ≤ 5 × 10⁴. All three keys pass easily. The question is
which key fits which situation.

**Way 1: sorted characters.**

<!-- file: code/ch25_string_hashing/valid_anagram.py -->

```python
class SolutionSorted:
    def isAnagram(self, s: str, t: str) -> bool:
        # Signature = sorted characters. O(n log n) time, O(n) auxiliary.
        # Works for any characters (Unicode included).
        return sorted(s) == sorted(t)
```

**Way 2: Counter (the follow-up's answer).**

<!-- file: code/ch25_string_hashing/valid_anagram.py -->

```python
class SolutionCounter:
    def isAnagram(self, s: str, t: str) -> bool:
        # Signature = multiset of characters. O(n) time, O(k) space for k distinct
        # characters. The follow-up answer: works for any alphabet.
        return Counter(s) == Counter(t)
```

**Way 3: one 26-slot count (optimal for lowercase).**

<!-- file: code/ch25_string_hashing/valid_anagram.py -->

```python
class Solution:
    def isAnagram(self, s: str, t: str) -> bool:
        # Signature = 26 counts. O(n) time, O(1) space (26 slots). Lowercase only.
        if len(s) != len(t):
            return False
        counts = [0] * 26
        for a, b in zip(s, t):
            counts[ord(a) - 97] += 1
            counts[ord(b) - 97] -= 1
        return not any(counts)
```

| Way | Time | Space | Alphabet |
|---|---|---|---|
| Sorted | O(n log n) | O(n) | Any |
| Counter | O(n) | O(k) | Any (the follow-up) |
| 26 counts | O(n) | O(1) | Lowercase ASCII only |

**Edge cases tested:**
- different lengths, a single character
- the same letters with different counts
- non-ASCII input for the general versions
- 1,000 random pairs checked across all three

**Common bugs:**
- Skipping the length check in way 3: `zip` silently stops at the shorter string.
- Using way 3 on Unicode input (Ch 24, 24.7).

**Built-in shortcut:** way 2, `Counter` **[KNOW-HOW]**.

### 25.6.2 438 Find All Anagrams in a String (Medium): <https://leetcode.com/problems/find-all-anagrams-in-a-string/>

**Problem.** Return the start indices of every substring of `s` that is an anagram of `p`
(any order).

**Signatures.** LeetCode: `def findAnagrams(self, s: str, p: str) -> List[int]:`. Book:
`list[int]`.

**Constraints → target.** n, m ≤ 3 × 10⁴, lowercase only. Rebuilding a key per window
costs O(n · m) or worse. Sliding counts cost O(26 · n).

**Approach 1: sort every window.**

<!-- file: code/ch25_string_hashing/find_anagrams.py -->

```python
class SolutionSortWindows:
    def findAnagrams(self, s: str, p: str) -> list[int]:
        # O(n * m log m) time: sort every window. O(m) auxiliary.
        key = sorted(p)
        m = len(p)
        return [i for i in range(len(s) - m + 1) if sorted(s[i:i + m]) == key]
```

**Approach 2: sliding 26-count signature.**

<!-- file: code/ch25_string_hashing/find_anagrams.py -->

```python
class SolutionSlidingCounts:
    def findAnagrams(self, s: str, p: str) -> list[int]:
        # O(26 * n) time: update two letters per step, compare 26 counts.
        # O(1) auxiliary (two 26-slot lists).
        m = len(p)
        if m > len(s):
            return []
        need = [0] * 26
        window = [0] * 26
        for c in p:
            need[ord(c) - 97] += 1
        out = []
        for i, c in enumerate(s):
            window[ord(c) - 97] += 1                   # enter
            if i >= m:
                window[ord(s[i - m]) - 97] -= 1        # leave
            if i >= m - 1 and window == need:
                out.append(i - m + 1)
        return out
```

**Approach 3: a "matches" counter, O(n) independent of the alphabet size.**

<!-- file: code/ch25_string_hashing/find_anagrams.py -->

```python
class Solution:
    def findAnagrams(self, s: str, p: str) -> list[int]:
        # O(n) time: keep `matches` = number of letters whose window count equals
        # the needed count; each step changes at most two letters, so updating
        # `matches` is O(1). O(1) auxiliary.
        m = len(p)
        if m > len(s):
            return []
        need = [0] * 26
        for c in p:
            need[ord(c) - 97] += 1
        window = [0] * 26
        matches = sum(1 for k in range(26) if need[k] == 0)   # empty window

        def change(k: int, delta: int) -> int:
            """Apply window[k] += delta; return the change in `matches`."""
            before = window[k] == need[k]
            window[k] += delta
            after = window[k] == need[k]
            return int(after) - int(before)

        out = []
        for i, c in enumerate(s):
            matches += change(ord(c) - 97, +1)
            if i >= m:
                matches += change(ord(s[i - m]) - 97, -1)
            if matches == 26:
                out.append(i - m + 1)
        return out
```

Why approach 3 is "better" on paper: comparing 26 slots every step is O(Σ) per step;
tracking how many letters already match avoids it. With a large alphabet (Unicode), that
matters. **With 26 letters in CPython, approach 2 measured faster** (25.7), because
`window == need` compares the lists in C. Traced (approach 2) in 25.4.

**Edge cases tested:**
- p longer than s
- all-equal letters (overlapping windows)
- p equal to s
- n = 3 × 10⁴
- 1,000 random cases across all three approaches

**Common bugs:**
- Removing the leaving character before the window is full.
- An off-by-one start index (`i - m + 1`).
- Comparing `Counter` objects per window: correct, but O(m) per window.

**Built-in shortcut:** a `Counter` window with `==`: correct, and simpler than approach 3.

### 25.6.3 187 Repeated DNA Sequences (Medium): <https://leetcode.com/problems/repeated-dna-sequences/>

**Problem.** Return every 10-letter substring that occurs more than once in a DNA string
(letters A, C, G, T), in any order.

**Signatures.** LeetCode: `def findRepeatedDnaSequences(self, s: str) -> List[str]:`. Book:
`list[str]`.

**Constraints → target.** n ≤ 10⁵, L = 10. O(n · L) = 10⁶ is fine. The problem is a
showcase for rolling keys.

**Approach 1: a set of substrings.**

<!-- file: code/ch25_string_hashing/repeated_dna.py -->

```python
class SolutionSubstringSet:
    def findRepeatedDnaSequences(self, s: str) -> list[str]:
        # O(n * L) time: each window is sliced (L chars) and hashed (L chars).
        # O(n * L) space: up to n stored substrings of length L.
        seen: set[str] = set()
        repeated: set[str] = set()
        for i in range(len(s) - L + 1):
            window = s[i:i + L]
            if window in seen:
                repeated.add(window)
            seen.add(window)
        return list(repeated)
```

**Approach 2: bit-packed rolling key (exact).** Four letters fit in 2 bits, so a window
of 10 is a 20-bit integer:

<!-- file: code/ch25_string_hashing/repeated_dna.py -->

```python
class SolutionRollingBits:
    def findRepeatedDnaSequences(self, s: str) -> list[str]:
        # O(n) time: 4 letters fit in 2 bits, so a 10-letter window is a 20-bit int,
        # updated in O(1) per step (shift in the new letter, mask off the old one).
        # O(n) space for the set of ints. Exact: no collisions possible.
        code = {"A": 0, "C": 1, "G": 2, "T": 3}
        mask = (1 << (2 * L)) - 1
        seen: set[int] = set()
        found: set[int] = set()
        out: list[str] = []
        key = 0
        for i, c in enumerate(s):
            key = ((key << 2) | code[c]) & mask
            if i >= L - 1:
                if key in seen and key not in found:
                    found.add(key)
                    out.append(s[i - L + 1:i + 1])
                seen.add(key)
        return out
```

**Approach 3: Rabin–Karp (general alphabet, with verification).**

<!-- file: code/ch25_string_hashing/repeated_dna.py -->

```python
class Solution:
    def findRepeatedDnaSequences(self, s: str) -> list[str]:
        # General Rabin-Karp rolling hash (works for any alphabet).
        # hash(window) = sum(c_k * B^(L-1-k)) mod M, updated in O(1) per step.
        # A hash hit is VERIFIED by comparing substrings (O(L)), since collisions
        # are possible in principle. Time: O(n) for the rolling plus O(L) per hit,
        # so O(n * L) worst case (every window repeats); L = 10 is fixed here.
        base, mod = 4, (1 << 61) - 1
        code = {"A": 0, "C": 1, "G": 2, "T": 3}
        top = pow(base, L - 1, mod)               # weight of the leaving letter
        starts: dict[int, list[int]] = {}         # hash -> window starts seen
        reported: set[str] = set()
        out: list[str] = []
        h = 0
        for i, c in enumerate(s):
            if i >= L:
                h = (h - code[s[i - L]] * top) % mod     # remove the leaving letter
            h = (h * base + code[c]) % mod                # add the entering letter
            if i >= L - 1:
                start = i - L + 1
                bucket = starts.get(h)
                if bucket is None:                         # new hash: no slicing at all
                    starts[h] = [start]
                    continue
                window = s[start:start + L]
                for prev in bucket:
                    if s[prev:prev + L] == window:        # verify: no false positives
                        if window not in reported:
                            reported.add(window)
                            out.append(window)
                        break
                else:
                    bucket.append(start)                  # true collision: keep both
```

(Ch 29 uses the same rolling hash to search for a pattern.)

**Edge cases tested:**
- a string shorter than 10, exactly one window
- all-A strings (every window repeats)
- n = 10⁵
- 500 random two-letter strings (many repeats), checked across all three approaches

**Common bugs:**
- Reporting the same sequence once per extra occurrence (use a "reported" set).
- Rolling hashes without verification, which can produce false positives.
- Forgetting the mask, so the key keeps growing beyond 20 bits (Python ints never
  overflow, so the bug is silent; Ch 3).

**Built-in shortcut:** approach 1, which is also the fastest measured (25.7).

### 25.6.4 271 Encode and Decode Strings (Medium, **Premium**): <https://leetcode.com/problems/encode-and-decode-strings/>

**Problem.** Design a `Codec` with `encode(strs) -> str` and `decode(s) -> list[str]` so
that `decode(encode(strs)) == strs` for any list of strings. Strings may contain any of
the 256 ASCII characters, so no separator character is safe. The follow-up asks for a
scheme that works for any character set.

**Availability.** This problem requires LeetCode Premium. **Free alternative:**
`code/ch25_string_hashing/codec.py` is the judge. It round-trips edge cases and 1,000
random lists over all 256 characters. Solve it there without a subscription.

**Signatures.** LeetCode template: `class Codec` with
`def encode(self, strs: List[str]) -> str:` and `def decode(self, s: str) -> List[str]:`.
Book: `list[str]`.

**Approach 1: join with a delimiter (broken).**

<!-- file: code/ch25_string_hashing/codec.py -->

```python
class CodecNaiveDelimiter:
    """Broken on purpose: joins with a separator that may occur inside a string."""

    def encode(self, strs: list[str]) -> str:
        return ",".join(strs)

    def decode(self, s: str) -> list[str]:
        return s.split(",")
```

It fails on `["a,b", "c"]`, which decodes to three strings. It also can't distinguish
`[]` from `[""]`.

**Approach 2: escape the delimiter.**

<!-- file: code/ch25_string_hashing/codec.py -->

```python
class CodecEscaping:
    """Correct: escape the separator inside strings. O(total) both ways."""

    def encode(self, strs: list[str]) -> str:
        # "/" escapes itself and ";" ; an unescaped ";" ends each string.
        return "".join(x.replace("/", "//").replace(";", "/;") + ";" for x in strs)

    def decode(self, s: str) -> list[str]:
        out: list[str] = []
        current: list[str] = []
        i = 0
        while i < len(s):
            c = s[i]
            if c == "/":                     # escaped character: take the next one
                current.append(s[i + 1])
                i += 2
            elif c == ";":                   # end of one string
                out.append("".join(current))
                current = []
                i += 1
            else:
                current.append(c)
                i += 1
        return out
```

**Approach 3: length prefix (optimal and simplest to prove).**

<!-- file: code/ch25_string_hashing/codec.py -->

```python
class Codec:
    """Length prefix: "<len>#<chars>" per string. Works for ANY characters (the
    follow-up), because the decoder never interprets the payload, it only counts.
    O(total) time both ways; O(total) for the output."""

    def encode(self, strs: list[str]) -> str:
        return "".join(f"{len(x)}#{x}" for x in strs)

    def decode(self, s: str) -> list[str]:
        out: list[str] = []
        i = 0
        while i < len(s):
            j = s.index("#", i)              # the length field contains only digits
            length = int(s[i:j])
            out.append(s[j + 1:j + 1 + length])
            i = j + 1 + length
        return out
```

Why length prefixes are robust: the decoder reads a run of digits up to `#`, then skips
**exactly** that many characters, whatever they are, `#` and digits included. Nothing
inside a string is ever interpreted, which is why it answers the follow-up for any
alphabet (Unicode included; the tests use an emoji).

**Edge cases tested:**
- `[]`, `[""]`, `["", ""]`
- strings containing `,`, `#`, `;`, `/`, digits, NUL, `\xff`, newlines and an emoji
- 200 strings of 200 characters
- 1,000 random lists over all 256 ASCII characters

**Common bugs:**
- Any delimiter-only scheme.
- Using `s.index("#")` from the start of the string instead of from `i`.
- Losing empty strings.

**Built-in shortcut:** `json.dumps` / `json.loads` would round-trip a list of strings, but
interviewers want the framing idea.

## 25.7 Python-specific notes

**Simple beats clever in CPython**, measured (`bench/ch25_string_hashing.py`, best of 3,
two runs):

| 187, n = 10⁵ | ms |
|---|---|
| Set of substrings (O(n · L), slicing and hashing in C) | 20.6–21.7 |
| Bit-packed rolling key (O(n), Python-level arithmetic) | 26.6–27.0 |
| Rabin–Karp with verification | 52.2–52.4 |

| 438, n = 3 × 10⁴, m = 100 | ms |
|---|---|
| Sort each window | 191.5–195.5 |
| Sliding 26-count compare (O(26 · n)) | 4.9–5.4 |
| "Matches" counter (O(n)) | 13.0–13.4 |

For both problems, the version with the best complexity on paper wasn't the fastest. With
a small fixed L (10) or alphabet (26), the "extra" factor is executed by C code (string
slicing and hashing, list comparison), while the clever version does more Python-level
work per character. **In interviews, present the rolling or matching technique and its
complexity; on LeetCode, the simple version is a legitimate choice. State the trade-off.**

**Hashable keys:** `tuple(counts)` and `str` work, `list` and `Counter` don't (Ch 7).
**String hashes are cached** after the first computation in CPython, but slicing creates a
new string each time, which gets hashed once (general knowledge, not benchmarked here).

**Rolling hash arithmetic never overflows** in Python, but without `% mod` the numbers
grow without bound and arithmetic slows down. Always reduce.

## 25.8 Practice set

Hints are in Appendix D, under "Chapter 25 practice". Reference solutions:
`code/ch25_string_hashing/practice_solutions.py`, and `code/ch14_hashing/group_anagrams.py`
for 49.

| # | Title | Difficulty | Link | Hint ref |
|---|---|---|---|---|
| 49 | Group Anagrams (revisit) | Medium | <https://leetcode.com/problems/group-anagrams/> | D.25.1 |
| 567 | Permutation in String | Medium | <https://leetcode.com/problems/permutation-in-string/> | D.25.2 |

## 25.9 Chapter cheat sheet

| | |
|---|---|
| **Signals** | Anagram/permutation; all anagram windows; repeated fixed-length substrings; serialize strings |
| **Keys** | Sorted string (any alphabet), 26-count tuple (lowercase), substring itself, bit-packed int (tiny alphabet), Rabin–Karp (general; verify hits) |
| **Sliding signature** | Add the entering char, remove the leaving char, compare with the target counts |
| **Framing** | `len#payload`, never delimiters alone |
| **Complexity** | O(n) to O(n · L) time; O(1) average per lookup |
| **Top 3 bugs** | Unhashable keys; unverified rolling-hash hits; delimiter-based encoding |
| **CPython** | Substring set and 26-count compare measured faster than the clever rolling versions |
