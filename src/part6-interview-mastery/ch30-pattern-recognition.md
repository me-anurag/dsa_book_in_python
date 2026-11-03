# Chapter 30: Pattern Recognition

## 30.0 At a glance

Parts IV and V taught each pattern inside a chapter that announced it. In an interview,
nobody tells you the chapter. This chapter is the index you search **from the problem
statement side**:

- the **signal phrases** that point to each pattern (44 rows)
- a **decision tree** that combines signals with the constraints
- the **look-alike** confusions, all in one place
- a six-step **attack method** for unseen problems

Every example problem is one this book solves or assigns, so you can check each signal
against a solution you've already seen. This is the front matter's "pattern notebook"
(0.4), filled in.

## 30.1 Complete reference: signal phrases

Read the left column as "if the statement says this…". Chapters point to the full
treatment.

| # | Signal in the statement | Candidate pattern | Ch | Example |
|---|---|---|---|---|
| 1 | "in-place", "O(1) extra memory", returns `None` | Write pointer / swaps | 13 | 283, 27 |
| 2 | "return k, the number of elements kept" | Write pointer; judge reads `nums[:k]` | 1, 13 | 26, 27 |
| 3 | "rotate by k" | Three reversals (`k %= n`) | 13 | 189 |
| 4 | "merge into the first array, which has room at the end" | Fill from the back | 13 | 88 |
| 5 | Digits stored in an array/string, "add one" / "add" | Carry from the right | 13, 28 | 66, 415, 67 |
| 6 | "pair with sum = target", input **unsorted** | Hash map of complements | 14 | 1 |
| 7 | "pair/triplet with sum = target", input **sorted** or sortable, O(1) space | Two pointers from both ends | 16 | 167, 15, 18 |
| 8 | "contains duplicate", "seen before" | Set | 14 | 217 |
| 9 | "anagram", "same characters", "permutation of" | Count signature (26-slot / Counter / sorted) | 14, 25 | 242, 49 |
| 10 | "group strings that…" | Dict from signature to list | 14 | 49 |
| 11 | "one-to-one mapping", "follows the pattern" | Two dicts (both directions) | 14 | 205, 290 |
| 12 | "top k most frequent" | Count + heap or buckets | 14 | 347 |
| 13 | "longest consecutive", O(n) required | Set + start-of-run walks | 14 | 128 |
| 14 | "sum of range [i, j]" asked many times | Prefix sums | 15 | 303 |
| 15 | "contiguous subarray" + "sum **equals** k" (negatives possible) | Prefix sum + hash map of counts | 15 | 560 |
| 16 | "subarray sum divisible by k" / "multiple of k" | Prefix remainders | 15 | 974, 523 |
| 17 | "product of all except self", "no division" | Prefix × suffix products | 15 | 238 |
| 18 | "add v to every element in [l, r)" many times | Difference array | 15 | 1094 |
| 19 | "largest area between two lines", "trapped water" | Two pointers, move the shorter side | 16 | 11, 42 |
| 20 | "squares of a sorted array" (negatives) | Two pointers, fill from the back | 16 | 977 |
| 21 | "subarray of size k" | Fixed sliding window | 17 | 643, 1456 |
| 22 | "longest/shortest contiguous" + non-negative values + sum condition | Variable sliding window | 17 | 209 |
| 23 | "at most k zeros / distinct / replacements" | Shrink-while-invalid window | 17, 27 | 1004, 904, 424 |
| 24 | "**exactly** k distinct" | atMost(k) − atMost(k − 1) | 17 | 992 |
| 25 | "longest substring without repeating" | Window + last index | 27 | 3 |
| 26 | "minimum window containing all of t" | need/have with `formed` | 27 | 76 |
| 27 | "all start indices of anagrams of p" | Sliding count signature | 25 | 438, 567 |
| 28 | "merge overlapping intervals" | Sort by start, sweep | 18 | 56, 57 |
| 29 | "remove the fewest intervals so none overlap" | Sort by end, greedy | 18 | 435 |
| 30 | "k-th largest", "without sorting" | Heap / quickselect / counting | 18 | 215 |
| 31 | Only values 0, 1, 2; "one pass" | Dutch national flag | 18 | 75 |
| 32 | "arrange to form the largest number" | Custom comparator | 6, 18 | 179 |
| 33 | "sorted" + "O(log n)" | Binary search (lower bound) | 19 | 704, 35, 34 |
| 34 | "rotated sorted array" | Binary search on the sorted half | 19 | 33, 153 |
| 35 | "minimum capacity/speed such that it's possible" | Binary search on the answer | 19 | 875, 1011 |
| 36 | "maximum subarray sum/product" | Kadane (track min for products) | 20 | 53, 152 |
| 37 | "circular" + max subarray | total − min subarray | 20 | 918 |
| 38 | "next greater/smaller element", "days until warmer" | Monotonic stack | 21 | 739, 496, 503 |
| 39 | "largest rectangle in histogram" | Increasing stack | 21 | 84 |
| 40 | "maximum of every window of size k" | Monotonic deque | 21 | 239 |
| 41 | "rotate the matrix in place", "spiral order" | Index formulas / boundaries | 22 | 48, 54 |
| 42 | "set row and column to 0", "update all cells simultaneously" | In-grid markers / spare bits | 22 | 73, 289 |
| 43 | Values in 1..n, "missing", "duplicate", O(1) space | Cyclic sort / sign marking / Floyd | 23 | 41, 448, 287 |
| 44 | "appears more than n/2 (n/3) times" | Boyer–Moore voting | 23 | 169, 229 |
| 45 | "every element twice except one" | XOR | 23 | 136, 268 |
| 46 | "palindrome", "at most one deletion" | Two pointers inward | 26 | 125, 680 |
| 47 | "longest palindromic substring", "count palindromic substrings" | Expand around center | 26 | 5, 647 |
| 48 | "parse the integer", "valid brackets", "decode k[...]", "evaluate without eval" | State machine / stack | 28 | 8, 20, 394, 227 |
| 49 | "multiply/add numbers given as strings" | Digit arithmetic | 28 | 43, 415 |
| 50 | "first occurrence without built-ins", "prefix that is also a suffix" | KMP / Z / Rabin–Karp | 29 | 28, 1392, 214, 459 |
| 51 | "repeated substrings of length 10" | Substring set / rolling hash | 25 | 187 |

### Constraints → target complexity (Ch 10, measured budget)

| Largest n | Target | Patterns that fit |
|---|---|---|
| ≤ 20 | O(2ⁿ) | Brute-force subsets (rare in this book) |
| ≤ 100–200 | O(n³) | Triple loops, k-sum with k = 4 (18) |
| ≤ 3,000 | O(n²) | 3Sum (15), expand-around-center (5), DP tables |
| ≤ 10⁵–10⁶ | O(n log n) / O(n) | Sorting, hashing, two pointers, windows, prefix sums, stacks |
| Value ≥ 10⁹ | O(log n) | Binary search on the answer |

## 30.2 Explanations by group

### 30.2.1 The decision tree

Walk the tree from the top. Each question is answerable from the statement and the
constraints alone.

```text
1. Is the input a STRING or ARRAY problem about...
   |
   +-- a single contiguous piece (subarray / substring)?
   |     +-- sum = k / divisible by k, negatives possible ..... prefix sums + map (15)
   |     +-- longest/shortest with a monotone constraint ..... sliding window (17, 27)
   |     +-- maximum sum / product .............................. Kadane (20)
   |     +-- palindromic .................................. expand / two pointers (26)
   |     +-- equals a fixed pattern ............................. KMP / Z / find (29)
   |
   +-- pairs / triples / k elements?
   |     +-- sorted (or sorting allowed) and O(1) space ......... two pointers (16)
   |     +-- unsorted, need indices ............................. hash map (14)
   |
   +-- each element vs. its NEAREST greater/smaller? ............ monotonic stack (21)
   +-- frequencies, duplicates, grouping, equivalence? .......... hashing (14, 25)
   +-- intervals, k-th, orderings, greedy choices? .............. sort first (18)
   +-- values in 1..n with O(1) space? .......................... index tricks (23)
   +-- a sorted/monotone search space? .......................... binary search (19)
   +-- a grid? ................................................ matrix techniques (22)
   +-- text with structure (numbers, brackets, operators)? ..... parsing + stack (28)
   +-- in-place rearrangement of one array? ..................... write pointer (13)

2. Check the constraints against the budget (30.1). If the candidate is too slow,
   look one level deeper: the follow-up usually names the faster pattern.
```

### 30.2.2 Look-alikes, all in one place

| You think | It might be | Deciding question |
|---|---|---|
| Sliding window | Prefix sum + map | Can values be negative? Is it "= k" rather than "≤ k"? |
| Hash map pair search | Two pointers | Is the input sorted, or is O(1) space required? |
| Two pointers | Sliding window | Do both pointers move in the same direction over a contiguous range? |
| Sorting | Hashing | Do you need order, or only equality/counts? Is O(n) required? |
| Binary search on an array | Binary search on the answer | Is the thing you're searching an index, or a numeric answer with a monotone check? |
| Monotonic stack | Two pointers | (Trapping water accepts both.) Do you need **each** element's nearest neighbor? |
| Substring | Subsequence | Must the characters be contiguous? |
| Kadane | Sliding window | Is there a length limit or an "at most k" constraint? |
| Index tricks | Hash set | Are values guaranteed in 1..n, and is O(1) space required? |

### 30.2.3 The attack method for an unseen problem

1. **Clarify.** Restate the problem. Ask about the input range, duplicates, negatives,
   empty input, return format, and whether the input may be modified.
2. **Examples.** Work one small example by hand, plus one edge case (empty, single, all
   equal, negatives).
3. **Brute force.** State it and its complexity out loud, even if it's hopeless. It's
   your fallback and your test oracle.
4. **Optimize.** Find the waste in the brute force (repeated sums → prefix sums; repeated
   scans → hashing; rescanning a window → updating it; re-reading text → KMP). Name the
   pattern and check it against the constraints table.
5. **Code.** Write the template you know (Parts IV–V), with readable names. State the
   invariant as a comment.
6. **Test.** Dry-run your own example through the code. Then test the edge cases from
   Ch 32's checklist. State the time and auxiliary space, each with its reason.

**Worked through once, on a problem from Ch 25.** "Return all starting positions in s
where some rearrangement of p occurs."

| Step | What you say / do |
|---|---|
| Clarify | Lowercase only? Overlapping matches count? Order of output? (Lowercase, yes, any order.) |
| Examples | s = "cbaebabacd", p = "abc" → [0, 6] |
| Brute force | Sort each window of size m and compare: O(n · m log m) |
| Optimize | Signals: "rearrangement" (#9) + "all positions" + fixed size (#27). Waste: rebuilding each window's signature. Fix: a sliding 26-count signature. O(26 · n) |
| Code | The Ch 25 template; invariant: `window` = counts of the last m characters |
| Test | p longer than s → []; s = p → [0]; all-equal letters → every start |

## 30.3 Gotchas & traps

**1. A sliding window on negative numbers.** The "sum ≤ k" shrink rule breaks; use prefix
sums (signal #15).

**2. "Subsequence" read as "substring".** The patterns differ completely (Ch 24, Ch 27).

**3. Sorting away information.** Sorting destroys indices (1 Two Sum) and positions (11).
Sort indices, or don't sort.

**4. Trusting the brute force's speed.** At n = 10⁵, O(n²) is about 10¹⁰ steps. Check the
budget before coding (Ch 10).

**5. Optimizing before having a correct version.** Keep the brute force as an oracle; every
chapter's test file does exactly that.

**6. Matching keywords without reading the constraints.** "Sorted" with n ≤ 100 doesn't
demand binary search; "anagram" over Unicode rules out the 26-slot list.

## 30.4 Benchmarks

None. This chapter organizes patterns already measured in Ch 13–29.

## 30.5 Exercises: name the pattern

For each statement (paraphrased; no problem numbers), name the pattern and the target
complexity. Answers, with the matching problem from this book, are in Appendix D.

1. Given a sorted array and a target, return the positions of two values that add up to
   it, using constant extra memory.
2. Count the contiguous subarrays whose sum is exactly k; values may be negative.
3. Find the length of the longest substring in which no character repeats.
4. For each day's temperature, report how many days until a warmer day.
5. Return the k values that appear most often, in better than O(n log n) time.
6. Given an array of length n + 1 with values in 1..n and one repeated value, find it
   without modifying the array, in constant space.
7. Find the smallest eating speed that finishes all piles within h hours.
8. Rotate an n × n image by 90° without allocating a second matrix.
9. Return the largest sum of any contiguous subarray.
10. Merge all overlapping intervals.
11. Decide whether two strings are anagrams.
12. Return the shortest substring of s containing every character of t, with multiplicity.
13. Evaluate an arithmetic expression with + − × ÷ without `eval`.
14. Find the maximum of every window of size k.
15. Return the first missing positive integer in O(n) time and O(1) space.
16. Find the longest proper prefix of s that is also a suffix.
17. Move all zeros to the end, keeping the order of the other values, in place.
18. Decide whether some rearrangement of s1 occurs as a substring of s2.
19. Find the minimum ship capacity to deliver packages in order within d days.
20. Count the palindromic substrings of a string.

## 30.6 Chapter cheat sheet

| Clue | Pattern |
|---|---|
| Unsorted pair → hash; sorted pair → two pointers | 14 / 16 |
| Contiguous + "= k" with negatives → prefix + map; contiguous + monotone constraint → window | 15 / 17, 27 |
| Max subarray → Kadane; nearest greater → monotonic stack; window max → deque | 20 / 21 |
| Sorted + log n → binary search; "minimum x such that" → search the answer | 19 |
| Intervals / k-th / greedy → sort; 0-1-2 → Dutch flag | 18 |
| 1..n values + O(1) space → index tricks; majority → voting; pairs cancel → XOR | 23 |
| Palindromes → two pointers / centers; pattern search → KMP / Z | 26 / 29 |
| Brackets, numbers, operators → stack + state machine | 28 |
| Always | Clarify → examples → brute force → optimize → code → test |
