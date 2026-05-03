# Problem: Longest Palindromic Substring

## Category
Strings

## Difficulty
Medium

---

## Problem Statement

Given a string `s`, return the **longest palindromic substring**.

A **palindrome** reads the same forwards and backwards (`"aba"`, `"abba"`).
A **substring** is a contiguous slice of `s` (not a subsequence).

This is **LeetCode #5**.

### Examples

```
"babad"   →  "bab"   (or "aba" — either is correct)
"cbbd"    →  "bb"
"a"       →  "a"
""        →  ""
```

---

## Clarifying Questions to Ask the Interviewer

1. **Substring or subsequence?** Substring — must be contiguous.
2. **If multiple longest palindromes exist, return any one?** Yes, typically.
3. **Case-sensitive?** Yes by default — `"Aa"` is not a palindrome.

---

## Approach

### Naive Approach — O(n³)

Try every substring and check if it's a palindrome. There are O(n²) substrings, each costing O(n) to check. Way too slow.

### The Big Idea — Expand Around Each Center

Every palindrome has a **center**. Two cases:

1. **Odd-length palindrome** has a single character at its center: e.g., `"a*b*a"` — center is the middle `b`.
2. **Even-length palindrome** has its center *between* two characters: e.g., `"ab|ba"` — center is between the two `b`s.

There are exactly `2n - 1` possible centers (n single-char centers + n-1 between-char centers).

For each center, **expand outward** as long as the characters on both sides match. Track the longest palindrome found across all centers.

### Walking through "babad"

```
Index: 0 1 2 3 4
Char:  b a b a d
```

Try every center (odd and even):

| Center | Expansion attempt | Palindrome found |
|---|---|---|
| (0, 0) — 'b' | b | "b" (length 1) |
| (0, 1) — 'b','a' | mismatch | "" (length 0) |
| (1, 1) — 'a' | bab → 'b' = 'b' ✓; expand more: out of bounds | "bab" (length 3) |
| (1, 2) — 'a','b' | mismatch | "" |
| (2, 2) — 'b' | aba → 'a' = 'a' ✓; expand: 'b' vs 'd' ✗ | "aba" (length 3) |
| (2, 3) — 'b','a' | mismatch | "" |
| (3, 3) — 'a' | bad → 'b' ≠ 'd' | "a" (length 1) |
| (3, 4) — 'a','d' | mismatch | "" |
| (4, 4) — 'd' | trivial | "d" (length 1) |

Longest: `"bab"` (or `"aba"`, also length 3).

### How "expand from center" works in code

A helper function takes left and right indices and walks outward:

```
expand(s, l, r):
    while l >= 0 and r < n and s[l] == s[r]:
        l--
        r++
    return r - l - 1     # length when expansion stopped
```

When the loop exits, `l` and `r` have walked one step too far. The length of the matching palindrome is `r - l - 1`.

For each index `i` in the string:
- Try odd center: `expand(s, i, i)`.
- Try even center: `expand(s, i, i+1)`.
- Take the longer of the two.

---

## Time Complexity

**O(n²)** — there are O(n) centers, each expansion is O(n) in the worst case.

## Space Complexity

**O(1)** — no extra arrays needed.

---

## Code (Java)

```java
public String longestPalindrome(String s) {
    if (s == null || s.isEmpty()) return "";

    int start = 0, end = 0;     // [start..end] inclusive of best palindrome

    for (int i = 0; i < s.length(); i++) {
        int len1 = expand(s, i, i);         // odd-length palindrome
        int len2 = expand(s, i, i + 1);     // even-length palindrome
        int len = Math.max(len1, len2);

        if (len > end - start + 1) {
            // Recompute the new boundaries.
            // For length 'len' centered at i:
            //   start = i - (len - 1) / 2
            //   end   = i + len / 2
            // (This handles both odd and even cases correctly.)
            start = i - (len - 1) / 2;
            end = i + len / 2;
        }
    }
    return s.substring(start, end + 1);
}

// Returns the length of the palindrome that can be formed by expanding
// around (l, r). For odd-length palindromes pass l = r = center.
// For even-length pass l, r = center, center + 1.
private int expand(String s, int l, int r) {
    while (l >= 0 && r < s.length() && s.charAt(l) == s.charAt(r)) {
        l--;
        r++;
    }
    // When the while loop ends, l and r are 1 past the palindrome bounds.
    // Length = (r - 1) - (l + 1) + 1 = r - l - 1.
    return r - l - 1;
}
```

---

## Code (Python)

```python
class Solution:
    def longestPalindrome(self, s: str) -> str:
        if not s:
            return ""

        start, end = 0, 0

        def expand(l: int, r: int) -> int:
            while l >= 0 and r < len(s) and s[l] == s[r]:
                l -= 1
                r += 1
            return r - l - 1

        for i in range(len(s)):
            len1 = expand(i, i)             # odd
            len2 = expand(i, i + 1)         # even
            length = max(len1, len2)

            if length > end - start + 1:
                start = i - (length - 1) // 2
                end = i + length // 2

        return s[start:end + 1]
```

---

## Edge Cases

| Case | Output |
|---|---|
| `""` | `""` |
| `"a"` | `"a"` |
| `"aaaa"` (all same) | `"aaaa"` |
| `"abcd"` (no palindrome longer than 1) | any single char (e.g., `"a"`) |
| `"abba"` (even-length winner) | `"abba"` — the even-center case catches it |

---

## Follow-up Questions an Interviewer Might Ask

1. **"Can you do it in O(n)?"**
   Yes — **Manacher's algorithm**. It's clever but tricky to implement; the expand-around-center solution is what most interviewers expect.

2. **"Count the number of palindromic substrings (LeetCode 647)."**
   Same expand-around-center approach. Instead of tracking the longest, count each successful expansion.

3. **"Find the longest palindromic *subsequence* (LeetCode 516)."**
   Different problem — subsequence allows skipping. Solve with DP: `dp[i][j]` = longest palindromic subsequence in `s[i..j]`.

4. **"What if the string is huge (gigabytes)?"**
   Manacher's algorithm becomes attractive at that scale. Or stream-process the string in chunks if you can guarantee no palindrome crosses chunk boundaries (rare).

---

## Key Lessons from This Problem

1. **Every palindrome has a center.** Iterating over centers gives a clean O(n²) algorithm.
2. **Two cases — odd and even.** Always handle both, or you'll miss palindromes like "abba".
3. **Expand-around-center is a recurring pattern** — also useful for counting palindromic substrings.
