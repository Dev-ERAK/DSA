# Problem: First Non-Repeating Character

## Category
Strings / Hashing

## Difficulty
Easy

---

## Problem Statement

Given a string `s`, find the **first non-repeating character** — a character that appears exactly once. Return its **index**, or `-1` if no such character exists.

This is **LeetCode #387**.

### Examples

```
"leetcode"     →  0    ('l' is the first character that appears only once)
"loveleetcode" →  2    ('v' at index 2)
"aabb"         →  -1   (every character repeats)
""             →  -1
```

---

## Clarifying Questions to Ask the Interviewer

1. **What characters?** Lowercase a-z is the typical assumption. For other alphabets, use a hashmap.
2. **Return the index, the character itself, or both?** Return the index (LeetCode convention).
3. **Case-sensitive?** Yes by default — `'A'` and `'a'` are different.

---

## Approach

### The Big Idea — Two Passes

You can't decide whether a character is "non-repeating" without looking at the **entire string**. So a single pass won't work — the first occurrence might or might not repeat later.

**Two-pass solution:**

1. **Pass 1:** Walk the string, count how many times each character appears.
2. **Pass 2:** Walk the string again. Return the index of the first character whose count is 1.

Total time: O(n + n) = O(n).

### Walking through "loveleetcode"

```
Index: 0  1  2  3  4  5  6  7  8  9 10 11
Char:  l  o  v  e  l  e  e  t  c  o  d  e
```

**Pass 1** — count frequencies:

```
l: 2
o: 2
v: 1
e: 4
t: 1
c: 1
d: 1
```

**Pass 2** — find the first index whose count is 1:

```
Index 0 'l': count 2 — skip
Index 1 'o': count 2 — skip
Index 2 'v': count 1 — return 2!
```

Answer: **2** ✓

### Why two passes is necessary

Imagine a single pass. At index 0 (`'l'`), you don't yet know that `'l'` will appear again at index 4. You can't make a decision until you've seen the whole string.

There's no way to fix this with a more complex single-pass — but two passes is already O(n), which is optimal.

---

## Time Complexity

**O(n)** — two passes, each linear.

## Space Complexity

**O(σ)** — where σ is the alphabet size (constant 26 for a-z).

---

## Code (Java)

```java
public int firstUniqChar(String s) {
    int[] count = new int[26];

    // Pass 1: count each character.
    for (int i = 0; i < s.length(); i++) {
        count[s.charAt(i) - 'a']++;
    }

    // Pass 2: find the first character with count == 1.
    for (int i = 0; i < s.length(); i++) {
        if (count[s.charAt(i) - 'a'] == 1) {
            return i;
        }
    }
    return -1;
}
```

---

## Code (Python)

```python
from collections import Counter

class Solution:
    def firstUniqChar(self, s: str) -> int:
        count = Counter(s)                  # Pass 1
        for i, c in enumerate(s):           # Pass 2
            if count[c] == 1:
                return i
        return -1
```

> `Counter(s)` creates a dict-like object mapping each character to its count, in O(n).

---

## Edge Cases

| Case | Result |
|---|---|
| Empty string | -1 |
| All identical (`"aaaa"`) | -1 |
| All unique (`"abcd"`) | 0 |
| Single character | 0 |
| Mix where the first char repeats but a later one doesn't | Index of the first non-repeater |

---

## Follow-up Questions an Interviewer Might Ask

1. **"What if the input is a stream — characters arrive one at a time, and at any moment you need to report the first non-repeater?"**
   This is **LeetCode 387 streaming variant**. Maintain:
   - A frequency hashmap.
   - A queue of candidate indices (each character's first appearance).
   When you read a new character, increment its count. Then while the front of the queue corresponds to a character with count > 1, pop it. The front of the queue is your answer (or `-1` if empty).

2. **"Unicode support?"**
   Replace `int[26]` with `HashMap<Integer, Integer>` over codepoints.

3. **"Can it be done in one pass?"**
   No — see the explanation above. You can't determine "non-repeating" without seeing the rest of the string.

4. **"Find the *last* non-repeating character?"**
   Same Pass 1, but Pass 2 walks the string right-to-left.

---

## Key Lessons from This Problem

1. **Two passes are sometimes necessary** — don't force everything into a single pass.
2. **Frequency counts** are the natural tool for "appears once / appears multiple times" questions.
3. **Streaming versions** of array/string problems typically introduce a queue or heap to maintain "the next answer" efficiently.
