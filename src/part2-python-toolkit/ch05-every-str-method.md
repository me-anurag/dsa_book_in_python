# Chapter 5: Every `str` Method

## 5.0 At a glance

Python 3.13.2 has **47 public `str` methods**. This chapter lists all of them,
grouped by job, with complexity and a tier tag, and then the operations that aren't
methods (indexing, slicing, `+`, `in`, comparison). `code/ch05_str/reference_check.py`
asserts that the tables below match `dir(str)` on the running interpreter exactly.

Three things matter more than the rest:

1. **Strings are immutable.** No method changes a string; every "change" returns a new
   one. Building a string piece by piece needs a list and `"".join` (5.2.7, measured).
2. **`split()` and `split(" ")` are different functions in disguise.** The second
   produces empty strings for repeated spaces (5.2.4).
3. **`isdigit`, `isdecimal` and `isnumeric` accept Unicode characters you didn't
   expect.** For "is this an ASCII digit", compare `"0" <= c <= "9"` (5.2.3).

Notation: n = `len(s)`, m = length of the argument (substring, prefix, ...),
k = length of the result.

## 5.1 Complete reference

**Mutates? Never.** `str` is immutable, so every method returns a new string, a
number, a bool, a list or a tuple. The column is kept for consistency with the other
reference chapters.

Version column: "all" means available in every Python 3 version this book supports.
Version facts are from the `stdtypes` docs.

### Case

| Method | Signature → returns | Mutates? | Time (reason) | Tier | Min version |
|---|---|---|---|---|---|
| `lower` | `s.lower() → str` | No | O(n): one pass, new string | **[USE-FREELY]** | all |
| `upper` | `s.upper() → str` | No | O(n): one pass | **[USE-FREELY]** | all |
| `casefold` | `s.casefold() → str` | No | O(n): one pass | **[USE-FREELY]** | 3.3 |
| `capitalize` | `s.capitalize() → str` | No | O(n): first char titlecased, rest lowered | **[KNOW-HOW]** | all (behavior changed 3.8) |
| `title` | `s.title() → str` | No | O(n): one pass | **[KNOW-HOW]** | all |
| `swapcase` | `s.swapcase() → str` | No | O(n): one pass | **[KNOW-HOW]** | all |

### Search

| Method | Signature → returns | Mutates? | Time (reason) | Tier | Min version |
|---|---|---|---|---|---|
| `find` | `s.find(sub[, start[, end]]) → int` (-1 if absent) | No | O(n + m) worst case: CPython's forward search uses a two-way algorithm (time-complexity docs; measured flat on an adversarial input, Ch 12) | **[KNOW-HOW]** | all |
| `rfind` | `s.rfind(sub[, start[, end]]) → int` | No | O(n) typical, O(n · m) worst case (per the docs) | **[KNOW-HOW]** | all |
| `index` | `s.index(sub[, start[, end]]) → int` | No | Same as `find`; raises `ValueError` if absent | **[KNOW-HOW]** | all |
| `rindex` | `s.rindex(sub[, start[, end]]) → int` | No | Same as `rfind`; raises `ValueError` | **[KNOW-HOW]** | all |
| `count` | `s.count(sub[, start[, end]]) → int` | No | Same bound as `find`; counts **non-overlapping** matches | **[KNOW-HOW]** | all |
| `startswith` | `s.startswith(prefix[, start[, end]]) → bool` | No | O(m): compares at most m chars | **[USE-FREELY]** | all |
| `endswith` | `s.endswith(suffix[, start[, end]]) → bool` | No | O(m) | **[USE-FREELY]** | all |

`startswith` and `endswith` also accept a **tuple** of options:
`s.startswith(("http", "ftp"))`.

### Testing

All return `bool` and return **`False` for the empty string**. Time is O(n) in the
worst case, because every character is checked (the scan stops at the first
failure).

| Method | True when every character is... | Mutates? | Tier | Min version |
|---|---|---|---|---|
| `isalpha` | a letter (any script) | No | **[USE-FREELY]** | all |
| `isdigit` | a digit, **including** superscripts like `²` | No | **[USE-FREELY]** | all |
| `isdecimal` | a decimal digit (Unicode category Nd, any script) | No | **[USE-FREELY]** | all |
| `isnumeric` | numeric, including fractions like `½` and Roman numerals like `Ⅻ` | No | **[USE-FREELY]** | all |
| `isalnum` | `isalpha() or isdecimal() or isdigit() or isnumeric()` | No | **[USE-FREELY]** | all |
| `isspace` | whitespace (space, `\t`, `\n`, `\r`, `\v`, `\f`, Unicode spaces) | No | **[USE-FREELY]** | all |
| `islower` | lowercase, and there's at least one cased character | No | **[USE-FREELY]** | all |
| `isupper` | uppercase, and there's at least one cased character | No | **[USE-FREELY]** | all |
| `istitle` | in title case (`"Hello World"`) | No | **[USE-FREELY]** | all |
| `isascii` | ASCII (code point < 128). **True for `""`** | No | **[USE-FREELY]** | 3.7 |
| `isidentifier` | a valid Python identifier, as a whole | No | **[USE-FREELY]** | all |
| `isprintable` | printable. **True for `""`** | No | **[USE-FREELY]** | all |

### Split and join

| Method | Signature → returns | Mutates? | Time (reason) | Tier | Min version |
|---|---|---|---|---|---|
| `split` | `s.split(sep=None, maxsplit=-1) → list[str]` | No | O(n): one scan, plus copying the pieces | **[KNOW-HOW]** | all |
| `rsplit` | `s.rsplit(sep=None, maxsplit=-1) → list[str]` | No | O(n); differs from `split` only with `maxsplit` | **[KNOW-HOW]** | all |
| `splitlines` | `s.splitlines(keepends=False) → list[str]` | No | O(n) | **[KNOW-HOW]** | all |
| `partition` | `s.partition(sep) → tuple[str, str, str]` | No | O(n): search, then two copies | **[KNOW-HOW]** | all |
| `rpartition` | `s.rpartition(sep) → tuple[str, str, str]` | No | O(n) | **[KNOW-HOW]** | all |
| `join` | `sep.join(iterable_of_str) → str` | No | O(total length): sizes the result, then copies each piece once | **[USE-FREELY]** | all |

### Trim and pad

| Method | Signature → returns | Mutates? | Time (reason) | Tier | Min version |
|---|---|---|---|---|---|
| `strip` | `s.strip(chars=None) → str` | No | O(n): scan both ends, copy the middle | **[USE-FREELY]** | all |
| `lstrip` | `s.lstrip(chars=None) → str` | No | O(n) | **[USE-FREELY]** | all |
| `rstrip` | `s.rstrip(chars=None) → str` | No | O(n) | **[USE-FREELY]** | all |
| `removeprefix` | `s.removeprefix(prefix) → str` | No | O(n): compare m chars, copy the rest | **[KNOW-HOW]** | 3.9 |
| `removesuffix` | `s.removesuffix(suffix) → str` | No | O(n) | **[KNOW-HOW]** | 3.9 |
| `center` | `s.center(width, fillchar=" ") → str` | No | O(max(n, width)) | **[KNOW-HOW]** | all |
| `ljust` | `s.ljust(width, fillchar=" ") → str` | No | O(max(n, width)) | **[KNOW-HOW]** | all |
| `rjust` | `s.rjust(width, fillchar=" ") → str` | No | O(max(n, width)) | **[KNOW-HOW]** | all |
| `zfill` | `s.zfill(width) → str` | No | O(max(n, width)); keeps a leading sign first | **[KNOW-HOW]** | all |
| `expandtabs` | `s.expandtabs(tabsize=8) → str` | No | O(n + k) | **[USE-FREELY]** | all |

### Transform

| Method | Signature → returns | Mutates? | Time (reason) | Tier | Min version |
|---|---|---|---|---|---|
| `replace` | `s.replace(old, new, count=-1) → str` | No | O(n + k) plus the search for `old` | **[KNOW-HOW]** | all; `count=` as keyword 3.13 |
| `translate` | `s.translate(table) → str` | No | O(n): one table lookup per char | **[KNOW-HOW]** | all |
| `maketrans` | `str.maketrans(x[, y[, z]]) → dict` (static) | No | O(len of arguments) | **[KNOW-HOW]** | all |
| `encode` | `s.encode(encoding="utf-8", errors="strict") → bytes` | No | O(n) | **[USE-FREELY]** | all |
| `format` | `s.format(*args, **kwargs) → str` | No | O(k) | **[USE-FREELY]** | all |
| `format_map` | `s.format_map(mapping) → str` | No | O(k) | **[USE-FREELY]** | 3.2 |

### Operations that aren't methods

| Operation | Example | Returns | Time (reason) | Tier |
|---|---|---|---|---|
| Index | `s[i]`, `s[-1]` | 1-char `str` | O(1): fixed-width internal storage (CPython) | **[USE-FREELY]** |
| Slice | `s[i:j]`, `s[::-1]` | `str` | O(k): copies k chars | **[USE-FREELY]** |
| Concatenate | `s + t` | `str` | O(n + m): new string | **[USE-FREELY]** |
| Repeat | `s * r` | `str` | O(n · r) | **[USE-FREELY]** |
| Membership | `sub in s` | `bool` | Same as `find` | **[USE-FREELY]** |
| Compare | `s < t`, `s == t` | `bool` | O(min(n, m)): stops at the first difference | **[USE-FREELY]** |
| Length | `len(s)` | `int` | O(1): stored | **[USE-FREELY]** |
| Iterate | `for c in s` | 1-char strings | O(n) | **[USE-FREELY]** |
| Code point | `ord(c)`, `chr(i)` | `int`, `str` | O(1) | **[USE-FREELY]** |
| Format | `f"{x:>5}"` (3.6+), `f"{x=}"` (3.8+) | `str` | O(k) | **[USE-FREELY]** |
| Sort characters | `sorted(s)` | **`list[str]`**, not `str` | O(n log n): Timsort | **[KNOW-HOW]** |
| Reverse | `s[::-1]`, `"".join(reversed(s))` | `str` | O(n) | **[KNOW-HOW]** |

### `string` module constants

| Constant | Value |
|---|---|
| `string.ascii_lowercase` | `'abcdefghijklmnopqrstuvwxyz'` |
| `string.ascii_uppercase` | `'ABCDEFGHIJKLMNOPQRSTUVWXYZ'` |
| `string.ascii_letters` | lowercase + uppercase |
| `string.digits` | `'0123456789'` |
| `string.hexdigits` | `'0123456789abcdefABCDEF'` |
| `string.octdigits` | `'01234567'` |
| `string.punctuation` | ``'!"#$%&\'()*+,-./:;<=>?@[\\]^_`{|}~'`` (32 characters) |
| `string.whitespace` | `' \t\n\r\x0b\x0c'` |
| `string.printable` | digits + letters + punctuation + whitespace |

The module also has `string.capwords(s)` (title-cases words split on whitespace,
5.2.1).

## 5.2 Explanations by group

### 5.2.1 Case

```python
"Hello World".lower()  # -> 'hello world'
"ß".lower()  # -> 'ß'
"ß".casefold()  # -> 'ss'
"ß".upper()  # -> 'SS'
```

`casefold` is the aggressive lowercase for **caseless comparison** across languages.
For LeetCode inputs (usually ASCII) `lower()` is enough. Note that `upper()` can change
the length.

`title()` capitalizes after **any** non-letter, apostrophes included.
`string.capwords` splits on whitespace instead:

```python
import string

"they're bill's".title()  # -> "They'Re Bill'S"
string.capwords("they're bill's")  # -> "They're Bill's"
"hello WORLD".capitalize()  # -> 'Hello world'
```

### 5.2.2 Search: `find` vs `index` vs `in`

| Need | Use | If absent |
|---|---|---|
| Is it there? | `sub in s` | `False` |
| Where is it, and absence is normal | `s.find(sub)` | `-1` |
| Where is it, and absence is a bug | `s.index(sub)` | `ValueError` |

```python
s = "abc"
s.find("z")  # -> -1
s.index("z")  # -> ValueError: substring not found
s[s.find("z")]  # -> 'c'
```

The last line is the `-1` trap: `-1` is a valid index, so a forgotten check quietly
reads the last character. Always test `!= -1` before using the result of `find`.

`count` counts **non-overlapping** occurrences. For overlapping ones, loop with
`find(sub, start + 1)` (Exercise 5):

```python
"aaaa".count("aa")  # -> 2
"abc".count("")  # -> 4
```

The empty string "occurs" at every one of the n + 1 boundaries. With `start`/`end`
arguments, `s.find(sub, i)` searches from index `i` **without** slicing, which saves an
O(n) copy per call.

### 5.2.3 Testing: `isdigit` vs `isdecimal` vs `isnumeric`

The docs define three widening sets: decimal ⊂ digit ⊂ numeric. All three are
Unicode-aware, which means they accept more than `0`–`9`:

| Character | `isdecimal` | `isdigit` | `isnumeric` | `int(c)` |
|---|---|---|---|---|
| `"7"` | True | True | True | 7 |
| `"²"` (superscript two) | False | **True** | True | `ValueError` |
| `"½"` (vulgar fraction) | False | False | **True** | `ValueError` |
| `"٣"` (Arabic-Indic three) | **True** | True | True | **3** |
| `"-1"` | False | False | False | -1 |
| `""` | False | False | False | `ValueError` |

```python
"²".isdigit()  # -> True
int("٣")  # -> 3
"-1".isdigit()  # -> False
```

Consequences:

- **`isdigit()` does not guarantee that `int()` will succeed.** `isdecimal()` is the
  method that matches what `int()` accepts as a digit.
- **None of them accept a sign.** `"-1".isdigit()` is `False`; atoi-style parsers
  (Ch 28) handle the sign themselves.
- **For "is this an ASCII digit", write `"0" <= c <= "9"`.** It's exact, and it's what an
  interviewer expects in a hand-written parser. LeetCode inputs are usually ASCII,
  where all four checks agree, but the explicit comparison states your intent.

The same applies to letters: `isalpha()` accepts `"é"` and `"ж"`. For "ASCII letter",
use `"a" <= c <= "z"` (or `c in string.ascii_letters`).

`isalnum()` is the go-to filter for "letters and digits" problems such as 125 Valid
Palindrome (Ch 26):

```python
"".join(c.lower() for c in "A man, a plan!" if c.isalnum())  # -> 'amanaplan'
```

### 5.2.4 Split and join

**`split()` vs `split(" ")`.** With no argument, `split` treats any run of whitespace
as one separator and drops leading and trailing whitespace. With an explicit
separator, every single occurrence splits, so repeated separators produce empty
strings:

```python
s = "  a  b "
s.split()  # -> ['a', 'b']
s.split(" ")  # -> ['', '', 'a', '', 'b', '']
"1,,2".split(",")  # -> ['1', '', '2']
```

151 Reverse Words in a String (Ch 24) has exactly this input shape. `split()` solves the
spacing for you; `split(" ")` gives you empty strings to filter out.

`maxsplit` limits the number of splits. `rsplit` differs from `split` only when
`maxsplit` is given:

```python
"a.b.c".split(".", 1)  # -> ['a', 'b.c']
"a.b.c".rsplit(".", 1)  # -> ['a.b', 'c']
```

**`partition` always returns three parts**, and never raises:

```python
"k=v=w".partition("=")  # -> ('k', '=', 'v=w')
"abc".partition("z")  # -> ('abc', '', '')
```

**`splitlines` vs `split("\n")`.** `splitlines` recognizes `\r\n` and other line
boundaries, and doesn't produce a trailing empty string:

```python
"a\nb\r\nc".splitlines()  # -> ['a', 'b', 'c']
"a\nb\n".split("\n")  # -> ['a', 'b', '']
```

**`join` is called on the separator** and needs strings:

```python
"-".join(["a", "b", "c"])  # -> 'a-b-c'
"".join(sorted("bca"))  # -> 'abc'
"".join([1, 2])  # -> TypeError: sequence item 0: expected str instance, int found
```

Convert first: `"".join(map(str, nums))` or `"".join(str(x) for x in nums)`.

### 5.2.5 Trim and pad

**`strip(chars)` removes any of those *characters*, not a prefix string:**

```python
"xxhixx".strip("x")  # -> 'hi'
"abcba".strip("ab")  # -> 'c'
"abcba".removeprefix("ab")  # -> 'cba'
```

`strip("ab")` keeps removing while the end character is in `{'a', 'b'}`. To remove an
exact prefix or suffix once, use `removeprefix`/`removesuffix` (3.9+).

Padding:

```python
"x".center(5, "*")  # -> '**x**'
"42".zfill(5)  # -> '00042'
"-42".zfill(5)  # -> '-0042'
f"{7:03d}"  # -> '007'
f"{'x':>3}|"  # -> '  x|'
f"{255:b}"  # -> '11111111'
```

`zfill` keeps the sign in front, which is exactly what you want for numbers. The
f-string format spec (`:03d`, `:>3`, `:b`) handles most formatting needs in one place.

### 5.2.6 Transform

```python
"banana".replace("a", "o")  # -> 'bonono'
"banana".replace("a", "o", 1)  # -> 'bonana'
```

`replace(old, new, count=1)` with `count` as a keyword works only on 3.13+. The
positional form works everywhere.

**`maketrans` + `translate`** map many characters in one O(n) pass. The third argument
lists characters to delete:

```python
table = str.maketrans("abc", "xyz", "!")
"a!b!c".translate(table)  # -> 'xyz'
```

That's the idiom for Caesar shifts and character substitutions (Exercise 8). It's
**[KNOW-HOW]**: the hand-written version is a loop with
`chr((ord(c) - ord("a") + k) % 26 + ord("a"))` (Ch 24).

### 5.2.7 Immutability and building strings

Strings can't change in place:

```python
s = "abc"
s[0] = "x"  # -> TypeError: 'str' object does not support item assignment
```

So `s += c` must, by the language rules, create a new string of length n + 1, which
means copying n characters. Doing that n times is O(n²). CPython has an
**implementation-specific optimization**: when nothing else references `s`, it may
resize the string in place instead. That optimization is not part of the language,
and it disappears as soon as another reference to the string exists.

Measured in `bench/ch05_string_concat.py` (CPython 3.13.2, best of 5, two runs within
10% of each other):

| Building N characters | N = 10,000 | 20,000 | 40,000 | Growth when N doubles |
|---|---|---|---|---|
| `s += "x"` | 0.7 ms | 1.3 ms | 2.8 ms | ~2× (linear: the optimization applied) |
| `s += "x"` while another name refers to `s` | 2.2 ms | 5.3 ms | 20 ms | → ~4× (quadratic) |
| `parts.append("x")`, then `"".join(parts)` | 0.24 ms | 0.47 ms | 1.1 ms | ~2× (linear) |

What this shows:

1. `+=` was linear here **only because of the optimization**, and it was still about
   2.5× slower than `join`.
2. One extra reference (here a variable `alias = s` inside the loop) made `+=`
   quadratic. At 40,000 characters it was ~19× slower than `join`, and the gap grows
   with N.
3. **`list` + `"".join` is linear by construction**, on every Python implementation.

**Rule: build strings with a list and `"".join`.** Use `+` for a fixed, small number
of pieces (`left + mid + right`), never in a loop over the input.

## 5.3 Gotchas & traps

**1. `split(" ")` on irregular spacing** produces empty strings (5.2.4).

**2. `find` returns -1, and -1 is a valid index** (5.2.2).

**3. `isdigit()` accepts `"²"`, which `int()` rejects** (5.2.3). Use `"0" <= c <= "9"`.

**4. `strip("ab")` strips characters, not the prefix "ab"** (5.2.5).

**5. `sorted(s)` returns a list.** `"".join(sorted(s))` for a string. This is the anagram
signature (Ch 14).

**6. Lexicographic comparison of digit strings:**

```python
"10" < "9"  # -> True
"apple" < "app"  # -> False
"Z" < "a"  # -> True
```

Comparison goes character by character by code point. `"1"` < `"9"` decides the first
case. A prefix is smaller than the longer string. All uppercase ASCII letters come
before all lowercase ones. Convert numbers with `int()` before comparing them.

**7. `s += c` in a loop** (5.2.7). Use a list and `join`.

**8. `title()` and apostrophes** (5.2.1).

**9. Out-of-range index vs slice:**

```python
"abc"[10:]  # -> ''
"abc"[10]  # -> IndexError: string index out of range
```

Slices clamp to the string's bounds and never raise; indexing raises.

**10. Empty-string truth values for `is*`.** All `is*` methods return `False` for `""`,
except `isascii()` and `isprintable()`, which return `True`. Check `if s and ...`
when emptiness matters.

## 5.4 Benchmarks

| File | Measures | Result (CPython 3.13.2) |
|---|---|---|
| `bench/ch05_string_concat.py` | `+=` vs `+=` with an alias vs `list` + `join`, for N = 10k, 20k, 40k | `join` fastest and linear; `+=` linear only while unaliased, and quadratic with an alias (5.2.7) |

## 5.5 Exercises

Answers are in Appendix D (verified in `code/ch05_str/exercises.py`).

**Warm-up**

1. What do `"  hello   world  ".split()` and `.split(" ")` return?
2. What does `"abc"["abc".find("z")]` evaluate to, and why is that dangerous?
3. Which check accepts exactly the ASCII digits `0`–`9`: `isdigit`, `isdecimal`,
   `isnumeric`, or something else?
4. Reduce `"A man, a plan, a canal: Panama"` to lowercase letters and digits only.

**Core**

5. `"aaaa".count("aa")` returns 2. Write `count_overlapping(s, sub)` that returns 3.
6. Reverse the word order of `" the sky  is blue "`, collapsing the spaces:
   `"blue is sky the"`. Use built-ins; Ch 24 does it by hand.
7. Format the integer 7 as `"007"`, in two ways.
8. Shift every lowercase letter forward by one (`z` wraps to `a`) using `maketrans`
   and `translate`: `"abz"` → `"bca"`.
9. What do `"k=v=w".partition("=")` and `.rpartition("=")` return?
10. Write `build(n)`, which returns the first n letters of `"abc...zabc..."` in O(n)
    time. Explain why `s += c` is not guaranteed to be O(n) overall.

**Tricky**

11. What does `"they're bill's".title()` return? How do you get `"They're Bill's"`?
12. What does `"abcba".strip("ab")` return? What about `.removeprefix("ab")`?
13. What type does `sorted("bca")` return, and how do you get `"abc"`?
14. Is `"10" < "9"` true? Why?

## 5.6 Chapter cheat sheet

| Task | Write | Avoid |
|---|---|---|
| Words from text | `s.split()` | `s.split(" ")` |
| Substring present? | `sub in s` | `s.find(sub) >= 0` (fine, but noisier) |
| Position, may be absent | `i = s.find(sub)`, then check `i != -1` | `s[s.find(sub)]` |
| ASCII digit / letter | `"0" <= c <= "9"`, `"a" <= c <= "z"` | `isdigit()` when Unicode matters |
| Letters and digits | `c.isalnum()` | |
| Remove an exact prefix | `s.removeprefix(p)` (3.9+) | `s.lstrip(p)` |
| Build a string | `parts.append(...)`, then `"".join(parts)` | `s += c` in a loop |
| Sorted characters | `"".join(sorted(s))` | `sorted(s)` as if it were a string |
| Many char substitutions | `s.translate(str.maketrans(...))` | Chained `replace` calls |
| Pad a number | `str(x).zfill(w)`, `f"{x:0{w}d}"` | Manual loops |
| Costs | Index O(1), slice O(k), `+` O(n + m), compare O(min), `len` O(1) | |
