# Chapter 31: Mixed Problem Sets

## 31.0 At a glance

Every problem below appeared earlier in the book. Here they're **shuffled, with no
chapter labels**, so the skill being tested is the one interviews test: recognizing the
pattern from the statement alone. The sets are:

- **Five sets of 8 problems**, each mixing patterns and difficulties
- **Two timed mock interviews**: 2 problems in 45 minutes each

Solutions come at the end (31.4), with the **recognition reasoning first** (which clue,
which pattern, which complexity), then the reference solution's file. All 44 references
are checked by `code/ch31_mixed/verify_sets.py`, which imports each solution and runs a
statement example.

**How to use these sets:**
1. Cover the solutions. For each problem, write down the pattern and target complexity
   **before** coding (Ch 30's attack method).
2. Give yourself 25–40 minutes per problem (front matter, 0.4). Code it from a blank
   file, with tests.
3. Only then compare with 31.4, and log the clue in your pattern notebook.

## 31.1 Problem sets

Statements are paraphrased in this book's words; follow the link for the original.

### Set A

| # | Title | Difficulty | Link | In brief |
|---|---|---|---|---|
| A1 | Contains Duplicate | Easy | <https://leetcode.com/problems/contains-duplicate/> | Does any value appear twice? |
| A2 | Squares of a Sorted Array | Easy | <https://leetcode.com/problems/squares-of-a-sorted-array/> | Sorted input (with negatives) → sorted squares, ideally in O(n) |
| A3 | Longest Substring Without Repeating Characters | Medium | <https://leetcode.com/problems/longest-substring-without-repeating-characters/> | Longest substring with all-distinct characters |
| A4 | Daily Temperatures | Medium | <https://leetcode.com/problems/daily-temperatures/> | Days until a warmer day, for every day |
| A5 | Merge Intervals | Medium | <https://leetcode.com/problems/merge-intervals/> | Merge every overlapping (or touching) interval |
| A6 | Valid Palindrome | Easy | <https://leetcode.com/problems/valid-palindrome/> | Palindrome check on letters and digits only, ignoring case |
| A7 | Product of Array Except Self | Medium | <https://leetcode.com/problems/product-of-array-except-self/> | Product of all other elements for each position, no division |
| A8 | Search in Rotated Sorted Array | Medium | <https://leetcode.com/problems/search-in-rotated-sorted-array/> | Find a target in a rotated sorted array in O(log n) |

### Set B

| # | Title | Difficulty | Link | In brief |
|---|---|---|---|---|
| B1 | Two Sum | Easy | <https://leetcode.com/problems/two-sum/> | Indices of two values summing to a target (unsorted) |
| B2 | Max Consecutive Ones III | Medium | <https://leetcode.com/problems/max-consecutive-ones-iii/> | Longest run of 1s if up to k zeros may be flipped |
| B3 | Valid Parentheses | Easy | <https://leetcode.com/problems/valid-parentheses/> | Are three kinds of brackets properly nested? |
| B4 | Rotate Image | Medium | <https://leetcode.com/problems/rotate-image/> | Rotate a square matrix 90° clockwise, in place |
| B5 | Majority Element | Easy | <https://leetcode.com/problems/majority-element/> | The value appearing more than n/2 times, in O(1) space |
| B6 | Binary Search | Easy | <https://leetcode.com/problems/binary-search/> | Index of a target in a sorted array, or −1 |
| B7 | Is Subsequence | Easy | <https://leetcode.com/problems/is-subsequence/> | Is s a subsequence of t? |
| B8 | Subarray Sum Equals K | Medium | <https://leetcode.com/problems/subarray-sum-equals-k/> | Count contiguous subarrays summing to k (negatives allowed) |

### Set C

| # | Title | Difficulty | Link | In brief |
|---|---|---|---|---|
| C1 | Move Zeroes | Easy | <https://leetcode.com/problems/move-zeroes/> | Zeros to the end, others keep their order, in place |
| C2 | Top K Frequent Elements | Medium | <https://leetcode.com/problems/top-k-frequent-elements/> | The k most frequent values, better than O(n log n) |
| C3 | Koko Eating Bananas | Medium | <https://leetcode.com/problems/koko-eating-bananas/> | Minimum eating speed to finish the piles within h hours |
| C4 | Longest Palindromic Substring | Medium | <https://leetcode.com/problems/longest-palindromic-substring/> | Any longest contiguous palindrome |
| C5 | Largest Rectangle in Histogram | Hard | <https://leetcode.com/problems/largest-rectangle-in-histogram/> | Largest rectangle under a bar chart |
| C6 | Find All Anagrams in a String | Medium | <https://leetcode.com/problems/find-all-anagrams-in-a-string/> | Every start where a rearrangement of p occurs in s |
| C7 | Plus One | Easy | <https://leetcode.com/problems/plus-one/> | Add one to a number stored as a digit array |
| C8 | Set Matrix Zeroes | Medium | <https://leetcode.com/problems/set-matrix-zeroes/> | Zero the row and column of every original zero, O(1) space |

### Set D

| # | Title | Difficulty | Link | In brief |
|---|---|---|---|---|
| D1 | Longest Consecutive Sequence | Medium | <https://leetcode.com/problems/longest-consecutive-sequence/> | Longest run of consecutive integers among the values, in O(n) |
| D2 | Container With Most Water | Medium | <https://leetcode.com/problems/container-with-most-water/> | Two lines holding the most water |
| D3 | Minimum Size Subarray Sum | Medium | <https://leetcode.com/problems/minimum-size-subarray-sum/> | Shortest subarray (positive values) with sum ≥ target |
| D4 | Decode String | Medium | <https://leetcode.com/problems/decode-string/> | Expand nested `k[...]` repetitions |
| D5 | Single Number | Easy | <https://leetcode.com/problems/single-number/> | Every value twice except one; O(1) space |
| D6 | Find Minimum in Rotated Sorted Array | Medium | <https://leetcode.com/problems/find-minimum-in-rotated-sorted-array/> | Minimum of a rotated sorted array in O(log n) |
| D7 | Palindromic Substrings | Medium | <https://leetcode.com/problems/palindromic-substrings/> | Count palindromic substrings by position |
| D8 | Subarray Sums Divisible by K | Medium | <https://leetcode.com/problems/subarray-sums-divisible-by-k/> | Count subarrays whose sum is divisible by k |

### Set E

| # | Title | Difficulty | Link | In brief |
|---|---|---|---|---|
| E1 | 3Sum | Medium | <https://leetcode.com/problems/3sum/> | All distinct triplets summing to zero |
| E2 | Minimum Window Substring | Hard | <https://leetcode.com/problems/minimum-window-substring/> | Shortest window of s covering t with multiplicity |
| E3 | Sliding Window Maximum | Hard | <https://leetcode.com/problems/sliding-window-maximum/> | Maximum of every window of size k, in O(n) |
| E4 | First Missing Positive | Hard | <https://leetcode.com/problems/first-missing-positive/> | Smallest missing positive, O(n) time and O(1) space |
| E5 | Multiply Strings | Medium | <https://leetcode.com/problems/multiply-strings/> | Product of two decimal strings, no integer conversion |
| E6 | Longest Happy Prefix | Hard | <https://leetcode.com/problems/longest-happy-prefix/> | Longest proper prefix that is also a suffix |
| E7 | Sort Array By Parity | Easy | <https://leetcode.com/problems/sort-array-by-parity/> | Evens before odds, any order |
| E8 | Find Peak Element | Medium | <https://leetcode.com/problems/find-peak-element/> | Any index larger than both neighbors, in O(log n) |

## 31.2 Timed mock interviews

Set a 45-minute timer per mock. Talk through steps 1–6 of the attack method (Ch 30,
30.2.3) out loud, as you would to an interviewer. Budget about 15 minutes for the first
problem and 30 for the second.

| Mock | Problem 1 | Problem 2 |
|---|---|---|
| Mock 1 | Group Anagrams (Medium): <https://leetcode.com/problems/group-anagrams/>. Group strings that are rearrangements of each other. | Trapping Rain Water (Hard): <https://leetcode.com/problems/trapping-rain-water/>. Water trapped between bars of given heights. |
| Mock 2 | Basic Calculator II (Medium): <https://leetcode.com/problems/basic-calculator-ii/>. Evaluate `+ - * /` with precedence, without eval. | Find the Duplicate Number (Medium): <https://leetcode.com/problems/find-the-duplicate-number/>. Find the repeated value; read-only array, O(1) space. |

**Scoring yourself** (interviewers grade along similar lines):

| Area | Full marks when |
|---|---|
| Communication | You restated the problem, asked about edge cases, and named the brute force first |
| Pattern | You named the pattern and why, before coding |
| Correctness | Your code passed your own edge cases without hints |
| Complexity | You stated time and auxiliary space, each with its reason |
| Time | Both problems were done within 45 minutes |

## 31.3 After each set

For every problem you missed or were slow on:

1. Write the **clue** you overlooked in your pattern notebook (front matter, 0.4).
2. Re-solve it from a blank file on the day 1 / 3 / 7 / 21 schedule (Appendix C).
3. Re-read the chapter's "Recognition signals" section for that pattern.

## 31.4 Solutions: don't read until you've attempted the set

Format: **clue → pattern → complexity**, then where the tested solution lives (`code/...`).

### Set A

| # | Clue → pattern | Complexity | Reference |
|---|---|---|---|
| A1 | "appears twice" → set | O(n) time, O(n) space | `ch14_hashing/practice_solutions.py` (`Solution217`) |
| A2 | Sorted, but the **largest squares sit at both ends** → two pointers, fill from the back | O(n), O(1) aux | `ch16_two_pointers/practice_solutions.py` (`Solution977`) |
| A3 | Longest contiguous + "no repeats" → window with last index | O(n), O(k) | `ch27_string_windows/longest_unique.py` |
| A4 | "Until a **warmer** day" = next greater → monotonic stack | O(n), O(n) | `ch21_monotonic/daily_temperatures.py` |
| A5 | Intervals + overlap → sort by start, sweep (touching merges) | O(n log n) | `ch18_sorting/merge_intervals.py` |
| A6 | Palindrome + ignore characters → two pointers that skip | O(n), O(1) | `ch26_palindromes/valid_palindrome.py` (`Solution125`) |
| A7 | "Except self" + "no division" → prefix × suffix products | O(n), O(1) aux (output excluded) | `ch15_prefix_sums/product_except_self.py` |
| A8 | Sorted + rotated + O(log n) → binary search on the sorted half | O(log n) | `ch19_binary_search/rotated_search.py` |

### Set B

| # | Clue → pattern | Complexity | Reference |
|---|---|---|---|
| B1 | Pair to a target, **unsorted**, indices → complement hash map | O(n) | `ch14_hashing/two_sum.py` |
| B2 | "At most k zeros" in a contiguous run → shrink-while-invalid window | O(n), O(1) | `ch17_sliding_window/max_ones.py` |
| B3 | Nesting, last opened closes first → stack | O(n) | `ch28_parsing/practice_solutions.py` (`Solution20`) |
| B4 | Rotate in place → transpose + reverse rows | O(n²), O(1) | `ch22_matrices/rotate_image.py` |
| B5 | "More than n/2" + O(1) space → Boyer–Moore voting | O(n), O(1) | `ch23_index_tricks/majority_element.py` |
| B6 | Sorted + find → binary search | O(log n) | `ch19_binary_search/practice_solutions.py` (`Solution704`) |
| B7 | Subsequence (not contiguous) → greedy two pointers | O(len t) | `ch24_strings/practice_solutions.py` (`Solution392`) |
| B8 | Contiguous + "= k" + **negatives** → prefix sum + map (not a window) | O(n) | `ch15_prefix_sums/subarray_sum_k.py` |

### Set C

| # | Clue → pattern | Complexity | Reference |
|---|---|---|---|
| C1 | In place + keep order → write pointer with swaps | O(n), O(1) | `ch13_array_basics/move_zeroes.py` |
| C2 | Top k by frequency, beat n log n → count + buckets | O(n) | `ch14_hashing/top_k_frequent.py` |
| C3 | "Minimum speed such that…" with a monotone check → binary search on the answer | O(n log max) | `ch19_binary_search/koko_bananas.py` |
| C4 | Longest palindromic **substring** → expand around centers | O(n²), O(1) | `ch26_palindromes/longest_palindromic_substring.py` |
| C5 | Each bar's nearest **lower** bars on both sides → increasing stack | O(n) | `ch21_monotonic/largest_rectangle.py` |
| C6 | Rearrangement + every position + fixed size → sliding count signature | O(n) | `ch25_string_hashing/find_anagrams.py` |
| C7 | Digits + add → carry from the right | O(d) | `ch13_array_basics/plus_one.py` |
| C8 | Row/column marking + O(1) space → first row/column as markers | O(m·n), O(1) | `ch22_matrices/set_matrix_zeroes.py` |

### Set D

| # | Clue → pattern | Complexity | Reference |
|---|---|---|---|
| D1 | Consecutive values, O(n), no sorting → set + start-of-run walks | O(n) | `ch14_hashing/longest_consecutive.py` |
| D2 | Area between two ends → two pointers, move the shorter side | O(n) | `ch16_two_pointers/container_most_water.py` |
| D3 | Shortest contiguous, **positive** values, sum ≥ target → window recording while valid | O(n) | `ch17_sliding_window/min_size_subarray.py` |
| D4 | Nested `k[...]` → stack of (text, count) | O(output) | `ch28_parsing/decode_string.py` |
| D5 | Pairs + one single + O(1) space → XOR | O(n), O(1) | `ch23_index_tricks/practice_solutions.py` (`Solution136`) |
| D6 | Rotated sorted + minimum → binary search comparing with `nums[hi]` | O(log n) | `ch19_binary_search/practice_solutions.py` (`Solution153`) |
| D7 | Count palindromic substrings → count while expanding centers | O(n²) | `ch26_palindromes/count_palindromes.py` |
| D8 | Contiguous + divisible by k → prefix remainders | O(n), O(k) | `ch15_prefix_sums/subarray_div_k.py` |

### Set E

| # | Clue → pattern | Complexity | Reference |
|---|---|---|---|
| E1 | Triplets, distinct, sum 0 → sort + fix one + two pointers, skip duplicates | O(n²) | `ch16_two_pointers/three_sum.py` |
| E2 | Shortest window covering t with multiplicity → need/have with `formed` | O(m + n) | `ch27_string_windows/min_window.py` |
| E3 | Max of every window, O(n) → monotonic deque | O(n), O(k) | `ch21_monotonic/sliding_window_max.py` |
| E4 | Values matter only in 1..n, O(1) space → cyclic sort | O(n), O(1) | `ch23_index_tricks/first_missing_positive.py` |
| E5 | "No integer conversion" + multiply → digit-position array | O(m·n) | `ch28_parsing/multiply_strings.py` |
| E6 | Prefix = suffix → prefix function, answer `pi[-1]` | O(n) | `ch29_string_matching/borders.py` (`Solution1392`) |
| E7 | Partition, any order → write pointer with swaps | O(n), O(1) | `ch13_array_basics/practice_solutions.py` (`Solution905`) |
| E8 | O(log n) on an **unsorted** array with −∞ edges → climb uphill by halves | O(log n) | `ch19_binary_search/peak_element.py` |

### Mocks

| Mock | Clue → pattern | Complexity | Reference |
|---|---|---|---|
| 1, problem 1 | Group by "same letters" → dict keyed by a signature (sorted string or count tuple) | O(N · L log L) or O(N · L) | `ch14_hashing/group_anagrams.py` |
| 1, problem 2 | Water above each bar = min(max left, max right) − height → two pointers, or prefix maxima | O(n), O(1) | `ch16_two_pointers/trapping_rain_water.py` |
| 2, problem 1 | Precedence without eval → stack of signed terms; truncating division (**not** `//`) | O(n) | `ch28_parsing/basic_calculator_ii.py` |
| 2, problem 2 | Read-only + O(1) space + values 1..n → Floyd's cycle detection (or binary search on values) | O(n), O(1) | `ch23_index_tricks/find_duplicate.py` |
