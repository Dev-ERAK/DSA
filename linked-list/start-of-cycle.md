# Problem: Find the Start of a Cycle in a Linked List

## Category
Linked List / Two Pointers

## Difficulty
Medium

---

## Problem Statement

Given the head of a linked list that **may** contain a cycle, return the **node where the cycle begins**. Return `null` if there's no cycle.

This is **LeetCode #142**.

### Example

```
List:  1 → 2 → 3 → 4 → 5
                ↑       │
                └───────┘   (5.next points back to 3)
Answer: node 3 (where the cycle starts)
```

---

## Clarifying Questions to Ask the Interviewer

1. **Modify the list?** No.
2. **Use O(1) space?** Yes — Floyd's algorithm extends from cycle detection to cycle-start finding.
3. **Return the node, not the index?** Yes (the indexing isn't well-defined inside a cycle).

---

## Approach — Floyd's Algorithm, Two Phases

### Phase 1 — Detect the cycle

Same as in `detect-cycle.md`. Use slow (1 step) and fast (2 steps) pointers. If they meet → cycle exists. If `fast` hits `null` → no cycle.

### Phase 2 — Find the cycle's start

Once they've met inside the cycle:
1. **Reset one pointer to head** (call it `p1`).
2. **Leave the other at the meeting point** (`p2`).
3. **Move both one step at a time.** They will meet at the **start of the cycle**.

It sounds magical. Let's see why it's true.

### Why Phase 2 works (the math)

Let:
- `L` = distance from head to the cycle's start (the "tail" before the cycle).
- `C` = length of the cycle.
- `K` = distance from the cycle's start to the meeting point (going around the cycle).

When slow enters the cycle, it has walked `L` steps. Fast has walked `2L` steps, so it's `L mod C` ahead inside the cycle.

For them to meet, fast needs to fully catch up to slow inside the cycle. Working through the math, when they meet:
- Slow has walked `L + K` total steps.
- Fast has walked `2(L + K)` total steps.
- The difference must be a multiple of C: `2(L + K) - (L + K) = L + K = nC` for some integer `n`.

So **`L + K` is a multiple of `C`**, i.e., `L = nC - K`.

Now in Phase 2:
- `p1` walks `L` steps from head — arriving at the cycle's start.
- `p2` walks `L = nC - K` steps from the meeting point. Within the cycle, that's `nC - K` steps forward, which lands you exactly at the cycle's start (`-K` steps from the meeting point, modulo cycle length).

So they meet at the cycle's start. ✓

(You don't need to reproduce this proof in an interview — just remember the result.)

### Walking through `1 → 2 → 3 → 4 → 5 → (back to 3)`

```
L = 2  (head=1, cycle starts at 3 — two steps from head)
C = 3  (cycle is 3 → 4 → 5 → 3)
```

#### Phase 1
| step | slow | fast |
|---|---|---|
| 0 | 1 | 1 |
| 1 | 2 | 3 |
| 2 | 3 | 5 |
| 3 | 4 | 4 |
| collision at 4 |   |   |

So `K = 1` (cycle start `3` to meeting point `4` is 1 step around the cycle).

Check the formula: `L + K = 3 = 1 × C = 3`. ✓

#### Phase 2
Reset `p1 = head = 1`. Leave `p2 = 4`.

| step | p1 | p2 |
|---|---|---|
| 0 | 1 | 4 |
| 1 | 2 | 5 |
| 2 | 3 | 3 |

They meet at node 3 — the cycle's start. ✓

---

## Time Complexity

**O(n)** — at most 2n total steps.

## Space Complexity

**O(1)** — just three pointers.

---

## Code (Java)

```java
public ListNode detectCycle(ListNode head) {
    ListNode slow = head;
    ListNode fast = head;

    // Phase 1: detect a cycle.
    while (fast != null && fast.next != null) {
        slow = slow.next;
        fast = fast.next.next;

        if (slow == fast) {
            // Phase 2: find the cycle's start.
            ListNode p = head;
            while (p != slow) {
                p = p.next;
                slow = slow.next;
            }
            return p;
        }
    }
    return null;             // no cycle
}
```

---

## Code (Python)

```python
class Solution:
    def detectCycle(self, head):
        slow = fast = head

        while fast and fast.next:
            slow = slow.next
            fast = fast.next.next

            if slow is fast:
                # Phase 2
                p = head
                while p is not slow:
                    p = p.next
                    slow = slow.next
                return p

        return None
```

---

## Edge Cases

| Case | Result |
|---|---|
| No cycle | `null` (Phase 1's loop exits) |
| Cycle starts at the head | Phase 2 returns head immediately (p1 and p2 collide at step 0) |
| Self-loop (single node pointing to itself) | Returns that node |
| Empty list | `null` |

---

## Follow-up Questions an Interviewer Might Ask

1. **"Detect AND break the cycle."**
   Find the cycle's start. Walk to the node whose `.next` is the start. Set its `.next = null`.

2. **"Find the length of the cycle."**
   After Phase 1's collision, fix one pointer and let the other walk one lap. Count steps.

3. **"Why does this algorithm work without computing L or C explicitly?"**
   The clever bit is that the algebra `L = nC - K` makes the two pointers in Phase 2 always converge at the start, regardless of n.

4. **"Solution using a HashSet — when would you prefer it?"**
   Hashset uses O(n) space but is easier to reason about. Use Floyd's when O(1) space is required or when you want to flex.

---

## Key Lessons from This Problem

1. **Floyd's algorithm is more powerful than just cycle detection** — it also pinpoints the cycle's start.
2. **The math is non-obvious** — but trust the result and remember the two-phase pattern.
3. **Two pointers, three problems** — Floyd's gives you cycle existence, cycle length, and cycle start, all with the same primitive.
