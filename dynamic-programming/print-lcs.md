# Problem: Print the Longest Common Subsequence

## Category
Dynamic Programming

## Difficulty
Medium

---

## Problem Statement

Given two strings, return one **actual** LCS string (not just its length).

(See `lcs.md` first if you haven't.)

### Example

```
text1 = "abcbdab", text2 = "bdcaba"
Output: "bcba"   (or "bdab", or "bcab" — multiple valid LCS strings)
```

---

## Clarifying Questions to Ask the Interviewer

1. **Any valid LCS, or all of them?** Any one is standard.
2. **Use the full DP table or two rows?** Full table — needed for reconstruction.

---

## Approach

### Step 1 — Build the full DP table (same as `lcs.md`)

Compute `dp[i][j]` = length of LCS of `text1[0..i-1]` and `text2[0..j-1]`.

We **need the full table** for reconstruction. The 2-row optimization doesn't work here.

### Step 2 — Backtrack from `dp[m][n]` to build the string

Starting at `(i, j) = (m, n)`:

- If `text1[i-1] == text2[j-1]`: this character is part of the LCS. Prepend it. Move to `(i-1, j-1)`.
- Else: move to whichever of `(i-1, j)` or `(i, j-1)` has the larger `dp` value. (Ties → pick either; produces a different valid LCS.)

Continue until `i == 0` or `j == 0`.

### Why backtracking works

Each cell `dp[i][j]` was computed from one of three predecessors:

- `dp[i-1][j-1] + 1` (match — character is in the LCS)
- `dp[i-1][j]` (drop `text1[i-1]`)
- `dp[i][j-1]` (drop `text2[j-1]`)

Walking backwards, we recover the matches that built up the LCS.

### Walking through "abcbdab" and "bdcaba"

Build the DP table (only some cells shown for brevity):

```
       ""  b  d  c  a  b  a
   ""   0  0  0  0  0  0  0
   a    0  0  0  0  1  1  1
   b    0  1  1  1  1  2  2
   c    0  1  1  2  2  2  2
   b    0  1  1  2  2  3  3
   d    0  1  2  2  2  3  3
   a    0  1  2  2  3  3  4
   b    0  1  2  2  3  4  4
```

`dp[7][6] = 4`. Backtrack from `(7, 6)`:

```
(7,6) = 4. text1[6]='b', text2[5]='a'. Mismatch.
        dp[6][6]=4 vs dp[7][5]=4. Tie — pick either; say up to (6,6).
(6,6) = 4. text1[5]='a', text2[5]='a'. MATCH! Add 'a'. Move to (5,5).
(5,5) = 3. text1[4]='d', text2[4]='b'. Mismatch.
        dp[4][5]=3 vs dp[5][4]=2. Up: (4,5).
(4,5) = 3. text1[3]='b', text2[4]='b'. MATCH! Add 'b'. Move to (3,4).
(3,4) = 2. text1[2]='c', text2[3]='a'. Mismatch.
        dp[2][4]=1 vs dp[3][3]=2. Left: (3,3).
(3,3) = 2. text1[2]='c', text2[2]='c'. MATCH! Add 'c'. Move to (2,2).
(2,2) = 1. text1[1]='b', text2[1]='d'. Mismatch.
        dp[1][2]=0 vs dp[2][1]=1. Left: (2,1).
(2,1) = 1. text1[1]='b', text2[0]='b'. MATCH! Add 'b'. Move to (1,0).
(1,0): j == 0 → stop.
```

Characters collected (in reverse order of building): `a, b, c, b`. Reverse: `"bcba"`. ✓

---

## Time Complexity

**O(m × n)** for the table; **O(m + n)** for the backtrack.

## Space Complexity

**O(m × n)** for the full table.

---

## Code (Java)

```java
public String printLCS(String a, String b) {
    int m = a.length(), n = b.length();
    int[][] dp = new int[m + 1][n + 1];

    // Step 1: fill the DP table.
    for (int i = 1; i <= m; i++) {
        for (int j = 1; j <= n; j++) {
            if (a.charAt(i - 1) == b.charAt(j - 1)) {
                dp[i][j] = dp[i - 1][j - 1] + 1;
            } else {
                dp[i][j] = Math.max(dp[i - 1][j], dp[i][j - 1]);
            }
        }
    }

    // Step 2: backtrack to build the string.
    StringBuilder sb = new StringBuilder();
    int i = m, j = n;
    while (i > 0 && j > 0) {
        if (a.charAt(i - 1) == b.charAt(j - 1)) {
            sb.append(a.charAt(i - 1));
            i--; j--;
        } else if (dp[i - 1][j] >= dp[i][j - 1]) {
            i--;
        } else {
            j--;
        }
    }
    return sb.reverse().toString();
}
```

---

## Code (Python)

```python
class Solution:
    def printLCS(self, a: str, b: str) -> str:
        m, n = len(a), len(b)
        dp = [[0] * (n + 1) for _ in range(m + 1)]

        for i in range(1, m + 1):
            for j in range(1, n + 1):
                if a[i - 1] == b[j - 1]:
                    dp[i][j] = dp[i - 1][j - 1] + 1
                else:
                    dp[i][j] = max(dp[i - 1][j], dp[i][j - 1])

        out = []
        i, j = m, n
        while i > 0 and j > 0:
            if a[i - 1] == b[j - 1]:
                out.append(a[i - 1])
                i -= 1
                j -= 1
            elif dp[i - 1][j] >= dp[i][j - 1]:
                i -= 1
            else:
                j -= 1

        return ''.join(reversed(out))
```

---

## Edge Cases

| Case | Result |
|---|---|
| Either string empty | `""` |
| Identical strings | The string itself |
| Multiple valid LCS strings | The tie-break (`>=` in the else branch) picks one specific path |

---

## Follow-up Questions an Interviewer Might Ask

1. **"Print *all* LCS strings."**
   When backtracking, follow **both** paths whenever there's a tie. Output may be exponential in the number of distinct LCS strings.

2. **"Memory-bounded reconstruction."**
   **Hirschberg's algorithm** computes LCS in **O(m × n) time, O(min(m, n)) space**. Recursion + divide-and-conquer.

3. **"Print the LCS as indices into `a`."**
   Track `i` (not `a[i-1]`) during backtrack — same logic, output indices.

4. **"What if instead of LCS, you wanted Longest Common Substring?"**
   Different DP, different reconstruction. Track the cell where the maximum was achieved; reconstruct backwards while values keep matching.

---

## Key Lessons from This Problem

1. **DP tables are reconstructable** — by walking backwards through them, you can recover the actual solution, not just its size.
2. **Tie-breaking gives one valid answer** — multiple LCS strings exist when there are choices.
3. **Building the answer in reverse** is natural during backtracking; reverse at the end.
