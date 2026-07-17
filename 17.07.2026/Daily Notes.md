## Part 3 — The GCD Pairs Problem (LeetCode)

### What is it asking? (plain English)
1. Take a list of numbers, e.g. `nums = [2, 3, 4]`.
2. Find the **GCD** of **every possible pair**.
3. Put those GCDs in a list, **sorted smallest → biggest**.
4. Answer questions like *"what's the value at index 0? index 2?"*

### What's a GCD?
**Greatest Common Divisor** = the biggest number that divides both numbers evenly.
- `gcd(2, 4)` = **2**
- `gcd(2, 3)` = **1**
- `gcd(6, 9)` = **3**

### Worked example: `nums = [2, 3, 4]`

| Pair | GCD |
|------|-----|
| (2, 3) | 1 |
| (2, 4) | 2 |
| (3, 4) | 1 |

Raw list `[1, 2, 1]` → sorted `[1, 1, 2]`.

Queries `[0, 2, 2]` (indexes start at **0**):
```
[1, 1, 2]
 ↑  ↑  ↑
 0  1  2
```
- index 0 → 1
- index 2 → 2
- index 2 → 2

**Answer: `[1, 2, 2]`** ✅

---

## Part 4 — Brute-force solution (correct, but slow)

```python
from math import gcd
from itertools import combinations

def solve(nums, queries):
    # make every pair's GCD, sort ascending
    gcd_pairs = sorted(gcd(a, b) for a, b in combinations(nums, 2))
    # look up each query index
    return [gcd_pairs[q] for q in queries]
```

- **`combinations(nums, 2)`** automatically gives every pair — no manual loops.
- **`gcd_pairs[q]`** reads the value at position `q`.

### Why it can go wrong 💥
The trap is *"every possible pair."* The number of pairs grows fast:

$$\frac{n \times (n-1)}{2}$$

| `n` | Number of pairs |
|---|---|
| 3 | 3 |
| 1,000 | ~500,000 |
| 100,000 | **~5,000,000,000** |

LeetCode allows `n` up to `100,000` → **5 billion pairs**. Two failures happen:
- ⏱️ **Time Limit Exceeded (TLE)** — too many operations for the few seconds allowed.
- 🧠 **Memory Limit Exceeded (MLE)** — a 5-billion-item list won't fit in memory.

> The small examples pass, so it *feels* right — but LeetCode secretly tests giant inputs where brute-force collapses.

---

## Part 5 — The fast solution (counting trick)

Instead of listing every pair, **count how many pairs share each GCD value** using number theory. The most it ever loops over is the biggest number (~50,000), not billions of pairs.

```python
from bisect import bisect_right

class Solution(object):
    def gcdValues(self, nums, queries):
        maxV = max(nums)

        # frequency of each value
        cnt = [0] * (maxV + 1)
        for v in nums:
            cnt[v] += 1

        # gcd_count[g] = number of pairs whose GCD is exactly g
        gcd_count = [0] * (maxV + 1)
        for g in range(maxV, 0, -1):        # high -> low
            d = 0                            # how many numbers divisible by g
            for multiple in range(g, maxV + 1, g):
                d += cnt[multiple]
            pairs = d * (d - 1) // 2         # pairs whose GCD is a MULTIPLE of g
            for k in range(2 * g, maxV + 1, g):
                pairs -= gcd_count[k]        # remove strict multiples
            gcd_count[g] = pairs

        # prefix sums -> cumulative count of pairs with GCD <= g
        prefix = [0] * (maxV + 1)
        running = 0
        for g in range(1, maxV + 1):
            running += gcd_count[g]
            prefix[g] = running

        # answer each query with binary search
        return [bisect_right(prefix, k) for k in queries]
```

### The counting crowd analogy 🏟️
- **Brute-force** = pointing at each person: "1, 2, 3…" — impossible for a stadium.
- **Fast version** = "40 rows × 100 seats, all full → 4,000" — instant, no matter the size.

### The one idea to remember
> When you can't enumerate pairs directly, **count them by a shared property** (their GCD). Count "multiples of g" pairs first, then **subtract strict multiples from high to low** to isolate "exactly g".

---

## Part 6 — Understanding the LeetCode template

```python
class Solution:
    def gcdValues(self, nums: List[int], queries: List[int]) -> List
```

LeetCode **calls your function for you** behind the scenes, so it needs a specific shape.

| Part | What it means |
|---|---|
| `class Solution:` | A container. Must be named `Solution`. |
| `def gcdValues(...)` | The function LeetCode calls. **Don't rename it.** |
| `self` | Required first word for methods in a class. You won't use it — just leave it. |
| `nums: List[int]` | The input `nums` is a **list of integers**. |
| `queries: List[int]` | The input `queries` is a **list of integers**. |
| `-> List[int]` | You must **return** a list of integers. |

### Type hints (the `: List[int]` and `-> List[int]` bits)
These are **type hints** — they tell you (and other readers) what type each input is and what the function returns:
- `nums: List[int]` → "`nums` is a list of ints"
- `-> List[int]` → "this function returns a list of ints"

They're just **notes for humans** — Python doesn't enforce them, and your code runs the same with or without them. LeetCode's Python3 template includes them because they're the modern, readable style.

### The #1 beginner gotcha 🎯
> **`return` the answer — never `print` it.** LeetCode reads what you `return`. If you `print`, it sees "nothing returned" and marks it wrong.

### Job-application analogy
- `class Solution` = the form (must use theirs).
- `def gcdValues(self, nums, queries)` = the labelled input boxes.
- `return` = handing the completed form back.
  
---

## Part 7 — Python 2 vs Python 3

### The error we hit
```
ImportError: cannot import name gcd
```
`math.gcd` only exists in **Python 3.5+**. In **Python 2** it doesn't → the import fails. The `(object)` template and this error mean the language was set to old **Python 2**.

### Which Python should you use?
> **Python3 — always.** 🙂

- Python 2 officially **died in January 2020** — no updates, no security fixes.
- Every job, tutorial, and library assumes Python 3.
- Fixes the `gcd` error instantly.

### On LeetCode's dropdown ⚠️
| Option | Use it? |
|---|---|
| **Python3** | ✅ Yes |
| **Python** | ❌ No — this is the old Python 2 |

The one *without* the "3" is the outdated one — sneaky!

### Three ways to fix the gcd error
```python
# Fix 1 (best): switch dropdown to Python3, then:
from math import gcd

# Fix 2: Python 2's home for gcd
from fractions import gcd

# Fix 3: write it yourself (works everywhere) - Euclidean algorithm
def gcd(a, b):
    while b:
        a, b = b, a % b
    return a
```

---
