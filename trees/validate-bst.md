# Problem: Validate a Binary Search Tree

## Category
Trees / BST

## Difficulty
Medium

---

## Problem Statement

Given the root of a binary tree, determine whether it's a **valid Binary Search Tree (BST)**.

A valid BST satisfies:
- The **left subtree** of every node contains only values **strictly less** than the node.
- The **right subtree** of every node contains only values **strictly greater** than the node.
- Both subtrees are themselves valid BSTs.

This is **LeetCode #98**.

### Example

Valid BST:
```
       5
      / \
     1   8
        / \
       7   9
```

Invalid (looks BST-ish, but the 4 below 7 violates the global rule):
```
       5
      / \
     1   8
        / \
       4   9      ← 4 < 5, but it's in the RIGHT subtree of 5!
```

---

## Clarifying Questions to Ask the Interviewer

1. **Strict comparisons?** Yes — duplicates are not allowed (LeetCode 98).
2. **Empty tree?** Counts as valid (vacuously).
3. **Worry about integer overflow at INT_MIN/INT_MAX?** Yes — use `long` for the bounds, not `int`.

---

## Approach

### Common Mistake — Local Comparison Only

A naïve attempt:

```
isValid(node):
    if node is null: return true
    if node.left  != null && node.left.val  >= node.val: return false
    if node.right != null && node.right.val <= node.val: return false
    return isValid(node.left) && isValid(node.right)
```

This **only checks immediate children**. It misses deeper violations (like the `4` example above — `4 < 5` but `4` is way down in the right subtree).

### Method 1 — Bounds Recursion (the right way)

Pass `(min, max)` bounds with each recursive call: every value in this subtree must lie strictly within `(min, max)`.

```
valid(node, lo, hi):
    if node is null: return true
    if node.val <= lo or node.val >= hi: return false

    return valid(node.left,  lo, node.val)        # left subtree: < node.val
        and valid(node.right, node.val, hi)       # right subtree: > node.val
```

Initial call: `valid(root, -∞, +∞)`.

#### Walking through

```
       5
      / \
     1   8
        / \
       4   9      (intentional violation)
```

- `valid(5, -∞, +∞)`: 5 in (−∞, ∞) ✓
- `valid(1, -∞, 5)`: 1 in (−∞, 5) ✓. Children null → return true.
- `valid(8, 5, +∞)`: 8 in (5, ∞) ✓
  - `valid(4, 5, 8)`: 4 ≤ 5 → **return false** ✓
- The whole tree is invalid because of node 4.

The bounds **propagate down** the tree, narrowing as we go. A node deep in a right-of-right-of-left subtree must lie within the cumulative bounds set by all its ancestors.

### Method 2 — Inorder Traversal

A BST's **inorder traversal** (left → root → right) yields values in **strictly increasing order**. So:

1. Do an inorder traversal.
2. Each value must be strictly greater than the previous.

If you encounter `cur.val ≤ prev.val`, return false. Otherwise return true after the traversal.

This is also O(n) but a different mental model.

---

## Time Complexity

**O(n)** — visit each node once.

## Space Complexity

**O(h)** — recursion stack height.

---

## Code (Java)

```java
public boolean isValidBST(TreeNode root) {
    return valid(root, Long.MIN_VALUE, Long.MAX_VALUE);
}

// Use long for bounds to handle nodes with value Integer.MIN_VALUE / MAX_VALUE.
private boolean valid(TreeNode node, long lo, long hi) {
    if (node == null) return true;
    if (node.val <= lo || node.val >= hi) return false;
    return valid(node.left,  lo, node.val)
        && valid(node.right, node.val, hi);
}

// Inorder version
public boolean isValidBSTInorder(TreeNode root) {
    Deque<TreeNode> stack = new ArrayDeque<>();
    TreeNode cur = root;
    Long prev = null;

    while (cur != null || !stack.isEmpty()) {
        // Walk left as far as possible, pushing along the way.
        while (cur != null) {
            stack.push(cur);
            cur = cur.left;
        }

        cur = stack.pop();
        if (prev != null && cur.val <= prev) return false;
        prev = (long) cur.val;

        cur = cur.right;
    }
    return true;
}
```

---

## Code (Python)

```python
class Solution:
    def isValidBST(self, root) -> bool:
        def valid(node, lo, hi):
            if node is None:
                return True
            if node.val <= lo or node.val >= hi:
                return False
            return valid(node.left, lo, node.val) and valid(node.right, node.val, hi)

        return valid(root, float('-inf'), float('inf'))

    def isValidBST_inorder(self, root) -> bool:
        stack = []
        cur = root
        prev = None
        while cur or stack:
            while cur:
                stack.append(cur)
                cur = cur.left
            cur = stack.pop()
            if prev is not None and cur.val <= prev:
                return False
            prev = cur.val
            cur = cur.right
        return True
```

---

## Edge Cases

| Case | Result |
|---|---|
| Empty tree | `true` |
| Single node | `true` (always valid) |
| Duplicates | `false` (strict BST rule) |
| Node with value `Integer.MIN_VALUE` | Use `long` bounds, not `int`, to avoid edge-case false negatives |
| All values in increasing left-skew | Valid BST if structure is correct |

---

## Follow-up Questions an Interviewer Might Ask

1. **"Why doesn't the local-comparison approach work?"**
   Because the BST property is global, not just about immediate children. A node deep in the tree can violate the property even if every parent-child relationship is locally OK.

2. **"Recover a BST where exactly two nodes are swapped (LeetCode 99)."**
   Inorder traversal will identify the two violation points. Swap the two nodes' values back.

3. **"Implement a BST iterator (LeetCode 173) — `next()` returns values in sorted order, in O(1) average."**
   Use the same controlled inorder traversal: maintain a stack of "left ancestors yet to visit."

4. **"Allow duplicates on one side?"**
   Switch the comparisons to `<=` on one direction. Be explicit and consistent.

---

## Key Lessons from This Problem

1. **Local checks aren't enough** for tree properties. Always think globally.
2. **Bounds propagation** is a powerful pattern: pass min/max constraints down the recursion.
3. **Inorder of a BST is sorted** — a property that solves many BST problems elegantly.
