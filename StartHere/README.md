# How to Get Started

Everyone arrives with a different background. Choose the path that matches your current level — feel free to jump around or combine them.

---

## 1. Absolute Beginner (No Coding Experience)
Goal: Go from zero to being able to read, write, and audit Move smart contracts.

1. **Programming basics**  
   Pick one:  
   - JavaScript or Python on [freeCodeCamp YouTube](https://www.youtube.com/c/Freecodecamp) (20–30 hours)  
   → Gives you variables, loops, functions, and problem-solving.

2. **Smart contract fundamentals**  
   Complete the free courses on [Cyfrin Updraft](https://updraft.cyfrin.io/) (Foundry + Solidity path).  
   Even though it’s Solidity-based, the security mindset, testing tools, and audit techniques transfer 1-to-1 to Move.

3. **Optional but helpful**  
   Read the first 6–8 chapters of [The Rust Book](https://doc.rust-lang.org/book/) – it will make Move’s ownership and borrowing concepts click instantly.

4. **Learn Move itself**  
   Work through the official [Move Book](https://move-book.com/) from start to finish (including the “Create a Coin” tutorial). This is the single best resource.

5. **Pick your chain**  
   - Sui → [Sui Developer Portal](https://docs.sui.io/guides) + [Sui Move Intro Course](https://github.com/sui-foundation/sui-move-intro-course)  
   - Aptos → [Aptos Build Docs](https://aptos.dev/) + [Aptos Shores](https://www.aptosshores.com/)

6. **Start auditing Move**  
   Jump to the Security Researcher path below.

---

## 2. You Already Know Smart Contracts (Solidity, Vyper, etc.)
You understand events, storage, reentrancy, access control, etc.

1. **Learn Move core**  
   [Move Book](https://move-book.com/) – focus on resources, abilities (`key`, `store`), and the object model differences.

2. **Choose Sui or Aptos (or both)**  
   - Sui: Object-centric model, parallel execution  
   - Aptos: Account-centric, Block-STM  
   Read the “Sui vs Aptos” comparison in this repo and pick one to start.

3. **Hands-on courses**  
   - Sui: blockchainBard YouTube series + official docs  
   - Aptos: net2dev YouTube series + Aptos Shores

4. **Move straight to security**  
   Continue to the Security Researcher path.

---

## 3. Security Researcher / Auditor (Any Background)
You want to find bugs in Move code today.

### Immediate actions
1. Read the entire **Move Security Best Practices** section of this repo.
2. Study past Move audits and Bug Bounties in the **Specific Move Vulnerabilities** section.  
3. Master the **Move Prover** – it’s the closest thing Move has to Certora/Formal Verification.
4. Practice on real code: fork popular Sui/Aptos GitHub repos and try to break them.

### Where to earn reputation & money
- Competitive audits: HackenProof, Cantina, Sherlock, Hats Finance, Code4rena (Move contests appear regularly now)  
- Bug bounties: Most major Sui/Aptos protocols on HackenProof or Immunefi  
- Private gigs: Many teams hire Move-specialized auditors directly (being early matters). To be able to get this you need to already have some portfolio or name.

### Key security resources
- Zellic’s “Move Fast and Break Things” blog series  
- Aptos official security guidelines  
- Move Prover tutorials  
- This repo’s checklists & vulnerability patterns

The Move ecosystem is still young — skilled security researchers are extremely rare and extremely well paid.

Welcome, and happy hacking (the white-hat kind)!