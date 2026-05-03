# Problem: Height (Maximum Depth) of a Binary Tree

## Category
Trees / Recursion

## Difficulty
Easy

---

## Problem Statement

Given the root of a binary tree, return its **maximum depth** — the number of nodes along the longest path from the root down to a leaf.

By convention, an **empty tree has depth 0**.

This is **LeetCode #104**.

### What's a binary tree?

A binary tree is a structure where each node has at most two children: `left` and `right`.

```
        1
       / \
      2   3
     / \   \
    4   5   6
```

Heights:
- Tree above: 3 (1 → 2 → 4 or 1 → 2 → 5 are length-3 paths).

### Example

```
       3
      / \
     9  20
        / \
      15   7

Maximum depth = 3   (path: 3 → 20 → 15 or 3 → 20 → 7)
```

---

## Clarifying Questions to Ask the Interviewer

1. **Is depth measured in nodes or edges?** Nodes (LeetCode convention). If you want edges, subtract 1.
2. **Empty tree?** Depth 0.
3. **Leaf node depth?** 1.

---

## Approach

### Method 1 — Recursion (DFS, the natural fit)

The depth of a tree rooted at `node` is:

```
0                                       if node is null
1 + max(depth(left), depth(right))     otherwise
```

In words: a tree's depth is 1 (for the current node) plus the deeper of its two subtrees.

This is **the recursion that mirrors the tree's structure** — almost magical in its simplicity.

#### Walking through

```
       3
      / \
     9  20
        / \
      15   7
```

- `depth(3) = 1 + max(depth(9), depth(20))`
- `depth(9) = 1 + max(depth(null), depth(null)) = 1 + max(0, 0) = 1`
- `depth(20) = 1 + max(depth(15), depth(7)) = 1 + max(1, 1) = 2`
- So `depth(3) = 1 + max(1, 2) = 3` ✓

### Method 2 — Iterative BFS (Level-Order Traversal)

Walk the tree **level by level** with a queue. Each iteration of the outer loop processes one full level. Count the levels.

This avoids deep recursion (helpful for very tall trees) and is the natural choice when you need level-related info.

---

## Time Complexity

**O(n)** — every node is visited once.

## Space Complexity

- **DFS:** O(h) for the recursion stack (h = tree height). Worst case (skewed tree) → O(n).
- **BFS:** O(w) for the queue (w = max width of any level).

---

## Code (Java)

```java
class TreeNode {
    int val;
    TreeNode left, right;
    TreeNode(int v) { val = v; }
}

// Recursive — the elegant version
public int maxDepth(TreeNode root) {
    if (root == null) return 0;
    return 1 + Math.max(maxDepth(root.left), maxDepth(root.right));
}

// Iterative BFS
public int maxDepthBFS(TreeNode root) {
    if (root == null) return 0;
    Deque<TreeNode> q = new ArrayDeque<>();
    q.offer(root);
    int depth = 0;

    while (!q.isEmpty()) {
        depth++;                            // entering a new level
        int sz = q.size();
        for (int i = 0; i < sz; i++) {       // process exactly this level
            TreeNode n = q.poll();
            if (n.left  != null) q.offer(n.left);
            if (n.right != null) q.offer(n.right);
        }
    }
    return depth;
}
```

---

## Code (Python)

```python
class TreeNode:
    def __init__(self, val=0, left=None, right=None):
        self.val = val
        self.left = left
        self.right = right

class Solution:
    def maxDepth(self, root) -> int:
        if root is None:
            return 0
        return 1 + max(self.maxDepth(root.left), self.maxDepth(root.right))

    def maxDepth_bfs(self, root) -> int:
        if root is None:
            return 0
        from collections import deque
        q = deque([root])
        depth = 0
        while q:
            depth += 1
            for _ in range(len(q)):       # process current level
                n = q.popleft()
                if n.left:  q.append(n.left)
                if n.right: q.append(n.right)
        return depth
```

---

## Edge Cases

| Case | Result |
|---|---|
| Empty tree | 0 |
| Single node | 1 |
| Skewed tree (linked-list shape) | Equal to number of nodes; recursion can crash on huge trees → use BFS |
| Wide tree | BFS uses O(w) memory; pick DFS if w >> h |

---

## Follow-up Questions an Interviewer Might Ask

1. **"Find the diameter of the tree (LeetCode 543)."**
   The longest path between any two nodes (which may not pass through the root). While computing height, also track `max(diameter, leftHeight + rightHeight)`.

2. **"Find the *minimum* depth (LeetCode 111)."**
   The depth of the closest leaf. BFS is more efficient here — return as soon as you encounter a leaf. With DFS, you'd need to take the min only over actual leaves.

3. **"Iterative DFS without recursion."**
   Use a stack of `(node, currentDepth)` pairs. Track max depth as you go.

4. **"Height-balanced check?"**
   See `balanced-tree.md`. We can check balance and compute height in one pass.

---

## Key Lessons from This Problem

1. **Trees love recursion.** Almost every tree problem becomes simple when expressed in terms of the recursive structure.
2. **DFS uses the call stack; BFS uses an explicit queue.** Both reach every node — pick based on whether you need level info.
3. **Skewed trees** are a real concern — they turn O(log n) recursion into O(n) — switch to BFS or iterative DFS in production code.
