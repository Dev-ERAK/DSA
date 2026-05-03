# Problem: Valid Anagram

## Category
Strings / Hashing

## Difficulty
Easy

---

## Problem Statement

Two strings are **anagrams** if they contain the same letters in the same quantities, just rearranged.

Given two strings `s` and `t`, return `true` if `t` is an anagram of `s`.

This is **LeetCode #242**.

### Examples

```
"listen", "silent"    →  true   (same letters)
"rat", "car"          →  false  (different letters)
"a", "b"              →  false
"", ""                →  true   (both empty — vacuously equal)
```

---

## Clarifying Questions to Ask the Interviewer

1. **Case-sensitive?** Usually yes — `"a"` ≠ `"A"`. Confirm.
2. **What characters?** Lowercase a-z is the typical assumption. For Unicode, use a hashmap.
3. **What if lengths differ?** Then they're definitely not anagrams — return `false` immediately.

---

## Approach

### Method 1 — Sort and Compare (O(n log n))

Sort both strings; if the sorted versions match, they're anagrams.

```
"listen" → sorted → "eilnst"
"silent" → sorted → "eilnst"
Match → true
```

Simple but slower than necessary.

### Method 2 — Frequency Counting (O(n))

Count how often each character appears in each string. If both counts match, they're anagrams.

For lowercase a-z, use an `int[26]` array. For larger alphabets, use a hashmap.

#### Trick — count once, decrement once

Instead of building two count arrays, walk both strings together:
- For each character in `s`, increment its count.
- For each character in `t`, decrement its count.

If they're anagrams, all counts end at 0. If any count is non-zero, they aren't.

This works because if both strings have the same length and the running net count is zero everywhere, they had identical character frequencies.

### Walking through "anagram", "nagaram"

Both have length 7.

Counts after processing:

| step | char in s, t | a | n | g | r | m |
|---|---|---|---|---|---|---|
| 1 | a, n | +1, -0 | -1 | 0 | 0 | 0 |
| ... after all 7 | | 0 | 0 | 0 | 0 | 0 |

All zero → they're anagrams.

---

## Time Complexity

**O(n)** with frequency counting.
**O(n log n)** with sorting.

## Space Complexity

**O(σ)** — where σ is the alphabet size (constant 26 for a-z).

---

## Code (Java)

```java
public boolean isAnagram(String s, String t) {
    if (s.length() != t.length()) return false;

    int[] count = new int[26];                  // 26 lowercase letters

    for (int i = 0; i < s.length(); i++) {
        count[s.charAt(i) - 'a']++;             // character to index 0..25
        count[t.charAt(i) - 'a']--;
    }

    for (int c : count) {
        if (c != 0) return false;
    }
    return true;
}
```

> **Note on `s.charAt(i) - 'a'`:** in Java, `char` is a numeric type. Subtracting `'a'` converts a letter into a 0..25 index. So `'a'` → 0, `'b'` → 1, `'z'` → 25.

### Unicode-safe variant (using a hashmap)

```java
public boolean isAnagramUnicode(String s, String t) {
    if (s.length() != t.length()) return false;

    Map<Character, Integer> count = new HashMap<>();
    for (char c : s.toCharArray()) count.merge(c, 1, Integer::sum);
    for (char c : t.toCharArray()) {
        if (count.merge(c, -1, Integer::sum) < 0) return false;
    }
    return true;
}
```

---

## Code (Python)

```python
from collections import Counter

class Solution:
    def isAnagram(self, s: str, t: str) -> bool:
        # Cleanest: use Counter
        return Counter(s) == Counter(t)
```

### Manual count

```python
class Solution:
    def isAnagram_manual(self, s: str, t: str) -> bool:
        if len(s) != len(t):
            return False
        count = [0] * 26
        for cs, ct in zip(s, t):
            count[ord(cs) - ord('a')] += 1
            count[ord(ct) - ord('a')] -= 1
        return all(c == 0 for c in count)
```

> `ord('a')` returns the integer value 97. `ord(c) - ord('a')` gives 0..25 for lowercase letters.

---

## Edge Cases

| Case | Result |
|---|---|
| Both empty | `true` (length match, no characters to mismatch) |
| Different lengths | `false` (immediate exit) |
| Same string | `true` (it's an anagram of itself) |
| Same letters, different cases (`"a"`, `"A"`) | `false` — case matters |

---

## Follow-up Questions an Interviewer Might Ask

1. **"What about Unicode? My input has emoji."**
   Use `String.codePoints()` (Java) or iterate over characters (Python — Python strings are already Unicode-aware) and use a `HashMap<Integer, Integer>` keyed on codepoints.

2. **"Group all anagram pairs from a list of strings."**
   See `strings/group-anagrams.md`. Use a hashmap from "canonical form" (sorted string or count tuple) to list of strings.

3. **"Is there a way without extra space?"**
   Sort one of the strings in place and compare — but most languages don't let you mutate strings. So you'd need character arrays.

4. **"What's the difference between an anagram and a permutation?"**
   In string problems they're the same: rearrangements of the same characters.

---

## Key Lessons from This Problem

1. **Frequency counts** are the natural representation for "anagram" or "same multiset" questions.
2. **A length check first** is a free O(1) shortcut.
3. **`int[26]`** is a fast, low-overhead alternative to a hashmap when the alphabet is fixed.
