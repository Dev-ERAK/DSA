# Problem: Longest Common Subsequence (LCS)

## Category
Dynamic Programming

## Difficulty
Medium

---

## Problem Statement

Given two strings `text1` and `text2`, return the **length** of their longest common subsequence (LCS).

A **subsequence** is a sequence you can derive from another by deleting some (or no) characters **without changing the order**. It does NOT have to be contiguous.

For example, in `"abcde"`:
- `"ace"` is a subsequence (kept a, c, e).
- `"aec"` is **not** (order changed).

This is **LeetCode #1143**.

### Examples

```
text1 = "abcde", text2 = "ace"   →  3   ("ace")
text1 = "abc",   text2 = "abc"   →  3   ("abc")
text1 = "abc",   text2 = "def"   →  0   (no common letters)
```

---

## Clarifying Questions to Ask the Interviewer

1. **Length only, or actual subsequence?** Usually length. Reconstruction is in `print-lcs.md`.
2. **Empty strings?** Return 0.
3. **Case-sensitive?** Yes by default.

---

## Approach — Dynamic Programming

### The DP table

Define `dp[i][j]` = length of LCS of `text1[0..i-1]` and `text2[0..j-1]`.

(Using length, not index, makes the recurrence cleaner. `dp[0][j]` and `dp[i][0]` are 0 — empty string vs anything has LCS 0.)

### The recurrence

For each pair of indices, two cases:

1. **Last characters match (`text1[i-1] == text2[j-1]`):**
   The LCS extends by 1: `dp[i][j] = dp[i-1][j-1] + 1`.

2. **Last characters don't match:**
   The LCS is the better of two choices: drop `text1[i-1]` or drop `text2[j-1]`.
   `dp[i][j] = max(dp[i-1][j], dp[i][j-1])`.

Final answer: `dp[m][n]`.

### Walking through `text1 = "abcde"`, `text2 = "ace"`

Build the DP table (rows = `text1` characters, columns = `text2` characters):

```
       ""  a   c   e
   ""   0  0   0   0
   a    0  1   1   1
   b    0  1   1   1
   c    0  1   2   2
   d    0  1   2   2
   e    0  1   2   3
```

Reading some cells:
- `dp[1][1]`: `'a' == 'a'`, so `dp[0][0] + 1 = 1`. ✓
- `dp[2][1]`: `'b' != 'a'`, take `max(dp[1][1], dp[2][0]) = max(1, 0) = 1`. ✓
- `dp[5][3]`: `'e' == 'e'`, so `dp[4][2] + 1 = 2 + 1 = 3`. ✓

Final answer: `dp[5][3] = 3`. ✓

### Why this works

LCS has the **optimal substructure** property: the answer for `(i, j)` only depends on smaller `(i', j')` cases. Each cell is computed from at most three already-known cells.

Each cell takes O(1), and there are O(m × n) cells → **O(m × n)** total.

### Space optimization

We only ever read from the previous row. So we can use just **two rows** (previous + current), reducing memory to **O(min(m, n))**.

---

## Time Complexity

**O(m × n)**

## Space Complexity

**O(m × n)**, reducible to **O(min(m, n))**.

---

## Code (Java)

```java
public int longestCommonSubsequence(String a, String b) {
    int m = a.length(), n = b.length();
    int[][] dp = new int[m + 1][n + 1];        // padded with 0s on row 0 and col 0

    for (int i = 1; i <= m; i++) {
        for (int j = 1; j <= n; j++) {
            if (a.charAt(i - 1) == b.charAt(j - 1)) {
                dp[i][j] = dp[i - 1][j - 1] + 1;
            } else {
                dp[i][j] = Math.max(dp[i - 1][j], dp[i][j - 1]);
            }
        }
    }
    return dp[m][n];
}

// Space-optimized to two rows
public int lcsTwoRows(String a, String b) {
    if (a.length() < b.length()) {
        // ensure b is the shorter one
        String t = a; a = b; b = t;
    }
    int m = a.length(), n = b.length();
    int[] prev = new int[n + 1];
    int[] cur  = new int[n + 1];

    for (int i = 1; i <= m; i++) {
        for (int j = 1; j <= n; j++) {
            cur[j] = (a.charAt(i - 1) == b.charAt(j - 1))
                    ? prev[j - 1] + 1
                    : Math.max(prev[j], cur[j - 1]);
        }
        int[] tmp = prev; prev = cur; cur = tmp;
        Arrays.fill(cur, 0);
    }
    return prev[n];
}
```

---

## Code (Python)

```python
class Solution:
    def longestCommonSubsequence(self, a: str, b: str) -> int:
        m, n = len(a), len(b)
        dp = [[0] * (n + 1) for _ in range(m + 1)]

        for i in range(1, m + 1):
            for j in range(1, n + 1):
                if a[i - 1] == b[j - 1]:
                    dp[i][j] = dp[i - 1][j - 1] + 1
                else:
                    dp[i][j] = max(dp[i - 1][j], dp[i][j - 1])

        return dp[m][n]
```

---

## Edge Cases

| Case | Result |
|---|---|
| Either string empty | 0 |
| Strings equal | Length of either |
| No common characters | 0 |
| One string is a subsequence of the other | Length of the smaller string |

---

## Follow-up Questions an Interviewer Might Ask

1. **"Reconstruct the actual LCS string."**
   See `print-lcs.md` — backtrack through the DP table from `dp[m][n]`.

2. **"Difference between LCS and longest common *substring* (contiguous)?"**
   Different DP. Substring requires contiguity:
   ```
   dp[i][j] = dp[i-1][j-1] + 1   if last chars match
            = 0                   otherwise
   ```
   Answer is the max value anywhere in the table, not just the corner.

3. **"Edit distance (LeetCode 72)?"**
   Same DP shape, different recurrence (insert / delete / replace).

4. **"Why use length-based indexing (`text1[i-1]`)?"**
   Lets `dp[0][*]` and `dp[*][0]` represent "empty string vs something" cleanly — no special-case handling at boundaries.

---

## Key Lessons from This Problem

1. **2D DP table** with rows = string1, columns = string2 is the standard layout.
2. **Match → extend by 1; mismatch → take the better of two options.** Memorize this recurrence.
3. **Space optimization** by keeping only the previous row is a recurring DP technique.
