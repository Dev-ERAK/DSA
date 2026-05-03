# Problem: Intersection of Two Linked Lists

## Category
Linked List / Two Pointers

## Difficulty
Easy

---

## Problem Statement

Given the heads of two singly linked lists, return the **node** at which they intersect — or `null` if they never share a node.

"Intersect" means the **same node** (by reference, not just same value). Two lists that share a node from some point onward continue together to the same end.

This is **LeetCode #160**.

### Visualizing intersection

```
A:   a1 → a2 → a3 ↘
                   c1 → c2 → c3 → null
B:        b1 → b2 ↗
```

Both lists share the tail `c1 → c2 → c3`. The answer is `c1`.

If they never merge, the answer is `null`.

---

## Clarifying Questions to Ask the Interviewer

1. **Do we compare by value or by reference?** By reference — the same node, not just same `val`.
2. **Can the lists have different lengths?** Yes (this is the standard case).
3. **Use O(1) extra memory?** Yes — that's the elegant solution.

---

## Approach

### Naive Approach — HashSet (O(m + n) space)

Walk list A; put every node into a hashset. Then walk list B; the first node already in the set is the intersection.

Works but uses O(n) memory.

### Length-Difference Approach (O(1) space, two passes)

If list A is longer than list B by `d` nodes, the intersection (if any) is at the same distance from the *end*. So:

1. Find the lengths of both lists.
2. Advance the longer list's pointer by `d = |lenA - lenB|`.
3. Now both pointers are equidistant from the end. Walk in lockstep until they meet (intersection) or both hit `null`.

### Pointer-Switch Trick (O(1) space, one pass — most elegant)

Use two pointers `a` and `b`, starting at the heads of A and B.

Move both one step at a time. **When a pointer hits the end of its list, jump it to the head of the *other* list.**

Why this works: after at most one switch each, both pointers will have walked exactly `lenA + lenB` steps. They are perfectly aligned at the same distance from any intersection — so they meet at the intersection (or both hit `null` if there is none).

#### Walking through an example

A: `4 → 1 → 8 → 4 → 5` (length 5)
B: `5 → 6 → 1 → 8 → 4 → 5` (length 6)
They intersect at the `8`.

```
Step  a's path                    b's path
 0    4 (A)                       5 (B)
 1    1 (A)                       6 (B)
 2    8 (A)                       1 (B)
 3    4 (A)                       8 (B)
 4    5 (A)                       4 (B)
 5    null → switch to B head     5 (B)
 6    5 (B head)                  null → switch to A head
 7    6 (B)                       4 (A head)
 8    1 (B)                       1 (A)
 9    8 (B)                       8 (A)  ← MEET at 8! ✓
```

Both pointers walked the same total distance (length A + length B = 11 steps), so they aligned perfectly.

#### What if the lists don't intersect?

Both pointers eventually walk through both lists once and reach `null` at the same time. The loop exits with `a == b == null`.

---

## Time Complexity

**O(m + n)** — each pointer walks at most `m + n` steps.

## Space Complexity

**O(1)** — just two pointers.

---

## Code (Java)

```java
public ListNode getIntersectionNode(ListNode headA, ListNode headB) {
    if (headA == null || headB == null) return null;

    ListNode a = headA;
    ListNode b = headB;

    // They will either meet at the intersection or both become null.
    while (a != b) {
        a = (a == null) ? headB : a.next;
        b = (b == null) ? headA : b.next;
    }
    return a;       // either intersection node or null
}
```

---

## Code (Python)

```python
class Solution:
    def getIntersectionNode(self, headA, headB):
        if not headA or not headB:
            return None

        a, b = headA, headB
        while a is not b:
            a = headB if a is None else a.next
            b = headA if b is None else b.next
        return a
```

---

## Edge Cases

| Case | Result |
|---|---|
| Either list is empty | `null` |
| Lists don't intersect | Both pointers reach `null` after `m + n` steps |
| Lists intersect at the head | Both pointers meet immediately on step 0 |
| Equal-length lists with no intersection | Both reach `null` simultaneously after `m` steps |

---

## Follow-up Questions an Interviewer Might Ask

1. **"What if the lists could contain cycles?"**
   First detect cycles. If both have cycles in the same loop, they intersect. If one has a cycle and the other doesn't, they can't intersect.

2. **"Use the length-difference method instead."**
   See `linked-list/length-difference.md` for that variant.

3. **"Why does the pointer switch work mathematically?"**
   Each pointer ends up walking `lenA + lenB` steps. If they intersect, they're aligned at the same distance from the meeting node. If not, they end at `null` at the same time.

4. **"Modifying the input?"**
   You could mark visited nodes in some way (with a flag), but that violates "don't modify input" and is generally considered a bad design.

---

## Key Lessons from This Problem

1. **The pointer-switch trick** is one of the most elegant ideas in linked-list problems. It's worth memorizing.
2. **Compare by reference, not by value** — when the question is about "same node," use `==` (Java) or `is` (Python).
3. **Equalizing path lengths** — the recurring theme: when two pointers must meet at a specific point, give them equal "runway" by advancing the longer one first or by switching lists.
