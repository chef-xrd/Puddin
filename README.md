# 🍮 Puddin – Developer Spec (Updated)

---

### 🧠 Overview

Puddin is a **community-centered, vault-backed meme coin** built on **Radix Babylon** using **Scrypto**. This badass treat is minted (Naturalized) using **LSUs (Liquid Staking Units)** and tracked via a unified **puddin_citizen_nft**.

The Puddin token grows in value naturally over time as staking and protocol rewards accumulate in the LSU vault. It's deflationary, backed by real assets, and ready for action. Deploying (burning Puddin or locking Puddin, or Puddin LP) earns XRD rewards passively. Alternatively, Puddin can be returned to the bakery (Retired) when you need the dough.

---

### ⚙️ Core Mechanics

  1. Before doing anything, users without a **puddin_citizen_nft** generate one.
  2. A dropdown allows users to choose a different aesthetic progression path.
  3. If user's already have an NFT, they can choose it to perform their actions.
     - This is to make tracking metrics easier for multiple NFTs in the same wallet.


#### 🎶 Minting (Naturalization)

- User selects a **puddin_citizen_nft**and an amount of LSU they'd like to lock in the bakery (**redemption_vault**).
- Mint rate is based on the **XRD value** of the LSU (queried from the LSU manager):
    - To begin with the ratio is 100 Puddin for every 1 XRD locked in the LSU.
    - For example: 1 LSU may be worth 1.08 XRD which equals 108 Puddin, but it may be that the same amount becomes worth 100 Puddin due to deflationary mechanisms.
- During the minting process, the minting component charges a 5% fee.
    - Of this 5%, 1% goes to a dev wallet, while the rest goes to the **emissions_vault.**
 - After mint, vault receives the LSU, naturally increasing the backing value of all Puddin.

---

#### 🎖️ Deploy Puddin – Mission System

- **Deploy Puddin** by sending them on missions. Through the Deploy System, puddin_citizen_nfts can evolve and users can claim emissions rewards.
- The Puddin_Citizen_NFT tracks the following data to determine how it changes and what emission rewards are given.
  - `puddin_points`: The total weighted value of the NFT that can be used to claim emissions rewards from the emissions_vault.
  - `badge_tier/rank`: Visual representation of the NFT's **puddin_points**.
  - `puddin_burned`: Total Puddin burned for this badge through the deploy method. Multiply by 1.5.
      - When adding to this value, give a 10% bonus if the amount burned in one transaction is above 50,000 Puddin.
  - `puddin_locked`: Total Puddin locked with this badge. Starts at a 1.0 multiplier, but increases with lock duration.
  - `puddin_LP_locked`: Total Puddin LP locked with this badge. 2.0 multiplier. May increase with lock duration.
  - `last_claim_epoch`: Last epoch when emissions were claimed.
  - `cumulative_rewards_per_puddin`: Accumulated claimable rewards per Puddin at last claim.
      - Each epoch begins a new balance sheet. At the end of the epoch, the amount on the sheet is split among all NFT holders depending on their **puddin_points.**
      - Value resets to 0 upon claim.
  - `puddin_deployed`: Rough estimate of how much puddin this NFT is worth.
      - This amount should be roughly equal to the value of burned puddin + locked puddin. 
- **Rank Promotions**: Every 100,000 puddin_points increases a rank, up to Rank 4 (would like to go all the way to rank 7).
- **Operation Merge**: Combine squads (badges) to form larger units.
- **Combining NFTs**:
  - Properly calculates pending rewards before merge. Runs the claim function on all the NFTs to claim any due amounts and sets `cumulative_rewards_per_puddin` to 0 on the resulting merged nft.
  - Sets both NFT ranks to 5.
  - Merges nft values.
  - Properly calculates pending rewards before merge.
  - 1 Epoch claim/merge cooldown is placed on the resulting badge.
  - Sets a new image which no longer has a rank progression from a random list, or alternatively users choose one. Not sure how to handle this honestly.

#### 💰 Mission Rewards (Fridge Intel)

- The Puddin Blueprint charges a commission fee on staking rewards that is taken straight from stored LSU as the LSU grows. It also charges a mint fee to mint Puddin.
- These fees are donated to the **Puddin HQ (emissions_vault)** in XRD.
- Deployed Puddin operatives obtain a portion of this LSU reward from their missions.

---

#### 🔄 Puddin Retirement Program (Retires/Redeems Puddin)

- Users can redeem Puddin for LSU at the backing rate  of **LSU Vault / Puddin Supply**.
- **2.5% fee** is applied:
  - **1.25% of LSU stays in the core vault**, increasing the backing of all remaining Puddin.
  - **1.25% goes to the community_vault** to fund events and rewards.
- Redeemed Puddin is **burned** and removed from circulation, maintaining the LSU/Puddin ratio.

This fee is a **feature**, not a penalty:
- 🔥 It supports long-term deflation.
- 💰 It boosts vault value and community capacity.
- 🔁 It opens **arbitrage opportunities** — if Puddin trades below vault value, redemptions can balance the price for a profit.
- 💀 It punishes anyone trying to drain the vault.

---
#### 📈 Epoch-Based Distribution Formula

```
epoch_rewards_per_puddin = epoch_deposits / total_participation_score
user_claimable = nft.puddin_points * (current_rewards_per_puddin - nft.cumulative_rewards_per_puddin)
Note: total_score includes weighted contributions from locks, burns, and LP
```

- Tracks `cumulative_rewards_per_puddin` globally (increases each epoch)
- Each NFT stores its `cumulative_rewards_per_puddin` at last claim
- New deposits to growth vault increase the global rate
- Claims update the NFT's stored rate to current

#### 🔐 Implementation

- System maintains:
  - `total_participation_score`: Sum of all NFT scores
  - `cumulative_rewards_per_puddin`: Running total of rewards per score point
  - `current_epoch`: Current epoch number
- NFT combining: Claims pending rewards first, then merges stats
- This prevents early claimers from draining the vault

---

### 🏛️ Vault System

#### Bakery (**core_vault**)
- Stores LSUs backing Puddin (Puddin Validator LSUs only)
- Used for redemption payouts
- Accepts LSU donations

#### HQ (**growth_vault**)
- Stores validator commission fees in LSU for Puddin Civilian NFT holders
- Funded by commission and mint fees.
- Accepts Puddin LSU donations.

#### Mess Hall (**community_vault**)
- Admin-controlled vault for events, giveaways, dev-hiring/funding
- Accepts donation from any token.

**Vault Rules**:
- Donations accepted by all vaults in their respective tokens.
- Only the Mess Hall is withdrawable by Admin.

---

### 🛡️ Admin Apron & Safety Mechanisms

- Non-soulbound badge held by deployer.
- Grants access only to the **community_vault**.
- **Activity Requirements**:
  - Withdrawals from community_vault count as activity.
  - `keep_alive()` method - admin-only function to signal presence.
- If no activity for 12 months, vault becomes **recyclable**

- The **Recycle** public method:
  - Transfers **Mess Hall** Puddin LSU to HQ.
  - Burns Puddin owned by **Mess Hall** vault.
  - Acts as failsafe if admin badge is lost.

#### 🚨 Emergency Mechanism (Not sure if these should be included to avoid exploits)
- `emergency_pause()`: Admin can pause minting/redemption/claims
- `emergency_unpause()`: Resume normal operations

### 🚀 Deployment Information

**Validator Address:**
`validator_rdx1sdfqxl8dle9k74vgggq5swpaqwhnvq08w4tvfdd7mddjrpr6frzksd`

**Claim NFT Resource Address:**
`resource_rdx1n2kck9rngtvrz80kma25hvacy8wse3zu8mej6h7e0qlnx6jhfjsgcd`

**LSU Resource Address:**
`resource_rdx1thxlh6vnxygg0c3xg4saju0gcch997f7x83az0luuvm73kslsdx8sv`
