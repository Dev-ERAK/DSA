# Problem: Lowest Common Ancestor in a Binary Tree

## Category
Trees

## Difficulty
Medium

---

## Problem Statement

Given the root of a binary tree (NOT a BST — no ordering rule) and two nodes `p` and `q`, find their **Lowest Common Ancestor (LCA)** — the deepest node that has both `p` and `q` as descendants (a node is its own descendant).

This is **LeetCode #236**.

### Example

```
       3
      / \
     5   1
    / \ / \
   6  2 0  8
      / \
     7   4
```

- LCA of 5 and 1 → **3**
- LCA of 5 and 4 → **5** (5 is the ancestor of 4)
- LCA of 6 and 4 → **5**

---

## Clarifying Questions to Ask the Interviewer

1. **Are p and q guaranteed to be in the tree?** Yes (LeetCode 236).
2. **Compare by reference, not value?** By reference (`p == node`), since values can repeat in a non-BST.
3. **Return the node, or its value?** The node.

---

## Approach — Recursion (Postorder DFS)

Without the BST ordering, we have to search the tree. The trick: **return useful information from each subtree** so the recursion can identify the LCA on the way back up.

### The recursion

```
lca(node, p, q):
    if node is null: return null
    if node == p or node == q: return node     # found one of the targets

    left  = lca(node.left,  p, q)
    right = lca(node.right, p, q)

    if left != null and right != null:
        return node           # one target is in the left subtree, the other in the right — node is the LCA
    return left != null ? left : right    # otherwise, propagate whichever side found something
```

### Reading the recursion's return value

`lca(...)` returns:
- `null` if neither `p` nor `q` is in this subtree.
- A pointer to `p` or `q` if exactly one of them is found.
- The actual LCA node if both are found in this subtree.

### Why this works

There are three cases at any node:

- **Both targets are in the same subtree.** The recursive call into that subtree returns the LCA; the other call returns null. We pass the non-null up.
- **Targets are in opposite subtrees.** Both recursive calls return non-null (each returns *some* node found below). The current node is the deepest ancestor with one in each subtree — it's the LCA.
- **One target is the current node.** We return the current node (which is one of the targets). On the way up, the parent will see this and (depending on what the other side returned) either pass it through or recognize the current as the LCA.

### Walking through "LCA of 5 and 4"

```
       3
      / \
     5   1
    / \ / \
   6  2 0  8
      / \
     7   4
```

```
lca(3, 5, 4):
  lca(5, 5, 4):
    return 5         (matched p=5, short-circuit)
  lca(1, 5, 4):
    lca(0, 5, 4):  → null
    lca(8, 5, 4):  → null
    return null
  Now: left = 5, right = null. Only left found something.
  Return 5.
```

So LCA = 5. ✓ (Note that the algorithm doesn't realize 4 is *underneath* 5 — but that's fine. Once it finds 5, it's enough; the convention is that a node is an ancestor of itself.)

### Walking through "LCA of 6 and 4"

```
lca(3, 6, 4):
  lca(5, 6, 4):
    lca(6, 6, 4):  return 6 (match p)
    lca(2, 6, 4):
      lca(7, ...): null
      lca(4, ...): return 4 (match q)
      → left null, right 4 → return 4
    → left = 6, right = 4. Both non-null!
    Return 5.
  lca(1, 6, 4):  null
  → left = 5, right = null. Return 5.
```

LCA = 5. ✓

---

## Time Complexity

**O(n)** — visit each node once.

## Space Complexity

**O(h)** — recursion stack height.

---

## Code (Java)

```java
public TreeNode lowestCommonAncestor(TreeNode root, TreeNode p, TreeNode q) {
    if (root == null || root == p || root == q) {
        return root;
    }

    TreeNode left  = lowestCommonAncestor(root.left,  p, q);
    TreeNode right = lowestCommonAncestor(root.right, p, q);

    if (left != null && right != null) {
        return root;       // one target on each side — root is the LCA
    }
    return (left != null) ? left : right;
}
```

---

## Code (Python)

```python
class Solution:
    def lowestCommonAncestor(self, root, p, q):
        if root is None or root is p or root is q:
            return root

        left  = self.lowestCommonAncestor(root.left,  p, q)
        right = self.lowestCommonAncestor(root.right, p, q)

        if left and right:
            return root
        return left if left else right
```

---

## Edge Cases

| Case | Result |
|---|---|
| `p` is the root | Root is LCA. |
| One is an ancestor of the other | The ancestor. |
| `p == q` | `p`. |
| Both are leaves of different subtrees | Their nearest common ancestor — the recursion finds it. |

---

## Follow-up Questions an Interviewer Might Ask

1. **"BST variant — can we use ordering?"**
   Yes — see `lca-bst.md`. Use the BST property to navigate without searching the whole tree.

2. **"What if there are parent pointers?"**
   Walk up from each node. Equalize depths first, then walk up together until they meet.

3. **"Many queries on the same tree?"**
   Preprocess with **Tarjan's offline LCA** (Union-Find based) — O((n + q) α(n)) for n nodes and q queries. For online queries: Euler tour + sparse-table RMQ → O(1) per query after O(n log n) preprocessing.

4. **"What if one of `p`, `q` might not be in the tree?"**
   Modify to count discoveries; only return the LCA if both were found. Otherwise return null.

---

## Key Lessons from This Problem

1. **Use the recursion's return value to encode information** — not just a yes/no, but "what was found below."
2. **The "one on each side" check** is the heart of the LCA algorithm.
3. **A node is its own descendant** — this convention simplifies the algorithm.
