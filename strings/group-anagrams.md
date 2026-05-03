# Problem: Group Anagrams

## Category
Strings / Hashing

## Difficulty
Medium

---

## Problem Statement

Given an array of strings, **group together** the ones that are anagrams of each other.

Two strings are **anagrams** if they have the exact same letters in the same quantities (just rearranged).

This is **LeetCode #49**.

### Example

```
Input:  ["eat", "tea", "tan", "ate", "nat", "bat"]

Output: [
  ["eat", "tea", "ate"],     ← all rearrangements of "aet"
  ["tan", "nat"],            ← all rearrangements of "ant"
  ["bat"]                    ← alone
]
```

(Order of groups, and order within a group, generally doesn't matter.)

---

## Clarifying Questions to Ask the Interviewer

1. **Does the order of groups matter?** Usually no.
2. **Lowercase only?** Often yes; otherwise use a hashmap-based key.
3. **Empty strings allowed?** Sure — they're all anagrams of each other.

---

## Approach

### The Big Idea — Find a "Canonical Form"

Two strings are anagrams iff they have the **same character multiset**. So we need a unique "fingerprint" that's the same for all anagrams of a word.

Two natural fingerprints:

#### Fingerprint 1: Sort the letters

```
"eat" → sorted → "aet"
"tea" → sorted → "aet"     ← same fingerprint, group together
"tan" → sorted → "ant"     ← different fingerprint, separate group
```

This is **O(k log k)** per word (where k is word length), but very simple.

#### Fingerprint 2: Character count tuple

Build an array of 26 counts (for a-z), and use that as the key.

```
"eat" → [1,0,0,0,1,0,0,0,0,0,0,0,0,0,0,0,0,0,0,1, ...]   ('a':1, 'e':1, 't':1)
```

This is **O(k)** per word — slightly faster, especially for long words.

### Walking through the algorithm

For each string in the input:
1. Compute its canonical key (sorted letters or count tuple).
2. Look up the key in a hashmap.
3. Append the original string to the list at that key.

When done, return the hashmap's values.

```
Input: ["eat", "tea", "tan", "ate", "nat", "bat"]

Map after each step:
"eat" → key "aet" → {"aet": ["eat"]}
"tea" → key "aet" → {"aet": ["eat", "tea"]}
"tan" → key "ant" → {"aet": ["eat","tea"], "ant": ["tan"]}
"ate" → key "aet" → {"aet": ["eat","tea","ate"], "ant": ["tan"]}
"nat" → key "ant" → ...
"bat" → key "abt" → {"aet": [...], "ant": [...], "abt": ["bat"]}

Final answer: list of values.
```

---

## Time Complexity

- Sort key: **O(n × k log k)** where n = number of words, k = max word length.
- Count key: **O(n × k)** — slightly faster.

## Space Complexity

**O(n × k)** for the hashmap.

---

## Code (Java)

```java
public List<List<String>> groupAnagrams(String[] strs) {
    Map<String, List<String>> groups = new HashMap<>();

    for (String s : strs) {
        // Sort the letters to make a canonical form.
        char[] chars = s.toCharArray();
        Arrays.sort(chars);
        String key = new String(chars);

        // computeIfAbsent: if key doesn't exist yet, put a new empty list.
        groups.computeIfAbsent(key, k -> new ArrayList<>()).add(s);
    }
    return new ArrayList<>(groups.values());
}
```

### Count-based key (faster for long words)

```java
public List<List<String>> groupAnagramsCount(String[] strs) {
    Map<String, List<String>> groups = new HashMap<>();

    for (String s : strs) {
        int[] count = new int[26];
        for (char c : s.toCharArray()) {
            count[c - 'a']++;
        }
        // Build a string from the count array, e.g., "#1#0#0...#1"
        StringBuilder sb = new StringBuilder();
        for (int i = 0; i < 26; i++) {
            sb.append('#').append(count[i]);
        }
        groups.computeIfAbsent(sb.toString(), k -> new ArrayList<>()).add(s);
    }
    return new ArrayList<>(groups.values());
}
```

> Why the `#` separators? Without them, `[1, 11]` and `[11, 1]` would both become `"111"`. The separators avoid collisions.

---

## Code (Python)

```python
from collections import defaultdict

class Solution:
    # Sort key — concise
    def groupAnagrams(self, strs):
        groups = defaultdict(list)
        for s in strs:
            key = ''.join(sorted(s))
            groups[key].append(s)
        return list(groups.values())

    # Count tuple — faster for long words
    def groupAnagrams_count(self, strs):
        groups = defaultdict(list)
        for s in strs:
            count = [0] * 26
            for c in s:
                count[ord(c) - ord('a')] += 1
            groups[tuple(count)].append(s)     # tuples are hashable
        return list(groups.values())
```

> Why a `tuple` and not a `list`? Lists in Python aren't hashable (they're mutable). Tuples are.

---

## Edge Cases

| Case | Result |
|---|---|
| Empty array | Empty list `[]`. |
| All same string | One group with duplicates `[["abc","abc","abc"]]`. |
| Mix of empty and non-empty strings | Empty strings group together; the others group normally. |
| Single string | One group with that one string. |

---

## Follow-up Questions an Interviewer Might Ask

1. **"Which key is faster — sort or count?"**
   Count is O(k) per word; sort is O(k log k). For very short words the difference is negligible. For longer words, count wins.

2. **"What if the alphabet is unbounded (Unicode)?"**
   Sort key still works: just sort the characters of the string. Count key would need a hashmap, not a fixed array.

3. **"What if strings can be very long but mostly unique?"**
   Sort key is fine. The bottleneck is rarely the key computation; it's the hashmap operations.

4. **"Memory-bound situation, must reduce footprint."**
   Sort key uses much less memory than a count tuple of 26 ints per word. Or, hash the key into a fixed-size integer.

---

## Key Lessons from This Problem

1. **"Canonical form" is a powerful idea.** When you need to group "things that are equivalent in some way," compute a key that's the same for all equivalent items.
2. **Hashmap of key → list** is the universal pattern for "group by X."
3. **In Python, use tuples (not lists) as hashmap keys** — lists are mutable and not hashable.
