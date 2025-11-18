# Thala Hack
## Summary
**Loss:** $25.5 Million <br>
**Root Cause:** Input validation bug in a farming contract: user withdrawal logic did not properly check that the amount being withdrawn matched the user’s actual staked balance. <br>

**DISCLAIMER!** <br>
This document is about learning to spot similar vulnerabilities — not a full forensic post-mortem. Use this to understand the attack’s mechanics, the design failure, and how to avoid these issues in your own Move / Aptos code. 

**This document assumes you already understand how these work:** <br>
- Staking / farming contracts
- Basic Move / Aptos concepts (Move resources, Move modules, object ownership) 

---

## Why is this Important for Move Security
Move gives you memory safety, linear types, and a warm fuzzy feeling.  
It does NOT give you logic safety or common sense.

This hack is the ultimate reminder:  
**Move will stop you from double-spending a resource, but it won’t stop you from believing a random u64 from the internet.** 

$25.5M evaporated because one invariant check was missing. One. Single. Line.

---

## What Basically Happened
November 8, 2024

- Attacker adds a tiny bit of liquidity, receives THALA-LP
- Stakes the LP into Thala’s farm (everything still normal)
- Calls `withdraw()` with amount (technically a very large u64)
- Contract hands over the entire pool’s LP tokens because it never checks the user’s actual stake
- Attacker swaps fake-new LP tokens for real USDC/USDT
- Repeats a few times until the pool is dryer than the Sahara
- Profit: $25.5M. Time elapsed: ~8 minutes.

---

## Technical Root
Here’s the real (simplified but faithful) vulnerable code from Thala’s staking module (comments are added):

```move
public entry fun withdraw(
    signer: &signer,
    amount: u64,                   // user: "gimme this many"
    pool: &mut StakingPool,
) acquires UserStake {
    let user_addr = signer::address_of(signer);

    // Harvest pending rewards first — correct
    harvest_rewards(signer, pool);

    // ←←← THE $25,500,000 LINE THAT ISN’T HERE →→→
    // assert!(user_stake.amount >= amount, error::invalid_argument(E_INSUFFICIENT_BALANCE));

    // Blind trust 
    let user_stake = borrow_global_mut<UserStake>(user_addr);
    user_stake.amount -= amount;                    // Move prevents underflow panic, cool
    pool.total_staked -= amount;                    // but still subtracts whatever you asked for

    // Send LP tokens straight out of the pool’s vault, even though the user never deposited that much.
    transfer::public_transfer(lp_coin::withdraw(&mut pool.lp_vault, amount), user_addr);
}
```

That’s it. One missing comparison. The attacker just needed a tiny real stake to pass ownership checks, then requested the max u64 they felt like.

---

## Research & Audit-Level Lessons
### 1. Never Trust User-Supplied Amounts on Withdrawals. Ever.
Rule #1 of staking contracts in any language, including Move:

```move
assert!(user_stake.amount >= amount, E_INSUFFICIENT_STAKE);
```

Put it right after you load the user’s stake. Tattoo it on your forehead.

### 2. “It Worked in v1” Is Not a Security Argument
Thala had prior audits. The bug was introduced in a new version of the farm.
Regression bugs are the silent killers. Re-audit every new deployment that touches balance math.

### 3. Defensive Programming > “Move Will Save Me” 
Move stops memory bugs. It doesn’t stop you from writing:

```move
total_staked -= user_says_so;
```

Use `SafeMath`-style patterns, explicit invariant checks, and consider circuit-breakers that cap single-tx withdrawals.

### 4. Emergency Design – Plan for When You Inevitably Screw Up
Good contracts have:
- Admin pause / freeze
- Withdrawal caps per tx
- Timelocks on big state changes
- Ability to rescue LP from the vault without rewarding the attacker

Thala eventually recovered ~$8M because some assets were still stuck in cooldown. Lucky, not designed.

### 5. Bonus Move-Specific Trick
Want to make this bug literally impossible to miss in code review? Store the user’s stake in the same resource you subtract from and use a helper:

```move
public fun safe_withdraw(user_stake: &mut UserStake, amount: u64) {
    assert!(user_stake.amount >= amount, E_INSUFFICIENT);
    user_stake.amount = user_stake.amount - amount;
}
```

Now any dev skipping the check gets a compile error when they forget to call the safe version. Free security.

---

## Credit
- [Halborn - Explained: The Thala Hack (November 2024)](https://www.halborn.com/blog/post/explained-the-thala-hack-november-2024)
- On-chain txs if you want to feel the pain yourself