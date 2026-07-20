# LeetCode 1260 – Shift 2D Grid

## Problem

Given a 2D `grid` of size `m x n` and an integer `k`. You need to shift the `grid` `k` times.

In one shift operation:

1. Element at `grid[i][j]` moves to `grid[i][j + 1]`.
2. Element at `grid[i][n - 1]` moves to `grid[i + 1][0]`.
3. Element at `grid[m - 1][n - 1]` moves to `grid[0][0]`.

Return the 2D grid after applying shift operation `k` times.

---

## Examples

### Example 1
**Input:** `grid = [[1,2,3],[4,5,6],[7,8,9]]`, `k = 1`
**Output:** `[[9,1,2],[3,4,5],[6,7,8]]`

### Example 2
**Input:** `grid = [[3,8,1,9],[19,7,2,5],[4,6,11,10],[12,0,21,13]]`, `k = 4`
**Output:** `[[12,0,21,13],[3,8,1,9],[19,7,2,5],[4,6,11,10]]`

### Example 3
**Input:** `grid = [[1,2,3],[4,5,6],[7,8,9]]`, `k = 9`
**Output:** `[[1,2,3],[4,5,6],[7,8,9]]`

> Note: There are 9 cells and `k = 9`, so every element makes one full loop and lands back where it started — the grid is unchanged.

---

## Constraints

| Constraint | Meaning |
|------------|---------|
| `m == grid.length` | `m` is the number of rows |
| `n == grid[i].length` | `n` is the number of columns |
| `1 <= m <= 50` | Up to 50 rows |
| `1 <= n <= 50` | Up to 50 columns |
| `-1000 <= grid[i][j] <= 1000` | Range of cell values |
| `0 <= k <= 100` | Number of shifts (can be 0) |
