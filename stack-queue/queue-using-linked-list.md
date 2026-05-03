# Problem: Implement a Queue Using a Linked List

## Category
Queue / Linked List / Design

## Difficulty
Easy

---

## Problem Statement

Implement a **First-In-First-Out (FIFO)** queue using a singly linked list, supporting these operations in **O(1)**:

- `enqueue(x)` — add `x` to the back.
- `dequeue()` — remove and return the front.
- `peek()` — look at the front without removing.
- `size()`, `isEmpty()`.

---

## Clarifying Questions to Ask the Interviewer

1. **Queue order is FIFO?** Yes — first added is first removed.
2. **Single-threaded?** Yes (a thread-safe queue is a separate problem).
3. **Why a linked list instead of an array?** A naïve array dequeue would be O(n) (shift everything left). With a linked list and a tail pointer, both ends are O(1).

---

## Approach

### The Big Idea

Maintain **two pointers**: `head` (the front) and `tail` (the back).

- `enqueue` adds to `tail`.
- `dequeue` removes from `head`.

That's the whole concept. Both operations touch only one or two pointers — O(1).

### Why both `head` and `tail`?

If we only kept `head`, we'd have to walk to the end of the list to enqueue — O(n). Keeping a `tail` pointer turns enqueue into O(1).

### Operations in detail

#### enqueue(x)

```
node = new Node(x)
if tail == null:                    # queue was empty
    head = tail = node
else:
    tail.next = node                # link to the back
    tail = node
size++
```

#### dequeue()

```
if head == null: error
val = head.val
head = head.next
if head == null:                    # queue is now empty
    tail = null                     # don't leave dangling tail
size--
return val
```

#### peek()

```
if head == null: error
return head.val
```

### Walking through some operations

Start: empty queue.

```
enqueue(1):  head = tail = (1)         queue: 1
enqueue(2):  tail.next = (2); tail = (2)
                                       queue: 1 → 2
enqueue(3):  tail.next = (3); tail = (3)
                                       queue: 1 → 2 → 3

dequeue():   val = head.val = 1
             head = head.next = (2)
                                       queue: 2 → 3
             returns 1

peek():      returns head.val = 2

dequeue():   val = 2; head = (3)       queue: 3
dequeue():   val = 3; head = null
             tail = null              queue: empty
             returns 3
```

### Critical detail — reset `tail` when emptying

When the last `dequeue` makes `head` null, we must also set `tail` null. Otherwise `tail` becomes a dangling reference to a node that's no longer in the queue. The next `enqueue` would mishandle it.

---

## Time Complexity

All operations: **O(1)**.

## Space Complexity

**O(n)** — one node per element.

---

## Code (Java)

```java
class LinkedQueue<T> {
    private static class Node<T> {
        T val;
        Node<T> next;
        Node(T v) { val = v; }
    }

    private Node<T> head;
    private Node<T> tail;
    private int size = 0;

    public void enqueue(T x) {
        Node<T> n = new Node<>(x);
        if (tail == null) {
            head = tail = n;            // first node — both pointers same
        } else {
            tail.next = n;
            tail = n;
        }
        size++;
    }

    public T dequeue() {
        if (head == null) {
            throw new NoSuchElementException("Queue is empty");
        }
        T val = head.val;
        head = head.next;
        if (head == null) {
            tail = null;                // queue is now empty — clear tail too
        }
        size--;
        return val;
    }

    public T peek() {
        if (head == null) {
            throw new NoSuchElementException("Queue is empty");
        }
        return head.val;
    }

    public int size()       { return size; }
    public boolean isEmpty() { return head == null; }
}
```

---

## Code (Python)

```python
class _Node:
    __slots__ = ("val", "next")
    def __init__(self, val):
        self.val = val
        self.next = None

class LinkedQueue:
    def __init__(self):
        self.head = None
        self.tail = None
        self._size = 0

    def enqueue(self, x):
        n = _Node(x)
        if self.tail is None:
            self.head = self.tail = n
        else:
            self.tail.next = n
            self.tail = n
        self._size += 1

    def dequeue(self):
        if self.head is None:
            raise IndexError("Queue is empty")
        val = self.head.val
        self.head = self.head.next
        if self.head is None:
            self.tail = None
        self._size -= 1
        return val

    def peek(self):
        if self.head is None:
            raise IndexError("Queue is empty")
        return self.head.val

    def __len__(self):
        return self._size

    def is_empty(self):
        return self.head is None
```

> `__slots__` saves memory in Python by avoiding the per-instance dict — useful for nodes you'll create millions of.

---

## Edge Cases

| Case | What happens |
|---|---|
| `dequeue` / `peek` on empty queue | Throw an exception. |
| Single-element queue | `enqueue` sets head = tail; `dequeue` clears both. |
| Many enqueues then many dequeues | All O(1). |
| Forgetting to reset `tail` when empty | Subtle bug — old `tail` reference becomes invalid. |

---

## Follow-up Questions an Interviewer Might Ask

1. **"Why a linked list instead of an array?"**
   With a naïve array-backed queue, dequeue from the front is O(n) (shift). To get O(1) on both ends with an array, you'd need a **circular buffer** (track head/tail indices that wrap around modulo capacity).

2. **"Doubly linked list for queues?"**
   Useful if you want O(1) on both ends bidirectionally (a **deque**). For a simple FIFO queue, singly linked is enough.

3. **"Thread-safe queue?"**
   Java has `LinkedBlockingQueue`, `ConcurrentLinkedQueue`. Python has `queue.Queue`. Building one from scratch involves locks or lock-free atomic ops — much harder.

4. **"How would you implement a queue using two stacks?"**
   See `queue-using-stacks.md` — a classic interview question.

---

## Key Lessons from This Problem

1. **Two pointers** (head and tail) make a linked list a perfect FIFO queue.
2. **Always update tail to null when the queue becomes empty** — easy to forget.
3. **Choose your backing structure** based on operations: random access → array; ends-only access → linked list.
