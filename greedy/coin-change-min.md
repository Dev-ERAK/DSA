# Problem: Minimum Coins to Make Change

## Category
Dynamic Programming (with greedy variant for canonical denominations)

## Difficulty
Medium

---

## Problem Statement

Given:
- A list of coin **denominations** (e.g., `[1, 5, 10, 25]`).
- An **amount** to make.

Return the **minimum number of coins** needed to make exactly that amount. Return `-1` if it's impossible.

You have an **unlimited** supply of each denomination.

This is **LeetCode #322**.

### Examples

```
coins = [1, 2, 5], amount = 11   →  3   (5 + 5 + 1)
coins = [2], amount = 3          →  -1  (impossible)
coins = [1], amount = 0          →  0
coins = [1, 3, 4], amount = 6   →  2   (3 + 3) — beats greedy's 4 + 1 + 1!
```

---

## Clarifying Questions to Ask the Interviewer

1. **Unlimited copies of each coin?** Yes (LeetCode 322).
2. **All denominations positive?** Yes.
3. **Amount can be 0?** Yes — return 0 (no coins needed).
4. **Can we use the greedy "always take the largest coin that fits"?** **No** in general — see `greedy-fails-coin.md`. Use DP.

---

## Approach — Dynamic Programming

### Why greedy doesn't work in general

Take `coins = [1, 3, 4], amount = 6`:
- **Greedy:** `4 + 1 + 1 = 3` coins.
- **Optimal:** `3 + 3 = 2` coins.

Greedy fails because picking the largest coin first commits us to a suboptimal path. With arbitrary denominations, we need DP.

### The DP idea

Define `dp[x]` = minimum number of coins needed to make amount `x`.

Base case: `dp[0] = 0`.

Recurrence:

```
dp[x] = 1 + min(dp[x - c]  for each coin c, if c ≤ x)
```

Translation: to make `x`, try each coin `c`. If we use `c`, we need `dp[x - c]` more coins. Plus 1 for the current coin. Take the minimum over all choices.

If no coin fits, `dp[x]` stays at "infinity" → return `-1`.

### Walking through `coins = [1, 2, 5], amount = 11`

```
dp[0] = 0

dp[1]: try coin 1: dp[0] + 1 = 1. → dp[1] = 1
dp[2]: try coin 1: dp[1] + 1 = 2; coin 2: dp[0] + 1 = 1. → dp[2] = 1
dp[3]: coin 1: 2; coin 2: dp[1]+1 = 2. → dp[3] = 2
dp[4]: coin 1: 3; coin 2: dp[2]+1 = 2. → dp[4] = 2
dp[5]: coin 1: 3; coin 2: dp[3]+1 = 3; coin 5: dp[0]+1 = 1. → dp[5] = 1
dp[6]: coin 1: 2; coin 2: dp[4]+1 = 3; coin 5: dp[1]+1 = 2. → dp[6] = 2
dp[7]: coin 5: dp[2]+1 = 2. → dp[7] = 2
dp[8]: coin 5: dp[3]+1 = 3. → dp[8] = 3
dp[9]: coin 5: dp[4]+1 = 3. → dp[9] = 3
dp[10]: coin 5: dp[5]+1 = 2. → dp[10] = 2
dp[11]: coin 5: dp[6]+1 = 3. → dp[11] = 3
```

Answer: **3**. Reconstructing: `5 + 5 + 1`.

### Why O(amount × |coins|)?

For each `dp[x]`, we try every coin. There are `amount + 1` values of x and `|coins|` coins per value.

This is **pseudo-polynomial** — polynomial in the *value* of amount, but not in the bit-length. For huge amounts, you'd need a different approach.

---

## Time Complexity

**O(amount × |coins|)**

## Space Complexity

**O(amount)**

---

## Code (Java)

```java
public int coinChange(int[] coins, int amount) {
    // Use amount + 1 as a sentinel for "infinity" — any valid answer is ≤ amount.
    int[] dp = new int[amount + 1];
    Arrays.fill(dp, amount + 1);
    dp[0] = 0;

    for (int x = 1; x <= amount; x++) {
        for (int c : coins) {
            if (c <= x) {
                dp[x] = Math.min(dp[x], dp[x - c] + 1);
            }
        }
    }

    return dp[amount] > amount ? -1 : dp[amount];
}
```

---

## Code (Python)

```python
class Solution:
    def coinChange(self, coins, amount: int) -> int:
        INF = amount + 1                     # sentinel for "impossible"
        dp = [INF] * (amount + 1)
        dp[0] = 0

        for x in range(1, amount + 1):
            for c in coins:
                if c <= x:
                    dp[x] = min(dp[x], dp[x - c] + 1)

        return -1 if dp[amount] == INF else dp[amount]
```

---

## Edge Cases

| Case | Result |
|---|---|
| `amount = 0` | 0 (no coins needed) |
| No combination works | -1 |
| A coin equals `amount` | 1 |
| Negative amount | Undefined — clarify or throw |
| Empty coin list | If amount > 0, return -1 |

---

## Follow-up Questions an Interviewer Might Ask

1. **"Number of *ways* to make change instead of minimum coins (LeetCode 518)."**
   Different DP. Iterate **coins outer, amount inner**, and **add** instead of taking min:
   ```
   ways[0] = 1
   for c in coins:
       for x in c..amount:
           ways[x] += ways[x - c]
   ```

2. **"Memoized top-down version?"**
   Same complexity. Useful when many `amount` values are unreachable.

3. **"What if each coin can be used at most K times?"**
   Add another dimension to the DP (per-coin usage count), or use bounded knapsack techniques.

4. **"Print which coins are used."**
   Track which coin produced each `dp[x]`. Reconstruct from `amount` backward.

---

## Key Lessons from This Problem

1. **DP > Greedy** for arbitrary coin sets. Greedy works only for canonical systems.
2. **`dp[x] = 1 + min(dp[x - c] for c in coins)`** — memorize this recurrence.
3. **Sentinels for "impossible"** like `amount + 1` are cleaner than special-casing `-1` in the loop.
