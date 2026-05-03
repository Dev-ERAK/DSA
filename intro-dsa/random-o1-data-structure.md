# Problem: Insert, Delete, GetRandom — All in O(1)

## Category
Intro to DSA / Design

## Difficulty
Medium

---

## Problem Statement

Design a data structure that supports three operations, **each in O(1) average time**:

1. `insert(val)` — add a value (only if it isn't already there).
2. `remove(val)` — remove a value (only if it's there).
3. `getRandom()` — return any one of the stored values, with **equal probability** for each.

This is LeetCode #380 ("Insert Delete GetRandom O(1)").

### Example

```
insert(1)       -> true   (added)
remove(2)       -> false  (2 wasn't there)
insert(2)       -> true
getRandom()     -> 1 or 2 (each with 50% chance)
remove(1)       -> true
insert(2)       -> false  (2 is already there)
getRandom()     -> 2      (only value left)
```

---

## Clarifying Questions to Ask the Interviewer

1. **Are duplicates allowed?** (No — classic version. There is a harder follow-up that allows them.)
2. **What does `getRandom` mean — does each value need an equal chance?** (Yes — uniform distribution.)
3. **Does `insert` / `remove` need to return whether the value was actually added/removed?** (Yes — boolean return.)
4. **What if `getRandom` is called on an empty structure?** (Usually the interviewer says "we won't call it then." Worth confirming.)

---

## Approach

### Why this problem is tricky

You probably already know two simple structures:

- **HashSet / HashMap** — `insert` and `remove` are O(1), but **picking a random element is hard**. There's no way to pick "the i-th key" of a hashmap; the keys aren't stored in any indexable order.
- **Array (ArrayList in Java, list in Python)** — picking a random element is easy: pick a random index. But `remove(val)` needs to find the value (O(n) scan) and shift everything after it (also O(n)). Too slow.

**Neither one alone is enough.** The trick is to **combine** them.

### The Big Idea — combine an array and a hashmap

- An **array** stores all the values. Random access by index is O(1), so `getRandom` is just "pick a random index."
- A **hashmap** maps each value to its index in the array. So we can look up "where is `val`?" in O(1).

### How does `remove(val)` stay O(1)?

Removing from the middle of an array is normally O(n) because everything after has to shift left. We avoid the shift with a clever swap:

1. Look up `val`'s index `i` in the hashmap.
2. **Swap** `array[i]` with the last element of the array.
3. Update the hashmap so the swapped-in last element now points to index `i`.
4. Pop the last element off the array (which is the one we wanted to remove). This is O(1).
5. Remove `val` from the hashmap.

### Walking through an example

Suppose we've inserted 10, 20, 30, 40 in that order.
```
array:    [10, 20, 30, 40]
indexMap: {10: 0, 20: 1, 30: 2, 40: 3}
```

Now `remove(20)`:
- `20` is at index 1.
- Swap `array[1]` (which is 20) with `array[3]` (which is 40):
  ```
  array: [10, 40, 30, 20]
  ```
- Update the map: `40` is now at index 1.
  ```
  indexMap: {10: 0, 20: 1, 30: 2, 40: 1}
  ```
- Pop the last element (which is now 20, the one we wanted to remove):
  ```
  array: [10, 40, 30]
  ```
- Remove `20` from the map:
  ```
  indexMap: {10: 0, 30: 2, 40: 1}
  ```

All in O(1). The array is still tightly packed, so `getRandom` (pick a random index from `0..size-1`) still gives a uniform distribution.

---

## Time Complexity

- `insert`: **O(1)** average (hashmap lookup + array append)
- `remove`: **O(1)** average (hashmap lookup + swap + pop)
- `getRandom`: **O(1)** (one random index)

Average is the right word because hashmap operations are O(1) **on average** but worst-case O(n) due to collisions. In practice, with a decent hash, this never matters.

## Space Complexity

**O(n)** — the array and the hashmap each hold one entry per value.

---

## Code (Java)

```java
import java.util.*;

class RandomizedSet {
    // The array holds all values. Order doesn't matter — only that
    // values are tightly packed so a random index gives uniform sampling.
    private final List<Integer> list = new ArrayList<>();

    // Maps each value to its index in 'list'.
    private final Map<Integer, Integer> indexMap = new HashMap<>();

    // Random number generator for getRandom().
    private final Random rand = new Random();

    public boolean insert(int val) {
        // Already present? Nothing to do.
        if (indexMap.containsKey(val)) return false;

        // Append the value to the end and record its index.
        indexMap.put(val, list.size());
        list.add(val);
        return true;
    }

    public boolean remove(int val) {
        Integer idx = indexMap.get(val);
        if (idx == null) return false;          // not present

        int lastIndex = list.size() - 1;
        int lastValue = list.get(lastIndex);

        // Move the last value into the slot we're vacating.
        list.set(idx, lastValue);
        indexMap.put(lastValue, idx);

        // Pop the last slot (which now contains a duplicate of lastValue
        // — the original copy lives at index 'idx' now).
        list.remove(lastIndex);
        indexMap.remove(val);
        return true;
    }

    public int getRandom() {
        // Uniform random index → uniform random value.
        return list.get(rand.nextInt(list.size()));
    }
}
```

---

## Code (Python)

```python
import random

class RandomizedSet:
    def __init__(self):
        self.arr = []        # list of values
        self.idx = {}        # value -> index in arr

    def insert(self, val: int) -> bool:
        if val in self.idx:
            return False
        self.idx[val] = len(self.arr)
        self.arr.append(val)
        return True

    def remove(self, val: int) -> bool:
        if val not in self.idx:
            return False

        i = self.idx[val]            # index of val
        last = self.arr[-1]          # value at the end

        # Move 'last' into 'i', then chop off the (now duplicate) end.
        self.arr[i] = last
        self.idx[last] = i
        self.arr.pop()
        del self.idx[val]
        return True

    def getRandom(self) -> int:
        return random.choice(self.arr)
```

---

## Edge Cases

| Case | Why it works |
|---|---|
| Removing the **last** element | Swap is a no-op (we'd be swapping with itself). The pop still works correctly. |
| Inserting a duplicate | `insert` returns false; nothing changes. |
| Removing something not present | `remove` returns false; nothing changes. |
| `getRandom` on empty | Behavior is up to the contract — most solutions assume the caller doesn't do this. Add a guard if needed. |

---

## Follow-up Questions an Interviewer Might Ask

1. **"What if duplicates are allowed?"** Each value now maps to a **set** of indices: `Map<Integer, Set<Integer>>`. On remove, pick any one of those indices, do the same swap-with-last trick, and update both index sets.

2. **"How do you guarantee true uniform randomness?"** The randomness comes from `rand.nextInt(size)` (or `random.choice`), which is uniform over `[0, size)`. Since each value occupies exactly one index in our array, each value gets equal probability.

3. **"What's the worst-case time, not just average?"** Worst case is O(n) for hashmap operations because of collisions. For interview purposes, "O(1) average" is the expected answer.

---

## Key Lessons from This Problem

1. **Combine data structures.** When no single structure has all the properties you need, two together often do.
2. **Swap-with-last** is a recurring trick to remove from a packed array in O(1).
3. **Hashmap for lookup, array for indexing** — this pairing shows up in LRU caches and other classic designs.
