# Problem: Activity Selection — Maximum Non-Overlapping Activities

## Category
Greedy

## Difficulty
Medium

---

## Problem Statement

You're given `n` activities, each with a **start time** and **end time**. Select the **maximum number** of activities a single person can attend without overlapping. (You can't attend two at the same time.)

This is the classic introduction to greedy algorithms.

### Example

```
Activities (start, end):
(1, 3), (2, 4), (3, 5), (0, 6), (5, 7), (8, 9), (5, 9)

Maximum non-overlapping: 4
One valid selection: (1, 3), (3, 5), (5, 7), (8, 9)
```

---

## Clarifying Questions to Ask the Interviewer

1. **Are end times exclusive?** "End at 3, start at 3" — is that overlapping or back-to-back? Common assumption: **back-to-back is OK**.
2. **Same start and end (zero-duration activity)?** Edge case — typically counts as a degenerate interval.
3. **Return the count or the actual activities?** Count usually; reconstruction is easy if needed.

---

## Approach — Greedy by Earliest Finish Time

### The intuition

If we pick the activity that **ends earliest** first, we leave the maximum amount of time for future activities. Then repeat: among remaining activities that start after the last one ended, pick the one that ends earliest. And so on.

### The algorithm

1. **Sort** activities by **end time** ascending.
2. Iterate through them. Pick the next activity if its **start time ≥ last picked activity's end time**.
3. Count how many you picked.

### Walking through the example

Sorted by end time:
```
(1, 3)
(2, 4)
(3, 5)
(0, 6)
(5, 7)
(8, 9)
(5, 9)
```

Iterate:
```
Pick (1, 3). last_end = 3. count = 1.
(2, 4): start 2 < 3. Skip.
(3, 5): start 3 ≥ 3. Pick. last_end = 5. count = 2.
(0, 6): start 0 < 5. Skip.
(5, 7): start 5 ≥ 5. Pick. last_end = 7. count = 3.
(8, 9): start 8 ≥ 7. Pick. last_end = 9. count = 4.
(5, 9): start 5 < 9. Skip.
```

Final: **4 activities**. ✓

### Why earliest-finish-time wins

This is the canonical greedy with an **exchange argument** proof: any optimal solution can be transformed to start with the earliest-finish activity without losing count. So greedy stays optimal.

See `why-greedy-works.md` for the proof.

### Why other greedies fail

- **Earliest start time:** could pick a long activity that blocks many short ones.
- **Shortest duration:** could pick a tiny activity that splits a long sequence awkwardly.
- **Greedy by latest start:** symmetric to earliest-finish-time and also works.

---

## Time Complexity

**O(n log n)** — sorting dominates.

## Space Complexity

**O(1)** auxiliary (besides the sort).

---

## Code (Java)

```java
public int maxActivities(int[][] activities) {
    // Sort by end time ascending.
    Arrays.sort(activities, (a, b) -> Integer.compare(a[1], b[1]));

    int count = 0;
    int lastEnd = Integer.MIN_VALUE;

    for (int[] a : activities) {
        if (a[0] >= lastEnd) {              // start time ≥ last end → no overlap
            count++;
            lastEnd = a[1];
        }
    }
    return count;
}
```

---

## Code (Python)

```python
class Solution:
    def maxActivities(self, activities):
        activities.sort(key=lambda x: x[1])     # sort by end time

        count = 0
        last_end = float('-inf')

        for s, e in activities:
            if s >= last_end:
                count += 1
                last_end = e

        return count
```

---

## Edge Cases

| Case | Result |
|---|---|
| Empty input | 0 |
| All overlapping | 1 |
| All disjoint | n |
| Equal end times | Pick any one; tie-breaking doesn't change the count. |
| Zero-duration activity | Counts as one interval; depends on inclusive vs exclusive convention. |

---

## Follow-up Questions an Interviewer Might Ask

1. **"Why earliest finish time, not earliest start?"**
   The earliest-finish-time activity leaves the most "runway" for future activities. See `why-greedy-works.md` for the formal exchange argument.

2. **"What if each activity has a value (weighted scheduling)?"**
   Greedy doesn't work. Use **DP**: sort by end time, define `dp[i]` = best value using activities 1..i, decide skip vs take. Find the latest non-conflicting activity with binary search. **O(n log n)** total.

3. **"Multiple persons / k rooms — schedule all activities across them."**
   Min-heap of "earliest free time per room." Assign each new activity to the earliest-free room. (LeetCode 253 — Meeting Rooms II.)

4. **"What if activities must be served exactly k of them, choose any k?"**
   Greedy or DP variants — depends on the constraints.

---

## Key Lessons from This Problem

1. **Greedy works when local optimal choices lead to global optimum.** Activity selection is the textbook example.
2. **Sort first** — most greedy algorithms start with a sort.
3. **Earliest-finish-time** is the right greedy here. Memorize the rule and the reason.
