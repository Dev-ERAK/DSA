# Problem: First Occurrence in a Sorted Array

## Category
Searching / Binary Search

## Difficulty
Easy

---

## Problem Statement

Given a **sorted** array (with possibly duplicate values) and a `target`, return the **index of the first occurrence** of the target. Return `-1` if it's not present.

### Example

```
nums = [1, 2, 2, 2, 3, 4], target = 2   →  1   (the leftmost 2)
nums = [1, 1, 1, 1],       target = 1   →  0
nums = [1, 2, 3],          target = 4   →  -1
```

---

## Clarifying Questions to Ask the Interviewer

1. **Sorted ascending?** Yes — required for binary search.
2. **Duplicates allowed?** Yes — that's why "first occurrence" matters.
3. **What if the target isn't there?** Return `-1`.

---

## Approach — Modified Binary Search

### Why standard binary search isn't enough

Plain binary search returns **some** matching index, but with duplicates, you might land on `nums[3] == 2` in `[1, 2, 2, 2, 3]` — that's a valid match, but not the *first*.

### The fix — keep searching left after a match

When you find a match, **don't return immediately**. Record the match, then **search the left half** to see if there's an earlier one.

```
lo = 0; hi = n - 1; ans = -1
while lo <= hi:
    mid = (lo + hi) / 2
    if nums[mid] == target:
        ans = mid          # remember this match
        hi = mid - 1       # but keep searching left
    elif nums[mid] < target:
        lo = mid + 1
    else:
        hi = mid - 1
return ans
```

### Walking through `nums = [1, 2, 2, 2, 3, 4], target = 2`

```
n = 6.

Step 1: lo=0, hi=5. mid=2. nums[2]=2. MATCH. ans=2. hi=mid-1=1.
Step 2: lo=0, hi=1. mid=0. nums[0]=1. 1<2. lo=mid+1=1.
Step 3: lo=1, hi=1. mid=1. nums[1]=2. MATCH. ans=1. hi=mid-1=0.
Step 4: lo=1, hi=0. Loop ends.

Return ans = 1.   ✓
```

### Equivalent — `lower_bound`

The "first index where `nums[i] >= target`" is called `lower_bound`. After finding it, check whether `nums[lo] == target`:

```
lo = 0; hi = n
while lo < hi:
    mid = (lo + hi) / 2
    if nums[mid] < target: lo = mid + 1
    else: hi = mid

if lo < n and nums[lo] == target: return lo
return -1
```

Both styles work — pick whichever feels cleaner.

---

## Time Complexity

**O(log n)**

## Space Complexity

**O(1)**

---

## Code (Java)

```java
public int firstOccurrence(int[] nums, int target) {
    int lo = 0, hi = nums.length - 1, ans = -1;

    while (lo <= hi) {
        int mid = lo + (hi - lo) / 2;

        if (nums[mid] == target) {
            ans = mid;             // record match
            hi = mid - 1;          // keep searching left
        } else if (nums[mid] < target) {
            lo = mid + 1;
        } else {
            hi = mid - 1;
        }
    }
    return ans;
}
```

---

## Code (Python)

```python
class Solution:
    def firstOccurrence(self, nums, target: int) -> int:
        lo, hi, ans = 0, len(nums) - 1, -1
        while lo <= hi:
            mid = (lo + hi) // 2
            if nums[mid] == target:
                ans = mid
                hi = mid - 1
            elif nums[mid] < target:
                lo = mid + 1
            else:
                hi = mid - 1
        return ans
```

---

## Edge Cases

| Case | Result |
|---|---|
| Empty array | `-1` (loop doesn't execute) |
| Target not present | `-1` |
| All elements equal target | `0` (the leftmost) |
| Target at first index | `0` |
| Target at last index | n - 1 |

---

## Follow-up Questions an Interviewer Might Ask

1. **"Find the *last* occurrence."**
   Symmetric: when matched, set `lo = mid + 1` and remember `mid`.

2. **"Count occurrences of target."**
   Find first and last. `count = last - first + 1` (if target is present).
   Or: `upper_bound(target) - lower_bound(target)`.

3. **"Sorted in descending order."**
   Flip the inequality directions in the comparisons.

4. **"Find target in rotated sorted array (LeetCode 33)."**
   Modified binary search: at each step, identify which half is properly sorted, then check whether the target lies in that half.

---

## Key Lessons from This Problem

1. **"Don't return on first match"** — keep narrowing until you've eliminated the possibility of an earlier match.
2. **Track `ans` separately** — your loop condition tracks the search range; `ans` tracks the best match found so far.
3. **Variants of binary search** are everywhere. Master the template, and many "find boundary" problems become trivial.
