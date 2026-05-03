# Problem: Rotate Array by K Steps

## Category
Arrays

## Difficulty
Medium

---

## Problem Statement

You're given an array of numbers and a number `k`. Move every element of the array `k` positions to the **right**. The elements that fall off the right end should wrap around and reappear at the start.

Think of the array like a row of people. Everyone takes `k` steps to the right. The last `k` people loop around and stand at the front.

### Example

```
Input:  nums = [1, 2, 3, 4, 5, 6, 7],  k = 3
Output: [5, 6, 7, 1, 2, 3, 4]
```

Why? Each element shifts 3 spots right, with wrap-around:
```
Original index: 0 1 2 3 4 5 6
Original value: 1 2 3 4 5 6 7

After rotating right by 3:
New index:      0 1 2 3 4 5 6
New value:      5 6 7 1 2 3 4
```

The element that was at index `i` is now at index `(i + k) % n`, where `n` is the array length and `%` is the remainder operator.

---

## Clarifying Questions to Ask the Interviewer

These questions show you're thinking carefully:

1. **Can `k` be larger than the array length?** (Yes — and rotating by `n` is the same as not rotating, so we'll use `k = k % n`.)
2. **Can `k` be negative?** (If yes, that's a left rotation. We can convert it: `k = ((k % n) + n) % n`.)
3. **Can the array be empty?** (Yes — return immediately, nothing to do.)
4. **Should we rotate in place, or can we use extra memory?** (In place — using O(1) extra memory — is the goal.)

---

## Approach

We'll build up to the optimal solution from the simplest idea.

### Method 1 — Brute Force (the obvious idea)

Rotate by 1 position, repeated `k` times. Each "rotate by 1" shifts every element right by 1, and wraps the last element around to index 0.

**Why this is slow:** Each one-step rotation costs O(n) work, and we do it `k` times. Total: O(n × k). For large `k`, this is unacceptable.

### Method 2 — Use an Extra Array (easy to write, uses extra memory)

Create a new array `result` of the same size. For each index `i` in the original array:
```
result[(i + k) % n] = nums[i]
```
Then copy `result` back into `nums`.

- Time: O(n)
- Space: O(n) — uses a second array

### Method 3 — Reverse Trick (the optimal in-place solution)

This is the classic interview answer. It uses **three reversals** and no extra array.

**The trick:** rotating right by `k` is equivalent to:
1. Reverse the **whole** array.
2. Reverse the **first `k`** elements.
3. Reverse the **remaining `n - k`** elements.

Sounds magical. Let's see why it works.

#### Why does the reversal trick work?

Think of the array as two pieces, `A` and `B`, where `B` is the **last `k`** elements:

```
[ A | B ]   →  we want →   [ B | A ]
```

For our example with `k = 3`:
```
A = [1, 2, 3, 4]      (the first n - k elements)
B = [5, 6, 7]         (the last k elements)
```

We want `[B | A]`, which is `[5, 6, 7, 1, 2, 3, 4]`. ✓

Now watch what the three reversals do:

**Step 1: Reverse the whole array.**
```
Before: [A | B]      = [1, 2, 3, 4, 5, 6, 7]
After:  [reverse(B) | reverse(A)] = [7, 6, 5, 4, 3, 2, 1]
```
(When you reverse `[A | B]`, the first part of the result is `reverse(B)` because what was at the end now comes first, in reversed order. Same for the second half.)

**Step 2: Reverse the first `k` elements** (which is currently `reverse(B)`).
```
Before: [reverse(B) | reverse(A)] = [7, 6, 5 | 4, 3, 2, 1]
After:  [B | reverse(A)]          = [5, 6, 7 | 4, 3, 2, 1]
```
Reversing a reversed thing gives back the original — so `reverse(reverse(B)) = B`.

**Step 3: Reverse the remaining `n - k` elements** (currently `reverse(A)`).
```
Before: [B | reverse(A)] = [5, 6, 7 | 4, 3, 2, 1]
After:  [B | A]          = [5, 6, 7 | 1, 2, 3, 4]
```
Same idea: `reverse(reverse(A)) = A`.

That's the answer. Three linear passes, no extra array.

#### Visualizing the trace step by step

Starting array, `k = 3`:
```
Index:   0  1  2  3  4  5  6
Value:   1  2  3  4  5  6  7
```

Step 1 — reverse entire array (index 0 to 6):
```
Value:   7  6  5  4  3  2  1
```

Step 2 — reverse first `k = 3` elements (index 0 to 2):
```
Value:   5  6  7  4  3  2  1
         ^^^^^^^ reversed
```

Step 3 — reverse remaining `n - k = 4` elements (index 3 to 6):
```
Value:   5  6  7  1  2  3  4
                  ^^^^^^^^^^^ reversed
```

Done!

---

## Time Complexity

**O(n)** — three reversals, each touching every element once.

## Space Complexity

**O(1)** — we modify the array in place; no second array needed.

---

## Code (Java)

```java
public void rotate(int[] nums, int k) {
    int n = nums.length;
    if (n == 0) return;                   // empty array — nothing to do

    // Normalize k. If k is larger than n, rotating by n is a no-op,
    // so the effective rotation is k % n. The extra "+ n) % n" is a
    // safety net in case k is negative.
    k = ((k % n) + n) % n;

    reverse(nums, 0, n - 1);              // Step 1: reverse the whole array
    reverse(nums, 0, k - 1);              // Step 2: reverse the first k elements
    reverse(nums, k, n - 1);              // Step 3: reverse the rest
}

// Helper: reverses nums[l..r] in place using two pointers that
// walk toward each other, swapping as they go.
private void reverse(int[] nums, int l, int r) {
    while (l < r) {
        int temp = nums[l];
        nums[l] = nums[r];
        nums[r] = temp;
        l++;
        r--;
    }
}
```

### How `reverse` works

We use two pointers: `l` starting at the left of the segment, `r` at the right. We swap `nums[l]` and `nums[r]`, then move them toward the middle. When `l` meets or crosses `r`, we're done.

Trace `reverse(nums, 0, 6)` on `[1, 2, 3, 4, 5, 6, 7]`:
```
l=0, r=6: swap 1 and 7 → [7, 2, 3, 4, 5, 6, 1]
l=1, r=5: swap 2 and 6 → [7, 6, 3, 4, 5, 2, 1]
l=2, r=4: swap 3 and 5 → [7, 6, 5, 4, 3, 2, 1]
l=3, r=3: stop (l is not < r anymore)
```

---

## Code (Python)

```python
class Solution:
    def rotate(self, nums, k: int) -> None:
        n = len(nums)
        if n == 0:                        # empty list — nothing to do
            return

        # Normalize k (handles k > n and negative k).
        k = k % n

        # Helper: reverse nums in-place between indices l and r (inclusive).
        def reverse(l: int, r: int) -> None:
            while l < r:
                nums[l], nums[r] = nums[r], nums[l]   # Pythonic swap
                l += 1
                r -= 1

        reverse(0, n - 1)                 # Step 1: reverse the whole list
        reverse(0, k - 1)                 # Step 2: reverse the first k items
        reverse(k, n - 1)                 # Step 3: reverse the rest
```

### Pythonic note

Python lets you swap two values in one line: `nums[l], nums[r] = nums[r], nums[l]`. Behind the scenes it builds a tuple of the right-hand-side values and unpacks them into the left-hand variables — no temporary variable needed.

---

## Edge Cases

These are the cases that often trip people up. Make sure each one works:

| Input | Why it's tricky |
|---|---|
| `k = 0` | No rotation. With `k % n`, the second reverse becomes `reverse(0, -1)` which our `while l < r` loop ignores → safe no-op. |
| `k = n` | A full rotation is the same as no rotation. `k % n = 0` handles this. |
| `k > n` (e.g., `k = 100`, `n = 7`) | `k % n` reduces it to a meaningful value (e.g., `100 % 7 = 2`). |
| Empty array | We `return` immediately. |
| One element | Any rotation leaves it unchanged. |
| Negative `k` | `((k % n) + n) % n` converts it to the equivalent positive rotation. |

---

## Follow-up Questions an Interviewer Might Ask

1. **"Can you do it using an extra array?"** Yes — Method 2 above. It's simpler but uses O(n) extra memory.
2. **"How would you rotate to the *left* by `k` instead?"** Rotating left by `k` is the same as rotating right by `n - k`. Or: same three-reversal trick with the segments swapped.
3. **"How would you rotate a linked list by `k`?"** Walk to find the new tail (at position `n - k - 1`), then splice: the old tail's `next` becomes the old head, and the new head is the node after the new tail.
4. **"What if the array is huge and stored on disk?"** Then minimizing passes matters more than memory; the reverse trick is still good (3 passes). Block-cyclic rotation can also be useful.

---

## Key Lessons from This Problem

1. **Modulo (`%`) tames out-of-range inputs.** Whenever you have a "wrap around" problem, think `% n`.
2. **The reverse trick** is a recurring idea — it shows up in linked-list problems and string-rotation problems too.
3. **In place + O(1) memory** is the gold standard for array problems. Always ask yourself: do I really need a second array?
