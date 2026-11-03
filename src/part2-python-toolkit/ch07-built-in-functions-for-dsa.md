# Chapter 7: Built-in Functions for DSA

## 7.0 At a glance

Thirty-five built-in functions cover nearly everything an array or string solution
calls. For each one this chapter gives the signature, what it returns (**an iterator or
a list** is the distinction that bites), its cost, its tier, and its most common
misuse.

The three facts that cause the most bugs:

1. **`map`, `filter`, `zip`, `enumerate` and `reversed` return one-shot iterators, not
   lists.** Only `sorted` returns a list.
2. **`round` rounds ties to even:** `round(2.5) == 2`.
3. **`int()` is more permissive than a LeetCode parser:** it accepts `" 42 "`, `"+7"`
   and `"4_2"`, so it can't validate input for problems like 8 String to Integer.

Notation: n = length of the input iterable.

## 7.1 Complete reference

"Iterator" in the Returns column means a lazy, single-use object (Ch 4). **Mutates?** is
"No" for every function here: built-ins don't change their arguments, though consuming
an iterator does exhaust it.

### Size and iteration

| Function | Signature | Returns | Time (reason) | Tier | Min version / note |
|---|---|---|---|---|---|
| `len` | `len(obj)` | `int` | O(1) for built-in containers: the size is stored | **[USE-FREELY]** | |
| `range` | `range(stop)`, `range(start, stop[, step])` | `range` (lazy sequence) | O(1) to create; `x in r` O(1) for ints | **[USE-FREELY]** | |
| `enumerate` | `enumerate(it, start=0)` | iterator of `(i, x)` | O(1) per item | **[USE-FREELY]** | |
| `zip` | `zip(*its, strict=False)` | iterator of tuples | O(1) per tuple of k items | **[USE-FREELY]** | `strict` 3.10 |
| `reversed` | `reversed(seq)` | iterator | O(1) to create, O(n) to consume | **[KNOW-HOW]** | dicts and dict views 3.8 |
| `iter` | `iter(obj)` | iterator | O(1) | **[USE-FREELY]** | |
| `next` | `next(it[, default])` | the next item | O(1) plus the producer's work | **[USE-FREELY]** | |
| `slice` | `slice(start, stop[, step])` | `slice` object | O(1) | **[USE-FREELY]** | |

### Ordering and aggregation

| Function | Signature | Returns | Time (reason) | Tier | Min version / note |
|---|---|---|---|---|---|
| `sorted` | `sorted(it, *, key=None, reverse=False)` | **`list`** | O(n log n): Timsort; O(n) auxiliary | **[KNOW-HOW]** | |
| `min` / `max` | `min(it, *, key=None, default=...)`, `min(a, b, ...)` | an element | O(n): one pass | **[KNOW-HOW]** | `default` 3.4; `key=None` allowed 3.8 |
| `sum` | `sum(it, start=0)` | number (or `start`'s type) | O(n) for numbers | **[USE-FREELY]** | `start=` keyword 3.8; more accurate float sum 3.12 |
| `any` | `any(it)` | `bool` | O(k): stops at the first truthy item | **[USE-FREELY]** | |
| `all` | `all(it)` | `bool` | O(k): stops at the first falsy item | **[USE-FREELY]** | |
| `map` | `map(f, *its)` | iterator | O(1) per item plus the cost of `f` | **[USE-FREELY]** | |
| `filter` | `filter(f_or_None, it)` | iterator | O(1) per item plus the cost of `f` | **[USE-FREELY]** | |

### Arithmetic

| Function | Signature | Returns | Time (reason) | Tier | Min version / note |
|---|---|---|---|---|---|
| `abs` | `abs(x)` | number | O(1) for word-size ints | **[USE-FREELY]** | |
| `divmod` | `divmod(a, b)` | `(a // b, a % b)` | O(1) for word-size ints | **[USE-FREELY]** | Floor semantics (Ch 3) |
| `pow` | `pow(base, exp[, mod])` | number | O(log exp) multiplications (square-and-multiply) | **[KNOW-HOW]** | Negative `exp` with `mod` (modular inverse) 3.8 |
| `round` | `round(x[, ndigits])` | `int` (no ndigits) or `float` | O(1) | **[USE-FREELY]** | Ties to even |

### Conversion and representation

| Function | Signature | Returns | Time (reason) | Tier | Min version / note |
|---|---|---|---|---|---|
| `int` | `int(x)`, `int(s, base=10)` | `int` | O(d) for d digits in bases that are powers of two; base 10 is non-linear and limited to 4300 digits (3.11, Ch 3) | **[KNOW-HOW]** | `_` separators 3.6 |
| `bin` / `oct` / `hex` | `bin(n)` | `str` with a `0b`/`0o`/`0x` prefix | O(d) | **[KNOW-HOW]** | |
| `ord` | `ord(c)` | `int` code point | O(1) | **[USE-FREELY]** | |
| `chr` | `chr(i)` | 1-char `str` | O(1) | **[USE-FREELY]** | |
| `str` | `str(x)` | `str` | O(size of the result) | **[USE-FREELY]** | |
| `hash` | `hash(x)` | `int` | O(size of x): hashes every character or element | **[USE-FREELY]** | Raises for lists, dicts, sets |

### Containers

| Function | Signature | Returns | Time (reason) | Tier |
|---|---|---|---|---|
| `list` | `list(it)` | new `list` | O(n): one pass | **[USE-FREELY]** |
| `tuple` | `tuple(it)` | new `tuple` | O(n) | **[USE-FREELY]** |
| `set` | `set(it)` | new `set` | O(n) average: n hash inserts | **[USE-FREELY]** |
| `frozenset` | `frozenset(it)` | new `frozenset` (hashable) | O(n) average | **[USE-FREELY]** |
| `dict` | `dict(pairs)`, `dict(**kw)`, `dict(zip(keys, values))` | new `dict` | O(n) average | **[USE-FREELY]** |

### Introspection

| Function | Signature | Returns | Time | Tier |
|---|---|---|---|---|
| `isinstance` | `isinstance(x, cls_or_tuple)` | `bool` | O(1) for one class | **[USE-FREELY]** |
| `id` | `id(x)` | `int` identity (memory address in CPython) | O(1) | **[USE-FREELY]** |

## 7.2 Explanations by group

### 7.2.1 Iterators vs lists

```python
type(map(str, [1, 2])).__name__  # -> 'map'
type(sorted([2, 1])).__name__  # -> 'list'
doubled = map(lambda x: 2 * x, [1, 2])
list(doubled)  # -> [2, 4]
list(doubled)  # -> []
```

| Returns a list | Returns a lazy object |
|---|---|
| `sorted`, `list(...)`, comprehensions | `map`, `filter`, `zip`, `enumerate`, `reversed`, `iter`, generator expressions, `range` (a re-iterable sequence, not single-use) |

To use a lazy result twice, or to index it, wrap it in `list(...)` once. `len(map(...))`
raises `TypeError`. A `range` is the exception: it's a real sequence, so it supports
`len`, indexing and repeated iteration.

### 7.2.2 `range`, `enumerate`, `zip`, `reversed`

```python
r = range(0, 10, 3)
list(r)  # -> [0, 3, 6, 9]
len(r)  # -> 4
r[-1]  # -> 9
list(enumerate("ab", start=1))  # -> [(1, 'a'), (2, 'b')]
dict(zip("ab", [1, 2]))  # -> {'a': 1, 'b': 2}
list(reversed({"a": 1, "b": 2}))  # -> ['b', 'a']
```

**`x in range(...)` is O(1) only for ints.** For an int, `range` checks the bounds and the
step with arithmetic. For anything else, including a float, it falls back to scanning
every element. Measured with `range(10**6)` (`bench/ch07_range_membership.py`, CPython
3.13.2): **0.3–0.7 µs** per int check vs **156–174 ms** per float check.

```python
10**9 in range(10**10)  # -> True
5.0 in range(10)  # -> True
```

The first line is instant despite the 10¹⁰-element range. The second scans, and it
finds `5`, because `5.0 == 5`.

`zip` stops at the shortest input. Pass `strict=True` (3.10+) when the lengths must match
(Ch 3, 3.2.10).

### 7.2.3 `sorted`, `min`, `max` with `key`

`key` maps each item to the value that gets compared. `min` and `max` return the
**item**, not the key:

```python
counts = {"a": 3, "b": 5, "c": 5}
max(counts, key=counts.get)  # -> 'b'
max("apple", "banana", key=len)  # -> 'banana'
min([3, 1, 2], key=lambda x: -x)  # -> 3
```

`max(counts, key=counts.get)` is the "key with the largest value" idiom. On ties,
`max` and `min` return the **first** maximal or minimal item they meet.

Tuples compare element by element, which gives tie-breaking for free:

```python
min((2, "b"), (2, "a"))  # -> (2, 'a')
```

**Empty input raises**, unless you pass `default` (3.4+):

```python
max([])  # -> ValueError: max() iterable argument is empty
max([], default=0)  # -> 0
```

The message above is from 3.13. Python 3.11 words it as
`max() arg is an empty sequence`, so don't match on the text.

`sorted` accepts any iterable and always returns a list:

```python
sorted("cba")  # -> ['a', 'b', 'c']
sorted({"b": 1, "a": 2})  # -> ['a', 'b']
```

Sorting a dict sorts its **keys**. Use `sorted(d.items(), key=...)` for pairs.

### 7.2.4 `sum`, `any`, `all`

```python
sum([1, 2], start=10)  # -> 13
sum([True, False, True])  # -> 2
isinstance(True, int)  # -> True
```

`bool` is a subclass of `int`, so summing booleans counts the `True`s.

**`sum` refuses strings** and points you to the right tool:

```python
sum(["a", "b"], "")  # -> TypeError: sum() can't sum strings [use ''.join(seq) instead]
```

**`sum(list_of_lists, [])` works but is O(total²).** Each `+` builds a new list and copies
everything accumulated so far. Flatten with a comprehension or
`itertools.chain.from_iterable` instead (O(total)):

```python
from itertools import chain

parts = [[1], [2, 3], [4]]
sum(parts, [])  # -> [1, 2, 3, 4]
list(chain.from_iterable(parts))  # -> [1, 2, 3, 4]
```

**Float sums changed in 3.12.** The docs: "Summation of floats switched to an algorithm
that gives higher accuracy and better commutativity on most builds." Run on this
book's machine:

| Expression | Python 3.11.4 | Python 3.13.2 |
|---|---|---|
| `sum([0.1] * 10)` | `0.9999999999999999` | `1.0` |

Don't compare float sums with `==`, on any version (Ch 3, 3.2.7).

`any([])` is `False` and `all([])` is `True` (Ch 4, 4.2.5).

### 7.2.5 Arithmetic: `abs`, `divmod`, `pow`, `round`

```python
divmod(-7, 2)  # -> (-4, 1)
pow(2, 10, 1000)  # -> 24
pow(3, -1, 7)  # -> 5
```

- `divmod` uses **floor** semantics, like `//` and `%` (Ch 3, 3.2.6).
- **`pow(base, exp, mod)` computes `base**exp % mod` without building the huge
  intermediate.** It uses O(log exp) multiplications by repeated squaring. This is the
  tool for "return the answer modulo 10⁹ + 7". It's **[KNOW-HOW]**: be ready to write
  square-and-multiply by hand.
- `pow(a, -1, m)` (3.8+) is the modular inverse: `3 * 5 % 7 == 1`.

**`round` uses banker's rounding: ties go to the even neighbor.**

```python
round(2.5)  # -> 2
round(3.5)  # -> 4
round(-0.5)  # -> 0
round(2.675, 2)  # -> 2.67
```

The last one is a float-representation effect, not a tie: `2.675` is stored as
2.67499999... (the docs give this exact example). For "round half up" on
non-negative numbers, use `int(x + 0.5)`. For integer division rounded up, use
`(a + b - 1) // b` with positive ints, or `-(-a // b)`.

### 7.2.6 `int`, `bin`, `hex`, `oct`, `ord`, `chr`

`int(s, base)` parses; `bin`/`oct`/`hex` format with a prefix:

```python
int("ff", 16)  # -> 255
int("0b101", 0)  # -> 5
bin(5)  # -> '0b101'
bin(5)[2:]  # -> '101'
format(5, "b")  # -> '101'
bin(-5)  # -> '-0b101'
hex(255)  # -> '0xff'
```

Base 0 means "infer the base from the prefix". `bin` of a negative number keeps the
minus sign; it does **not** produce two's complement. Bit problems that need 32-bit
two's complement mask with `x & 0xFFFFFFFF`.

**`int()` accepts more than LeetCode parsers allow:**

| Input | `int(...)` | Typical problem rule |
|---|---|---|
| `" 42 "` | 42 | Often allowed (leading spaces only) |
| `"+7"` | 7 | Allowed in atoi |
| `"4_2"` | 42 (3.6+) | Invalid |
| `"1e3"` | `ValueError` | Depends on the problem |
| `"3.0"` | `ValueError` | |
| `3.9` (a float) | 3 (truncates toward zero) | |

```python
int(" 42 ")  # -> 42
int("4_2")  # -> 42
int(-3.9)  # -> -3
int("1e3")  # -> ValueError: invalid literal for int() with base 10: '1e3'
```

For parsing problems (Ch 28) you write the character loop yourself. Calling `int()`
on a substring is the shortcut interviewers forbid.

**`ord`/`chr` map characters to code points and back.** The 26-letter index trick
(Ch 24):

```python
ord("c") - ord("a")  # -> 2
chr(ord("a") + 25)  # -> 'z'
ord("A") - ord("a")  # -> -32
```

### 7.2.7 Constructors and `hash`

```python
list("ab")  # -> ['a', 'b']
tuple([1])  # -> (1,)
sorted(set("aab"))  # -> ['a', 'b']
frozenset([1]) | {2}  # -> frozenset({1, 2})
dict([("a", 1)])  # -> {'a': 1}
```

`set(...)` iteration order is arbitrary. For strings it can change between runs,
because string hashing is randomized per process (see `PYTHONHASHSEED` in the docs).
Sort a set before printing it or comparing it in order. Dicts keep **insertion order**,
which has been guaranteed by the language since 3.7.

`hash` works only on immutable (hashable) objects:

```python
hash((1, 2)) == hash((1, 2))  # -> True
hash([1])  # -> TypeError: unhashable type: 'list'
```

To use a list as a dict key or set element, convert it: `tuple(lst)`. To use a set as a
key, use `frozenset(s)`.

### 7.2.8 `iter`, `next`, `slice`, `isinstance`, `id`

```python
it = iter([10, 20])
next(it)  # -> 10
next(it)  # -> 20
next(it, "done")  # -> 'done'
next(iter([]))  # -> StopIteration
```

`next(gen, default)` is the clean way to find a first match:

```python
nums = [3, 8, 5]
next((x for x in nums if x % 2 == 0), None)  # -> 8
```

`slice` objects let you name a slice: `middle = slice(1, -1)`, then `a[middle]`.
`isinstance(x, (int, float))` takes a tuple of types. `id(x)` is identity; `a is b` is
`id(a) == id(b)`.

## 7.3 Gotchas & traps

**1. Treating an iterator as a list.** Indexing, `len`, or a second pass fail (7.2.1).

**2. `round(2.5) == 2`.** Banker's rounding (7.2.5).

**3. `max([])` raises.** Use `default=` (7.2.3).

**4. `int()` as a validator.** It accepts spaces, `+`, `_` (7.2.6).

**5. `bin(-5)` isn't two's complement** (7.2.6).

**6. `sum(lists, [])` is quadratic** (7.2.4).

**7. `x in range(...)` with a float is O(n).** Measured 156–174 ms vs under 1 µs for an
int at 10⁶ (7.2.2).

**8. `isinstance(True, int)` is `True`.** Code that branches on "is it an int" also
catches booleans.

**9. Relying on set order** (7.2.7).

**10. `max(d)` returns the largest key, not the key with the largest value.** Use
`max(d, key=d.get)` (7.2.3).

**11. Float `sum` differs between 3.11 and 3.12+** (7.2.4).

## 7.4 Benchmarks

| File | Measures | Result (CPython 3.13.2) |
|---|---|---|
| `bench/ch07_range_membership.py` | `x in range(10**6)` for an int vs a float | 0.3–0.7 µs vs 156–174 ms per check |

Version comparison (not a timing): `sum([0.1] * 10)` on 3.11.4 vs 3.13.2 (7.2.4).

## 7.5 Exercises

Answers are in Appendix D (verified in `code/ch07_builtins/exercises.py`, on 3.13.2
and 3.11.4).

**Warm-up**

1. Which of `map`, `filter`, `zip`, `enumerate`, `reversed`, `range`, `sorted` return a
   list?
2. What does `max([])` do? Make it return 0 instead.
3. For `counts = {"a": 3, "b": 5, "c": 5}`, what does `max(counts, key=counts.get)`
   return?
4. Evaluate `round(2.5)`, `round(3.5)` and `round(2.675, 2)`.

**Core**

5. Why can't `int()` decide whether a string is a valid integer in the sense of 8 String
   to Integer? Give three inputs it accepts that a strict parser might reject, and
   three it rejects.
6. Produce `"101"` from the int 5 in two ways, and parse it back.
7. Compute the inverse of 3 modulo 7 with one built-in call, and verify it.
8. Count the `True` values in `[True, False, True]` in one call. Why does it work?
9. Flatten `[[1], [2, 3], [4]]`. What's wrong with `sum(lists, [])`?
10. Get the next item of an iterator, or `"empty"` if it's exhausted, without
    `try`/`except`.

**Tricky**

11. What is `sum([0.1] * 10)` on Python 3.11, and on 3.13? Why?
12. What does `min((2, "b"), (2, "a"))` return?
13. Is `10**9 in range(10**10)` fast? What about `10**9 + 0.5 in range(10**10)`?
14. `doubled = map(lambda x: 2 * x, [1, 2])`. What are `list(doubled)` and then
    `list(doubled)` again?

## 7.6 Chapter cheat sheet

| Function | Remember |
|---|---|
| `sorted` | Only built-in that returns a list; works on any iterable; stable |
| `map`/`filter`/`zip`/`enumerate`/`reversed` | One-shot iterators |
| `min`/`max` | `key=` returns the item; `default=` for empty; first wins on ties |
| `sum` | Counts bools; refuses strings (use `join`); lists → quadratic; float algorithm improved in 3.12 |
| `any`/`all` | Short-circuit; `any([]) == False`, `all([]) == True` |
| `round` | Ties to even; `2.675` → `2.67` |
| `divmod`, `//`, `%` | Floor semantics |
| `pow(b, e, m)` | Fast modular power; `pow(a, -1, m)` modular inverse (3.8+) |
| `int(s, base)` | Accepts spaces, `+`, `_`; not a validator; 4300-digit limit (3.11+) |
| `bin`/`hex` | Prefixed; `[2:]` or `format(x, "b")` to strip; negatives keep `-` |
| `ord`/`chr` | `ord(c) - ord("a")` → 0..25 |
| `range` | Lazy; `in` O(1) for ints only |
| `hash` | Immutable objects only; `tuple(lst)`, `frozenset(s)` as keys |
