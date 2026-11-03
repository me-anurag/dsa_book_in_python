# Appendix A: Cheat Sheets

Each chapter's cheat sheet, collected for printing. Generated from the chapters by
`build/gen_appendices.py`; the chapter is always the authoritative version.

## A.5 Every `str` Method


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

## A.6 Every `list` Operation


| Operation | Cost | Note |
|---|---|---|
| `a[i]`, `a[i] = x`, `len(a)` | O(1) | |
| `append`, `pop()` | O(1) amortized | Use the end of the list as a stack |
| `insert(0, x)`, `pop(0)`, `del a[0]` | O(n) | 1,400× slower at 10⁵ (`bench/ch06_list_ends.py`); use `deque` |
| `x in a`, `index`, `count`, `remove` | O(n) | Use a `set` for repeated lookups |
| `a[i:j]` | O(j − i) copy | Not free inside loops |
| `a[:] = ...` | O(n + k) | In-place replacement (Ch 1) |
| `a.sort()` / `sorted(a)` | O(n log n), stable | `sort` returns `None` |
| Multi-key | `key=lambda p: (-p[1], p[0])` | Or two stable passes, secondary first |
| Pairwise order | `cmp_to_key(cmp)` | 179 Largest Number |
| `a += it` | Mutates (extend) | `a + it` needs a list and creates a new one |

## A.9 Built-in vs Hand-Written: The Tier Map


| Tier | Rule | Examples |
|---|---|---|
| **[USE-FREELY]** | Use anywhere | `len`, `enumerate`, `zip`, `dict`, `set`, `deque`, `sum`, `"".join` |
| **[KNOW-HOW]** | Use, state the cost, be able to write it | `sorted`, `reversed`, `Counter`, `bisect`, `accumulate`, `heapq`, `str.find`, `split`, `min`/`max`, `int()` |
| **[HAND-WRITE]** | It's the question | Two pointers, sliding window, Kadane, monotonic stack, Dutch flag, cyclic sort, Boyer–Moore, rotation, spiral, expand-around-center, KMP/Z/Rabin–Karp, string arithmetic, atoi |
| Signals | "in-place", "O(1) extra", "without division", "don't convert to integer", "no built-in sort", "don't modify" | 344, 189, 283, 238, 43, 912, 41, 287 |
| Sorts | Merge (stable, O(n) space), quick (random pivot, 3-way, smaller side first), counting (small range) | Ch 18 |

## A.12 Operation Complexity Cheat Sheet


| O(1) | O(log n) | O(n) | O(n log n) |
|---|---|---|---|
| `a[i]`, `len`, `append`, `pop()`, dict/set get/add/in (avg), deque ends, `h[0]`, int in `range` | `heappush`/`heappop`, `bisect_left`/`right` | `x in list/str/tuple`, `pop(0)`, `insert(0)`, `insort`, slices, `+`, `min`/`max`/`sum`, copies, `join`, `split`, `heapify`, `Counter(it)`, forward substring search | `sorted`, `list.sort` |

## A.13 Array Fundamentals & In-Place Manipulation


| | |
|---|---|
| **Signals** | in-place, O(1) extra, `-> None`, "return k", remove/move/keep, rotate, merge into the first array, digit arrays |
| **Template** | `write = 0; for read ...: if keep(nums[read]): nums[write] = nums[read]; write += 1` |
| **Invariant** | `nums[:write]` = kept elements of `nums[:read]`, in order; `write <= read` |
| **Variants** | Fill from the back (88); three reversals (189); carry from the right (66); swap to carry the rejects along (283) |
| **Complexity** | O(n) time, O(1) auxiliary |
| **Top 3 bugs** | Rebinding instead of mutating; forgetting `k %= n`; forward-merging over unread values |
| **CPython note** | Slicing is ~5× faster than O(1)-space loops (measured); state the trade-off |

## A.14 Hashing & Frequency Counting


| | |
|---|---|
| **Signals** | seen/duplicate, count/frequency, pair to a target (unsorted), anagram, group by, bijection/pattern, top k, O(n) where sorting is O(n log n) |
| **Templates** | Count: `d[x] = d.get(x, 0) + 1`. Seen: check, then `add`. Group: `defaultdict(list)[signature(x)].append(x)` |
| **Keys** | Value; signature (sorted string or 26-tuple); derived (complement, x − 1, `(box, digit)`) |
| **Complexity** | O(n) average time; O(k) space for k distinct keys (O(1) for a fixed alphabet) |
| **Top 3 bugs** | Insert-before-check in Two Sum; unhashable keys (list, Counter); `[[]] * n` buckets |
| **Optimal tricks** | Bucket by frequency (347, O(n)); walk only from run starts (128, O(n)) |
| **CPython** | Sorted key faster for short strings; count tuple faster at L = 100 (measured) |

## A.15 Prefix Sums & Difference Arrays


| | |
|---|---|
| **Signals** | Range sums asked repeatedly; "contiguous subarray" + "sum = k / divisible by k" with **negatives**; product except self; many range updates |
| **Identity** | `sum(nums[i:j]) = prefix[j] - prefix[i]`, with `prefix[0] = 0` |
| **Templates** | Prefix array; prefix + map of counts seeded `{0: 1}`; remainder counts; difference array `diff[l] += v; diff[r] -= v` |
| **Complexity** | O(n) build, O(1) query; O(n) average counting with a map; O(n + u) for u range updates |
| **Top 3 bugs** | Missing the `{0: 1}` seed; updating the map before the lookup; closed vs half-open range ends |
| **Python** | `%` is already non-negative; no overflow; never `sum(slice)` in a loop |

## A.16 Two Pointers


| | |
|---|---|
| **Signals** | Sorted input; pair/triplet sum; O(1) extra space; area between two ends; extremes at the ends |
| **Template** | `left, right = 0, n - 1`; `while left < right`: compare, move the side that can't be in any answer |
| **Invariant** | Any valid answer lies in `[left, right]` |
| **k-sum** | Sort; fix k − 2 values; two pointers; skip duplicates at **every** level → O(n^(k−1)) |
| **Complexity** | O(n) per scan, O(1) space (+ O(n log n) to sort) |
| **Top 3 bugs** | `<=` instead of `<`; wrong duplicate-skip direction; moving the taller side (11) |
| **CPython** | 3Sum two pointers measured 2–5× faster than the set variant at n = 3000 |

## A.17 Sliding Window


| | |
|---|---|
| **Signals** | Contiguous + longest/shortest/count; size k; at most k; non-negative sums |
| **Template** | `for right`: add; `while invalid`: remove `nums[left]`, `left += 1`; record |
| **Longest vs shortest** | Longest: record after shrinking. Shortest: record inside the shrink loop |
| **Exactly k** | `at_most(k) - at_most(k - 1)`; count with `right - left + 1` |
| **Complexity** | O(n) time (amortized pointer moves); O(1) or O(k) space |
| **Top 3 bugs** | Window with negative values; not deleting zero-count keys; recording at the wrong point |
| **Not a window** | Negatives with a sum condition (Ch 15); max inside the window (Ch 21) |

## A.18 Sorting as a Tool (+ Writing Sorts by Hand)


| | |
|---|---|
| **Signals** | Intervals, k-th, "no built-in sort", few categories, greedy selection, custom order |
| **Sorts** | Merge: O(n log n) always, stable, O(n) space. Quick: expected O(n log n), random pivot + 3-way + smaller side first. Counting: O(n + k) for a small range |
| **Partition** | `< pivot | == pivot | unknown | > pivot`; don't advance `mid` after swapping with `high` |
| **Intervals** | Merge: sort by start, `start <= last_end` (56: touching merges). Max kept: sort by **end**, `start >= last_end` (435: touching is fine) |
| **k-th** | Heap O(n log k); quickselect expected O(n); counting O(n + R) |
| **Top 3 bugs** | Fixed pivot on sorted input; `<` vs `<=` at interval boundaries; k-th distinct vs k-th position |
| **CPython** | `sorted` beats every Python-level sort; quickselect measured slower than sorting |

## A.19 Binary Search


| | |
|---|---|
| **Signals** | Sorted + O(log n); first/last/insert position; rotated; "minimum k such that"; any peak |
| **Template** | `while lo < hi: mid = (lo + hi) // 2; if pred(mid): hi = mid else: lo = mid + 1` → first True |
| **Invariant** | Below `lo`: False; from `hi` on: True; the range shrinks every step |
| **Bounds** | Lower bound searches `[0, len]`; answer search uses `[smallest possible, largest possible]` |
| **Rotated** | One half is always sorted; compare with `nums[hi]` to find the minimum |
| **Complexity** | O(log n), or O(log R) checks × the check's cost |
| **Top 3 bugs** | `lo = mid` (infinite loop); `hi = len - 1` when the answer can be `len`; non-monotone predicates |

## A.20 Kadane & Subarray Optimization


| | |
|---|---|
| **Signals** | Max/min contiguous sum or product; negatives present; circular |
| **Recurrence** | `E(i) = max(nums[i], E(i-1) + nums[i])`; answer = max E(i) |
| **Template** | `e = best = nums[0]`; for the rest: `e = max(x, e + x)`; `best = max(best, e)` |
| **Product** | Track max and min ending here; candidates `(x, hi·x, lo·x)` |
| **Circular** | `max(kadane_max, total - kadane_min)`, unless all values are negative |
| **Complexity** | O(n) time, O(1) space; divide and conquer O(n log n) |
| **Top 3 bugs** | Initializing to 0 (empty subarray); updating `lo` from the new `hi`; missing the all-negative guard in 918 |

## A.21 Monotonic Stack & Monotonic Deque


| | |
|---|---|
| **Signals** | Next/previous greater or smaller; days until; span; histogram; window max/min; survivors |
| **Template** | `for i, x: while stack and nums[stack[-1]] < x: answer[stack.pop()] = i; stack.append(i)` |
| **Orders** | Next greater → decreasing stack; next smaller → increasing stack; previous = the new top after popping |
| **Deque** | Decreasing deque of indices; pop the back while `<= x`; pop the front when expired |
| **Complexity** | O(n) time (each index pushed and popped once), O(n) space |
| **Top 3 bugs** | Values instead of indices; `<` vs `<=` (strict vs non-strict); missing sentinel / second pass |
| **CPython** | Deque measured 1.4–3.1× faster than the heap for 239 |

## A.22 Matrices (2D Arrays)


| | |
|---|---|
| **Signals** | Rotate/transpose, spiral, in-place grid, row/column marking, simultaneous update, sorted rows and columns |
| **Formulas** | Transpose (r, c)→(c, r); clockwise (r, c)→(c, n−1−r) = transpose + reverse rows |
| **Directions** | `[(0, 1), (1, 0), (0, -1), (-1, 0)]`; turn with `d = (d + 1) % 4`; bounds-check every neighbor |
| **Space ladder** | Copy O(mn) → marker sets O(m+n) → first row/col markers O(1) → spare bits O(1) |
| **Complexity** | O(m · n) time; staircase search O(m + n) |
| **Top 3 bugs** | Negative indices wrapping silently; full-square transpose (double swap); missing spiral guards |

## A.23 Index Tricks & Voting


| | |
|---|---|
| **Signals** | Values in 1..n; missing/duplicate/majority; O(1) space or read-only |
| **Cyclic sort** | `while 1 <= v <= n and nums[v-1] != v`: swap into place (compute `j` first) |
| **Sign marking** | `nums[abs(x) - 1] = -abs(...)`; positive slots = missing values |
| **XOR** | Pairs cancel: `x ^ x == 0`; works on negatives in Python |
| **Read-only duplicate** | Binary search on value counts O(n log n); Floyd O(n) |
| **Voting** | > n/2: one candidate. > n/3: two candidates + verification pass |
| **Complexity** | O(n) time, O(1) space |
| **Top 3 bugs** | Self-referencing swap; missing the duplicate guard (infinite loop); skipping verification in 229 |

## A.24 String Fundamentals for DSA


| | |
|---|---|
| **Signals** | Lowercase-only alphabet; words and spaces; prefix/subsequence; symbol tables; "in place" on a string |
| **Char ↔ index** | `ord(c) - ord("a")`, `chr(ord("a") + k)`; ASCII lowercase only |
| **Counting** | `counts = [0] * 26`; compare two counts in 26 steps |
| **Building** | `parts.append(...)`, then `"".join(parts)`; edit via `list(s)` |
| **Tokenizing** | `split()` (not `split(" ")`), or scan with skip-spaces / walk-word loops |
| **Complexity** | O(n) time; O(1) space for a fixed alphabet, O(n) to build output |
| **Top 3 bugs** | `split(" ")`; the forgotten last word; `ord` arithmetic on non-ASCII input |

## A.25 Hashing on Strings


| | |
|---|---|
| **Signals** | Anagram/permutation; all anagram windows; repeated fixed-length substrings; serialize strings |
| **Keys** | Sorted string (any alphabet), 26-count tuple (lowercase), substring itself, bit-packed int (tiny alphabet), Rabin–Karp (general; verify hits) |
| **Sliding signature** | Add the entering char, remove the leaving char, compare with the target counts |
| **Framing** | `len#payload`, never delimiters alone |
| **Complexity** | O(n) to O(n · L) time; O(1) average per lookup |
| **Top 3 bugs** | Unhashable keys; unverified rolling-hash hits; delimiter-based encoding |
| **CPython** | Substring set and 26-count compare measured faster than the clever rolling versions |

## A.26 Palindromes & Two Pointers on Strings


| | |
|---|---|
| **Signals** | Palindrome; ignore non-alnum / case; at most one deletion; longest/count palindromic substrings; build from letters |
| **Validate** | Two pointers inward; skip what doesn't count |
| **One deletion** | At the first mismatch, check `s[l+1..r]` or `s[l..r-1]` |
| **Discover** | Expand around 2n − 1 centers (odd and even) |
| **Advanced** | Manacher: O(n) via mirrored radii |
| **Complexity** | Validate O(n); expand O(n²) worst, near O(n) typical (measured); DP O(n²) time and space |
| **Top 3 bugs** | Missing even centers; trying only one skip direction; `isalpha` instead of `isalnum` |

## A.27 Sliding Window on Strings


| | |
|---|---|
| **Signals** | Longest/shortest substring with a character constraint; minimum covering window; fixed length; equal-length word concatenation |
| **Summaries** | Last index (no repeats); counts + max (replacements); `formed` crossings (cover t) |
| **need/have** | `formed += 1` when `window[x] == need[x]`; `formed -= 1` when it drops `< need[x]`; record inside the shrink loop |
| **Complexity** | O(n) time (O(m + n) for 76; O(n · w) for 30); O(k) space |
| **Top 3 bugs** | Stale last-index moving `left` backwards; ignoring multiplicity in t; one offset only in 30 |

## A.28 Parsing, Simulation & Stacks on Strings


| | |
|---|---|
| **Signals** | atoi/parse; brackets and nesting; evaluate without eval; "don't convert to integer"; paths; zigzag/count-and-say simulation |
| **Parsing** | Explicit states; ASCII digit test `"0" <= c <= "9"`; overflow check before `value * 10 + d` |
| **Stacks** | Brackets, nested repeats `(text, k)`, directory names, signed terms |
| **Digit arithmetic** | Right to left with a carry; multiplication into position `i + j`, then carry |
| **Complexity** | O(n) scans; O(m · n) multiplication; O(output) for decoding |
| **Top 3 bugs** | `//` on negative terms; forgetting the last number or the last carry; `strip()`/`int()` accepting what the problem rejects |

## A.29 String Matching Algorithms


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
