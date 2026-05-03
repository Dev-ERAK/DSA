# Problem: Search a Sorted 2D Matrix

## Category
Arrays / Binary Search

## Difficulty
Medium

---

## Problem Statement

You're given a 2D matrix where:

- Each **row** is sorted left to right (ascending).
- Each **column** is sorted top to bottom (ascending).

Given a `target` value, return whether it exists in the matrix.

### Example

```
Matrix:                       target = 5  →  true
[[ 1,  4,  7, 11],            target = 8  →  false
 [ 2,  5,  8, 12],
 [ 3,  6,  9, 16],
 [10, 13, 14, 17]]
```

This is **LeetCode #240** ("Search a 2D Matrix II").

> ⚠️ Don't confuse this with LeetCode #74 — that one has a stronger condition (the matrix reads as a single sorted list when flattened). For #74 you can binary-search the matrix as a 1D array.

---

## Clarifying Questions to Ask the Interviewer

1. **Are the rows AND columns sorted, or is the matrix sorted as one big sorted list?** This problem assumes the weaker condition: rows sorted, columns sorted, but **not** globally sorted. (The matrix above shows this: row 2 starts with `3`, but row 1 ends with `12` — so it's not "left-to-right, top-to-bottom" sorted as a flat list.)
2. **Can the matrix be empty?** Yes — handle `null`/empty input.
3. **Are values distinct?** Doesn't really matter; the algorithm works either way.

---

## Approach

### Why naive linear search is O(m × n)

If you scan every cell, you check up to `m × n` values. For a 1000 × 1000 matrix that's a million comparisons. We can do better.

### Why simple binary search per row is O(m log n)

Binary search each row — O(log n) per row, m rows → O(m log n). Better, but still doesn't use the column-sorted property.

### The Optimal Trick — Staircase Search (O(m + n))

**Start at the top-right corner** of the matrix. From there, every move is forced:

- If `current == target` → found!
- If `current > target` → the target must be **smaller**, so move **left** (the column we're in has only larger numbers below — no good).
- If `current < target` → the target must be **larger**, so move **down** (the row we're in has only smaller numbers to the left — no good).

Each step eliminates either a whole row or a whole column. So we make at most `m + n` steps.

### Why does starting top-right work?

At the top-right corner, the two directions you can move have **opposite meanings**:

- "Smaller" lies to the **left**.
- "Larger" lies **down**.

This gives a clean, unambiguous decision rule. You can also start at the bottom-left for the same reason. **Never** start at top-left or bottom-right — both directions there mean "larger" (or both mean "smaller"), so you can't tell which way to go.

### Walking through an example — searching for 5

```
[[ 1,  4,  7, *11*],   ← start at top-right (11)
 [ 2,  5,  8,  12 ],
 [ 3,  6,  9,  16 ],
 [10, 13, 14,  17 ]]
```

Step 1: at (0, 3) = 11. Target 5 < 11 → go **left**.

```
[[ 1,  4, *7*, 11],
 [ 2,  5,  8, 12],
 ...
```

Step 2: at (0, 2) = 7. 5 < 7 → go **left**.

```
[[ 1, *4*,  7, 11],
 [ 2,  5,  8, 12],
 ...
```

Step 3: at (0, 1) = 4. 5 > 4 → go **down**.

```
[[ 1,  4,  7, 11],
 [ 2, *5*,  8, 12],
 ...
```

Step 4: at (1, 1) = 5. **Found!**

Total: 4 steps. We never touched row 2 or row 3.

---

## Time Complexity

**O(m + n)** — at most `m` downward moves and `n` leftward moves before we run out of matrix.

## Space Complexity

**O(1)** — just two index variables.

---

## Code (Java)

```java
public boolean searchMatrix(int[][] matrix, int target) {
    if (matrix == null || matrix.length == 0 || matrix[0].length == 0) {
        return false;
    }

    int m = matrix.length;
    int n = matrix[0].length;

    // Start at top-right corner.
    int row = 0;
    int col = n - 1;

    while (row < m && col >= 0) {
        int current = matrix[row][col];

        if (current == target) {
            return true;
        } else if (current > target) {
            col--;          // go left — target is smaller
        } else {
            row++;          // go down — target is larger
        }
    }
    return false;
}
```

---

## Code (Python)

```python
class Solution:
    def searchMatrix(self, matrix, target: int) -> bool:
        if not matrix or not matrix[0]:
            return False

        m, n = len(matrix), len(matrix[0])
        row, col = 0, n - 1               # start top-right

        while row < m and col >= 0:
            current = matrix[row][col]
            if current == target:
                return True
            elif current > target:
                col -= 1                  # go left
            else:
                row += 1                  # go down
        return False
```

---

## Edge Cases

| Case | What happens |
|---|---|
| Empty matrix or empty row | We return `false` immediately. |
| Single cell matrix | Compared once and returned. |
| Target smaller than minimum | Loop walks left then runs out — return `false`. |
| Target larger than maximum | Loop walks down then runs out — return `false`. |
| Many duplicates | Algorithm still works — finds *some* matching cell or returns false. |

---

## Follow-up Questions an Interviewer Might Ask

1. **"Why start top-right and not top-left?"**
   At the top-left, "left" doesn't exist and "down" only means larger. There's no way to go to a *smaller* value, so you can't eliminate when `current > target`. Top-right (or bottom-left) gives you the two-direction decision rule.

2. **"What if the matrix is sorted as a flat sequence (LeetCode 74)?"**
   Treat the m×n cells as one 1D sorted array of length `m*n`. Binary-search with `index = mid`, `row = index / n`, `col = index % n`. Time: **O(log(mn))**.

3. **"Can we do better than O(m + n)?"**
   With divide & conquer you can get O((m+n) · log(m+n)) but with worse constants. The staircase is provably optimal for the comparison-based version — for *any* algorithm, an adversary can force at least `m + n - 1` comparisons by carefully arranging values around the path.

4. **"What if rows are sorted but columns aren't?"**
   Then the column property is gone. Binary-search each row independently (O(m log n)), or use a different structure entirely.

---

## Key Lessons from This Problem

1. **Pick your starting position carefully.** The corner choice unlocks the algorithm.
2. **"Each step eliminates a row or column"** is the standard way to argue O(m + n).
3. **Sortedness can be 1D or 2D** — and it gives you very different powers. Recognize which condition you have.
