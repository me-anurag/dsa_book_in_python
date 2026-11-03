# Appendix E: Glossary

Terms as this book uses them, with the chapter that explains each.

| Term | Meaning | Ch |
|---|---|---|
| Aliasing | Two names referring to the same mutable object, so a change through one is visible through the other | 3 |
| Amortized cost | Average cost per operation over any sequence of operations (`list.append` is O(1) amortized) | 6, 10 |
| Auxiliary space | Extra memory an algorithm uses beyond its input; stated separately from time, and excluding the output unless noted | 10 |
| Big-O / Θ / Ω | Upper bound / tight bound / lower bound on growth, ignoring constant factors | 10 |
| Binary search on the answer | Searching a numeric answer range for the first value where a monotone check becomes true | 19 |
| Border | A string that is both a proper prefix and a suffix of another string | 29 |
| Boyer–Moore voting | Finding a majority value by cancelling pairs of different values, in O(1) space | 23 |
| Brute force | The most direct correct solution, usually checking all candidates; your first idea and your test oracle | 30, 32 |
| Carry | The value passed to the next digit position in hand-written arithmetic | 13, 28 |
| Complement lookup | For each x, checking a hash map for `target − x` | 14 |
| CPython | The reference Python implementation; all measurements in this book use CPython 3.13.2 | 11 |
| Cyclic sort | Placing each value v (in 1..n) at index v − 1 by swapping | 23 |
| Difference array | Recording range additions at their endpoints and recovering values with a prefix sum | 15 |
| Dutch national flag | One-pass three-way partition into "less", "equal", "greater" regions | 18 |
| Dynamic array | A contiguous array with spare capacity that grows geometrically; how CPython lists are built | 6 |
| Expand around center | Discovering palindromes by growing outward from each of the 2n − 1 centers | 26 |
| Floyd's cycle detection | Tortoise-and-hare pointers that find a cycle and its entry in O(1) space | 23 |
| Generator | A lazy, single-use iterator that produces items on demand | 4 |
| Generic alias | The runtime object behind `list[int]` (3.9+); used only in type hints | 2 |
| Hashable | Usable as a dict key or set element; immutable built-ins like `str`, `int`, `tuple` | 7, 14 |
| Immutable | Unchangeable after creation (`str`, `tuple`, `int`); every "change" makes a new object | 3, 5 |
| In place | Modifying the given input without allocating a copy proportional to its size | 1, 13 |
| Invariant | A statement that holds at a fixed point of every loop iteration and proves the code correct | 13–29 |
| Judge | LeetCode's program that calls your method and checks the return value or the mutated argument | 1 |
| Kadane's algorithm | Maximum subarray sum via "best sum ending here" in O(n) | 20 |
| KMP | Knuth–Morris–Pratt matching using the prefix function; O(n + m) worst case | 29 |
| Lower bound | The first index whose value is ≥ x in a sorted array (`bisect_left`) | 8, 19 |
| Manacher's algorithm | O(n) palindrome radii via mirrored centers (advanced) | 26 |
| Monotonic deque | A deque kept in sorted order so its front is always the window max (or min) | 21 |
| Monotonic stack | A stack kept in sorted order; popping answers "next greater/smaller" queries | 21 |
| need/have | Window technique tracking how many required characters are currently satisfied (`formed`) | 27 |
| Oracle | A simple, obviously correct solution used to check a more efficient one | 32 |
| Prefix function | `pi[i]` = length of the longest proper border of `p[:i+1]` | 29 |
| Prefix sums | Running totals with a leading 0, so any range sum is one subtraction | 15 |
| Rabin–Karp | Matching by rolling hashes, verifying candidate matches | 25, 29 |
| Rolling hash | A hash of a sliding window updated in O(1) as characters enter and leave | 25 |
| Shrinking | Reducing a failing test input to a minimal one that still fails | 32 |
| Signature | A key equal for all members of an equivalence class (sorted letters, count tuple) | 14, 25 |
| Sliding window | A contiguous range `[left, right]` updated incrementally as both ends move forward | 17, 27 |
| Stable sort | A sort that keeps equal elements in their original relative order (Timsort is stable) | 6 |
| Stress testing | Comparing a solution with an oracle on many random inputs | 32 |
| Tier tags | **[USE-FREELY]**, **[KNOW-HOW]**, **[HAND-WRITE]**: how freely a tool may be used in interviews | 9 |
| Timsort | CPython's adaptive, stable merge-sort hybrid behind `sorted` and `list.sort` | 6 |
| Truncating division | Rounding a quotient toward zero (as C, Java and most statements do), unlike Python's floor `//` | 3 |
| Two pointers | Two indices moving toward each other (or in step) to eliminate candidates | 16 |
| Write pointer | An index marking where the next kept element goes during an in-place filter | 13 |
| XOR trick | Using `x ^ x == 0` to cancel pairs | 23 |
| Z-function | `z[i]` = length of the longest common prefix of s and `s[i:]` | 29 |
