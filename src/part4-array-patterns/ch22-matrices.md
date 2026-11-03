# Chapter 22: Matrices (2D Arrays)

## 22.0 At a glance

- **What this pattern solves:** grid traversal and transformation: rotating,
  transposing, walking in a spiral, marking rows and columns, simulating cell updates
  in place, and searching sorted grids.
- **Prerequisites:** Ch 3 (the `[[0] * n] * m` trap, copying grids), Ch 4 (grid
  comprehensions, `zip(*grid)`), Ch 13 (in-place writes), Ch 19 (searching sorted data).
- **Typical complexity:** O(m · n) time, since every cell is touched a constant number
  of times. The question is usually the **auxiliary space**: O(m · n) copy → O(m + n)
  markers → O(1).
- **Tools used in this chapter:**
  - indexing, `range`, comprehensions **[USE-FREELY]**
  - `zip(*grid)` for transposing **[KNOW-HOW]**
  - direction vectors, layer-by-layer rotation, boundary-shrinking spirals, in-place
    marker rows and bit-encoded states **[HAND-WRITE]**

## 22.1 Recognition signals

- "**rotate** the image / matrix", "**transpose**"
- "return the elements in **spiral** order"
- "**in place**", "**without a second 2D array**", "**O(1) extra space**" on a grid
- "if a cell is 0, set its whole **row and column**"
- "update **all cells simultaneously** from the current state" (cellular automata)
- "rows **and** columns are sorted" → staircase search

**Look-alikes:**

| Looks like | Actually | Tell |
|---|---|---|
| Grid traversal | Graph search (BFS/DFS) | "Connected regions", "islands", "shortest path in a grid": out of this book's scope |
| Sorted-grid search (240, staircase) | Binary search over m · n (74, Ch 19) | Each row starts after the previous row ends → one sorted sequence |
| In-place grid update | Copy + update | Small grid, no follow-up about space → a copy is fine |

## 22.2 Core idea

A matrix is a list of row lists. `matrix[r][c]` is row r, column c, with
`m = len(matrix)` rows and `n = len(matrix[0])` columns. Three ideas cover this chapter:

**1. Coordinates as arithmetic.** Each transformation is a formula for where cell (r, c)
goes:

| Transformation | (r, c) → |
|---|---|
| Transpose | (c, r) |
| Rotate 90° clockwise | (c, n − 1 − r) |
| Rotate 90° counterclockwise | (n − 1 − c, r) |
| Flip left-right | (r, n − 1 − c) |
| Flip up-down | (m − 1 − r, c) |

Clockwise rotation = transpose, then flip left-right. That's the in-place trick in 48.

**2. Direction vectors** turn "move right, then down, then left, then up" into data:

```python
DIRECTIONS = [(0, 1), (1, 0), (0, -1), (-1, 0)]   # right, down, left, up
r, c, d = 0, 0, 0
dr, dc = DIRECTIONS[d]
(r + dr, c + dc)  # -> (0, 1)
d = (d + 1) % 4                                    # turn clockwise
DIRECTIONS[d]  # -> (1, 0)
```

The 8-neighbor version (with diagonals) drives 289 Game of Life. Always bounds-check
with a chained comparison: `0 <= r < m and 0 <= c < n` (Ch 3).

**3. Reuse the grid as its own scratch space.** Before overwriting a cell, decide where
its old information goes:
- a copy (O(m · n))
- marker sets (O(m + n))
- the first row and column themselves (73)
- spare bits in each cell (289)

## 22.3 The template

**Neighbor iteration with bounds:**

```python
NEIGHBORS_4 = [(-1, 0), (1, 0), (0, -1), (0, 1)]


def neighbors(grid: list[list[int]], r: int, c: int) -> list[tuple[int, int]]:
    m, n = len(grid), len(grid[0])
    return [(r + dr, c + dc) for dr, dc in NEIGHBORS_4
            if 0 <= r + dr < m and 0 <= c + dc < n]


grid = [[1, 2, 3], [4, 5, 6]]
neighbors(grid, 0, 0)  # -> [(1, 0), (0, 1)]
len(neighbors(grid, 1, 1))  # -> 3
```

**Safe grid construction and copying** (Ch 3):

```python
m, n = 2, 3
grid = [[0] * n for _ in range(m)]                 # never [[0] * n] * m
copy = [row[:] for row in grid]                    # rows are int lists: enough
copy[0][0] = 9
(grid[0][0], copy[0][0])  # -> (0, 9)
```

- **Complexity:** construction and copying are O(m · n) time and space; one neighbor
  query is O(1) (4 or 8 checks).

## 22.4 Dry run

54 Spiral Matrix, boundary version (22.6.2), on a 3 × 4 matrix:

```text
 1  2  3  4
 5  6  7  8
 9 10 11 12
```

| Step | Boundaries before (top, bottom, left, right) | Walk | Appended |
|---|---|---|---|
| 1 | 0, 2, 0, 3 | top row → | 1 2 3 4 |
| 2 | 1, 2, 0, 3 | right column ↓ | 8 12 |
| 3 | 1, 2, 0, 2 | bottom row ← (top ≤ bottom) | 11 10 9 |
| 4 | 1, 1, 0, 2 | left column ↑ (left ≤ right) | 5 |
| 5 | 1, 1, 1, 2 | top row → | 6 7 |
| 6 | 2, 1, 1, 2 | right column ↓: empty range | |
| — | 2, 1, 1, 1 | bottom row skipped (top > bottom); left column skipped (empty range) | |

Result: `[1, 2, 3, 4, 8, 12, 11, 10, 9, 5, 6, 7]`. The two `if` guards stop a single
leftover row or column from being walked twice.

## 22.5 Variations

| Variant | What changes | Example |
|---|---|---|
| Transpose (any shape) | New n × m grid; `out[c][r] = grid[r][c]` | 867 Transpose Matrix |
| Rotate in place (square) | Transpose + reverse rows, or four-way swaps by layer | 48 Rotate Image |
| Spiral walk | Four shrinking boundaries, or directions + visited | 54 Spiral Matrix |
| Row/column marking | Marker sets → first row/column as markers + 2 flags | 73 Set Matrix Zeroes |
| Simultaneous update | Copy → encode the next state in a spare bit | 289 Game of Life |
| Search sorted rows and columns | Staircase from the top-right corner, O(m + n) | 240 Search a 2D Matrix II |

## 22.6 Worked problems

### 22.6.1 48 Rotate Image (Medium): <https://leetcode.com/problems/rotate-image/>

**Problem.** Rotate an n × n matrix 90° clockwise, **in place**. Allocating another 2D
matrix isn't allowed.

**Signatures.** LeetCode: `def rotate(self, matrix: List[List[int]]) -> None:`. Book:
`list[list[int]]`.

**Constraints → target.** n ≤ 20, so time doesn't matter. The point is O(1) extra space.

**Approach 1: build a rotated copy (the baseline; violates the rule).**

<!-- file: code/ch22_matrices/rotate_image.py -->

```python
class SolutionCopy:
    def rotate(self, matrix: list[list[int]]) -> None:
        # O(n^2) time. O(n^2) auxiliary: a full rotated copy (not allowed by the
        # problem; shown as the baseline). Cell (r, c) moves to (c, n - 1 - r).
        n = len(matrix)
        rotated = [[0] * n for _ in range(n)]
        for r in range(n):
            for c in range(n):
                rotated[c][n - 1 - r] = matrix[r][c]
        matrix[:] = rotated                       # mutate the caller's list (Ch 1)
```

**Approach 2: transpose, then reverse each row.**

<!-- file: code/ch22_matrices/rotate_image.py -->

```python
class SolutionTransposeReverse:
    def rotate(self, matrix: list[list[int]]) -> None:
        # O(n^2) time, O(1) auxiliary. Clockwise = transpose, then reverse each row.
        n = len(matrix)
        for r in range(n):
            for c in range(r + 1, n):             # upper triangle only
                matrix[r][c], matrix[c][r] = matrix[c][r], matrix[r][c]
        for row in matrix:
            row.reverse()
```

Why: the transpose sends (r, c) to (c, r), and reversing a row sends (c, r) to
(c, n − 1 − r). Composed, that's exactly the clockwise formula. The transpose loop
covers only the upper triangle (`c > r`); visiting every cell would swap each pair
twice and undo itself.

**Approach 3: four-way swaps, layer by layer.**

<!-- file: code/ch22_matrices/rotate_image.py -->

```python
class Solution:
    def rotate(self, matrix: list[list[int]]) -> None:
        # O(n^2) time, O(1) auxiliary. Rotate layer by layer; each 4-cycle of cells
        # moves with one temporary: top <- left <- bottom <- right <- top.
        n = len(matrix)
        for layer in range(n // 2):
            first, last = layer, n - 1 - layer
            for i in range(first, last):
                j = last - (i - first)                   # mirror of i in this layer
                top = matrix[first][i]
                matrix[first][i] = matrix[j][first]      # left -> top
                matrix[j][first] = matrix[last][j]       # bottom -> left
                matrix[last][j] = matrix[i][last]        # right -> bottom
                matrix[i][last] = top                    # top -> right
```

Approaches 2 and 3 are both O(n²) time and O(1) space. Approach 2 is easier to get right
under pressure. Approach 3 moves each cell exactly once, which is what some
interviewers want to see.

**Edge cases tested:**
- 1 × 1, 2 × 2
- the two statement examples
- 300 random matrices up to 8 × 8, checked against `zip(*m[::-1])`

**Common bugs:**
- Transposing the full square, which double-swaps and returns the original.
- `matrix = rotated`, which rebinds the name; the judge sees nothing (Ch 1).
- Index slips in the four-way swap. Write the four corner positions for one layer
  first.

**Built-in shortcut:** `matrix[:] = [list(row) for row in zip(*matrix[::-1])]`
**[KNOW-HOW]**. It's one line and correct, but it builds a new grid (O(n²) space), so it
violates the rule.

### 22.6.2 54 Spiral Matrix (Medium): <https://leetcode.com/problems/spiral-matrix/>

**Problem.** Return all elements of an m × n matrix in clockwise spiral order, starting
at the top-left.

**Signatures.** LeetCode: `def spiralOrder(self, matrix: List[List[int]]) -> List[int]:`.
Book: `list[list[int]]` → `list[int]`.

**Constraints → target.** m, n ≤ 10. Every approach is O(m · n). The difference is
correctness and extra space.

**Approach 1: walk with directions and a visited grid.**

<!-- file: code/ch22_matrices/spiral_matrix.py -->

```python
class SolutionVisited:
    def spiralOrder(self, matrix: list[list[int]]) -> list[int]:
        # O(m * n) time: each cell visited once. O(m * n) auxiliary: the visited grid.
        # Walk; turn clockwise when the next cell is outside or already visited.
        m, n = len(matrix), len(matrix[0])
        seen = [[False] * n for _ in range(m)]
        r = c = d = 0
        out = []
        for _ in range(m * n):
            out.append(matrix[r][c])
            seen[r][c] = True
            dr, dc = DIRECTIONS[d]
            nr, nc = r + dr, c + dc
            if not (0 <= nr < m and 0 <= nc < n) or seen[nr][nc]:
                d = (d + 1) % 4                        # turn right
                dr, dc = DIRECTIONS[d]
                nr, nc = r + dr, c + dc
            r, c = nr, nc
        return out
```

**Approach 2: four shrinking boundaries (optimal space).**

<!-- file: code/ch22_matrices/spiral_matrix.py -->

```python
class Solution:
    def spiralOrder(self, matrix: list[list[int]]) -> list[int]:
        # O(m * n) time, O(1) auxiliary besides the output: four shrinking
        # boundaries instead of a visited grid.
        top, bottom = 0, len(matrix) - 1
        left, right = 0, len(matrix[0]) - 1
        out = []
        while top <= bottom and left <= right:
            for c in range(left, right + 1):           # top row, left -> right
                out.append(matrix[top][c])
            top += 1
            for r in range(top, bottom + 1):           # right column, down
                out.append(matrix[r][right])
            right -= 1
            if top <= bottom:                          # bottom row still exists?
                for c in range(right, left - 1, -1):
                    out.append(matrix[bottom][c])
                bottom -= 1
            if left <= right:                          # left column still exists?
                for r in range(bottom, top - 1, -1):
                    out.append(matrix[r][left])
                left += 1
        return out
```

Why the guards: after the top row and right column, the remaining region may be empty
or a single line. Without `if top <= bottom`, a single remaining row would be read
twice (once left to right, once right to left). Traced in 22.4.

**Edge cases tested:**
- 1 × 1, a single row, a single column
- 300 random shapes up to 10 × 10, checked between both approaches and for length
  m · n

**Common bugs:**
- Missing the two guards, which duplicates the middle row or column of non-square
  matrices.
- Off-by-one in reversed ranges (`range(right, left - 1, -1)`).
- Mutating the input (popping rows), which works but destroys the caller's data.

**Built-in shortcut:** "pop the first row, rotate the rest counterclockwise, repeat"
with `zip`. It's correct, but O((m · n) · min(m, n)), because it rebuilds the matrix
every round. It's a clever aside, not an answer.

### 22.6.3 73 Set Matrix Zeroes (Medium): <https://leetcode.com/problems/set-matrix-zeroes/>

**Problem.** For every cell that is **originally** 0, set its whole row and column to 0,
in place. The follow-up asks for constant extra space.

**Signatures.** LeetCode: `def setZeroes(self, matrix: List[List[int]]) -> None:`. Book:
`list[list[int]]`.

**Constraints → target.** m, n ≤ 200. O(m · n) time; the follow-up wants O(1) space.

**Approach 1: copy, then clear (O(m · n) space).**

<!-- file: code/ch22_matrices/set_matrix_zeroes.py -->

```python
class SolutionCopy:
    def setZeroes(self, matrix: list[list[int]]) -> None:
        # O(m * n * (m + n)) time: for each original zero, clear a row and a column.
        # O(m * n) auxiliary: a copy, so new zeros don't trigger more clearing.
        original = [row[:] for row in matrix]
        m, n = len(matrix), len(matrix[0])
        for r in range(m):
            for c in range(n):
                if original[r][c] == 0:
                    for k in range(n):
                        matrix[r][k] = 0
                    for k in range(m):
                        matrix[k][c] = 0
```

The copy matters: clearing in place as you scan creates new zeros that would wrongly
trigger more clearing.

**Approach 2: remember the rows and columns (O(m + n) space).**

<!-- file: code/ch22_matrices/set_matrix_zeroes.py -->

```python
class SolutionSets:
    def setZeroes(self, matrix: list[list[int]]) -> None:
        # O(m * n) time: two passes. O(m + n) auxiliary: the marked rows and columns.
        zero_rows: set[int] = set()
        zero_cols: set[int] = set()
        for r, row in enumerate(matrix):
            for c, v in enumerate(row):
                if v == 0:
                    zero_rows.add(r)
                    zero_cols.add(c)
        for r, row in enumerate(matrix):
            for c in range(len(row)):
                if r in zero_rows or c in zero_cols:
                    row[c] = 0
```

**Approach 3: the first row and column as markers (O(1) space, optimal).**

<!-- file: code/ch22_matrices/set_matrix_zeroes.py -->

```python
class Solution:
    def setZeroes(self, matrix: list[list[int]]) -> None:
        # O(m * n) time, O(1) auxiliary (the follow-up). Use row 0 and column 0 as
        # the marker arrays; two flags remember whether row 0 / column 0 themselves
        # must be zeroed (their cells are about to be overwritten by markers).
        m, n = len(matrix), len(matrix[0])
        first_row_zero = any(matrix[0][c] == 0 for c in range(n))
        first_col_zero = any(matrix[r][0] == 0 for r in range(m))
        for r in range(1, m):
            for c in range(1, n):
                if matrix[r][c] == 0:
                    matrix[r][0] = 0                 # mark row r
                    matrix[0][c] = 0                 # mark column c
        for r in range(1, m):
            for c in range(1, n):
                if matrix[r][0] == 0 or matrix[0][c] == 0:
                    matrix[r][c] = 0
        if first_row_zero:
            for c in range(n):
                matrix[0][c] = 0
        if first_col_zero:
            for r in range(m):
                matrix[r][0] = 0
```

Why the order matters: the markers live in row 0 and column 0, so those must be cleared
**last**, after the interior has read them. Their own original zero state is saved in
the two flags first.

**Edge cases tested:**
- 1 × 1, a single row, a single column
- no zeros
- a zero in the corner (the statement's second example)
- 500 random grids checked across all three approaches

**Common bugs:**
- Clearing row 0 / column 0 before processing the interior, which wipes the markers.
- One shared flag for `matrix[0][0]` (it belongs to both the first row and the first
  column).
- Clearing in place during the scan.

**Built-in shortcut:** none; the marker trick is **[HAND-WRITE]**.

### 22.6.4 289 Game of Life (Medium): <https://leetcode.com/problems/game-of-life/>

**Problem.** Each cell is alive (1) or dead (0). Every cell's next state depends on its
8 neighbors: a live cell survives with 2 or 3 live neighbors, and a dead cell comes
alive with exactly 3. All cells update **simultaneously**, in place. The follow-up asks
for an in-place solution.

**Signatures.** LeetCode: `def gameOfLife(self, board: List[List[int]]) -> None:`. Book:
`list[list[int]]`.

**Constraints → target.** m, n ≤ 25. O(m · n). The challenge is the simultaneous update
without a copy.

**Approach 1: compute from a copy.**

<!-- file: code/ch22_matrices/game_of_life.py -->

```python
class SolutionCopy:
    def gameOfLife(self, board: list[list[int]]) -> None:
        # O(m * n) time (8 neighbors per cell). O(m * n) auxiliary: a copy of the
        # current state, so updates don't leak into neighbor counts.
        m, n = len(board), len(board[0])
        current = [row[:] for row in board]
        for r in range(m):
            for c in range(n):
                live = sum(current[r + dr][c + dc] for dr, dc in NEIGHBORS
                           if 0 <= r + dr < m and 0 <= c + dc < n)
                board[r][c] = next_state(current[r][c], live)
```

**Approach 2: two bits per cell (O(1) space, optimal).**

<!-- file: code/ch22_matrices/game_of_life.py -->

```python
class Solution:
    def gameOfLife(self, board: list[list[int]]) -> None:
        # O(m * n) time, O(1) auxiliary (the follow-up). Two bits per cell:
        # bit 0 = current state, bit 1 = next state. Reading `cell & 1` always gives
        # the current state, even after bit 1 has been written. Then shift right.
        m, n = len(board), len(board[0])
        for r in range(m):
            for c in range(n):
                live = 0
                for dr, dc in NEIGHBORS:
                    rr, cc = r + dr, c + dc
                    if 0 <= rr < m and 0 <= cc < n:
                        live += board[rr][cc] & 1
                if next_state(board[r][c] & 1, live):
                    board[r][c] |= 2                 # set the next-state bit
        for r in range(m):
            for c in range(n):
                board[r][c] >>= 1                    # next state becomes current
```

Both use the same rule helper:

<!-- file: code/ch22_matrices/game_of_life.py -->

```python
def next_state(alive: int, live_neighbors: int) -> int:
    if alive:
        return 1 if live_neighbors in (2, 3) else 0
    return 1 if live_neighbors == 3 else 0
```

Why the bit encoding works: each cell stores two states at once. Neighbor counts read
only bit 0 (`& 1`), which the first pass never changes. The second pass shifts every
cell, making bit 1 the new state. The same idea appears as "use values 2 and 3 to mean
dead→alive and alive→dead" in many write-ups.

**Edge cases tested:**
- the two statement examples
- 1 × 1 (a lonely cell dies)
- all dead
- 300 random boards checked against the copy version

**Common bugs:**
- Updating in place with plain 0/1, which corrupts neighbor counts for later cells.
- Counting the cell itself as its own neighbor.
- Forgetting the bounds check at edges (Python's negative indexing silently wraps to
  the other side: `board[-1]` is the last row, not an error).

**Built-in shortcut:** none needed.

## 22.7 Python-specific notes

- **Negative indices wrap silently.** `grid[r - 1]` with r = 0 reads the **last** row
  without any error. That's the classic grid bug in Python, where C++ would at least
  sometimes crash. Always bounds-check neighbors explicitly.
- **`zip(*grid)` transposes** in one line (Ch 4), returning tuples. Use
  `[list(col) for col in zip(*grid)]` when you need lists. Rotations: clockwise is
  `zip(*grid[::-1])`, counterclockwise is `list(zip(*grid))[::-1]`.
- **Rows are separate lists.** `grid[r][c]` costs two index operations. In hot loops,
  binding `row = grid[r]` once per row saves one per cell (general knowledge, not
  benchmarked here; the grids in this chapter are small).
- **Copy rows, not the outer list:** `[row[:] for row in grid]`. `grid[:]` shares the
  rows (Ch 3).

```python
grid = [[1, 2, 3], [4, 5, 6]]
[list(row) for row in zip(*grid[::-1])]  # -> [[4, 1], [5, 2], [6, 3]]
[list(row) for row in zip(*grid)][::-1]  # -> [[3, 6], [2, 5], [1, 4]]
grid[-1][0]  # -> 4
```

## 22.8 Practice set

Hints are in Appendix D, under "Chapter 22 practice". Reference solutions:
`code/ch22_matrices/practice_solutions.py`.

| # | Title | Difficulty | Link | Hint ref |
|---|---|---|---|---|
| 867 | Transpose Matrix | Easy | <https://leetcode.com/problems/transpose-matrix/> | D.22.1 |
| 240 | Search a 2D Matrix II | Medium | <https://leetcode.com/problems/search-a-2d-matrix-ii/> | D.22.2 |

## 22.9 Chapter cheat sheet

| | |
|---|---|
| **Signals** | Rotate/transpose, spiral, in-place grid, row/column marking, simultaneous update, sorted rows and columns |
| **Formulas** | Transpose (r, c)→(c, r); clockwise (r, c)→(c, n−1−r) = transpose + reverse rows |
| **Directions** | `[(0, 1), (1, 0), (0, -1), (-1, 0)]`; turn with `d = (d + 1) % 4`; bounds-check every neighbor |
| **Space ladder** | Copy O(mn) → marker sets O(m+n) → first row/col markers O(1) → spare bits O(1) |
| **Complexity** | O(m · n) time; staircase search O(m + n) |
| **Top 3 bugs** | Negative indices wrapping silently; full-square transpose (double swap); missing spiral guards |
