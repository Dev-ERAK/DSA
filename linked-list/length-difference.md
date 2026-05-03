# Problem: Intersection of Two Lists — Length-Difference Method

## Category
Linked List

## Difficulty
Easy

---

## Problem Statement

Find the intersection node of two singly linked lists, using the **length-difference** technique. (See `intersection-two-lists.md` for the elegant pointer-switch one-pass version.)

This is the classic, easy-to-explain approach.

### Example

```
A:   4 → 1 → 8 → 4 → 5     (length 5)
B:   5 → 6 → 1 → 8 → 4 → 5 (length 6)
                  ↑
              intersection at node "8"
```

---

## Clarifying Questions to Ask the Interviewer

Same as the standard intersection problem — the two-pointer-switch and the length-difference methods are interchangeable.

1. Compare by reference, not value? (Yes.)
2. O(1) extra space, two passes acceptable? (Yes — that's the trade-off.)

---

## Approach

### The Big Idea

If the two lists intersect, they share the **same tail** — once they merge, they go together to the same `null`. So if list A is `d` nodes longer than list B, the intersection (if any) is at the same distance `d` from each list's start... no wait, that's not quite right.

Let me restate: if A is longer than B by `d` nodes, the **first `d` nodes of A cannot be part of the intersection**. Because B has fewer nodes, the intersection has to start no earlier than B's head.

So we **skip the first `d` nodes of the longer list**. After that, both lists have the same number of remaining nodes — they're aligned. Walking them together, they either meet at the intersection or both reach `null`.

### The algorithm

1. Walk each list to find its length (`lenA`, `lenB`).
2. Advance the longer list's pointer by `|lenA - lenB|` nodes.
3. Walk both pointers together, one step at a time, until they meet or both hit `null`.

### Walking through the example

```
lenA = 5, lenB = 6. Difference = 1, B is longer.
Advance B's pointer by 1: B now points to 6.

  A pointer    B pointer
     4            6
     1            1
     8            8     ← they meet at node 8!  ✓
```

### Why this is "two passes"

Pass 1: count lengths. Pass 2: walk in lockstep. Both passes are O(m + n) → total **O(m + n)**.

### Comparison to the pointer-switch method

Both are O(m + n) time and O(1) space. The pointer-switch is more elegant (single pass), but the length-difference method is easier to explain and debug. Knowing both is a plus.

---

## Time Complexity

**O(m + n)** — at most one length-counting pass plus one walking pass.

## Space Complexity

**O(1)** — just length counters and pointers.

---

## Code (Java)

```java
public ListNode getIntersectionNode(ListNode headA, ListNode headB) {
    int lenA = length(headA);
    int lenB = length(headB);

    ListNode a = headA;
    ListNode b = headB;

    // Advance the longer list's pointer by the difference.
    while (lenA > lenB) {
        a = a.next;
        lenA--;
    }
    while (lenB > lenA) {
        b = b.next;
        lenB--;
    }

    // Now both have the same number of nodes remaining.
    // Walk together until they meet or both reach null.
    while (a != b) {
        a = a.next;
        b = b.next;
    }
    return a;          // intersection node, or null
}

private int length(ListNode head) {
    int n = 0;
    while (head != null) {
        n++;
        head = head.next;
    }
    return n;
}
```

---

## Code (Python)

```python
class Solution:
    def getIntersectionNode(self, headA, headB):
        def length(node):
            n = 0
            while node:
                n += 1
                node = node.next
            return n

        la = length(headA)
        lb = length(headB)
        a, b = headA, headB

        # Advance the longer list's pointer.
        while la > lb:
            a = a.next
            la -= 1
        while lb > la:
            b = b.next
            lb -= 1

        # Walk together.
        while a is not b:
            a = a.next
            b = b.next
        return a
```

---

## Edge Cases

| Case | Result |
|---|---|
| Either list empty | Length is 0; the alignment loops don't run; `a` and `b` start equal (both null) → returns `null`. ✓ |
| Same length, no intersection | Walk together to `null` — return `null`. |
| Same length, intersect at head | Both pointers start equal → return head. |
| One list is entirely contained in the other | Length difference is the size of the prefix; works correctly. |

---

## Follow-up Questions an Interviewer Might Ask

1. **"Can you do it in one pass?"**
   Yes — see the pointer-switch method in `intersection-two-lists.md`.

2. **"Why two passes?"**
   We need to know the lengths before we can align the pointers. There's no way to know the length without walking the list.

3. **"What if the lists are linked lists *with cycles*?"**
   Tricky — length is undefined. First detect cycles (see `linked-list/detect-cycle.md`). If both have cycles in the same loop → they intersect. If only one has a cycle → no intersection.

4. **"What if lists could be very long but have a known max length?"**
   No change — both passes are still bounded by the list size.

---

## Key Lessons from This Problem

1. **"Equalize before walking together"** — a recurring trick in linked-list problems where two pointers must converge.
2. **Two passes are perfectly fine** when both are linear. Trading clarity for cleverness is sometimes the wrong call.
3. **Length-difference vs pointer-switch** — same complexity, different style. Pick the one you can explain best in an interview.
