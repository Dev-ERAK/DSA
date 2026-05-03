# Problem: Longest Increasing Subsequence (LIS) — O(n log n)

## Category
Dynamic Programming / Binary Search

## Difficulty
Medium

---

## Problem Statement

Same as `lis.md` — find the length of the longest strictly increasing subsequence — but in **O(n log n)** time.

This is a clever algorithm called **patience sorting** that uses binary search.

---

## Clarifying Questions to Ask the Interviewer

1. **Strictly increasing?** Yes — for non-decreasing, we'd use `bisect_right` instead of `bisect_left`.
2. **Length only?** Yes here. Reconstruction is harder but possible.

---

## Approach — Patience Sorting / Binary Search Replace

### The Big Idea

We maintain an array `tails` where `tails[k]` = **the smallest possible "tail value"** for an increasing subsequence of length `k + 1`.

### How `tails` evolves

For each new value `x` in `nums`:

1. **Find the leftmost position `i` in `tails` where `tails[i] >= x`** (binary search — `lower_bound`).
2. **If found:** replace `tails[i] = x`.
3. **If not found** (i.e., `x` is larger than every value in `tails`): **append** `x` to `tails`.

The **length** of `tails` at the end is the LIS length.

### Why does this work?

- When we **append**, we've extended an LIS by 1 — that's clearly OK.
- When we **replace** an existing tail with a smaller value, we're saying "an LIS of this length can end at a smaller value," which gives us **more room** for future extensions. The current LIS length doesn't change, but our future flexibility improves.
- `tails` always stays **strictly increasing** (a sorted array we can binary-search).

### Important caveat

`tails` is **not** necessarily a valid LIS itself! It's a structural witness for the *length*, not an actual subsequence. If you need the actual LIS, you need extra bookkeeping (parent pointers).

### Walking through `[10, 9, 2, 5, 3, 7, 101, 18]`

```
x = 10: tails empty → append. tails = [10]
x = 9:  find first ≥ 9 in [10]. Index 0. Replace. tails = [9]
x = 2:  find first ≥ 2 in [9]. Index 0. Replace. tails = [2]
x = 5:  find first ≥ 5 in [2]. None — append. tails = [2, 5]
x = 3:  find first ≥ 3 in [2, 5]. Index 1 (5). Replace. tails = [2, 3]
x = 7:  find first ≥ 7 in [2, 3]. None — append. tails = [2, 3, 7]
x = 101: append. tails = [2, 3, 7, 101]
x = 18: find first ≥ 18 in [2, 3, 7, 101]. Index 3 (101). Replace. tails = [2, 3, 7, 18]

Final length: 4   ✓
```

Note that the final `tails` is `[2, 3, 7, 18]` — which **is** a valid LIS in this case. But it's not always: in some inputs, the contents of `tails` at the end don't correspond to any actual LIS in the array.

### Where does the O(n log n) come from?

For each of the n elements, we do a binary search in `tails` (size at most n) → **O(log n) per element** → **O(n log n)** total.

---

## Time Complexity

**O(n log n)**

## Space Complexity

**O(n)** — `tails` can grow up to size n.

---

## Code (Java)

```java
public int lengthOfLIS(int[] nums) {
    int n = nums.length;
    int[] tails = new int[n];
    int size = 0;

    for (int x : nums) {
        // Binary search for the first index where tails[index] >= x.
        int lo = 0, hi = size;
        while (lo < hi) {
            int mid = (lo + hi) >>> 1;     // unsigned right shift to avoid overflow
            if (tails[mid] < x) {
                lo = mid + 1;
            } else {
                hi = mid;
            }
        }
        tails[lo] = x;                      // replace or append
        if (lo == size) size++;             // appended
    }
    return size;
}
```

---

## Code (Python)

```python
from bisect import bisect_left

class Solution:
    def lengthOfLIS(self, nums) -> int:
        tails = []
        for x in nums:
            i = bisect_left(tails, x)        # leftmost index where tails[i] >= x
            if i == len(tails):
                tails.append(x)
            else:
                tails[i] = x
        return len(tails)
```

> `bisect_left([2, 5], 3)` returns 1 — the index where 3 would be inserted to keep the list sorted (and where it'd land if equal to an existing value, on the left).

---

## Edge Cases

| Case | Result |
|---|---|
| Empty array | 0 |
| Strictly decreasing | 1 (every value just replaces `tails[0]`) |
| Non-decreasing variant | Use `bisect_right` (Python) — allows duplicates to extend |
| Single element | 1 |

---

## Follow-up Questions an Interviewer Might Ask

1. **"Reconstruct the actual LIS."**
   Maintain a parent pointer for each element. Also track which "slot" in `tails` each element fills. Walk back from the last appended element, following parents.

2. **"Why doesn't `tails` itself always represent the LIS?"**
   Because we replace tails with values that may come **after** the original tail — they're not part of the same subsequence. `tails` stores the **best tails per length**, not an actual subsequence.

3. **"Longest non-decreasing subsequence?"**
   Use `bisect_right` instead of `bisect_left`. This finds the first index `> x` (strictly), allowing duplicates to extend.

4. **"Russian Doll Envelopes (LeetCode 354)?"**
   Sort envelopes by width ascending, then by height descending (clever tie-break). Then run LIS on heights. The descending-on-tie prevents same-width envelopes from chaining.

---

## Key Lessons from This Problem

1. **Patience sorting** is the elegant insight here — a card-game-inspired algorithm.
2. **Binary search + greedy replacement** turns a simple O(n²) DP into O(n log n).
3. **The "answer length" is preserved even though `tails` isn't the actual LIS.** Be careful about that distinction in interviews.
