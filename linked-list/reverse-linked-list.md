# Problem: Reverse a Linked List

## Category
Linked List

## Difficulty
Easy

---

## Problem Statement

Given the head of a singly linked list, reverse it and return the new head.

This is **LeetCode #206** — the most common linked-list interview question. Master it and many others get easier.

### Example

```
Input:  1 → 2 → 3 → 4 → 5 → null
Output: 5 → 4 → 3 → 2 → 1 → null
```

### What's a singly linked list?

A linked list is a chain of **nodes**, each holding a value and a pointer to the next node:

```
class ListNode {
    int val;
    ListNode next;
}
```

The first node is called the **head**. The last node's `next` is `null`. There's no random access — to find the 5th element, you walk from the head four times.

---

## Clarifying Questions to Ask the Interviewer

1. **Singly or doubly linked?** This problem assumes singly linked. (Doubly linked would also let us go backward via `prev`.)
2. **Reverse in place, or return a new list?** In place — that's the standard contract.
3. **Iterative or recursive?** Either is fine; both are good to know.

---

## Approach

### The Big Idea (Iterative) — Walk the list and flip each pointer

For each node, change its `next` pointer to point to the **previous** node instead of the **next** one. We need three pointers to do this without losing track:

- `prev` — the node we just processed (will become the new "next" of the current).
- `cur` — the node we're working on.
- `next` — saved before we overwrite `cur.next`, so we don't lose the rest of the list.

#### The 4-step dance

For each node:
1. Save `next = cur.next` (so we don't lose the rest of the list).
2. Reverse the link: `cur.next = prev`.
3. Move forward: `prev = cur`.
4. Move forward: `cur = next`.

Repeat until `cur` is `null`. Then `prev` is the new head.

### Walking through `1 → 2 → 3 → null`

Start: `prev = null`, `cur = 1`.

```
Step 1: cur = 1, prev = null, next = 2
        Flip: 1.next = null
        After:  null ← 1   2 → 3 → null
        prev = 1, cur = 2

Step 2: cur = 2, prev = 1, next = 3
        Flip: 2.next = 1
        After:  null ← 1 ← 2   3 → null
        prev = 2, cur = 3

Step 3: cur = 3, prev = 2, next = null
        Flip: 3.next = 2
        After:  null ← 1 ← 2 ← 3
        prev = 3, cur = null

Loop ends. New head = prev = 3.
```

Result: `3 → 2 → 1 → null` ✓

### Why we save `next` first

If we flipped `cur.next = prev` first, we'd lose access to the rest of the list (the part we haven't reversed yet). Saving `next` is non-negotiable.

### Recursive Approach

The recursive version is elegant but uses O(n) stack space:

```
reverse(head):
    if head == null or head.next == null: return head
    newHead = reverse(head.next)
    head.next.next = head           # the next node points back to me
    head.next = null                # I now point to nothing (will be overridden by my caller, except for the original head)
    return newHead
```

The trick: we recurse all the way to the last node (the new head), then on the way back up, each node makes the next one point to itself.

---

## Time Complexity

**O(n)** — each node is visited once.

## Space Complexity

- Iterative: **O(1)** — just three pointers.
- Recursive: **O(n)** — one stack frame per node.

---

## Code (Java)

```java
class ListNode {
    int val;
    ListNode next;
    ListNode(int v) { this.val = v; }
}

// Iterative — preferred
public ListNode reverseList(ListNode head) {
    ListNode prev = null;
    ListNode cur = head;

    while (cur != null) {
        ListNode next = cur.next;     // 1. save the rest of the list
        cur.next = prev;              // 2. flip the pointer
        prev = cur;                   // 3. advance prev
        cur = next;                   // 4. advance cur
    }
    return prev;                       // prev is now the new head
}

// Recursive
public ListNode reverseListRec(ListNode head) {
    if (head == null || head.next == null) return head;
    ListNode newHead = reverseListRec(head.next);
    head.next.next = head;             // the next node should point back to me
    head.next = null;                  // I now point to nothing
    return newHead;
}
```

---

## Code (Python)

```python
class ListNode:
    def __init__(self, val=0, nxt=None):
        self.val = val
        self.next = nxt

class Solution:
    # Iterative
    def reverseList(self, head):
        prev, cur = None, head
        while cur:
            nxt = cur.next             # save the rest
            cur.next = prev            # flip
            prev = cur                 # advance
            cur = nxt
        return prev

    # Recursive
    def reverseListRec(self, head):
        if head is None or head.next is None:
            return head
        new_head = self.reverseListRec(head.next)
        head.next.next = head
        head.next = None
        return new_head
```

---

## Edge Cases

| Case | Result |
|---|---|
| Empty list (`head = null`) | Loop doesn't run; return `null`. |
| Single node | Loop runs once: `cur.next = null` (already was), `prev = cur`, `cur = null`. Returns the same node. |
| Two nodes | Standard case. |

---

## Follow-up Questions an Interviewer Might Ask

1. **"Reverse only between positions m and n (LeetCode 92)."**
   Walk to position `m - 1`. Then perform the local reversal on the next `n - m + 1` nodes, while carefully reattaching the reversed segment to the surrounding list.

2. **"Reverse in groups of k (LeetCode 25)."**
   Reverse each block of k nodes; if fewer than k remain at the end, leave them as-is (or reverse, depending on the variant).

3. **"What if it were a doubly linked list?"**
   Just swap each node's `prev` and `next` pointers in one pass. Even simpler.

4. **"Why not use a stack to reverse?"**
   You could push every node onto a stack, then pop them to rebuild the list. That works but uses O(n) extra space. The pointer-flipping approach is O(1).

---

## Key Lessons from This Problem

1. **Three-pointer technique** (`prev`, `cur`, `next`) — comes up in many linked-list problems.
2. **Save before you overwrite.** When mutating pointers, always grab what you need before changing it.
3. **Recursion-on-linked-lists** has a "build on the way back up" pattern that's worth understanding.
