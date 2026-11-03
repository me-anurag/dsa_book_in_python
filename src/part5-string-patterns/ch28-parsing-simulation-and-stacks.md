# Chapter 28: Parsing, Simulation & Stacks on Strings

## 28.0 At a glance

- **What this pattern solves:** turning text into structure or values, character by
  character:
  - number parsing with signs and overflow
  - matching and nested brackets
  - expression evaluation with precedence
  - path normalization
  - arithmetic on digit strings without `int()`
  - row-by-row simulations
- **Prerequisites:** Ch 3 (truncating division, overflow-free ints), Ch 7 (why `int()`
  isn't a parser), Ch 13 (carry propagation in 66), Ch 21 (stacks).
- **Typical complexity:** O(n) time for one scan; O(n) space for a stack or the output;
  O(m · n) for schoolbook multiplication.
- **Tools used in this chapter:**
  - `list` as a stack, `"".join` **[USE-FREELY]**
  - `str.split`, `re`, `int()` for reference checks **[KNOW-HOW]**
  - state-machine parsing, digit-by-digit arithmetic and stack-based evaluation
    **[HAND-WRITE]**

## 28.1 Recognition signals

- "**implement atoi**", "parse", "convert the string to an integer", with clamping rules
- **Brackets / nesting**: `()[]{}`, `k[...]` → a stack
- "**evaluate** the expression", "**without eval**" → operator precedence with a stack
- "**do not convert** to integer" / "no BigInteger" → digit arithmetic with carries
- Paths with `.` and `..` → a stack of directory names
- "**write in a zigzag** / spiral", "**count and say**" → direct simulation

**Look-alikes:**

| Looks like | Actually | Tell |
|---|---|---|
| Bracket matching (stack) | Counter | One bracket type only and no order check: a running balance is enough |
| Expression with only + and − | Running sum | No precedence: a stack isn't needed |
| Digit arithmetic | `int()` | Allowed only if the statement doesn't forbid it, and only below the 4300-digit limit (Ch 3) |

## 28.2 Core idea

Parsing problems are **state machines**: you read one character, and what it means
depends on where you are (before the sign, inside a number, after a bracket). Write the
states down, and each character becomes a small `if`/`elif` decision.

Stacks enter whenever something **opened earlier must be closed later**, in reverse order:
brackets, nested repetitions, directories you can go back out of. The most recent
unclosed item is always on top.

```text
decode "3[a2[c]]" with a stack of (text-so-far, repeat):

'3'   k = 3
'['   push ([], 3); current = []
'a'   current = [a]
'2'   k = 2
'['   push ([a], 2); current = []
'c'   current = [c]
']'   pop ([a], 2): current = [a] + "c" * 2 = [a, cc]
']'   pop ([], 3):  current = [] + "acc" * 3 -> "accaccacc"
```

Digit arithmetic is grade-school arithmetic made explicit: process digits from the right,
keep a carry, and build the output in reverse.

## 28.3 The template

**Read an unsigned integer starting at index i:**

```python
def read_number(s: str, i: int) -> tuple[int, int]:
    """Parse ASCII digits from s[i:]; return (value, index after the digits)."""
    value = 0
    while i < len(s) and "0" <= s[i] <= "9":
        value = value * 10 + (ord(s[i]) - ord("0"))
        i += 1
    return value, i


read_number("ab123cd", 2)  # -> (123, 5)
```

**Add two digit strings from the right:**

```python
def add_digit_strings(a: str, b: str) -> str:
    i, j, carry, out = len(a) - 1, len(b) - 1, 0, []
    while i >= 0 or j >= 0 or carry:
        total = carry
        if i >= 0:
            total += ord(a[i]) - 48
            i -= 1
        if j >= 0:
            total += ord(b[j]) - 48
            j -= 1
        out.append(chr(total % 10 + 48))
        carry = total // 10
    return "".join(reversed(out))


add_digit_strings("999", "1")  # -> '1000'
```

- **Invariant (addition):** after each step, `out` holds the lowest digits of the sum,
  reversed, and `carry` is what the next column must add.
- **Complexity:** O(max(m, n)) time; O(max(m, n)) output.

## 28.4 Dry run

227 Basic Calculator II on `"14-3/2"`. The stack holds **signed terms**; `op` is the
operator before the current number.

| Read | value | op (pending) | Action at the end of a number | stack |
|---|---|---|---|---|
| `14` then `-` | 14 | + | push 14 | [14] |
| `3` then `/` | 3 | − | push −3 | [14, −3] |
| `2` (end) | 2 | / | pop −3, push trunc(−3 / 2) = **−1** | [14, −1] |

Sum: 13. With Python's `//`, the last step would compute `-3 // 2 = -2` and return 12:
**wrong**. The problem says division truncates toward zero, and the stack makes negative
dividends possible (Ch 3, 3.2.6):

```python
-3 // 2  # -> -2
int(-3 / 2)  # -> -1
```

## 28.5 Variations

| Variant | What changes | Example |
|---|---|---|
| Bracket matching | Push openers; a closer must match the top | 20 Valid Parentheses |
| Cancelling pairs | Stack of survivors; equal to the top → pop | 1047 Remove All Adjacent Duplicates In String |
| Path normalization | Split on `/`; `..` pops, `.` and `''` skip | 71 Simplify Path |
| Nested repetition | Stack of (text so far, count) | 394 Decode String |
| Number parsing with rules | State machine + clamping | 8 String to Integer (atoi) |
| Binary / decimal addition | Column addition with a carry | 67 Add Binary, 415 Add Strings |
| Multiplication | Digit-position array, then carries | 43 Multiply Strings |
| Precedence | Stack of signed terms; * and / combine with the top | 227 Basic Calculator II |
| Simulation | Row pointer bouncing between 0 and numRows − 1 | 6 Zigzag Conversion |
| Run-length rewriting | `groupby` runs, iterated | 38 Count and Say |

## 28.6 Worked problems

### 28.6.1 8 String to Integer (atoi) (Medium): <https://leetcode.com/problems/string-to-integer-atoi/>

**Problem.** Parse an integer the way C's `atoi` does:
1. Skip leading spaces (only `' '`).
2. Read one optional `+` or `-`.
3. Read digits until the first non-digit. No digits means 0.
4. Clamp the result to [−2³¹, 2³¹ − 1].

**Signatures.** LeetCode: `def myAtoi(self, s: str) -> int:`. Book: identical.

**Constraints → target.** n ≤ 200; letters, digits, spaces, `+`, `-`, `.`. O(n). The
difficulty is getting every rule exactly right.

**Approach 1: regex + `int()` (a [KNOW-HOW] aside).**

<!-- file: code/ch28_parsing/atoi.py -->

```python
class SolutionRegex:
    def myAtoi(self, s: str) -> int:
        # O(n) time. Regex + int(): [KNOW-HOW] aside. Note: str.lstrip(" ") strips
        # only spaces, matching the problem (lstrip() would also strip tabs).
        match = re.match(r"[+-]?\d+", s.lstrip(" "))
        if not match:
            return 0
        return max(INT_MIN, min(INT_MAX, int(match.group())))
```

**Approach 2: hand-written state machine with pre-overflow checks (optimal).**

<!-- file: code/ch28_parsing/atoi.py -->

```python
class Solution:
    def myAtoi(self, s: str) -> int:
        # Hand-written parser. O(n) time, O(1) space. Clamp BEFORE the value would
        # leave the range (the C/Java discipline; in Python it just stops early).
        i, n = 0, len(s)
        while i < n and s[i] == " ":
            i += 1
        sign = 1
        if i < n and s[i] in "+-":
            sign = -1 if s[i] == "-" else 1
            i += 1
        limit = INT_MAX if sign == 1 else -INT_MIN       # 2147483647 or 2147483648
        value = 0
        while i < n and "0" <= s[i] <= "9":               # ASCII digits only (Ch 5)
            digit = ord(s[i]) - ord("0")
            if value > (limit - digit) // 10:             # value * 10 + digit > limit
                return INT_MAX if sign == 1 else INT_MIN
            value = value * 10 + digit
            i += 1
        return sign * value
```

The states, in order: spaces → sign (at most one) → digits → stop. The overflow check is
the same rearranged inequality as Ch 3's 7 Reverse Integer. In Python, nothing would
overflow if you parsed everything first, but stopping early also caps the work on inputs
like `"99999999999999999999"`. `int()` on the whole substring would accept `"4_2"` and
`" 42 "` (Ch 7), and the `\d` regex class also matches non-ASCII digits. The explicit
`"0" <= c <= "9"` test says exactly what the problem means.

**Edge cases tested:**
- the statement examples
- `""`, `"   "`, `"+"`, `"-"`, `"+-12"`
- `"  +0 123"` (stops at the space)
- both 32-bit extremes and values just past them, 20 nines
- `"00000123"`, `".1"`, `"3.14"`
- `"\t12"` (tab isn't skipped)
- 3,000 random strings checked against the regex version

**Common bugs:**
- `s.strip()`, which also strips tabs and trailing text.
- Accepting a second sign.
- Clamping only negative overflow, or clamping −2³¹ to −2³¹ + 1.
- `isdigit()` on Unicode digits (Ch 5).

**Built-in shortcut:** approach 1, `re` + `int()` **[KNOW-HOW]**. Interviewers want
approach 2.

### 28.6.2 394 Decode String (Medium): <https://leetcode.com/problems/decode-string/>

**Problem.** In a valid encoded string, `k[encoded]` means `encoded` repeated k times,
and brackets can nest. Return the decoded string.

**Signatures.** LeetCode: `def decodeString(self, s: str) -> str:`. Book: identical.

**Constraints → target.** n ≤ 30 and counts ≤ 300, so the output size dominates. O(output).

**Approach 1: recursive descent.**

<!-- file: code/ch28_parsing/decode_string.py -->

```python
class SolutionRecursive:
    def decodeString(self, s: str) -> str:
        # Recursive descent. O(output) time and space; recursion depth = nesting depth.
        def parse(i: int) -> tuple[str, int]:
            """Decode from s[i] until a ']' or the end; return (text, next index)."""
            parts: list[str] = []
            while i < len(s) and s[i] != "]":
                if s[i].isdigit():
                    k = 0
                    while s[i].isdigit():
                        k = k * 10 + int(s[i])
                        i += 1
                    inner, i = parse(i + 1)          # skip '['
                    parts.append(inner * k)
                    i += 1                           # skip ']'
                else:
                    parts.append(s[i])
                    i += 1
            return "".join(parts), i

        return parse(0)[0]
```

**Approach 2: explicit stack (optimal, no recursion).**

<!-- file: code/ch28_parsing/decode_string.py -->

```python
class Solution:
    def decodeString(self, s: str) -> str:
        # Explicit stack. On '[' save (text so far, repeat count) and start fresh;
        # on ']' pop and splice. O(output) time and space.
        stack: list[tuple[list[str], int]] = []
        current: list[str] = []
        k = 0
        for c in s:
            if c.isdigit():
                k = k * 10 + int(c)                  # counts can have several digits
            elif c == "[":
                stack.append((current, k))
                current, k = [], 0
            elif c == "]":
                outer, repeat = stack.pop()
                outer.append("".join(current) * repeat)
                current = outer
            else:
                current.append(c)
        return "".join(current)
```

Why the stack mirrors recursion: each `[` is a recursive call (save the caller's state),
and each `]` is a return (splice the result into the caller). Traced in 28.2.

**Edge cases tested:**
- the statement examples
- no brackets
- a multi-digit count (`10[a]`)
- nesting depth 3
- an empty body `1[]`

**Common bugs:**
- Reading only one digit of the count.
- Building strings with `+=` (Ch 5); use lists and `join`.
- Forgetting to reset `k` after `[`.

**Built-in shortcut:** none.

### 28.6.3 43 Multiply Strings (Medium): <https://leetcode.com/problems/multiply-strings/>

**Problem.** Multiply two non-negative integers given as decimal strings (up to 200
digits), returning a string, **without** a big-integer library and **without** converting
the inputs to integers.

**Signatures.** LeetCode: `def multiply(self, num1: str, num2: str) -> str:`. Book:
identical.

**Constraints → target.** m, n ≤ 200. O(m · n) = 4 × 10⁴ digit products.

**Approach 1: schoolbook rows, added as strings.**

<!-- file: code/ch28_parsing/multiply_strings.py -->

```python
class SolutionRepeatedAddition:
    def multiply(self, num1: str, num2: str) -> str:
        # Schoolbook by rows: num1 * (each digit of num2), shifted, added as
        # strings. O(m * n) digit operations, plus O(n) string additions of
        # length O(m + n): O(n * (m + n)) overall. O(m + n) space.
        def add(a: str, b: str) -> str:
            i, j, carry, out = len(a) - 1, len(b) - 1, 0, []
            while i >= 0 or j >= 0 or carry:
                total = carry
                if i >= 0:
                    total += ord(a[i]) - 48
                    i -= 1
                if j >= 0:
                    total += ord(b[j]) - 48
                    j -= 1
                out.append(chr(total % 10 + 48))
                carry = total // 10
            return "".join(reversed(out))

        def times_digit(a: str, d: int) -> str:
            carry, out = 0, []
            for c in reversed(a):
                total = (ord(c) - 48) * d + carry
                out.append(chr(total % 10 + 48))
                carry = total // 10
            if carry:
                out.append(chr(carry + 48))
            return "".join(reversed(out))

        if num1 == "0" or num2 == "0":
            return "0"
        result = "0"
        for shift, c in enumerate(reversed(num2)):
            row = times_digit(num1, ord(c) - 48) + "0" * shift
            result = add(result, row)
        return result
```

**Approach 2: accumulate by digit position, carry once (optimal).**

<!-- file: code/ch28_parsing/multiply_strings.py -->

```python
class Solution:
    def multiply(self, num1: str, num2: str) -> str:
        # Digit-position array: digit i of num1 times digit j of num2 (from the
        # right) contributes to position i + j. Accumulate, then propagate carries.
        # O(m * n) time, O(m + n) space.
        if num1 == "0" or num2 == "0":
            return "0"
        m, n = len(num1), len(num2)
        pos = [0] * (m + n)                   # pos[k] = value at 10^k (pre-carry)
        for i in range(m):
            a = ord(num1[m - 1 - i]) - 48
            for j in range(n):
                pos[i + j] += a * (ord(num2[n - 1 - j]) - 48)
        carry = 0
        for k in range(m + n):
            total = pos[k] + carry
            pos[k] = total % 10
            carry = total // 10
        while len(pos) > 1 and pos[-1] == 0:  # strip leading zeros (stored last)
            pos.pop()
        return "".join(chr(d + 48) for d in reversed(pos))
```

Why position i + j: digit i (weight 10ⁱ) times digit j (weight 10ʲ) has weight 10ⁱ⁺ʲ. The
product has at most m + n digits. Deferring all carries to one final pass keeps the inner
loop to a single multiply-add. (Python ints can't overflow, so the uncarried sums are
safe. In C++ they stay below 81 · min(m, n) anyway.)

**Edge cases tested:**
- `"0"` with any number
- single digits with a carry
- `"999" × "999"`
- 1,000 random pairs up to 30 digits (the **tests** use `int()` as a reference; the
  solutions never do)
- 200 nines × 200 nines

**Common bugs:**
- Forgetting the zero shortcut, which leaves leading zeros like `"000"`.
- Reversing indices inconsistently.
- Calling `int(num1)` "just to check": that's exactly what's forbidden.

**Built-in shortcut:** `str(int(num1) * int(num2))`, which is **forbidden** here (Ch 9's
signals table).

### 28.6.4 227 Basic Calculator II (Medium): <https://leetcode.com/problems/basic-calculator-ii/>

**Problem.** Evaluate a valid expression of non-negative integers with `+ - * /` and
spaces. `*` and `/` bind tighter, and division truncates toward zero. `eval()` isn't
allowed.

**Signatures.** LeetCode: `def calculate(self, s: str) -> int:`. Book: identical.

**Constraints → target.** n ≤ 3 × 10⁵. O(n).

**Approach 1: tokenize, then apply precedence in a second pass.**

<!-- file: code/ch28_parsing/basic_calculator_ii.py -->

```python
class SolutionTwoPass:
    def calculate(self, s: str) -> int:
        # Tokenize, then evaluate * and / left to right into a list of terms, then
        # sum. O(n) time, O(n) space.
        tokens: list[str | int] = []
        i = 0
        while i < len(s):
            c = s[i]
            if c.isdigit():
                value = 0
                while i < len(s) and s[i].isdigit():
                    value = value * 10 + ord(s[i]) - 48
                    i += 1
                tokens.append(value)
                continue
            if c in "+-*/":
                tokens.append(c)
            i += 1
        terms: list[int] = [tokens[0]]          # type: ignore[list-item]
        for k in range(1, len(tokens), 2):
            op, value = tokens[k], tokens[k + 1]
            if op == "+":
                terms.append(value)              # type: ignore[arg-type]
            elif op == "-":
                terms.append(-value)             # type: ignore[operator]
            elif op == "*":
                terms[-1] *= value               # type: ignore[operator]
            else:
                terms[-1] = trunc_div(terms[-1], value)  # type: ignore[arg-type]
        return sum(terms)
```

**Approach 2: one pass with a stack of signed terms (optimal).**

<!-- file: code/ch28_parsing/basic_calculator_ii.py -->

```python
class Solution:
    def calculate(self, s: str) -> int:
        # One pass with a stack of signed terms. O(n) time, O(n) space.
        # `op` is the operator BEFORE the number just finished: + and - push a new
        # term; * and / combine with the last term (higher precedence).
        stack: list[int] = []
        value = 0
        op = "+"
        for i, c in enumerate(s):
            if c.isdigit():
                value = value * 10 + ord(c) - 48
            if (not c.isdigit() and c != " ") or i == len(s) - 1:
                if op == "+":
                    stack.append(value)
                elif op == "-":
                    stack.append(-value)
                elif op == "*":
                    stack.append(stack.pop() * value)
                else:
                    stack.append(trunc_div(stack.pop(), value))  # NOT // (negatives)
                op = c
                value = 0
        return sum(stack)
```

with Ch 3's exact truncating division:

<!-- file: code/ch28_parsing/basic_calculator_ii.py -->

```python
def trunc_div(a: int, b: int) -> int:
    """a / b rounded toward zero, exact (Ch 3, 3.2.6)."""
    q = abs(a) // abs(b)
    return q if (a >= 0) == (b > 0) else -q
```

Why the stack works: an expression without parentheses is a sum of **terms**, where each
term is a product or quotient chain. `+` and `−` start a new term (pushed with its sign);
`*` and `/` modify the current term (the top). Summing the stack at the end applies the
lowest-precedence operators last. Traced in 28.4, including the truncation trap.

**Edge cases tested:**
- the statement examples
- `"14-3/2"` (a negative dividend: 13, not 12)
- a single number
- `0*7-2147483647`
- 2,000 random expressions with and without spaces, checked against an independent
  reference that uses `int(a / b)`

**Common bugs:**
- Python's `//` on a negative term (28.4).
- Forgetting to process the last number (the `i == len(s) - 1` check).
- Treating spaces as operators.
- Multi-digit numbers read one digit at a time.

**Built-in shortcut:** `eval`, which is **forbidden**, and dangerous on untrusted input
anyway.

## 28.7 Python-specific notes

- **`//` floors; problem statements truncate.** Any parser that can produce a negative
  dividend needs `trunc_div` or `int(a / b)` (the latter only below 2⁵³; Ch 3).
- **`int()` accepts more than atoi does** (`" 42 "`, `"4_2"`, Unicode digits; Ch 7). It's
  fine as a test oracle, but not as a parser.
- **`ord(c) - 48`** is `ord(c) - ord("0")`. Write `ord("0")` for readability; the code
  above uses 48 in tight loops, with the meaning noted.
- **Stacks are lists** (`append` / `pop()` are O(1), Ch 6). Recursion depth equals nesting
  depth, which is fine here (n ≤ 30 in 394), but prefer the explicit stack for long
  inputs (Ch 10).
- **`itertools.groupby`** is exactly run-length encoding (38; Ch 8).

## 28.8 Practice set

Hints are in Appendix D, under "Chapter 28 practice". Reference solutions:
`code/ch28_parsing/practice_solutions.py`.

| # | Title | Difficulty | Link | Hint ref |
|---|---|---|---|---|
| 20 | Valid Parentheses | Easy | <https://leetcode.com/problems/valid-parentheses/> | D.28.1 |
| 1047 | Remove All Adjacent Duplicates In String | Easy | <https://leetcode.com/problems/remove-all-adjacent-duplicates-in-string/> | D.28.2 |
| 67 | Add Binary | Easy | <https://leetcode.com/problems/add-binary/> | D.28.3 |
| 415 | Add Strings | Easy | <https://leetcode.com/problems/add-strings/> | D.28.4 |
| 71 | Simplify Path | Medium | <https://leetcode.com/problems/simplify-path/> | D.28.5 |
| 6 | Zigzag Conversion | Medium | <https://leetcode.com/problems/zigzag-conversion/> | D.28.6 |
| 38 | Count and Say | Medium | <https://leetcode.com/problems/count-and-say/> | D.28.7 |

## 28.9 Chapter cheat sheet

| | |
|---|---|
| **Signals** | atoi/parse; brackets and nesting; evaluate without eval; "don't convert to integer"; paths; zigzag/count-and-say simulation |
| **Parsing** | Explicit states; ASCII digit test `"0" <= c <= "9"`; overflow check before `value * 10 + d` |
| **Stacks** | Brackets, nested repeats `(text, k)`, directory names, signed terms |
| **Digit arithmetic** | Right to left with a carry; multiplication into position `i + j`, then carry |
| **Complexity** | O(n) scans; O(m · n) multiplication; O(output) for decoding |
| **Top 3 bugs** | `//` on negative terms; forgetting the last number or the last carry; `strip()`/`int()` accepting what the problem rejects |
