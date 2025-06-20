# 🍮 Puddin – Developer Spec (Updated v2)

---

### 🧠 Overview

Puddin is a **community-powered, vault-backed meme coin** built on **Radix Babylon** using **Scrypto**. It uses food metaphors to celebrate warmth, sharing, and self-control. Puddin is minted using **LSUs (Liquid Staking Units)** from the official **Puddin Validator** and tracked via soul-bound **Portion Badges** (NFTs).

Puddin grows in value naturally over time as the staking rewards accumulate in the LSU used to mint it. It's deflationary, backed by real assets, and fun.

---

### ⚙️ Core Mechanics

#### 🎶 Minting

- Users mint Puddin by depositing **LSUs** from the **Puddin Validator only** into the Pantry (**core_vault**).
- Mint price updates **instantly** after every mint, redemption, or burn:
  - Price = Vault XRD Value ÷ Total Puddin Supply
  - Initial price: 100 Puddin per 1 XRD (when no supply exists)
  - Prevents sandwich attacks by updating immediately
- **Slippage protection**: Optional max price parameter prevents failed transactions
- Vault receives the LSU, increasing the backing value of all Puddin.
- Contract **rejects** LSUs from other validators.

#### 🔄 Redemption

- Users can redeem Puddin for LSUs at the backing rate: **T / P** (vault / supply).
- **2.5% fee** is applied:
  - **1.25% stays in the core vault**, increasing the backing of all remaining Puddin
  - **1.25% goes to the community_vault** to fund events and rewards
- Redeemed Puddin is **burned** (tracked separately from "calorie burns").

This fee is a **feature**, not a penalty:
- 🔥 It supports long-term deflation
- 💰 It boosts vault value and community capacity
- 🔁 It opens **arbitrage opportunities** — if Puddin trades below vault value, redemptions can balance the price for a profit

> **Note:** All redemptions return Puddin Validator LSUs. Users can unstake these through the standard Radix network unbonding process (typically 7–14 days).

---

### 🧮 Portion Badges – Soul-Bound Calorie Trackers

- Minted by **burning Puddin** ("burn calories" - tracked separately from redemption burns)
- Non-transferable
- Track:
  - `puddin_burned`: Total Puddin burned for this badge (includes burn bonus)
  - `mint_sources`: Tracks LSU validator (always Puddin Validator)
  - `badge_tier`: Visual representation of burn amount
  - `last_claim_epoch`: Last epoch when emissions were claimed
  - `cumulative_rewards_per_puddin`: Accumulated rewards per Puddin at last claim
- **NFT Upgrades**: Burn more Puddin to level up
- **Combining NFTs**: 
  - Merges `puddin_burned` totals
  - Takes the max of `last_claim_epoch` to prevent double claiming
  - Properly calculates pending rewards before merge
  - 24h cooldown on the resulting badge
- **Burn Bonus**: Burning 100,000+ Puddin at once adds 10% to your burn credit
  - Example: Burn 150,000 → Badge shows 165,000 `puddin_burned`
  - This 165,000 is used for ALL emission calculations

---

### 🧊 Emissions Return System (Fridge Rewards)

The Puddin Validator charges a commission fee on staking rewards. These fees are donated to the **Fridge (growth_vault)** in XRD. Portion Badge holders claim emissions based on epochs and accumulated rewards.

#### 📈 Epoch-Based Distribution Formula

```text
epoch_rewards_per_puddin = epoch_deposits / total_puddin_burned_for_badges
user_claimable = badge.puddin_burned * (current_rewards_per_puddin - badge.cumulative_rewards_per_puddin)

Note: badge.puddin_burned includes any burn bonus earned
```

- Tracks `cumulative_rewards_per_puddin` globally (increases each epoch)
- Each badge stores its `cumulative_rewards_per_puddin` at last claim
- New deposits to Fridge increase the global rate
- Claims update the badge's stored rate to current

#### 🔐 Implementation

- System maintains:
  - `total_puddin_burned_for_badges`: Sum of all Puddin burned for badges (excludes redemption burns)
  - `cumulative_rewards_per_puddin`: Running total of rewards per burned Puddin
  - `current_epoch`: Current epoch number
- Badge combining: Claims pending rewards first, then merges stats
- This prevents early claimers from draining the fridge

---

### 🏛️ Vault System

#### Pantry (**core_vault**)
- Stores LSUs backing Puddin (Puddin Validator LSUs only)
- Used for redemption payouts
- Accepts LSU donations

#### Fridge (**growth_vault**)
- Stores validator commission fees in XRD for Portion Badge holders
- Funded by Puddin Validator commission donations
- Accepts XRD donations

#### Party Table (**community_vault**)
- Admin-controlled vault for events, giveaways, dev-hiring/funding
- Stores Puddin
- Accepts Puddin donations

**Vault Rules**:
- Donations accepted by all vaults in their respective tokens
- Only Party Table is withdrawable by Admin

---

### 🛡️ Admin Apron & Safety Mechanisms

- Non-soulbound badge held by deployer
- Grants access only to the **community_vault**
- **Activity Requirements**:
  - Withdrawals from community_vault count as activity
  - `keep_alive()` method - admin-only function to signal presence
- If no activity for 12 months, vault becomes **recyclable**
- The **Recycle** public method:
  - Transfers Party Table XRD to Fridge
  - Burns any Puddin in Party Table
  - Acts as failsafe if admin badge is lost

#### 🚨 Emergency Mechanism
- `emergency_pause()`: Admin can pause minting/redemption/claims
- `emergency_unpause()`: Resume normal operations
- Cannot pause for more than 30 days without automatic unpause

---

### ✔️ Scrypto Requirements

- Accept LSU via `mint_with_lsu(Bucket, Option<Decimal>)` - validate it's from Puddin Validator
  - Optional max price parameter for slippage protection
- Implement instant mint pricing:
  - Update price after every mint, redemption, or burn
  - Initial price: 0.01 XRD per Puddin
- Vault logic for redemption with separate burn tracking
- Portion Badge NFT logic:
  - Track burns, epochs, accumulated rewards
  - Proper merge logic for combining badges
  - Claim calculation based on epoch system
  - 10% burn bonus when burning 100,000+ Puddin at once
- Global tracking:
  - `total_puddin_burned_for_badges`
  - `cumulative_rewards_per_puddin`
  - `current_mint_price`
  - Separate tracking for redemption burns
- Admin methods:
  - `withdraw_community_vault(amount)`
  - `keep_alive()`
  - `emergency_pause()` / `emergency_unpause()`
- Public `recycle()` method with time check
- Validator commission deposit method: `deposit_validator_commission(Bucket)`

---

### 🖥️ Frontend Requirements

- **Scoop Puddin** (Mint with LSU - shows live mint price + slippage settings)
- **Burn Calories** (Burn Puddin → Portion Badge)
- **Combine Badges** (Shows pending rewards, handles claim during merge)
- **Claim Fridge Share** (Shows pending emissions based on epochs)
- **Pantry View** (Vault status, backing ratio, current mint price)
- **Badge Dashboard** (Burn history, pending rewards, tier progress)
- **Emergency Status** (Shows if paused, time until auto-unpause)

---

### 🚀 Future Considerations

**Liquidity Challenge**: Initial liquidity provision requires funding. Consider:
- Community-funded liquidity pool with LP incentives
- Initial "Puddin Party" event where early supporters provide liquidity
- Partnership with existing Radix DEXs

**Batch Operations** (if needed):
- `batch_claim(Vec<NonFungibleLocalId>)`: Claim for multiple badges in one transaction
- Useful for users managing multiple badges or services claiming on behalf of users

**Potential Future Features**:
- Badge tier perks could include:
  - Multipliers on certain community events
  - Governance weight (if governance is added)
  - Visual flair/recognition
  - Priority access to new features
- These would be social/community benefits rather than affecting core tokenomics

**Note on Burn Bonus**: The MVP includes a 10% burn bonus when burning 100,000+ Puddin at once to incentivize larger burns and create buy pressure spikes.
