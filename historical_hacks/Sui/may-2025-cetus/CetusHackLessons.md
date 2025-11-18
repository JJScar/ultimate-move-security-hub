# Cetus Hack
## Summary
**Loss:** ≈ $223 million <br>
**Root Cause:** Overflow-check bypass in math library `checked_shlw`, allowing massive liquidity for minimal token deposit. <br>

**DISCLAIMER!** <br>
This document is about learning to spot similar vulnerabilities - not repeating existing deep dives. If you wish to deepen your understanding please do further research and look at the credit section below to learn more. <br>
This write-up is meant to get you the context of the attack and learn how to spot similar issues in your future audits/bug bounties/protocol building. 

**This document assumes you already understand how these work:** <br>
- AMMs/CLMMs
- Tick Math for AMM protocols
- Flashloans 
- Bit Shifting and Masking

If you are not familiar with the above, there are plenty of great resources that are not Move-specific (and do not need to be). You can head over to Cyfrin Updraft or search on YouTube for these terms to learn more. 
---

## Why is this Important for Move Security
This is the biggest hack in the Move ecosystem to date. Learning about this hack is super important for all of us to stay more security-focused as engineers. It also involves a misconception about Move which is described below. 

## What Basically Happened
- The exploit was executed in May 2025
- The attacker took a flash loan
- They open a very narrow tick range position of [300000, 300200]
- The above caused an overflow as the vulnerability lied in the `checked_shlw()` function
- The overflow caused the liquidity calculation to collapse to a tiny denominator, tricking the CLMM into giving huge LP liquidity for a ~1 token input.
- The attacker drained ~$223 million worth of tokens. 

## Technical Root
The hack exploited a logical bug. The overflow-check was incorrect, so the function silently allowed an overflow instead of reverting.

The `checked_shlw()` function (truncated for simplicity):
```move
public fun checked_shlw(x: u256): u256 {
    let threshold = (0xffffffffffffffffu256 << 192);
    assert!(x <= threshold, E_OVERFLOW);
    x << 64
}
```

The function was supposed to catch an overflow if a left shift of 64 bits. However, the mask that was used was too large, checking the overflow incorrectly. The threshold `0xffffffffffffffff << 192` is an extremely large value. Therefore almost any x will pass x <= threshold. The function was never actually checking for overflow.

This caused the system to believe the LP provided far more liquidity than they actually deposited. The extremely narrow tick range produced massive liquidity values, and these large intermediates triggered the faulty overflow-check.

Again, this write-up is not here to help you understand the hack, you can read other articles for that. But we as engineers want to know how to not make historical mistakes. Keep reading for that. 

---

## Research & Audit-Level Lessons
### 1. Never Assume Move Is "Safe" From Overflows
Arithmetic in Move generally reverts on overflow, but bit-level integer operations don’t get this automatic safety.

So always look extra hard for bugs in lower bits operations.

### 2. Library Code is AS Critical as Protocol Logic
As security researchers or protocols engineers, we tend to over "trust" library code as it doesn't feel as important. Every piece of code can 'make or break' the entire protocol. 

A good rule is this: "If it's hard to understand, most likely it can break easy". 

### 3. Look for the Edge Cases When Dealing with Bit Operations
The masking logic simply used the wrong threshold. This should be caught with good unit or fuzz testing. 

Try to see how unexpected inputs ruin the code.

### 4. CLMMs Tick Range is Super Important
The attacker used a very narrow tick range to exploit the protocol. So when auditing CLMMs, always test extreme tick placements—especially very narrow ranges—because they dramatically increase intermediate values.

---

## Credit
- [Halborn](https://www.halborn.com/blog/post/explained-the-cetus-hack-may-2025)
- [Rekt](https://rekt.news/cetus-rekt)