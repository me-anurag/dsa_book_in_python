# Chapter 2: Type Hints, Completely

## 2.0 At a glance

Every LeetCode template carries type hints: `nums: List[int]`, `-> Optional[ListNode]`.
This book writes `nums: list[int]` and `-> ListNode | None`. Both are correct. This
chapter is the only place the book explains why two spellings exist, what each needs,
and what breaks on older Python. Other chapters refer back here.

What you need to take away:

- Type hints are **documentation for humans and tools**. CPython doesn't check them
  when your code runs.
- `List[int]` (from `typing`) and `list[int]` mean the same thing. The capitalized
  form exists for historical reasons.
- `Optional[int]` and `int | None` mean the same thing. **Neither is deprecated.**
- Book rule: **read both, write `list[int]`, never mix the two styles in one file.**

## 2.1 Complete reference

### Old form → modern form

All rows were checked against the `typing` documentation, and run on Python 3.13.2.

| `typing` form | Modern form | Modern form needs | Status of the `typing` form |
|---|---|---|---|
| `List[int]` | `list[int]` | 3.9+ (PEP 585) | Deprecated since 3.9; still works; no removal planned |
| `Dict[str, int]` | `dict[str, int]` | 3.9+ | Deprecated since 3.9; still works |
| `Set[int]` | `set[int]` | 3.9+ | Deprecated since 3.9; still works |
| `FrozenSet[int]` | `frozenset[int]` | 3.9+ | Deprecated since 3.9; still works |
| `Tuple[int, int]` | `tuple[int, int]` | 3.9+ | Deprecated since 3.9; still works |
| `Tuple[int, ...]` | `tuple[int, ...]` | 3.9+ | Same; `...` means "any length" |
| `Deque[int]` | `collections.deque[int]` | 3.9+ | Deprecated since 3.9; still works |
| `DefaultDict[str, int]` | `collections.defaultdict[str, int]` | 3.9+ | Deprecated since 3.9; still works |
| `Counter[str]` | `collections.Counter[str]` | 3.9+ | Deprecated since 3.9; still works |
| `OrderedDict[str, int]` | `collections.OrderedDict[str, int]` | 3.9+ | Deprecated since 3.9; still works |
| `Callable[[int], bool]` | `collections.abc.Callable[[int], bool]` | 3.9+ | Deprecated since 3.9; still works |
| `Iterable[int]`, `Iterator[int]`, `Sequence[int]` | `collections.abc.Iterable[int]`, etc. | 3.9+ | Deprecated since 3.9; still works |
| `Type[Foo]` | `type[Foo]` | 3.9+ | Deprecated since 3.9; still works |
| `Optional[int]` | `int \| None` | 3.10+ (PEP 604) | **Not deprecated.** The `\|` form is a style choice |
| `Union[int, str]` | `int \| str` | 3.10+ (PEP 604) | **Not deprecated.** The docs recommend the `\|` form |
| `Any` | `typing.Any` (no built-in equivalent) | 3.5+ | Current |
| `TypeAlias` (3.10+) | `type Grid = list[list[int]]` | 3.12+ (PEP 695) | `TypeAlias` deprecated since 3.12 |

What "deprecated" means here, from the `typing` docs: the aliases became redundant in
3.9, removal is **not currently planned**, the interpreter issues **no warnings** for
them, and they're guaranteed to stay without warnings until at least 3.14. Type
checkers may flag them in code that targets 3.9+.

### Timeline

| Version | Change | Why it matters on LeetCode |
|---|---|---|
| 3.5 | `typing` module added (PEP 484). Built-in types can't take `[]`, hence `List`, `Dict`, ... | Origin of LeetCode's template style |
| 3.7 | `from __future__ import annotations` (PEP 563): annotations stored as strings, not evaluated | Lets `list[int]` appear in annotations on 3.7 and 3.8 |
| 3.9 | Built-in generics (PEP 585): `list[int]`, `dict[str, int]`, `collections.deque[int]` | The book's style becomes possible without imports |
| 3.10 | Union operator (PEP 604): `int \| None` | The book's `\| None` style becomes possible |
| 3.12 | `type` statement for aliases (PEP 695) | Optional; rarely used in solutions |
| 3.14 | Annotations evaluated lazily by default (PEP 649/749) | Undefined names in annotations no longer fail at `def` time |

The 3.7 date for the future import comes from Python itself:

```python
import __future__

__future__.annotations.optional  # -> (3, 7, 0, 'beta', 1)
```

## 2.2 Explanations by group

### 2.2.1 What type hints are, and what they're not

A hint is an expression attached to a parameter or return value. Python evaluates it
(at `def` time up to 3.13, lazily from 3.14) and stores it. Then it **ignores it**:

```python
def add(a: int, b: int) -> int:
    return a + b


add("x", "y")  # -> 'xy'
add([1], [2])  # -> [1, 2]
add.__annotations__["a"]  # -> <class 'int'>
```

No error. The hints only inform:

| Who reads hints | What they do with them |
|---|---|
| You and the interviewer | Understand the input and output types at a glance |
| Your editor (VS Code, PyCharm) | Autocomplete, inline warnings |
| Static checkers (mypy, pyright) | Report type errors **before** running |
| LeetCode's template | Tells you the input and output types, which are part of the contract (Ch 1) |
| CPython at run time | Nothing |

### 2.2.2 Generic types: the subscript syntax

A **generic** type takes parameters in square brackets. The shapes you'll write most:

| You have | Hint |
|---|---|
| A list of ints | `list[int]` |
| A grid of characters | `list[list[str]]` |
| A dict from character to count | `dict[str, int]` |
| A dict from value to all its indices | `dict[int, list[int]]` |
| A set of visited cells | `set[tuple[int, int]]` |
| A pair, e.g. (row, col) | `tuple[int, int]` |
| A tuple of any length | `tuple[int, ...]` |
| A queue of cells | `collections.deque[tuple[int, int]]` |
| A grouping dict built with `defaultdict(list)` | `collections.defaultdict[str, list[str]]` |
| A key function | `collections.abc.Callable[[str], int]` |
| A value or "nothing" | `int \| None` |

Hints on local variables are optional. Add one when the starting value doesn't tell
the reader the type, as with an empty container:

```python
index_of: dict[int, int] = {}          # value -> index
seen: set[tuple[int, int]] = set()
index_of[5] = 0
seen.add((0, 1))
(index_of, seen)  # -> ({5: 0}, {(0, 1)})
```

At run time, `list[int]` is a real object, a "generic alias". It can be printed and
compared, but it isn't the class `list`:

```python
type(list[int])  # -> <class 'types.GenericAlias'>
list[int] == list  # -> False
isinstance([1, 2], list[int])  # -> TypeError: isinstance() argument 2 cannot...
```

To test the shape of a value at run time, check the container and the elements
separately: `isinstance(x, list) and all(isinstance(v, int) for v in x)`.

### 2.2.3 `Optional`, `Union` and `| None`

`Optional[X]` is defined as `X | None` (the `typing` docs say so verbatim), and the two
compare equal:

```python
from typing import Optional, Union

Optional[int] == (int | None)  # -> True
Union[int, str] == (int | str)  # -> True
```

**`Optional` does not mean "optional argument".** It means "this value may be `None`".
The argument is still required unless it has a default:

```python
def first_or_default(k: int | None) -> int:
    return 0 if k is None else k


first_or_default(None)  # -> 0
first_or_default()  # -> TypeError: first_or_default() missing 1 required...
```

### 2.2.4 What happens on older Python

The modern forms are **expressions**. When Python evaluates an annotation that the
running version doesn't support, the `def` statement itself fails.

| Code | 3.8 | 3.9 | 3.10–3.13 |
|---|---|---|---|
| `def f(x: list[int]): ...` | `TypeError` at `def` time | Works | Works |
| `def f(x: int \| None): ...` | `TypeError` at `def` time | `TypeError` at `def` time | Works |
| Either one, with `from __future__ import annotations` as the file's first statement | Works | Works | Works |

The 3.8 and 3.9 cells come from the version notes in the docs (built-in generics
added in 3.9, `X | Y` in 3.10). This book's machine doesn't have 3.8 or 3.9, so the
exact error messages aren't reproduced here. The *mechanism* can be shown on any
version by annotating with something that isn't subscriptable:

```python
def f(x: int[str]) -> None: pass  # -> TypeError: type 'int' is not subscriptable
```

That is the 3.8 situation: `list` wasn't subscriptable yet, so `list[int]` raised the
same kind of error. (On 3.14+ this `def` succeeds, because annotations are no longer
evaluated at `def` time; it fails later, only if something reads the annotations.)

**Why the future import fixes it.** With `from __future__ import annotations`, Python
stores every annotation as a string and never evaluates it:

```python
source = '''
from __future__ import annotations

def f(x: int[str]) -> list[int]:
    pass
'''
namespace = {}
exec(source, namespace)
namespace["f"].__annotations__  # -> {'x': 'int[str]', 'return': 'list[int]'}
```

The bad annotation is now just text. It costs nothing until a tool like mypy reads
it.

### 2.2.5 Why LeetCode shows `List[int]`

- **The templates predate 3.9.** Many were written when `List[int]` was the only option,
  and they still work. Changing thousands of templates gains nothing.
- **LeetCode pre-imports `typing`.** That's why `List`, `Optional` and the rest work in the
  editor without an import line.
- **Both styles work in LeetCode's Python 3 environment**, as long as its version is
  3.10 or newer for `| None`. LeetCode's Python version changes over time; check the
  language information in LeetCode's help center rather than trusting a number in a
  book. (This book couldn't fetch that page to verify it.)

The trap is the other direction: **code copied from LeetCode fails locally**, because
your file doesn't pre-import anything:

```python
from typing import List   # remove this line and the def below fails on 3.13


def f(nums: List[int]) -> int:
    return len(nums)


f([1, 2])  # -> 2
```

Without the import, Python 3.13 raises `NameError: name 'List' is not defined` at the
`def` line (verified in `code/ch02_type_hints/exercises.py`). On 3.14+, the `def`
succeeds and the error appears only if something evaluates the annotations.

### 2.2.6 The book's rule

> **Read both. Write `list[int]`. Never mix the two styles in one file.**

| Situation | What to write |
|---|---|
| Your own code, Python 3.10+ (this book) | `list[int]`, `dict[str, int]`, `int \| None` |
| Pasting into LeetCode | Either. Modern style works if LeetCode's Python is 3.10+ |
| Editing a LeetCode template in place | Keep its style for the whole file, or convert the whole file |
| Code that must run on 3.8 | `typing` forms, or `from __future__ import annotations` + modern forms |

Each worked problem in this book shows LeetCode's original signature once, then the
book's. For example, from `code/ch02_type_hints/two_styles.py`:

<!-- file: code/ch02_type_hints/two_styles.py -->

```python
class SolutionLeetCodeStyle:
    def twoSum(self, nums: List[int], target: int) -> List[int]:
```

<!-- file: code/ch02_type_hints/two_styles.py -->

```python
class SolutionBookStyle:
    def twoSum(self, nums: list[int], target: int) -> list[int]:
```

### 2.2.7 LeetCode-specific hints: a preview

Linked-list and tree problems are outside this book's scope, but their signatures
appear everywhere, so you should be able to read them. LeetCode puts the node class in
a comment above the template and defines it for you on the judge:

```python
class ListNode:
    def __init__(self, val: int = 0, next: "ListNode | None" = None):
        self.val = val
        self.next = next


def length(head: ListNode | None) -> int:      # LeetCode: Optional[ListNode]
    count = 0
    while head is not None:
        count += 1
        head = head.next
    return count


length(ListNode(1, ListNode(2)))  # -> 2
length(None)  # -> 0
```

| LeetCode writes | Book writes | Meaning |
|---|---|---|
| `head: Optional[ListNode]` | `head: ListNode \| None` | A node, or `None` for an empty list |
| `root: Optional[TreeNode]` | `root: TreeNode \| None` | A node, or `None` for an empty tree |
| `-> Optional[ListNode]` | `-> ListNode \| None` | Return a node, or `None` |

**Why `"ListNode | None"` is quoted inside the class.** While Python is still running the
`class ListNode:` body, the name `ListNode` doesn't exist yet. A string annotation (a
*forward reference*) defers the lookup. On 3.14+, or with the future import, the quotes
aren't needed.

## 2.3 Gotchas & traps

**1. Hints don't validate anything.** `add("x", "y")` returns `'xy'` (2.2.1). A wrong type
fails later, somewhere else, or not at all.

**2. `isinstance` rejects parameterized generics** (2.2.2). Check `list`, then the
elements.

**3. `Optional` ≠ "has a default"** (2.2.3).

**4. Copying LeetCode code without `from typing import List`.** `NameError` locally on
3.13 and earlier (2.2.5).

**5. `Tuple[int]` is a 1-tuple, not "a tuple of ints":**

```python
from typing import Tuple

Tuple[int]  # -> typing.Tuple[int]
tuple[int, ...]  # -> tuple[int, ...]
```

`tuple[int]` means exactly one `int`. For "any number of ints" write
`tuple[int, ...]`. For a list, use `list[int]`.

**6. A list literal is not a type hint.** `def f(x: [int])` runs, because hints aren't
checked, but every type checker rejects it:

```python
def f(x: [int]) -> None:
    pass


f.__annotations__["x"]  # -> [<class 'int'>]
```

Write `list[int]`.

**7. A hint doesn't fix a mutable default.** `def f(acc: list[int] = [])` still shares one
list across calls (Ch 1, Gotcha 5). Use `acc: list[int] | None = None`.

**8. Mixing styles in one file.** `List[int]` in one signature and `list[int]` in the
next is legal and confusing. Pick one per file.

**9. Using `int | None` with the future import, then evaluating it on 3.9.** The future
import only protects annotations. `x = int | None` in ordinary code is still evaluated
and fails on 3.9 (from the docs' version notes; not run here).

## 2.4 Benchmarks

None. This chapter makes no performance claims.

## 2.5 Exercises

Answers are in Appendix D (verified in `code/ch02_type_hints/exercises.py`).

**Warm-up**

1. Rewrite in the book's style:
   `def f(a: List[int], m: Dict[str, List[int]], o: Optional[Tuple[int, int]]) -> Set[str]:`
2. What is the minimum Python version for each of these, with no imports?
   (a) `def f(x: list[int])` (b) `def f(x: int | None)`
   (c) both, with `from __future__ import annotations` as the first statement
3. What does `add("x", "y")` return for `def add(a: int, b: int) -> int: return a + b`?

**Core**

4. What is the difference between `tuple[int]`, `tuple[int, int]` and `tuple[int, ...]`?
5. You paste a LeetCode solution into a local file and run it on Python 3.13. It fails
   with `NameError: name 'List' is not defined`. Give two fixes.
6. Why does `isinstance(x, list[int])` fail? Write a check that works.
7. Write hints for: (a) a dict mapping each character to the list of its indices;
   (b) a 2D grid of `"0"`/`"1"` strings; (c) a deque of `(row, col)` pairs;
   (d) a function argument that is a key function from `str` to `int`.

**Tricky**

8. `def g(k: Optional[int]) -> int:` is called as `g()`. What happens, and why?
9. Explain why `def f(x: list[int])` fails on 3.8 but works there with
   `from __future__ import annotations`.
10. Write the hint for a `defaultdict(list)` whose keys are `(row, col)` pairs and whose
    values are lists of strings.

## 2.6 Chapter cheat sheet

| Fact | Detail |
|---|---|
| Hints are not checked at run time | Wrong types run; tools and readers use hints |
| `List[int]` ≡ `list[int]` | Built-in generics 3.9+ (PEP 585); `typing` aliases deprecated, still work, no warnings |
| `Optional[T]` ≡ `T \| None` | `\|` needs 3.10+ (PEP 604); `Optional`/`Union` not deprecated |
| Collections | `collections.deque[int]`, `collections.Counter[str]`, `collections.abc.Callable[[int], bool]` (3.9+) |
| `tuple[int, ...]` | Any length; `tuple[int]` is exactly one |
| Old Python | `from __future__ import annotations` (3.7+) stores hints as strings |
| 3.14+ | Annotations evaluated lazily; forward references need no quotes |
| LeetCode | Templates use `typing` forms; `typing` is pre-imported there, **not** locally |
| Book rule | Read both, write `list[int]`, never mix styles in a file |
