# Problem: Detect a Cycle in a Linked List

## Category
Linked List / Two Pointers

## Difficulty
Easy

---

## Problem Statement

Given a linked list, return `true` if it contains a **cycle** (a loop where some node's `next` points back to an earlier node), and `false` otherwise.

This is **LeetCode #141**.

### Visualizing a cycle

```
1 → 2 → 3 → 4 → 5
        ↑       │
        └───────┘
```

If you start at node 1 and keep following `next`, you go `1, 2, 3, 4, 5, 3, 4, 5, 3, 4, 5, ...` forever. That's a cycle.

A list **without** a cycle ends at `null`:
```
1 → 2 → 3 → null
```

---

## Clarifying Questions to Ask the Interviewer

1. **Modify the list?** No — leave it intact.
2. **Constant memory required?** Yes — Floyd's algorithm uses O(1).
3. **Just detect, or find the cycle's start?** Detect here. See `linked-list/start-of-cycle.md` for the start-finding variant.

---

## Approach

### Naive Approach — Use a HashSet

Walk the list. Add each node's reference to a hashset. If you encounter a node already in the set, there's a cycle.

- Time: O(n)
- Space: **O(n)** — we want better.

### Optimal — Floyd's Tortoise and Hare (O(1) space)

Use two pointers:

- `slow` (tortoise) — moves 1 step at a time.
- `fast` (hare) — moves 2 steps at a time.

If there's no cycle, `fast` eventually hits `null`. If there **is** a cycle, `fast` keeps going around forever, and so does `slow` — but `fast` gains 1 position per step on `slow` while inside the cycle. Eventually they collide.

It's like two runners on a circular track: the faster one always laps the slower one.

#### Walking through a cyclic list

`1 → 2 → 3 → 4 → 5 → (back to 3)`

| step | slow | fast |
|---|---|---|
| 0 | 1 | 1 |
| 1 | 2 | 3 |
| 2 | 3 | 5 |
| 3 | 4 | 4 |
| (collision!) |   |   |

They meet at node 4 → return `true`.

#### Walking through a non-cyclic list

`1 → 2 → 3 → null`

| step | slow | fast |
|---|---|---|
| 0 | 1 | 1 |
| 1 | 2 | 3 |
| 2 | 3 | null (`fast.next.next` after fast=3 hits null) |

Loop exits because `fast == null`. Return `false`.

### Why fast must catch slow if there's a cycle

Once both pointers are inside the cycle, the relative speed of `fast` over `slow` is 1 step per iteration. With cycle length `c`, `fast` closes the gap and meets `slow` within `c` iterations. Mathematically, this is like the runners-on-track argument: the faster one always laps the slower.

---

## Time Complexity

**O(n)** — at most n + cycle length steps.

## Space Complexity

**O(1)** — just two pointers.

---

## Code (Java)

```java
public boolean hasCycle(ListNode head) {
    ListNode slow = head;
    ListNode fast = head;

    while (fast != null && fast.next != null) {
        slow = slow.next;            // 1 step
        fast = fast.next.next;       // 2 steps

        if (slow == fast) {          // they collided — cycle!
            return true;
        }
    }
    return false;                    // fast escaped to null — no cycle
}
```

> Note: we compare references (`slow == fast`), not values. Two different nodes could hold the same `val`; we want to detect identical nodes.

---

## Code (Python)

```python
class Solution:
    def hasCycle(self, head) -> bool:
        slow = fast = head
        while fast and fast.next:
            slow = slow.next
            fast = fast.next.next
            if slow is fast:        # 'is' compares references
                return True
        return False
```

---

## Edge Cases

| Case | Result |
|---|---|
| Empty list | `false` (loop doesn't run) |
| Single node, no self-loop | `false` (`fast.next` is null) |
| Single node with self-loop (`node.next = node`) | `true` (slow and fast both stay at the node and "collide" immediately) |
| Two nodes, second points to first (cycle) | `true` |

---

## Follow-up Questions an Interviewer Might Ask

1. **"Find the *start* of the cycle."**
   See `linked-list/start-of-cycle.md`. After the meeting point, reset one pointer to head and walk both at speed 1; they meet at the cycle's start.

2. **"Find the *length* of the cycle."**
   After the collision, fix one pointer and let the other walk one full lap. Count steps until they meet again.

3. **"Detect cycle without two pointers — using a hashset?"**
   Walk the list and store each node in a hashset; if you see a duplicate, cycle. O(n) space — works but worse than Floyd.

4. **"Why exactly 1 and 2 steps?"**
   Any pair of relatively prime speeds works (e.g., 1 and 3). 1 and 2 is the simplest with the smallest gap, ensuring they meet in the cycle.

---

## Key Lessons from This Problem

1. **Floyd's tortoise and hare** gives O(1) space cycle detection — a beautiful application of two pointers.
2. **Compare references, not values** — `slow == fast` (or `slow is fast` in Python) checks identity.
3. **Two pointers** also detect midpoints, k-from-end positions, and cycle lengths — the same template, different uses.
