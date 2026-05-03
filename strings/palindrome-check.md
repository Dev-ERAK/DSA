# Problem: Valid Palindrome (Ignoring Special Characters)

## Category
Strings / Two Pointers

## Difficulty
Easy

---

## Problem Statement

A **palindrome** is a string that reads the same forwards and backwards (e.g., "racecar", "level").

Given a string `s`, return `true` if it's a palindrome **after**:
- Ignoring spaces, punctuation, and any non-alphanumeric characters.
- Treating uppercase and lowercase as the same.

This is **LeetCode #125**.

### Examples

```
"A man, a plan, a canal: Panama"  →  true
After cleaning: "amanaplanacanalpanama" — reads the same both ways.

"race a car"                      →  false
After cleaning: "raceacar" — not a palindrome.

" "                               →  true
After cleaning: "" — empty string is vacuously a palindrome.
```

---

## Clarifying Questions to Ask the Interviewer

1. **What counts as "alphanumeric"?** Letters (a-z, A-Z) and digits (0-9). Anything else is ignored.
2. **Is comparison case-insensitive?** Yes.
3. **What about the empty string?** Conventionally, yes — it's a palindrome.
4. **Unicode support?** Usually ASCII for this problem; for true Unicode you'd need locale-aware lowercase and definitions of "letter."

---

## Approach

### Naive Approach — Build a cleaned string, compare with reverse

1. Build a new string with only lowercase letters and digits.
2. Reverse it.
3. Compare with the cleaned original.

This works but uses **O(n) extra space** for the cleaned string.

### Optimal Approach — Two Pointers (O(1) extra space)

Walk inward from both ends:

- Left pointer `l` starts at index 0.
- Right pointer `r` starts at the last index.
- Skip non-alphanumeric characters on each side.
- Compare lowercase versions of the characters at `l` and `r`. If they don't match → not a palindrome.
- Move `l` right and `r` left.

Continue until `l >= r`.

### Walking through the example

`"A man, a plan, a canal: Panama"`

```
l = 0 ('A'), r = 29 ('a')      → both alpha, lowercase compare 'a' == 'a' ✓
l = 1 (' '), skip               → l = 2 ('m')
                                → r = 28 ('m'), compare 'm' == 'm' ✓
l = 3 ('a'), r = 27 ('a')      → ✓
l = 4 ('n'), r = 26 ('n')      → ✓
l = 5 (','), skip → l = 6 (' '), skip → l = 7 ('a')
                                → r = 25 ('a') ✓
... and so on
```

All comparisons succeed → return `true`.

### Why two pointers work

If the string is a palindrome, then after stripping non-alphanumerics, the **i-th character from the left** equals the **i-th from the right**. We're checking exactly that, in O(n) time.

---

## Time Complexity

**O(n)** — each character is examined at most twice (once by left pointer, once by right).

## Space Complexity

**O(1)** — no extra string built; just two index variables.

---

## Code (Java)

```java
public boolean isPalindrome(String s) {
    int l = 0, r = s.length() - 1;

    while (l < r) {
        // Skip non-alphanumeric on the left.
        while (l < r && !Character.isLetterOrDigit(s.charAt(l))) {
            l++;
        }
        // Skip non-alphanumeric on the right.
        while (l < r && !Character.isLetterOrDigit(s.charAt(r))) {
            r--;
        }

        // Compare in lowercase.
        if (Character.toLowerCase(s.charAt(l))
                != Character.toLowerCase(s.charAt(r))) {
            return false;
        }
        l++;
        r--;
    }
    return true;
}
```

---

## Code (Python)

```python
class Solution:
    def isPalindrome(self, s: str) -> bool:
        l, r = 0, len(s) - 1
        while l < r:
            while l < r and not s[l].isalnum():
                l += 1
            while l < r and not s[r].isalnum():
                r -= 1
            if s[l].lower() != s[r].lower():
                return False
            l += 1
            r -= 1
        return True
```

---

## Edge Cases

| Case | Result | Why |
|---|---|---|
| `""` (empty) | `true` | Vacuously a palindrome. Loop doesn't execute. |
| `"a"` (single char) | `true` | Single character reads the same both ways. |
| `".,!"` (all non-alpha) | `true` | After cleaning it's the empty string. |
| `"Aa"` | `true` | Case-insensitive. |
| `"0P"` | `false` | `'0'` and `'P'` differ — they're not the same letter even lowercased. |

---

## Follow-up Questions an Interviewer Might Ask

1. **"What if you can delete *one* character to make it a palindrome (LeetCode 680)?"**
   When you find a mismatch at `(l, r)`, try two repair options:
   - Skip the left character: check if `s[l+1..r]` is a palindrome.
   - Skip the right character: check if `s[l..r-1]` is a palindrome.
   Return true if either works.

2. **"Find the longest palindromic substring."**
   Different problem — see `strings/longest-palindrome.md`. Use "expand around center" technique.

3. **"What about Unicode strings with combining characters?"**
   Normalize to NFC or NFKC, lowercase using `Character.toLowerCase` (Java) or `str.casefold()` (Python). Then run the same algorithm.

4. **"Could you do it without modifying input or building strings?"**
   Yes — the two-pointer code above doesn't allocate any string, only compares characters.

---

## Key Lessons from This Problem

1. **Two pointers** are perfect when you need to compare items from both ends.
2. **Don't build a cleaned copy** if you can skip "bad" characters in place — saves memory.
3. **`while l < r`** in the inner loops is critical — it prevents both pointers from crossing during the skip phase.
