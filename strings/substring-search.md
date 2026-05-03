# Problem: Substring Search (Implement strStr / indexOf)

## Category
Strings / Pattern Matching

## Difficulty
Easy

---

## Problem Statement

Given two strings:
- `haystack` — the bigger string we're searching in.
- `needle` — the smaller string we're searching for.

Return the **starting index** of the first occurrence of `needle` in `haystack`, or `-1` if it doesn't appear.

This is **LeetCode #28** — like writing your own version of `String.indexOf`.

### Examples

```
haystack = "sadbutsad", needle = "sad"     →  0   (matches at index 0)
haystack = "leetcode",  needle = "leeto"   →  -1  (not found)
haystack = "hello",     needle = ""        →  0   (empty needle matches at start)
```

---

## Clarifying Questions to Ask the Interviewer

1. **What if `needle` is empty?** Return 0 (matches the convention of Java's `indexOf` and Python's `find`).
2. **Case-sensitive?** Yes by default.
3. **Are we allowed to use built-in functions like `String.indexOf`?** Usually no — the question is testing whether you can implement it yourself.

---

## Approach

We'll cover two methods: the simple naive one (which is what most interviews accept) and KMP (the optimal version, asked sometimes).

### Method 1 — Naive (Sliding Compare)

For each possible starting index `i` in `haystack`, check whether `haystack[i..i+m-1]` matches `needle` character by character.

```
for i in 0..n-m:
    j = 0
    while j < m and haystack[i+j] == needle[j]:
        j++
    if j == m:
        return i              # full match
return -1
```

#### Walking through `haystack = "abxabcab"`, `needle = "abc"`

```
i=0: 'a','a' ✓; 'b','b' ✓; 'x','c' ✗ — fail
i=1: 'b','a' ✗ — fail
i=2: 'x','a' ✗
i=3: 'a','a' ✓; 'b','b' ✓; 'c','c' ✓ — MATCH at i=3!
```

Returns 3.

#### Why naive is slow in the worst case

Worst-case input: `haystack = "aaaaaaab"`, `needle = "aaab"`. At every starting position you compare almost all of `needle` before failing. Total: O(n × m).

For typical inputs (random text, short patterns), naive is very fast in practice.

### Method 2 — KMP (Knuth-Morris-Pratt) — O(n + m)

The **insight** of KMP: when a mismatch happens after partial success, we already know what part of the pattern matched. We don't need to re-check those characters in `haystack`.

KMP precomputes a "failure table" (sometimes called the **LPS** array — Longest Proper Prefix that's also a Suffix). When a mismatch occurs at pattern position `j`, the table tells us "you can safely jump to pattern position `lps[j-1]`" — without backing up the haystack pointer.

This is more involved to implement but gives a guaranteed **O(n + m)** runtime.

For most interviews, **naive is accepted**. KMP is typically only required when interviewers explicitly ask for the optimal solution.

---

## Time Complexity

- Naive: **O(n × m)** worst case, fast in practice.
- KMP: **O(n + m)** worst case.

## Space Complexity

- Naive: **O(1)**.
- KMP: **O(m)** for the LPS array.

---

## Code (Java) — Naive

```java
public int strStr(String haystack, String needle) {
    int n = haystack.length();
    int m = needle.length();

    if (m == 0) return 0;
    if (m > n) return -1;

    // Try every possible starting index in haystack.
    for (int i = 0; i <= n - m; i++) {
        int j = 0;
        // Try to match needle starting at i.
        while (j < m && haystack.charAt(i + j) == needle.charAt(j)) {
            j++;
        }
        if (j == m) {
            return i;       // matched the entire needle
        }
    }
    return -1;
}
```

### KMP (advanced)

```java
public int strStrKMP(String haystack, String needle) {
    int n = haystack.length(), m = needle.length();
    if (m == 0) return 0;

    int[] lps = computeLPS(needle);

    int i = 0;     // pointer in haystack
    int j = 0;     // pointer in needle

    while (i < n) {
        if (haystack.charAt(i) == needle.charAt(j)) {
            i++;
            j++;
            if (j == m) return i - m;       // full match
        } else if (j > 0) {
            j = lps[j - 1];                  // jump using failure table
        } else {
            i++;
        }
    }
    return -1;
}

// Build the failure table: lps[i] = length of the longest proper prefix
// of needle[0..i] that is also a suffix of needle[0..i].
private int[] computeLPS(String p) {
    int[] lps = new int[p.length()];
    int len = 0, i = 1;
    while (i < p.length()) {
        if (p.charAt(i) == p.charAt(len)) {
            lps[i++] = ++len;
        } else if (len > 0) {
            len = lps[len - 1];
        } else {
            lps[i++] = 0;
        }
    }
    return lps;
}
```

---

## Code (Python) — Naive

```python
class Solution:
    def strStr(self, haystack: str, needle: str) -> int:
        n, m = len(haystack), len(needle)
        if m == 0:
            return 0

        for i in range(n - m + 1):
            if haystack[i:i + m] == needle:
                return i
        return -1
```

> Python's `haystack[i:i+m]` slicing is O(m) per call, so the total stays at O(n × m).

### KMP

```python
def strStr_kmp(haystack: str, needle: str) -> int:
    if not needle:
        return 0

    # Build failure table
    lps = [0] * len(needle)
    length = 0
    i = 1
    while i < len(needle):
        if needle[i] == needle[length]:
            length += 1
            lps[i] = length
            i += 1
        elif length > 0:
            length = lps[length - 1]
        else:
            lps[i] = 0
            i += 1

    # Search
    i = j = 0
    while i < len(haystack):
        if haystack[i] == needle[j]:
            i += 1
            j += 1
            if j == len(needle):
                return i - j
        elif j > 0:
            j = lps[j - 1]
        else:
            i += 1
    return -1
```

---

## Edge Cases

| Case | Result |
|---|---|
| Empty needle | 0 |
| Needle longer than haystack | -1 |
| Multiple occurrences | Return the first |
| Equal strings | 0 |
| Needle = single character | Same as scanning `haystack` for that character |

---

## Follow-up Questions an Interviewer Might Ask

1. **"Find *all* occurrences."**
   Continue scanning after each match. With KMP, after a match at index `i`, set `j = lps[j - 1]` instead of stopping.

2. **"Can you do it without using `String.indexOf` or slicing?"**
   The naive code above uses neither. KMP also doesn't.

3. **"Pattern with wildcards (e.g., `?` matches any char)?"**
   Modify the comparison to allow `?` to match anything; or use regex/DP for full wildcard support.

4. **"Multiple patterns at once?"**
   Use **Aho-Corasick** — builds a finite automaton from all patterns and processes haystack in O(n + total pattern lengths + matches).

---

## Key Lessons from This Problem

1. **Naive is good enough most of the time.** Don't over-engineer unless asked.
2. **KMP's insight: pattern self-similarity lets you skip work.** The same idea (failure functions) shows up in regex engines and trie-based pattern matchers.
3. **String slicing is O(m) in most languages** — be careful when assessing the complexity of "Pythonic" solutions.
