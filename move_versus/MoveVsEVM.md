# Move Versus EVM
So you are a Solidity dev or you are a security researcher who wants to start auditing Move (Sui/Aptos). Congrats. 

This doc will walk you through *"converting"* your EVM knowledge, and start understanding the things Move does better and worse than Solidity. 

This doc assumes you are already familiar with security concepts like reentrancy, storage collisions etc. In the case you are not, please search online for smart contracts security concepts. 

## 1. Mental Model Switch
The most significant change is simply the way EVM engineers *think* about smart contracts. The whole process of *"owning"* storage is different, which means that the design is different. 

Really, once you get passed this point, Move becomes extremely simple. 

Bellow is a concise table to compare the core differences I mentioned above. This is not enough to be able to fully understand the differences. This is simply to get you to start thinking differently. 

To learn the differences in a deeper level, in the credits section you will find a link to the Move Book which explains a fair bit. You will also find an X post written by a known Move Auditor (MoveJay/ @movebrah) that goes in depth.

| EVM | Move | Why? (EVM Pitfall Fixed) |
|-----|------|--------------------------|
| Contracts own storage | Objects own themselves (resources) | No accidental `balanceOf[addr] = 0` wipes (fixes storage collisions) |
| Global mutable state | Immutable-by-default + explicit moves | No storage mutation races (no more unchecked mappings) |
| Anyone can call anything (external/public) | Entry functions + capability checks | No surprise external calls (kills delegatecall proxies) |
| Reentrancy = your problem | Reentrancy basically impossible | No fallback/callbacks in the VM (DAO-style drains? Nope) |
| Upgrades = proxy patterns | Upgrades = publish new module + migrate | No `selfdestruct` surprises (Sui: object migration; Aptos: admin caps) |

If you only remember one thing:
In Move, value can never be duplicated or accidentally dropped. The type system enforces it at compile time. That single feature deletes half the lines in your Solidity checklist.

---
## 2. Your New Audit Radar: 4 Shifts from EVM 
Move abstracts away reentrancy and overflows, so pivot to these—think of them as the "capability era" of bugs.

1. **Capability / Key Discipline** – Capabilities are like singleton keys (no spoofing via tx.origin). EVM equivalent: Ownable roles, but enforced at type level.  
   *Example:* In a Sui lending protocol, ensure the `AdminCap` isn't leaked to a public module—audit the publish fn for `friend` restrictions.  
   *Tip:* Run Move Prover on cap flows; it'll flag 80% of leaks before runtime.

2. **Ownership Chains** – Functions passing resources can orphan assets if not explicit. (EVM parallel: unsafe transfers, but compiler-proofed here.)  
   *Example:* Post-Cetus, watch for unhandled returns in shared objects—e.g., a borrow that doesn't `return` properly aborts the tx but leaves dangling refs<grok-card data-id="ea5520" data-type="citation_card"></grok-card>.  
   *Tip:* Fuzz with random resource moves; test "what if this tx partially fails?"

3. **Logic & Emission Bugs** – General "messing" with the chain is no more, so zoom out to business logic.  
   *Example:* Aptos DeFi: Ensure view fns don't expose internal state (like pending yields) that could front-run emissions.  
   *Tip:* Simulate multi-tx sequences—Move's determinism makes this easier than EVM gas wars.

4. **Ecosystem Youth (Get Creative)** – Move's maturing fast, but watch for chain-specific gotchas. Sui: Parallelism risks in shared objects. Aptos: Dynamic dispatch (post-2024) reintroduces mild reentrancy—treat it like guarded callbacks.  
   *2025 Update:* With Sui's $10M security fund, expect more formal verif tools; Aptos added ZK post-quantum sigs for tx safety<grok-card data-id="879159" data-type="citation_card"></grok-card><grok-card data-id="6dcf8f" data-type="citation_card"></grok-card>. *Tip:* Join whitehat bounties for real-world practice.

---

## 3. More Resources
This doc is not meant to teach you much. It only functions as an entry point. To help guide you in your Move journey. To continue learning more please, as usual, read. 

Specifically, the [article](https://medium.com/@udohaniebiet/movevm-vs-evm-the-battle-of-virtual-machines-dbd4d2b48844) by Anitech is great to understand the difference between MoveVM and EVM. 

For broader understanding, the [Move Book](https://move-book.com/concepts/) is the best starting point. 
Once you get the hang of "Pure Move" you can start reading [Sui](https://docs.sui.io/guides)/[Aptos](https://aptos.dev/) "versions" of it.

Really X is the best place. Follow accounts that know what they're talking about. My favorite is [MoveJay](https://x.com/movebrah/), who has many posts on going from EVM to MoveVM. You can also follow [me](https://x.com/JJS_OnChain), I also post many Move posts 