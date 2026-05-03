# Problem: When Does Greedy Fail for Coin Change?

## Category
Greedy / Theory

## Difficulty
Easy

---

## Problem Statement

Show — with a clean counterexample — that the **largest-coin-first** greedy strategy doesn't always work for the Coin Change problem.

This question tests whether you understand **why** Coin Change usually needs DP rather than greedy.

---

## Clarifying Questions to Ask the Interviewer

1. **What's "greedy"?** "Always take the largest coin that fits, then repeat with the remaining amount."
2. **Counterexample sufficient?** Yes — to disprove "greedy is optimal," one counterexample suffices.

---

## Approach — Counterexample

### Denominations `[1, 3, 4]`, amount `6`

**Greedy** (largest-first):
```
6 - 4 = 2     (used 1 coin: 4)
2 - 1 = 1     (used 2 coins: 4, 1)
1 - 1 = 0     (used 3 coins: 4, 1, 1)

Total: 3 coins.
```

**Optimal** (DP):
```
6 = 3 + 3.
Total: 2 coins.
```

Greedy used **3 coins**; optimal uses **2 coins**. Greedy is suboptimal here.

### Why greedy fails

Greedy commits to the largest coin first (`4`). That leaves `2`, which can only be made with `1 + 1`. The optimal solution skips `4` entirely and uses two `3`s.

In other words, "always take the biggest" prevents seeing better combinations that don't use the biggest coin.

### When *does* greedy work?

A coin denomination set is called **canonical** if greedy always gives the optimal answer.

**Examples of canonical systems:**
- US coins: `[1, 5, 10, 25, 50, 100]` ✓
- UK coins: also canonical ✓

**Examples of non-canonical systems:**
- `[1, 3, 4]` — counterexample above.
- `[1, 7, 10]` — try amount 14: greedy says `10 + 1 + 1 + 1 + 1 = 5` coins; optimal is `7 + 7 = 2` coins.

There's no easy pattern. Whether a system is canonical is non-trivial — there's an algorithm (**Pearson's test**) to check, by comparing greedy vs DP on small amounts up to roughly `2 × max_coin`.

### The lesson

For arbitrary denominations, **always use DP** unless you've proven your specific coin set is canonical.

---

## Time Complexity / Space Complexity

(Theory — not applicable.)

---

## Code (Java) — demonstrate the failure

```java
// Greedy (largest-first):
public int greedyCoinChange(int[] coins, int amount) {
    Arrays.sort(coins);                              // ascending; we'll iterate from the right
    int count = 0;
    for (int i = coins.length - 1; i >= 0 && amount > 0; i--) {
        count += amount / coins[i];
        amount %= coins[i];
    }
    return amount == 0 ? count : -1;
}

// On coins = {1, 3, 4}, amount = 6:
//   greedy: 4 + 1 + 1     → 3 coins
//   DP:     3 + 3         → 2 coins (better)
//
// On coins = {1, 7, 10}, amount = 14:
//   greedy: 10 + 1*4      → 5 coins
//   DP:     7 + 7         → 2 coins (better)
```

---

## Code (Python)

```python
def greedy_coin_change(coins, amount: int) -> int:
    coins = sorted(coins, reverse=True)
    count = 0
    for c in coins:
        if amount <= 0:
            break
        count += amount // c
        amount %= c
    return -1 if amount > 0 else count

# Compare with optimal DP from coin-change-min.md:
# greedy_coin_change([1, 3, 4], 6)   == 3   (suboptimal)
# DP                                 == 2

# greedy_coin_change([1, 7, 10], 14) == 5   (suboptimal)
# DP                                 == 2
```

---

## Edge Cases

- **Greedy might also fail to find any solution** if the smallest coin > 1. E.g., `coins = [3], amount = 10` — greedy uses `3 + 3 + 3` and is stuck with `1` left. DP correctly returns `-1`.
- **Edge case: amount 0.** Greedy returns 0 (no iterations); DP also returns 0.

---

## Follow-up Questions an Interviewer Might Ask

1. **"Is there a way to test whether a denomination system is canonical?"**
   Yes — **Pearson's algorithm** finds the smallest counterexample if one exists, by comparing greedy vs optimal on small amounts. Alternatively, just run DP and greedy on amounts up to ~`2 × max_coin` and check for disagreement.

2. **"Why does the US coin system happen to be canonical?"**
   Each denomination is a "round" multiple of smaller ones, in a way that makes the largest-coin strategy provably optimal. There's no general rule — it just happens to work for `{1, 5, 10, 25, ...}`.

3. **"Is there a polynomial-time greedy that works for all coin systems?"**
   No — it's been studied, and any "fixed greedy" can be defeated by adversarial denomination sets. Use DP.

4. **"Why is fractional knapsack greedy-optimal but 0/1 knapsack not?"**
   Fractional knapsack lets you take any fraction of an item, breaking the "all-or-nothing" constraint that makes 0/1 knapsack hard. With fractions, the greedy by value/weight ratio is exactly optimal.

---

## Key Lessons from This Problem

1. **One counterexample disproves a claim.** Greedy advocates carry the burden of proof — show it works on a specific system, or use DP.
2. **"It works for US coins"** isn't a general argument. Different denominations behave differently.
3. **DP is the safe default** for Coin Change. Use greedy only when you've **proven** it's correct for your specific system.
