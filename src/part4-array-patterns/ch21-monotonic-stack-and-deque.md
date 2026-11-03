# Chapter 21: Monotonic Stack & Monotonic Deque

## 21.0 At a glance

- **What this pattern solves:** "for each element, find the **nearest** element to its
  left or right that is greater (or smaller)", and everything built on that: waiting
  times, spans, histogram rectangles, sliding-window maxima. Brute force scans ahead
  from every element (O(n²)). A stack that keeps its contents **sorted** answers every
  query in O(n) total.
- **Prerequisites:** Ch 8 (`deque`), Ch 10 (amortized analysis), Ch 17 (windows).
- **Typical complexity:** O(n) time (each index pushed and popped at most once); O(n)
  auxiliary space.
- **Tools used in this chapter:**
  - `list` as a stack, `collections.deque` **[USE-FREELY]**
  - `heapq` **[KNOW-HOW]**
  - monotonic stacks and monotonic deques **[HAND-WRITE]**

## 21.1 Recognition signals

- "**next greater / next smaller** element", "**how many days until** warmer"
- "**span**", "how many consecutive previous values are ≤ this one"
- "**largest rectangle**", "**trapping water**" (the bar-by-bar view)
- "**maximum / minimum of every window** of size k"
- Collisions or cancellations that only interact with the **most recent survivor**
  (735)

**Look-alikes:**

| Looks like | Actually | Tell |
|---|---|---|
| Next greater (stack) | Two pointers (Ch 16) | The answer depends on both ends at once (containers, trapped water) |
| Window maximum (deque) | Plain sliding window (Ch 17) | Sums and counts can be undone when an element leaves; a max can't |
| Stack of survivors | Simple counter | Matching brackets with one kind only: a counter suffices (Ch 28) |

## 21.2 Core idea

Brute force: from every index, scan forward until something bigger appears, which is
O(n²) on a falling sequence. The waste: when a big value arrives, it answers the
question for **every** smaller element still waiting, all at once.

Keep the **waiting elements on a stack**. While the new value is greater than the top,
the top has found its answer, so pop it. Then push the new value. The stack is always
**decreasing** from bottom to top, because anything smaller than a newer element has
already been popped.

```text
temperatures: 73 74 75 71 69 72 76 73

i=0 73   stack [73]
i=1 74   74 > 73 -> 73 answered (1 day)          stack [74]
i=2 75   75 > 74 -> 74 answered (1)              stack [75]
i=3 71                                            stack [75 71]
i=4 69                                            stack [75 71 69]
i=5 72   72 > 69 -> 69 (1); 72 > 71 -> 71 (2)    stack [75 72]
i=6 76   76 > 72 -> 72 (1); 76 > 75 -> 75 (4)    stack [76]
i=7 73                                            stack [76 73]   (no answer: 0)
```

Each index is pushed once and popped at most once, so the total work is O(n) even
though there's a `while` inside the `for` (Ch 10, 10.2.4).

## 21.3 The template

**Next greater element (indices):**

```python
def next_greater_index(nums: list[int]) -> list[int]:
    """For each i, the index of the next strictly greater value, or -1."""
    answer = [-1] * len(nums)
    stack: list[int] = []                    # indices; values decreasing upward
    for i, x in enumerate(nums):
        while stack and nums[stack[-1]] < x:
            answer[stack.pop()] = i
        stack.append(i)
    return answer


next_greater_index([2, 1, 2, 4, 3])  # -> [3, 2, 3, -1, -1]
```

- **Invariant:** the stack holds exactly the indices whose answer hasn't been found yet,
  in increasing index order, with values non-increasing from bottom to top.
- **Complexity:** O(n) time (each index is pushed once and popped at most once); O(n)
  auxiliary space (the stack, worst case on falling input).

| Want | Stack order (bottom → top) | Pop while |
|---|---|---|
| Next **greater** | decreasing | `nums[top] < x` |
| Next **greater or equal** | strictly decreasing | `nums[top] <= x` |
| Next **smaller** | increasing | `nums[top] > x` |
| **Previous** greater/smaller | the same stack: after popping, the new top is the previous one | |

**Monotonic deque** (window maximum): a decreasing deque of indices. Pop the back while
it's ≤ the new value, and pop the front when it leaves the window. 21.6.4 has the code.

## 21.4 Dry run

84 Largest Rectangle in Histogram on `[2, 1, 5, 6, 2, 3]` (21.6.3). The stack is
**increasing**. When a lower bar arrives, each popped bar's rectangle spans from just
after the new stack top to just before i. A sentinel height 0 at i = 6 flushes the
stack.

| i | h | pops (height × width) | area | stack after (indices) | best |
|---|---|---|---|---|---|
| 0 | 2 | | | [0] | 0 |
| 1 | 1 | bar 0: 2 × (1 − (−1) − 1) = 2 × 1 | 2 | [1] | 2 |
| 2 | 5 | | | [1, 2] | 2 |
| 3 | 6 | | | [1, 2, 3] | 2 |
| 4 | 2 | bar 3: 6 × (4 − 2 − 1) = 6; bar 2: 5 × (4 − 1 − 1) = 10 | 10 | [1, 4] | 10 |
| 5 | 3 | | | [1, 4, 5] | 10 |
| 6 | 0 | bar 5: 3 × 1 = 3; bar 4: 2 × (6 − 1 − 1) = 8; bar 1: 1 × 6 = 6 | | [6] | 10 |

Answer: 10 (heights 5 and 6, width 2).

## 21.5 Variations

| Variant | What changes | Example |
|---|---|---|
| Next greater via a lookup dict | Stack over values; record `value → next greater` | 496 Next Greater Element I |
| Circular | Loop over 2n indices with `i % n`; push only in the first pass | 503 Next Greater Element II |
| Distance to the next greater | Store indices; answer = `i - j` | 739 Daily Temperatures |
| Online span | Stack of `(price, span)` pairs; absorb popped spans | 901 Online Stock Span |
| Survivors after collisions | Stack of survivors; resolve against the top | 735 Asteroid Collision |
| Both nearest smaller bars | Increasing stack; pop = right boundary, new top = left boundary | 84 Largest Rectangle in Histogram |
| Window maximum | Decreasing **deque**; drop expired indices from the front | 239 Sliding Window Maximum |

## 21.6 Worked problems

### 21.6.1 739 Daily Temperatures (Medium): <https://leetcode.com/problems/daily-temperatures/>

**Problem.** For each day, return how many days you'd wait for a strictly warmer
temperature, or 0 if none comes.

**Signatures.** LeetCode:
`def dailyTemperatures(self, temperatures: List[int]) -> List[int]:`. Book: `list[int]`.

**Constraints → target.** n ≤ 10⁵. O(n²) on falling temperatures is 5 × 10⁹ steps:
hopeless. O(n).

**Approach 1: scan ahead from each day.**

<!-- file: code/ch21_monotonic/daily_temperatures.py -->

```python
class SolutionBruteForce:
    def dailyTemperatures(self, temperatures: list[int]) -> list[int]:
        # O(n^2) time worst case (falling temperatures): scan ahead from each day.
        # O(1) auxiliary besides the output.
        n = len(temperatures)
        answer = [0] * n
        for i in range(n):
            for j in range(i + 1, n):
                if temperatures[j] > temperatures[i]:
                    answer[i] = j - i
                    break
        return answer
```

**Approach 2: monotonic stack (optimal).**

<!-- file: code/ch21_monotonic/daily_temperatures.py -->

```python
class Solution:
    def dailyTemperatures(self, temperatures: list[int]) -> list[int]:
        # O(n) time: each index is pushed once and popped at most once.
        # O(n) auxiliary: the stack. Invariant: the stack holds indices of days
        # still waiting for a warmer day, with temperatures non-increasing from
        # bottom to top (a monotonic decreasing stack).
        answer = [0] * len(temperatures)
        stack: list[int] = []
        for i, t in enumerate(temperatures):
            while stack and temperatures[stack[-1]] < t:   # day i resolves them
                j = stack.pop()
                answer[j] = i - j
            stack.append(i)
        return answer                                      # leftovers keep 0
```

Why it works: the trace in 21.2. A day leaves the stack exactly when its first warmer day
arrives, which is the answer it needs. Store **indices**, not temperatures, so the
distance can be computed.

**Edge cases tested:**
- a single day, all equal (equal isn't warmer → 0)
- strictly rising
- n = 10⁵
- 1,000 random cases checked against brute force

**Common bugs:**
- `<=` in the pop condition, which treats equal as warmer.
- Pushing temperatures instead of indices.
- Initializing answers to −1 instead of 0.

**Built-in shortcut:** none (**[HAND-WRITE]**).

### 21.6.2 503 Next Greater Element II (Medium): <https://leetcode.com/problems/next-greater-element-ii/>

**Problem.** In a **circular** array, return for each element the first strictly greater
value found by moving forward and wrapping around, or −1.

**Signatures.** LeetCode: `def nextGreaterElements(self, nums: List[int]) -> List[int]:`.
Book: `list[int]`.

**Constraints → target.** n ≤ 10⁴. O(n²) is 10⁸: too slow in Python. O(n).

**Approach 1: look at the next n − 1 positions modulo n.**

<!-- file: code/ch21_monotonic/next_greater_circular.py -->

```python
class SolutionBruteForce:
    def nextGreaterElements(self, nums: list[int]) -> list[int]:
        # O(n^2) time: look at the next n - 1 positions (mod n). O(1) auxiliary.
        n = len(nums)
        answer = [-1] * n
        for i in range(n):
            for step in range(1, n):
                if nums[(i + step) % n] > nums[i]:
                    answer[i] = nums[(i + step) % n]
                    break
        return answer
```

**Approach 2: two passes over a monotonic stack (optimal).**

<!-- file: code/ch21_monotonic/next_greater_circular.py -->

```python
class Solution:
    def nextGreaterElements(self, nums: list[int]) -> list[int]:
        # O(n) time: two passes over the indices (2n steps), each index pushed
        # once. O(n) auxiliary: the stack. The second pass only resolves elements
        # left from the first; it pushes nothing new.
        n = len(nums)
        answer = [-1] * n
        stack: list[int] = []
        for k in range(2 * n):
            i = k % n                                   # Ch 3: % for wrap-around
            while stack and nums[stack[-1]] < nums[i]:
                answer[stack.pop()] = nums[i]
            if k < n:
                stack.append(i)
        return answer
```

Why two passes suffice: after the first pass, the waiting elements need a greater value
from **earlier** in the array (the wrap-around). The second pass replays every element
once more as a possible answer. Pushing nothing in the second pass keeps the work at
2n.

**Edge cases tested:**
- a single element, all equal, negatives
- 1,000 random cases checked against brute force

**Common bugs:**
- Doubling the array with `nums + nums` (works, but it's an O(n) copy and invites
  index confusion).
- Pushing in the second pass, which creates duplicate answers.
- A single pass, which misses the wrap-around answers.

**Built-in shortcut:** none.

### 21.6.3 84 Largest Rectangle in Histogram (Hard): <https://leetcode.com/problems/largest-rectangle-in-histogram/>

**Problem.** Bars of width 1 have the given heights. Return the area of the largest
rectangle that fits inside the histogram.

**Signatures.** LeetCode: `def largestRectangleArea(self, heights: List[int]) -> int:`.
Book: `list[int]`.

**Constraints → target.** n ≤ 10⁵, so O(n).

**Key fact.** The best rectangle has some bar as its **shortest** bar. For bar i, the
widest rectangle at height `heights[i]` stretches from just after the nearest
lower bar on the left to just before the nearest lower bar on the right. Two "nearest
smaller" queries, and that's a monotonic stack.

**Approach 1: every left edge, extend right.**

<!-- file: code/ch21_monotonic/largest_rectangle.py -->

```python
class SolutionBruteForce:
    def largestRectangleArea(self, heights: list[int]) -> int:
        # O(n^2) time: every left edge, extend right tracking the minimum height.
        # O(1) auxiliary space.
        best = 0
        for i in range(len(heights)):
            low = heights[i]
            for j in range(i, len(heights)):
                low = min(low, heights[j])
                best = max(best, low * (j - i + 1))
        return best
```

**Approach 2: increasing stack (optimal).**

<!-- file: code/ch21_monotonic/largest_rectangle.py -->

```python
class Solution:
    def largestRectangleArea(self, heights: list[int]) -> int:
        # O(n) time: each bar pushed and popped once. O(n) auxiliary: the stack.
        # Increasing stack of indices. When bar i is lower than the top, the top
        # bar can't extend right past i: its rectangle is bounded by i on the right
        # and by the new top of the stack (the previous lower bar) on the left.
        best = 0
        stack: list[int] = []
        for i in range(len(heights) + 1):
            h = heights[i] if i < len(heights) else 0      # sentinel flushes the stack
            while stack and heights[stack[-1]] >= h:
                height = heights[stack.pop()]
                left = stack[-1] if stack else -1          # first lower bar on the left
                best = max(best, height * (i - left - 1))
            stack.append(i)
        return best
```

Why one pop gives both boundaries: the stack is increasing, so the element below a
popped bar is the nearest bar to its left that's lower (or equal, see below). The bar
causing the pop is the nearest lower bar to its right. Width = `i - left - 1`. Popping on
`>=` means equal-height bars are resolved by the later one, which computes the full
width correctly. Traced in 21.4.

**Edge cases tested:**
- a single zero, all equal
- increasing, decreasing
- n = 10⁵ of height 10⁴ (area 10⁹)
- 1,000 random cases checked against brute force

**Common bugs:**
- Forgetting the sentinel, which leaves bars on the stack unprocessed.
- Width `i - left` instead of `i - left - 1`.
- Using the popped index as the left boundary instead of the **new** top.

**Built-in shortcut:** none (**[HAND-WRITE]**).

### 21.6.4 239 Sliding Window Maximum (Hard): <https://leetcode.com/problems/sliding-window-maximum/>

**Problem.** For each window of k consecutive elements, report its maximum. (Ch 17
solved this with a heap.)

**Signatures.** LeetCode: `def maxSlidingWindow(self, nums: List[int], k: int) -> List[int]:`.
Book: `list[int]`.

**Constraints → target.** n ≤ 10⁵ and k ≤ n. O(n · k) can reach 2.5 × 10⁹. The heap is
O(n log n); the deque is O(n).

**Approach 1: max of every window.**

<!-- file: code/ch21_monotonic/sliding_window_max.py -->

```python
class SolutionBruteForce:
    def maxSlidingWindow(self, nums: list[int], k: int) -> list[int]:
        # O(n * k) time: max of each window. O(k) auxiliary for the slice.
        return [max(nums[i:i + k]) for i in range(len(nums) - k + 1)]
```

**Approach 2: max-heap with lazy deletion (Ch 17).**

<!-- file: code/ch21_monotonic/sliding_window_max.py -->

```python
class SolutionHeap:
    def maxSlidingWindow(self, nums: list[int], k: int) -> list[int]:
        # O(n log n) time; O(n) auxiliary (stale entries linger). Ch 17's version.
        heap: list[tuple[int, int]] = []
        out = []
        for i, x in enumerate(nums):
            heapq.heappush(heap, (-x, i))
            while heap[0][1] <= i - k:
                heapq.heappop(heap)
            if i >= k - 1:
                out.append(-heap[0][0])
        return out
```

**Approach 3: monotonic deque (optimal).**

<!-- file: code/ch21_monotonic/sliding_window_max.py -->

```python
class Solution:
    def maxSlidingWindow(self, nums: list[int], k: int) -> list[int]:
        # O(n) time: each index enters and leaves the deque once. O(k) auxiliary.
        # Invariant: the deque holds indices of the current window whose values
        # are strictly decreasing front to back, so the front is the window max.
        # An element smaller than a newer one can never be a max again: drop it.
        window: deque[int] = deque()
        out = []
        for i, x in enumerate(nums):
            while window and nums[window[-1]] <= x:     # newer, at least as big
                window.pop()
            window.append(i)
            if window[0] <= i - k:                      # front slid out
                window.popleft()
            if i >= k - 1:
                out.append(nums[window[0]])
        return out
```

Why discarding is safe: if j < i and `nums[j] <= nums[i]`, then j leaves the window
before i does, and while both are inside, i is at least as large. So j can never be the
maximum again. Dropping such elements keeps the deque decreasing, and the front is
always the maximum of the current window. Expired indices can only be at the front,
because the deque is in index order.

**Edge cases tested:**
- k = 1, k = n
- all equal, decreasing
- n = 10⁵ with k = 500 (deque vs heap)
- 1,000 random cases across all three approaches

**Common bugs:**
- Storing values instead of indices, which makes it impossible to tell when the front
  expires.
- A `list` with `pop(0)` instead of a `deque` (O(n) per pop, Ch 6).
- Checking expiry before pushing, with an off-by-one in `i - k`.

**Built-in shortcut:** none for O(n). The heap version is **[KNOW-HOW]**.

## 21.7 Python-specific notes

**Deque vs heap, measured** (`bench/ch21_window_max.py`, 239 with n = 10⁵, best of 3, two
runs):

| Input, k | Deque, O(n) | Heap, O(n log n) |
|---|---|---|
| Random, k = 10 | 23–25 ms | 68–77 ms |
| Random, k = 10,000 | 27–28 ms | 38–40 ms |
| Increasing, k = 10 | 20–21 ms | 60–61 ms |
| Increasing, k = 10,000 | 21 ms | 62–65 ms |

The deque measured 1.4–3.1× faster in every case. Here the complexity difference does show
up in CPython, unlike quickselect vs `sorted` in Ch 18, because both versions run
Python-level loops.

**Stacks are lists.** `append` and `pop()` at the end are O(1) (Ch 6). Use a `deque` only
when you also remove from the front.

**Sentinels avoid special cases.** The extra height-0 bar in 84 (`i == len(heights)`)
flushes the stack without a second loop. It's a common, readable trick; state it when
you use it.

## 21.8 Practice set

Hints are in Appendix D, under "Chapter 21 practice". Reference solutions:
`code/ch21_monotonic/practice_solutions.py`.

| # | Title | Difficulty | Link | Hint ref |
|---|---|---|---|---|
| 496 | Next Greater Element I | Easy | <https://leetcode.com/problems/next-greater-element-i/> | D.21.1 |
| 901 | Online Stock Span | Medium | <https://leetcode.com/problems/online-stock-span/> | D.21.2 |
| 735 | Asteroid Collision | Medium | <https://leetcode.com/problems/asteroid-collision/> | D.21.3 |

## 21.9 Chapter cheat sheet

| | |
|---|---|
| **Signals** | Next/previous greater or smaller; days until; span; histogram; window max/min; survivors |
| **Template** | `for i, x: while stack and nums[stack[-1]] < x: answer[stack.pop()] = i; stack.append(i)` |
| **Orders** | Next greater → decreasing stack; next smaller → increasing stack; previous = the new top after popping |
| **Deque** | Decreasing deque of indices; pop the back while `<= x`; pop the front when expired |
| **Complexity** | O(n) time (each index pushed and popped once), O(n) space |
| **Top 3 bugs** | Values instead of indices; `<` vs `<=` (strict vs non-strict); missing sentinel / second pass |
| **CPython** | Deque measured 1.4–3.1× faster than the heap for 239 |
