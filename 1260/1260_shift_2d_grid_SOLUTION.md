# LeetCode 1260 – Shift 2D Grid (Solution Notes)

## 💡 The key insight

The shift rules describe a **1D rotation in disguise**:

- Move right one cell.
- At the end of a row → wrap to the start of the next row.
- At the very last cell → wrap back to the very first cell.

That is *exactly* what happens if you **flatten the grid into one long list**, rotate it by `k`, then fold it back into rows. So the whole problem is just: **rotate a 1D array `k` steps to the right**.

---

## ✅ Full solution

```python
from typing import List

class Solution:
    def shiftGrid(self, grid: List[List[int]], k: int) -> List[List[int]]:
        m, n = len(grid), len(grid[0])
        total = m * n

        # 1. Flatten the grid into a 1D list
        flat = [grid[i][j] for i in range(m) for j in range(n)]

        # 2. Build the new flat list: each element moves k spots forward
        new_flat = [0] * total
        for i in range(total):
            new_flat[(i + k) % total] = flat[i]

        # 3. Fold the 1D list back into an m x n grid
        result = []
        for i in range(m):
            row = new_flat[i * n : (i + 1) * n]
            result.append(row)

        return result
```

---

## 🔍 Line-by-line explanation

### `m, n = len(grid), len(grid[0])`

This is **tuple unpacking** — assigning two variables on one line.

- `len(grid)` = how many **rows** the grid has → stored in `m`.
- `len(grid[0])` = the length of the **first row**, i.e. how many **columns** → stored in `n`.

Think of `grid` as a list of lists. The outer list's length is the row count. Any inner list's length is the column count (every row has the same length, so we just check row `0`).

```python
grid = [[1,2,3],
        [4,5,6],
        [7,8,9]]

len(grid)      # 3  -> m (3 rows)
len(grid[0])   # 3  -> n (3 columns, from row [1,2,3])
```

It's just shorthand for:

```python
m = len(grid)
n = len(grid[0])
```

---

### `total = m * n`

The total number of cells in the grid. For a 3×3 grid that's `9`.

We need this for the wrap-around maths. When an element goes past the last position, `% total` sends it back to the front.

---

### `flat = [grid[i][j] for i in range(m) for j in range(n)]`

A **list comprehension** that reads every cell, row by row, into one flat 1D list.

- Outer loop `for i in range(m)` → walks through each row.
- Inner loop `for j in range(n)` → walks through each column in that row.
- `grid[i][j]` → the value at row `i`, column `j`.

The order matters: it goes left-to-right along row 0, then row 1, etc. — which is the same order the shift "wraps" in.

```python
# For the 3x3 grid above:
flat = [1, 2, 3, 4, 5, 6, 7, 8, 9]
```

Equivalent long form:

```python
flat = []
for i in range(m):
    for j in range(n):
        flat.append(grid[i][j])
```

---

### `new_flat = [0] * total`

Creates a fresh list of the right size, pre-filled with zeros. This is our destination — we'll overwrite every slot with the shifted value.

```python
new_flat = [0, 0, 0, 0, 0, 0, 0, 0, 0]  # 9 zeros
```

---

### `for i in range(total): new_flat[(i + k) % total] = flat[i]`

This is the heart of the rotation.

- `i` = the element's **original** position in `flat`.
- `(i + k)` = where it should go after shifting `k` steps forward.
- `% total` = wrap it around if it goes past the end.

So each value from `flat[i]` gets placed into its new home in `new_flat`.

**Example** with `k = 1`, `total = 9`:

| i | flat[i] | (i + k) % total | lands at |
|---|---------|-----------------|----------|
| 0 | 1 | 1 | new_flat[1] |
| 1 | 2 | 2 | new_flat[2] |
| ... | ... | ... | ... |
| 8 | 9 | (8+1)%9 = 0 | new_flat[0] |

Result: `new_flat = [9, 1, 2, 3, 4, 5, 6, 7, 8]` ✅

> The `%` is also why `k = 9` on 9 cells changes nothing: `(i + 9) % 9 == i`.

---

### The folding loop

```python
result = []
for i in range(m):
    row = new_flat[i * n : (i + 1) * n]
    result.append(row)
```

This turns the flat 1D list **back into a 2D grid** of `m` rows, each `n` long.

**`result = []`** → start with an empty list that will hold our rows.

**`for i in range(m):`** → loop once per row (`m` rows total).

**`row = new_flat[i * n : (i + 1) * n]`** → this is Python **slicing**: `list[start:stop]` grabs elements from index `start` up to *but not including* `stop`.

- `i * n` = the start index of row `i`.
- `(i + 1) * n` = the start index of the *next* row (the stop point).

So each slice pulls out exactly `n` consecutive elements = one row.

With `n = 3` and `new_flat = [9,1,2,3,4,5,6,7,8]`:

| i | `i*n` | `(i+1)*n` | slice | row |
|---|-------|-----------|-------|-----|
| 0 | 0 | 3 | `new_flat[0:3]` | `[9, 1, 2]` |
| 1 | 3 | 6 | `new_flat[3:6]` | `[3, 4, 5]` |
| 2 | 6 | 9 | `new_flat[6:9]` | `[6, 7, 8]` |

**`result.append(row)`** → add each row to the result.

Final: `result = [[9,1,2],[3,4,5],[6,7,8]]` ✅

---

## 🧪 Full trace (Example 1)

```
grid = [[1,2,3],[4,5,6],[7,8,9]], k = 1

m, n     = 3, 3
total    = 9
flat     = [1,2,3,4,5,6,7,8,9]
new_flat = [9,1,2,3,4,5,6,7,8]
result   = [[9,1,2],[3,4,5],[6,7,8]]   ✅
```

---

## ⏱️ Complexity

| | Value | Why |
|---|-------|-----|
| **Time** | `O(m * n)` | Each cell is touched a constant number of times |
| **Space** | `O(m * n)` | We build `flat` and `new_flat` |

This is optimal — you can't do better than visiting every element, since every element potentially moves.

---

## 🎯 Takeaway pattern

Whenever a 2D problem has **row-wrapping** behaviour, try **flatten → solve in 1D → reshape**. It usually turns a fiddly 2D problem into a simple 1D one. The two reusable tricks here are:

1. **`(i + k) % total`** for any circular/rotation shift.
2. **`list[i*n : (i+1)*n]`** to chop a flat list back into rows of length `n`.
