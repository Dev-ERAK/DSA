# Problem: Lowest Common Ancestor in a BST

## Category
Trees / BST

## Difficulty
Easy

---

## Problem Statement

Given a Binary Search Tree (BST) and two of its nodes `p` and `q`, find their **Lowest Common Ancestor (LCA)** — the deepest node that has both `p` and `q` in its subtree (a node may be considered a descendant of itself).

This is **LeetCode #235**.

### Example

```
       6
      / \
     2   8
    / \ / \
   0  4 7  9
     / \
    3   5
```

- LCA of 2 and 8 → **6** (root)
- LCA of 2 and 4 → **2** (a node is its own descendant)
- LCA of 3 and 5 → **4**

---

## Clarifying Questions to Ask the Interviewer

1. **Both p and q exist in the tree?** Yes (LeetCode 235 guarantees this).
2. **Are values unique?** Yes (typical BST).
3. **Return the LCA node, or its value?** The node.

---

## Approach — Use the BST Ordering

In a general binary tree, finding the LCA needs a full search. But a BST gives us a powerful shortcut: at each node, we know whether `p` and `q` go left or right just by comparing values.

### The decision rule

For the current node:

- If both `p.val` and `q.val` are **less than** the node, the LCA is in the **left** subtree.
- If both are **greater than** the node, the LCA is in the **right** subtree.
- Otherwise — meaning one is on the left and the other on the right (or one equals the current node) — the **current node is the LCA**.

Why? The first time `p` and `q` "split" — going to different subtrees — that node must be their common ancestor. And it's the deepest one, because any node above it would also have them split.

### Walking through "LCA of 2 and 4"

Tree (subset):
```
       6
      /
     2
    / \
   0   4
```

- Start at 6. `2 < 6` and `4 < 6` — both less. Go left to 2.
- At 2. `2 < 2`? No (it's equal). `4 > 2`. Not "both less," not "both greater" → split (well, p == current). Return 2.

Correct: LCA of 2 and 4 is 2 (because 2 is itself an ancestor of 4 — and of itself).

### Walking through "LCA of 2 and 8"

- Start at 6. `2 < 6`, `8 > 6`. Split → return 6. ✓

### Walking through "LCA of 3 and 5"

- Start at 6. `3 < 6`, `5 < 6`. Both less → go left to 2.
- At 2. `3 > 2`, `5 > 2`. Both greater → go right to 4.
- At 4. `3 < 4`, `5 > 4`. Split → return 4. ✓

---

## Time Complexity

**O(h)** where `h` is the tree's height. Balanced: O(log n). Skewed: O(n).

## Space Complexity

- Iterative: **O(1)**
- Recursive: **O(h)** stack

---

## Code (Java)

```java
public TreeNode lowestCommonAncestor(TreeNode root, TreeNode p, TreeNode q) {
    while (root != null) {
        if (p.val < root.val && q.val < root.val) {
            root = root.left;            // both in left subtree
        } else if (p.val > root.val && q.val > root.val) {
            root = root.right;           // both in right subtree
        } else {
            return root;                 // split, or current node IS p or q
        }
    }
    return null;
}
```

---

## Code (Python)

```python
class Solution:
    def lowestCommonAncestor(self, root, p, q):
        while root:
            if p.val < root.val and q.val < root.val:
                root = root.left
            elif p.val > root.val and q.val > root.val:
                root = root.right
            else:
                return root
        return None
```

---

## Edge Cases

| Case | Result |
|---|---|
| One of `p`, `q` is the root | Root is the LCA. |
| `p` is an ancestor of `q` | `p` itself. |
| `p == q` | `p` (a node is its own descendant). |
| Tree only has p and q | The deeper node's parent (or one of them, if they're parent-child). |

---

## Follow-up Questions an Interviewer Might Ask

1. **"What if it's a general binary tree, not a BST?"**
   See `lca-binary-tree.md`. You can't use ordering — you have to search both subtrees.

2. **"What if nodes have parent pointers?"**
   Walk up from each node to root, collecting the path. Compare paths from the top until they diverge — the last common node is the LCA. Or: equalize depths and walk up in lockstep.

3. **"Handle the case where one of p/q might not be in the tree."**
   Modify to count finds; only return the LCA if both were found. Otherwise return null.

4. **"Find the LCA of multiple nodes."**
   For BST: use bounds — `lo = min(values)`, `hi = max(values)`. Walk down: take left if all < node, right if all > node, else this is the LCA.

---

## Key Lessons from This Problem

1. **The BST ordering is gold** — use it whenever you can.
2. **"Split point" is the LCA** — recognize this pattern.
3. **Iterative vs recursive** — iterative wins on space here; both run in O(h) time.
