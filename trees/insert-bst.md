# Problem: Insert into a Binary Search Tree

## Category
Trees / BST

## Difficulty
Medium

---

## Problem Statement

Given the root of a **Binary Search Tree (BST)** and a value `val`, insert `val` into the tree and return the (possibly updated) root. Maintain the BST property.

This is **LeetCode #701**.

### What's a BST?

A binary search tree has this rule for **every** node:

- All values in the **left** subtree are **less than** the node.
- All values in the **right** subtree are **greater than** the node.

```
       4
      / \
     2   7
    / \
   1   3
```

This is a BST: 1 < 2 < 3 < 4 < 7. The key property is that an **inorder traversal** (left → root → right) yields a sorted sequence.

### Example

```
Insert 5 into:
       4
      / \
     2   7
    / \
   1   3

Result:
       4
      / \
     2   7
    / \  /
   1  3 5
```

---

## Clarifying Questions to Ask the Interviewer

1. **Are duplicates allowed?** Often disallowed in BST insert problems. If they are, choose a side (e.g., always go right) and document.
2. **Self-balancing?** No — plain BST insertion. The result might be unbalanced.
3. **Empty tree?** Return a single new node.

---

## Approach

### The Big Idea — Walk Down to a Leaf

The new value always becomes a **leaf** in plain BST insertion. Walk down the tree:

- If `val < node.val`, go left.
- If `val > node.val`, go right.
- When you find a `null` child, attach the new node there.

### Recursive version

```
insert(node, val):
    if node is null: return new Node(val)
    if val < node.val: node.left = insert(node.left, val)
    else:              node.right = insert(node.right, val)
    return node
```

The recursion **returns the (possibly new) subtree** to the parent, who reassigns its child pointer. This handles both "the subtree changed" and "the subtree was null" cases uniformly.

### Walking through "insert 5"

```
insert(4, 5):
  5 > 4 → go right
  insert(7, 5):
    5 < 7 → go left
    insert(null, 5):
      return new Node(5)
    7.left = Node(5)
    return 7
  4.right = 7
  return 4
```

Tree after:
```
       4
      / \
     2   7
    / \  /
   1  3 5
```

### Iterative version

For O(1) extra space:

```
node = new Node(val)
if root is null: return node
cur = root
while True:
    if val < cur.val:
        if cur.left is null: cur.left = node; return root
        cur = cur.left
    else:
        if cur.right is null: cur.right = node; return root
        cur = cur.right
```

### Time complexity considerations

- **Balanced BST:** depth ≈ log n → insertion is **O(log n)**.
- **Skewed BST** (e.g., inserting sorted values 1, 2, 3, ...): depth = n → insertion is **O(n)**.

For guaranteed O(log n), you'd use a self-balancing tree (AVL, Red-Black) — but that's a separate problem.

---

## Time Complexity

- Average / balanced: **O(log n)**
- Worst case (skewed): **O(n)**

## Space Complexity

- Recursive: **O(h)** stack
- Iterative: **O(1)**

---

## Code (Java)

```java
class TreeNode {
    int val;
    TreeNode left, right;
    TreeNode(int v) { val = v; }
}

// Recursive — concise
public TreeNode insertIntoBST(TreeNode root, int val) {
    if (root == null) return new TreeNode(val);

    if (val < root.val) {
        root.left = insertIntoBST(root.left, val);
    } else {
        root.right = insertIntoBST(root.right, val);
    }
    return root;
}

// Iterative — O(1) extra space
public TreeNode insertIterative(TreeNode root, int val) {
    TreeNode node = new TreeNode(val);
    if (root == null) return node;

    TreeNode cur = root;
    while (true) {
        if (val < cur.val) {
            if (cur.left == null) {
                cur.left = node;
                return root;
            }
            cur = cur.left;
        } else {
            if (cur.right == null) {
                cur.right = node;
                return root;
            }
            cur = cur.right;
        }
    }
}
```

---

## Code (Python)

```python
class Solution:
    def insertIntoBST(self, root, val: int):
        if root is None:
            return TreeNode(val)
        if val < root.val:
            root.left = self.insertIntoBST(root.left, val)
        else:
            root.right = self.insertIntoBST(root.right, val)
        return root

    def insertIterative(self, root, val: int):
        node = TreeNode(val)
        if root is None:
            return node
        cur = root
        while True:
            if val < cur.val:
                if cur.left is None:
                    cur.left = node
                    return root
                cur = cur.left
            else:
                if cur.right is None:
                    cur.right = node
                    return root
                cur = cur.right
```

---

## Edge Cases

| Case | Result |
|---|---|
| Empty tree (`root = null`) | Return new node as the entire tree. |
| Insert smallest value | Walks left all the way down. |
| Insert largest value | Walks right all the way down. |
| Inserting many sorted values | Tree degenerates into a linked list — O(n) per operation. |

---

## Follow-up Questions an Interviewer Might Ask

1. **"How would you maintain balance?"**
   Use **AVL** or **Red-Black** trees, which auto-rotate on insert/delete to keep height O(log n). Implementation is significantly more involved.

2. **"How would you allow duplicates?"**
   Pick a convention (always go left, or always go right, or store a count per node). Be consistent.

3. **"How do you delete from a BST?"**
   Three cases:
   - Leaf: just remove.
   - One child: replace with that child.
   - Two children: replace with the **inorder successor** (smallest value in the right subtree), then delete that successor from the right subtree.

4. **"Why does this insertion always create a leaf?"**
   Walking down with the BST rule ensures every step goes to the unique slot the new value should occupy — and that slot is always at a `null` (leaf position).

---

## Key Lessons from This Problem

1. **Recursive insertion mirrors the tree structure** — return the modified subtree, parent reassigns.
2. **BST insertion is O(log n) only when the tree is balanced.** For arbitrary input, expect O(h).
3. **Keep the property invariant** — every operation should leave the tree as a valid BST.
