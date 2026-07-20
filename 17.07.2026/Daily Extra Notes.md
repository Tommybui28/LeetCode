# 🧠 GCD Pair Queries — Line-by-Line Explanation

A full walkthrough of the fast (memory-efficient) solution to **LeetCode 3312 – Sorted GCD Pair Queries**.

---

## The full code

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

---

## The big idea first 🎯

We can't list every pair (there could be billions → Memory Limit Exceeded). So instead we **count how many pairs share each GCD value**, then use those counts to answer any query instantly.

There are **4 phases**:
1. Count how often each number appears.
2. Work out how many pairs have each *exact* GCD.
3. Turn those counts into a running (prefix sum) total.
4. Answer each query with a binary search.

---

## Phase 0 — Setup

```python
from bisect import bisect_right
```
Imports a fast **binary search** helper from Python's standard library. We use it in the last line to jump straight to the answer instead of scanning.

```python
class Solution(object):
    def gcdValues(self, nums, queries):
```
The standard LeetCode wrapper. `nums` is our list of numbers; `queries` are the index positions we need to look up.

```python
maxV = max(nums)
```
Finds the **largest number** in the list. Every GCD must be `<= maxV`, so this tells us how big our helper arrays need to be. (This is why memory stays small — sized by the *biggest value* ~50,000, not by the *number of pairs* ~billions.)

---

## Phase 1 — Count how often each value appears

```python
cnt = [0] * (maxV + 1)
```
Creates a list full of zeros, with one slot for every possible value from `0` to `maxV`. `cnt[v]` will hold *"how many times does value `v` appear in nums?"*

```python
for v in nums:
    cnt[v] += 1
```
Goes through each number and bumps its counter. 

**Example:** `nums = [2, 3, 4]` → `cnt[2]=1`, `cnt[3]=1`, `cnt[4]=1`.

---

## Phase 2 — Count pairs by their exact GCD (the clever part)

```python
gcd_count = [0] * (maxV + 1)
```
Another zero-filled list. `gcd_count[g]` will end up meaning *"how many pairs have a GCD of exactly `g`?"*

```python
for g in range(maxV, 0, -1):        # high -> low
```
Loop through every possible GCD value `g`, going **from big down to small**. ⚠️ The direction matters — we'll see why in a second.

```python
    d = 0                            # how many numbers divisible by g
    for multiple in range(g, maxV + 1, g):
        d += cnt[multiple]
```
Here `d` counts how many numbers in `nums` are **divisible by `g`**. We do this by adding up the counts of `g`, `2g`, `3g`, … (every multiple of `g` up to `maxV`).

Why? Because if a number is divisible by `g`, it *could* form a pair with GCD `g`.

```python
    pairs = d * (d - 1) // 2         # pairs whose GCD is a MULTIPLE of g
```
If `d` numbers are all divisible by `g`, the number of pairs you can make from them is the combination formula:

$$\binom{d}{2} = \frac{d \times (d-1)}{2}$$

⚠️ But this counts pairs whose GCD is `g`, `2g`, `3g`, … — i.e. **any multiple of `g`**, not exactly `g`.

```python
    for k in range(2 * g, maxV + 1, g):
        pairs -= gcd_count[k]        # remove strict multiples
```
So we **subtract** the pairs we've already counted for the strict multiples (`2g`, `3g`, …). Because we looped **high → low**, those `gcd_count[k]` values are already finished and correct. This leaves only the pairs whose GCD is **exactly `g`**. *(This subtraction trick is called inclusion-exclusion.)*

```python
    gcd_count[g] = pairs
```
Store the final "exactly `g`" count.

---

## Phase 3 — Prefix sums (running totals)

```python
prefix = [0] * (maxV + 1)
running = 0
for g in range(1, maxV + 1):
    running += gcd_count[g]
    prefix[g] = running
```
This builds a **cumulative total**. `prefix[g]` = *"how many pairs have a GCD of `g` or less?"*

Since `gcdPairs` is sorted ascending, this is essentially describing the sorted list *without storing it*. 

**Example:** if `gcd_count = [_, 2, 1]` (2 pairs with GCD 1, 1 pair with GCD 2), then:
- `prefix[1] = 2` → 2 pairs have GCD ≤ 1
- `prefix[2] = 3` → 3 pairs have GCD ≤ 2

That perfectly describes the sorted list `[1, 1, 2]`.

---

## Phase 4 — Answer each query

```python
return [bisect_right(prefix, k) for k in queries]
```
For each query index `k`, `bisect_right(prefix, k)` finds the **smallest `g`** where `prefix[g] > k`.

In plain words: *"which GCD value sits at position `k` in the sorted list?"* Binary search finds it in `O(log maxV)` — super fast, even for 100,000 queries.

**Example:** sorted list is `[1, 1, 2]` (prefix `[_, 2, 3]`).
- query `k=0` → smallest `g` with `prefix[g] > 0` → `g = 1` ✅
- query `k=2` → smallest `g` with `prefix[g] > 2` → `g = 2` ✅

---

## Full walkthrough: `nums = [2, 3, 4]`

`maxV = 4`, `cnt = [0, 0, 1, 1, 1]`

| g | d (divisible by g) | pairs = C(d,2) | subtract multiples | `gcd_count[g]` |
|---|---|---|---|---|
| 4 | 1 (just 4) | 0 | – | 0 |
| 3 | 1 (just 3) | 0 | – | 0 |
| 2 | 2 (2 and 4) | 1 | – | **1** |
| 1 | 3 (all) | 3 | −(0+0+1) | **2** |

`gcd_count = [_, 2, 1, 0, 0]` → `prefix = [_, 2, 3, 3, 3]`

This means sorted `gcdPairs = [1, 1, 2]`. Queries `[0, 2, 2]` → `[1, 2, 2]` ✅

---

## Why this beats brute-force 💾

| | Brute-force | This version |
|---|---|---|
| Biggest thing stored | List of ~5 billion pairs | Arrays sized ~50,000 |
| Memory | 💥 Exceeded | 🟢 Tiny |
| Passes giant tests | ❌ No | ✅ Yes |

---

## Complexity

| Step | Cost |
|---|---|
| Counting divisibles for all `g` | `O(maxV log maxV)` (harmonic sum) |
| Prefix sums | `O(maxV)` |
| Each query | `O(log maxV)` binary search |

---

## The one line to remember 🎯

> When you can't enumerate pairs directly, **count them by a shared property** (their GCD). Count "multiples of g" first, then **subtract strict multiples from high to low** to isolate "exactly g".
