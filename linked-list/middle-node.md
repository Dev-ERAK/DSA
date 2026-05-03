# Problem: Middle of a Linked List

## Category
Linked List / Two Pointers

## Difficulty
Easy

---

## Problem Statement

Given the head of a singly linked list, return the **middle node**.

If the list has an even number of nodes, return the **second** middle node.

This is **LeetCode #876**.

### Examples

```
1 → 2 → 3 → 4 → 5            →  middle is 3
1 → 2 → 3 → 4 → 5 → 6        →  middle is 4 (second of two middles)
```

---

## Clarifying Questions to Ask the Interviewer

1. **For an even-length list, do you want the first or second middle?**
   - LeetCode says **second** middle (e.g., 4 in `[1,2,3,4,5,6]`).
   - The other convention picks the first middle (3).
2. **Empty list?** Return `null` / `None`.
3. **Can we do it in one pass?** Yes — that's the whole point of the slow/fast trick.

---

## Approach

### Naive Two-Pass Approach

1. **Pass 1:** count the number of nodes, `n`.
2. **Pass 2:** walk `n / 2` steps to reach the middle.

This works but takes two passes.

### Optimal — Slow and Fast Pointers (One Pass)

Use two pointers that move at different speeds:

- `slow` advances by **1** node per step.
- `fast` advances by **2** nodes per step.

When `fast` reaches the end (or runs out of nodes), `slow` is at the middle.

#### Why does this work?

`fast` moves twice as fast as `slow`. So when `fast` has traveled `n` steps, `slow` has traveled `n / 2` steps. By the time `fast` reaches the end, `slow` is exactly at the midpoint.

#### Walking through `1 → 2 → 3 → 4 → 5`

```
Initial:  slow=1, fast=1
Step 1:   slow=2, fast=3
Step 2:   slow=3, fast=5
fast.next is null → stop. Return slow = 3. ✓
```

#### Walking through `1 → 2 → 3 → 4 → 5 → 6`

```
Initial:  slow=1, fast=1
Step 1:   slow=2, fast=3
Step 2:   slow=3, fast=5
Step 3:   slow=4, fast=null  (fast went off the end)
Loop exits. Return slow = 4 (the second middle). ✓
```

#### The loop condition

```
while (fast != null && fast.next != null) {
    slow = slow.next;
    fast = fast.next.next;
}
```

We need `fast.next` to be non-null *before* we read `fast.next.next`. If we wrote just `while (fast != null)`, we'd crash on `fast.next.next` for an even-length list.

---

## Time Complexity

**O(n)** — `slow` walks half the list.

## Space Complexity

**O(1)** — just two pointers.

---

## Code (Java)

```java
public ListNode middleNode(ListNode head) {
    ListNode slow = head;
    ListNode fast = head;

    // Both conditions matter:
    // - fast != null  : we haven't fallen off the list
    // - fast.next != null : we have at least 2 more steps for fast
    while (fast != null && fast.next != null) {
        slow = slow.next;          // 1 step
        fast = fast.next.next;     // 2 steps
    }

    return slow;
}
```

---

## Code (Python)

```python
class Solution:
    def middleNode(self, head):
        slow = fast = head
        while fast and fast.next:
            slow = slow.next
            fast = fast.next.next
        return slow
```

---

## Edge Cases

| Case | Result |
|---|---|
| Empty list | `slow = fast = null`. Loop doesn't run. Return `null`. |
| Single node | Loop doesn't run (`fast.next` is null). Return the only node. |
| Two nodes | Loop runs once: `slow` moves to second node, `fast` moves to null. Return the second node. ✓ (matches "second middle" convention) |
| Three nodes | Loop runs once: `slow` ends at node 2, `fast` ends at node 3. Return node 2. |

---

## Follow-up Questions an Interviewer Might Ask

1. **"Return the *first* middle for even-length lists, not the second."**
   Stop one step earlier:
   ```java
   while (fast.next != null && fast.next.next != null) {
       slow = slow.next;
       fast = fast.next.next;
   }
   ```
   Now for `[1,2,3,4]`, `slow` ends at 2 (the first middle).

2. **"Use this to detect a cycle in the list."**
   The same slow/fast technique — if there's a cycle, the two pointers will meet inside it. See `linked-list/detect-cycle.md`.

3. **"Split the list into two halves at the middle."**
   Find the middle, then set `middle.prev.next = null` (you'd need to track the previous node) to break the chain.

4. **"How does this generalize to other 'k-th from end' problems?"**
   The two-pointer technique generalizes: to find the k-th from end, advance `fast` by k first, then move both pointers together until `fast` hits null. `slow` is now at the k-th from end.

---

## Key Lessons from This Problem

1. **Slow and fast pointers** give a one-pass O(1)-space algorithm whenever you need to relate "halfway" or "from the end" positions.
2. **Watch the loop condition** — `fast != null && fast.next != null` is the right guard against null pointer crashes for fast advancing by 2.
3. **Even vs odd length** affects where `slow` ends up. Be deliberate about the convention you want.
