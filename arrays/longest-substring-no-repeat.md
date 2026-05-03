# Problem: Longest Substring Without Repeating Characters

## Category
Arrays / Strings / Sliding Window

## Difficulty
Medium

---

## Problem Statement

Given a string `s`, return the **length** of the longest substring that has **no repeating characters**.

A **substring** is a contiguous slice of the string (not a subsequence — order *and* contiguity matter).

This is **LeetCode #3** — one of the most common interview questions.

### Examples

```
"abcabcbb"  →  3   (the substring "abc")
"bbbbb"     →  1   (just one "b")
"pwwkew"    →  3   ("wke" — note that "pwke" is a subsequence, not a substring, so it doesn't count)
""          →  0
```

---

## Clarifying Questions to Ask the Interviewer

1. **Substring or subsequence?** Substring — must be contiguous.
2. **What characters? ASCII, lowercase letters, Unicode?** A hashmap handles all of them; an `int[128]` works for ASCII.
3. **Return the length, or the substring itself?** Length is standard.

---

## Approach

### Brute force — O(n³) or O(n²)

Try every substring and check uniqueness. Even with smart checking, you're still doing nested loops over starts, ends, and a uniqueness scan. Way too slow for `n = 10⁵`.

### The Big Idea — Variable-Size Sliding Window

Imagine the substring as a **window** with a left pointer `l` and a right pointer `r`. We grow the window to the right; if a duplicate sneaks in, we shrink from the left until the window is duplicate-free again.

#### What we track

A hashmap `lastIndex` mapping each character to the **most recent index** where we saw it.

#### The algorithm

For each `r` from `0` to `n-1`:

1. If `s[r]` is already in `lastIndex` **AND** that index is `≥ l` (meaning the duplicate is **inside** our current window), move `l` to `lastIndex[s[r]] + 1`.
2. Update `lastIndex[s[r]] = r`.
3. Update `best = max(best, r - l + 1)`.

The key insight: **`l` never moves backward**. Each character is processed once when added to the window and once when forgotten. Total: O(n).

### Walking through "abcabcbb"

```
Index:  0  1  2  3  4  5  6  7
Char:   a  b  c  a  b  c  b  b
```

| r | char | lastIndex says? | l adjusts? | window | best |
|---|---|---|---|---|---|
| 0 | a | not seen | l = 0 | "a" (length 1) | 1 |
| 1 | b | not seen | l = 0 | "ab" (length 2) | 2 |
| 2 | c | not seen | l = 0 | "abc" (length 3) | 3 |
| 3 | a | seen at 0, 0 ≥ l=0 → l = 1 | l = 1 | "bca" (length 3) | 3 |
| 4 | b | seen at 1, 1 ≥ l=1 → l = 2 | l = 2 | "cab" (length 3) | 3 |
| 5 | c | seen at 2, 2 ≥ l=2 → l = 3 | l = 3 | "abc" (length 3) | 3 |
| 6 | b | seen at 4, 4 ≥ l=3 → l = 5 | l = 5 | "cb" (length 2) | 3 |
| 7 | b | seen at 6, 6 ≥ l=5 → l = 7 | l = 7 | "b" (length 1) | 3 |

Answer: **3** ✓

### Why we check `lastIndex[c] ≥ l`

A character may have been seen before but **outside** the current window — in which case it's no longer relevant. The check `lastIndex[c] ≥ l` ensures we only react to duplicates that are actually inside the window.

---

## Time Complexity

**O(n)** — each character is examined a constant number of times.

## Space Complexity

**O(min(n, σ))** — where σ is the size of the alphabet (e.g., 26, 128, or all Unicode codepoints). The hashmap holds at most one entry per distinct character.

---

## Code (Java)

```java
public int lengthOfLongestSubstring(String s) {
    Map<Character, Integer> lastIndex = new HashMap<>();
    int best = 0;
    int l = 0;                                  // window's left bound

    for (int r = 0; r < s.length(); r++) {
        char c = s.charAt(r);

        // If we've seen this character AND it's inside the window,
        // jump l past the previous occurrence.
        if (lastIndex.containsKey(c) && lastIndex.get(c) >= l) {
            l = lastIndex.get(c) + 1;
        }

        // Update the last-seen position.
        lastIndex.put(c, r);

        // Track the longest window seen so far.
        best = Math.max(best, r - l + 1);
    }
    return best;
}
```

---

## Code (Python)

```python
class Solution:
    def lengthOfLongestSubstring(self, s: str) -> int:
        last_index = {}
        best = 0
        l = 0

        for r, c in enumerate(s):
            # If c is in the current window, jump l past it.
            if c in last_index and last_index[c] >= l:
                l = last_index[c] + 1

            last_index[c] = r
            best = max(best, r - l + 1)

        return best
```

---

## Edge Cases

| Case | What happens |
|---|---|
| `""` (empty) | Loop doesn't run. Returns 0. |
| `"aaaa"` (all same) | Window keeps shrinking to length 1. Returns 1. |
| `"abcdef"` (all unique) | Window grows to full length. Returns n. |
| Single character | Returns 1. |
| Mix of cases (`"aA"`) | They're distinct characters — case-sensitive. Returns 2. |

---

## Follow-up Questions an Interviewer Might Ask

1. **"What if you can have at most K distinct characters?"** (LeetCode 340)
   Same shape: track a count of each character in the window. While `distinct > K`, shrink from the left, decrementing counts and removing characters whose count hits 0.

2. **"Return the actual substring, not just the length."**
   Track `bestStart` whenever you update `best`. The substring is `s[bestStart..bestStart+best-1]`.

3. **"Why don't we move `l` step by step?"**
   We *could*, but jumping `l` to `lastIndex[c] + 1` skips ahead in O(1) instead of walking one character at a time. Both are O(n) overall, but the jump is cleaner.

4. **"What if the alphabet is fixed (only lowercase letters)?"**
   Use an `int[26]` (or `int[128]` for ASCII) instead of a hashmap. Faster constants, no hashing overhead.

---

## Key Lessons from This Problem

1. **Variable-size sliding window** is the go-to technique for "longest/shortest substring with property X."
2. **The left pointer never moves backward** — this is what makes the algorithm O(n).
3. **Always check whether the duplicate is *inside the window***. Without this guard, you'd needlessly skip past characters that are no longer relevant.
