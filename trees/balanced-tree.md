# Problem: Is the Binary Tree Height-Balanced?

## Category
Trees / Recursion

## Difficulty
Easy

---

## Problem Statement

A binary tree is **height-balanced** if, for **every** node, the heights of its left and right subtrees differ by at most 1.

Given the root of a binary tree, return `true` if it's balanced.

This is **LeetCode #110**.

### Examples

Balanced (every node's subtrees differ by at most 1):
```
       3
      / \
     9  20
        / \
      15   7
```

Not balanced (the leftmost path is way deeper than the rightmost):
```
        1
       /
      2
     /
    3
   /
  4
```

---

## Clarifying Questions to Ask the Interviewer

1. **Empty tree?** Counts as balanced.
2. **What's the exact definition?** "Heights of left and right subtrees differ by at most 1, **for every node** in the tree."
3. **Does this mean AVL-balanced?** Yes — same rule.

---

## Approach

### Naive Approach — O(n²)

For every node, compute the heights of its two subtrees and check the difference. Computing heights is O(n), so doing it at every node is O(n × n) = O(n²). Wasteful — we recompute heights of subtrees over and over.

### Optimal — Single DFS with a "Height + Imbalance Sentinel" — O(n)

Compute the height of each subtree exactly once. While computing, **return a sentinel value (e.g., -1) if any imbalance is detected**.

```
height(node):
    if node is null: return 0

    leftH = height(node.left)
    if leftH == -1: return -1                # imbalance found in left subtree

    rightH = height(node.right)
    if rightH == -1: return -1               # imbalance found in right subtree

    if |leftH - rightH| > 1: return -1       # this node is unbalanced
    return 1 + max(leftH, rightH)            # otherwise, return height
```

Then `isBalanced` is just `return height(root) != -1`.

### Why this is O(n)

We touch each node exactly once. We "short-circuit" via the -1 sentinel, never revisiting a subtree.

### Walking through

```
       3
      / \
     9  20
        / \
      15   7
```

- `height(9) = 1`
- `height(15) = 1`, `height(7) = 1`
- `height(20) = 1 + max(1, 1) = 2`. Imbalance? |1 - 1| = 0 ≤ 1 ✓
- `height(3)`: leftH = 1, rightH = 2. Imbalance? |1 - 2| = 1 ≤ 1 ✓. Return 3.
- Final: `height(root) = 3 ≠ -1` → return `true`.

For the unbalanced example (`1 → 2 → 3 → 4`):
- `height(4) = 1`
- `height(3) = 1 + max(1, 0) = 2`. Imbalance? |1 - 0| = 1 ≤ 1 ✓
- `height(2) = 1 + max(2, 0) = 3`. Imbalance? |2 - 0| = 2 > 1 — **return -1**.
- `height(1)` sees leftH = -1 → return -1.
- Final: -1 → return `false`. ✓

---

## Time Complexity

**O(n)** — single DFS, O(1) per node.

## Space Complexity

**O(h)** — recursion stack, where h is the height of the tree.

---

## Code (Java)

```java
public boolean isBalanced(TreeNode root) {
    return check(root) != -1;
}

// Returns the height of the subtree rooted at node, or -1 if it's unbalanced.
private int check(TreeNode node) {
    if (node == null) return 0;

    int lh = check(node.left);
    if (lh == -1) return -1;        // already unbalanced — short-circuit

    int rh = check(node.right);
    if (rh == -1) return -1;

    if (Math.abs(lh - rh) > 1) return -1;
    return 1 + Math.max(lh, rh);
}
```

---

## Code (Python)

```python
class Solution:
    def isBalanced(self, root) -> bool:
        def check(node):
            if node is None:
                return 0

            lh = check(node.left)
            if lh == -1:
                return -1

            rh = check(node.right)
            if rh == -1:
                return -1

            if abs(lh - rh) > 1:
                return -1
            return 1 + max(lh, rh)

        return check(root) != -1
```

---

## Edge Cases

| Case | Balanced? |
|---|---|
| Empty tree | true |
| Single node | true |
| Two nodes (root + 1 child) | true (heights 1 vs 0) |
| Three-node "L-shape" (root → left → left.left) | false (heights 2 vs 0) |
| Perfect binary tree | true |

---

## Follow-up Questions an Interviewer Might Ask

1. **"Convert an unbalanced BST to a balanced one."**
   Inorder traversal gives a sorted array; rebuild a balanced BST from it (recursively pick the middle as the root). LeetCode 1382.

2. **"What's a self-balancing data structure?"**
   AVL trees and Red-Black trees automatically rebalance on insert/delete. Each operation is O(log n).

3. **"Why DFS, not BFS?"**
   Heights are bottom-up: a node's height depends on its children. DFS naturally goes to leaves first and computes heights on the way back up.

4. **"Is the strictness of '≤ 1' standard?"**
   AVL: yes. There are weaker definitions (e.g., red-black trees ensure a height bound of `2 log n`).

---

## Key Lessons from This Problem

1. **Combining "compute" and "validate" in one pass** is a powerful pattern for tree problems.
2. **Sentinel values** (like -1 here) let you short-circuit recursion without throwing exceptions.
3. **Bottom-up DFS** for tree problems where the answer depends on subtrees' answers.
