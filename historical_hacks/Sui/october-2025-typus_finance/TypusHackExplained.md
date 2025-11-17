# Typus Hack
## Summary
**Loss:** ≈ $3.44 Million <br>
**Root Cause:** Permission-validation logic bug in the oracle module allowed an attacker to update oracle prices; this then enabled large arbitrage/swaps driven via manipulated prices. <br> 

**DISCLAIMER!** <br>
This document is about learning to spot similar vulnerabilities - not repeating existing deep dives. If you wish to deepen your understanding please do further research and look at the credit section below. <br>
This write-up is meant to get you the context of the attack and learn how to spot similar issues in your future audits/bug bounties/protocol building. 

**This article assumes you already understand how these work:** <br>
- Oracles and price-feeds in DeFi protocols
- Arbitrage / flash-swap
- Basic Move / Sui concepts (objects, capabilities, shared vs owned objects)

If you are not familiar with the above, there are plenty of great resources that are not Move-specific (and do not need to be). You can search for “oracle price manipulation” or “Sui Move permission validation” on YouTube or via blog posts.

---

## Why is this Important for Move Security
Move stops an entire class of low-level bugs — memory safety issues, resource duplication, accidental mutation.
But it cannot protect you from your own logic mistakes: access-control failures, wrong assumptions, and oracle manipulation.
This hack is a reminder that Move is powerful, not invincible. 

Access control mistakes still kill protocols — even in Move. 
The attack showed us that even the oldest DeFi attacks are still present today (oracle-manipulation). This attack also showed that shared objects aren’t automatic security checks, we still need to verify everything. 

This is an important reminder that even though Move is safer, it is still vulnerable. Developing smart contracts is risky and security should be the most paramount value at any development stage. 

## What Basically Happened
- The attack executed 16 Oct 2025 
- The attacker realised that the `update_v2()` function did not actually validate who the caller was. 
- The `UpdateAuthority` object is shared (globally accessible). 
- Using this, the attacker repeatedly called `update_v2` to push the oracle to extreme, fake prices.
- With the prices being corrupted, the attacker then proceeded to execute multiple swaps.
- Typus lost ~$3.44 Million

## Technical Root
The attacker used a number of things to make the attack possible. However, the root cause is actually quite simple. The caller check didn’t do anything at all. Nothing. Nada. 

Here is a snippet of the code:
```move
public fun update_v2(
  oracle: &mut Oracle,
  update_authority: & UpdateAuthority,
  price: u64,
  twap_price: u64,
  clock: &Clock,
  ctx: &mut TxContext
) {
  // check authority
  vector::contains(&update_authority.authority, &tx_context::sender(ctx));
  version_check(oracle);
  update_(oracle, price, twap_price, clock, ctx);
}
```

As we can see, the function does "try" to check if the `sender` has been added to the `update_authority` vector. However, the return value is ignored. `contains` returns true if the sender is authorized — and false otherwise. But nothing checks the returned answer.

It was by design to have the `update_authority` publicly shared as it needs to be. Shared objects behaved exactly as designed; the protocol’s permission logic did not. 

One ignored boolean — $3.44M vanished. 

The check is supposed to look like this:
```move
assert!(
    vector::contains(&update_authority.authority, &tx_context::sender(ctx)),
    E_NOT_AUTHORIZED
);
```
---

## Research & Audit-Level Lessons
### 1. Don’t Assume Move/Sui Handles All Access Control “Automatically”
Yes, Move design makes it easier for the protocol to stay safer. However, we still need to understand how everything works. 

All it took was noticing that contains returns a boolean — and nobody checked it. And because the `update_authority` object is available (as it should be) the attack was possible. 

### 2. Always Assert the Result of Authority Checks
When functions return a boolean to control permissions, always verify the result. Many protocols wrap boolean-returning helpers inside proper assert functions — but Typus didn’t. 

So if you see these functions, always make sure the returned answer is checked. 

### 3. Audit Shared Objects for Unexpected Reachability
Many protocols share objects publicly as it is needed for the protocols functionality. Audit how shared objects flow through the system. If anyone can access them, your permission checks must be airtight.

---

## Credit
- [SlowMist-Is the Move Language Secure? The Typus Permission-Validation Vulnerability](https://slowmist.medium.com/is-the-move-language-secure-the-typus-permission-validation-vulnerability-755a5175f7c3)
- [Halborn](https://www.halborn.com/blog/post/explained-the-typus-finance-hack-october-2025)