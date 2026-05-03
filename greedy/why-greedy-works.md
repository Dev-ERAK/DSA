# Problem: Why Does Greedy Work for Activity Selection?

## Category
Greedy / Theory

## Difficulty
Easy

---

## Problem Statement

Explain — in plain English and with a proof — why the **earliest-finish-time greedy** is **always optimal** for the Activity Selection problem.

---

## Clarifying Questions to Ask the Interviewer

1. **Conceptual or formal proof?** Both — start informal, then make it formal with an exchange argument.

---

## Approach — The Exchange Argument

### Setup

Activities are sorted by end time: `f₁ ≤ f₂ ≤ ... ≤ fₙ`.

Suppose `OPT = {o₁, o₂, ..., o_k}` is **any** optimal solution, also sorted by end time.

The greedy picks `g₁` = the activity with the earliest finish time overall.

### Claim — We can replace `o₁` with `g₁` in OPT and stay optimal

#### Why?

1. By definition, `g₁` finishes **no later than** `o₁` (since g₁ has the earliest finish among all activities).
2. So if `o₁` doesn't conflict with `o₂..o_k`, then `g₁` (which ends earlier or equal) also doesn't conflict with them.
3. Replace `o₁` with `g₁`. The new set has size `k` and is conflict-free. **Still optimal.**

This shows: **there exists an optimal solution that starts with the greedy's first choice.**

### Recurse on the smaller subproblem

After picking `g₁`, ignore all activities that conflict with it. The remaining problem is structurally the same: pick the maximum non-overlapping subset from the rest.

By induction, we can repeat the argument: there's an optimal solution that starts with the greedy's next choice. And so on.

Conclusion: **the greedy choice (earliest finish time at each step) is always optimal.**

### This proof style — "Greedy stays ahead"

The exchange argument proves: at any step, the greedy's partial solution is **at least as good as** any optimal partial solution. So the greedy's final answer is at least as large.

### Why other greedies fail — concrete counterexamples

#### Shortest duration first

Activities: `(1, 5), (4, 6), (5, 9)`.

- **Shortest duration:** picks `(4, 6)` (duration 2). Now `(1, 5)` and `(5, 9)` both conflict with `(4, 6)`. Result: **1**.
- **Earliest finish time:** picks `(1, 5)`, then `(5, 9)`. Result: **2**.

#### Earliest start time first

Activities: `(0, 100), (1, 2), (2, 3)`.

- **Earliest start:** picks `(0, 100)`. That blocks everything else. Result: **1**.
- **Earliest finish:** picks `(1, 2)`, then `(2, 3)`. Result: **2**.

#### Latest start first

Could be made to fail with a similar construction. **Only earliest-finish-time is provably optimal.**

### General principles for "when does greedy work?"

A greedy algorithm is correct iff the problem has these two properties:

1. **Greedy choice property** — a globally optimal solution can be reached by making locally optimal choices.
2. **Optimal substructure** — the optimal solution to the problem contains optimal solutions to subproblems.

Activity selection has both. The Coin Change problem (with arbitrary denominations) has only the second — see `greedy-fails-coin.md`.

---

## Time Complexity / Space Complexity

(Theory — not applicable.)

---

## Code (Java) — counterexamples for failed greedies

```java
// Demonstrate that "shortest duration" greedy is not optimal:
// Activities: (1,5), (4,6), (5,9)
//   Shortest-duration picks (4,6) (length 2) — only 1 activity total.
//   Optimal (earliest-finish-time) picks (1,5), (5,9) — 2 activities.
```

---

## Code (Python) — same counterexamples documented

```python
def counterexample_shortest_duration():
    activities = [(1, 5), (4, 6), (5, 9)]
    # Shortest-duration greedy picks (4,6), conflicts with the others → 1.
    # Earliest-finish-time picks (1,5) then (5,9) → 2.
    return None

def counterexample_earliest_start():
    activities = [(0, 100), (1, 2), (2, 3)]
    # Earliest-start picks (0,100), blocks everything → 1.
    # Earliest-finish picks (1,2) then (2,3) → 2.
    return None
```

---

## Edge Cases

| Case | Why earliest-finish still wins |
|---|---|
| Single activity | Trivially optimal. |
| All disjoint | Every greedy picks them all. |
| Equal finish times | Either choice; the proof still works (exchange leaves the count unchanged). |

---

## Follow-up Questions an Interviewer Might Ask

1. **"When does greedy *not* work?"**
   Greedy fails when **optimal substructure exists but the greedy choice property doesn't**. Example: 0/1 Knapsack — locally best item may not be in the optimal set.

2. **"How do you formally prove a greedy?"**
   Two main techniques:
   - **Exchange argument** — show you can swap any non-greedy choice with the greedy one without losing optimality.
   - **Induction on solution size** — show that after the greedy's first choice, the remaining problem is identical in structure but smaller.

3. **"Greedy choice property in plain English?"**
   "There's an optimal solution that includes the greedy's choice." That's the heart of the proof.

4. **"Are exchange arguments always for greedy?"**
   No — they're a general proof technique used in matroid theory, scheduling, and more.

---

## Key Lessons from This Problem

1. **Earliest-finish-time greedy is optimal for activity selection** — proved via exchange argument.
2. **Other greedies fail with concrete counterexamples.** Don't assume any local choice rule works.
3. **Greedy choice + optimal substructure** are the two properties to verify before trusting a greedy algorithm.
