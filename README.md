# 🍮 Puddin – Developer Spec (Updated)

---

### 🧠 Overview

Puddin is a **community-powered, vault-backed meme coin** built on **Radix Babylon** using **Scrypto**. It uses food metaphors to celebrate warmth, sharing, and self-control. Puddin is minted using **LSUs (Liquid Staking Units)** and tracked via soul-bound **Portion Badges** (NFTs).

Puddin grows in value naturally over time as the staking rewards accumulate in the LSU used to mint it. It's deflationary, backed by real assets, and fun.

---

### ⚙️ Core Mechanics

#### 🎶 Minting

- Users mint Puddin by depositing **LSUs** into the Pantry (**core_vault**).
- Mint rate is based on the **XRD value** of the LSU (queried from the LSU manager):
  - LSU XRD value determines Puddin minted (1 LSU worth 1.08 XRD → 1.08 Puddin)
- Vault receives the LSU, increasing the backing value of all Puddin.

#### 🔄 Redemption

- Users can redeem Puddin for XRD at the backing rate: **T / P** (vault / supply).
- **2.5% fee** is applied:
  - **1.25% stays in the core vault**, increasing the backing of all remaining Puddin
  - **1.25% goes to the community_vault** to fund events and rewards
- Redeemed Puddin is **burned**.

This fee is a **feature**, not a penalty:
- 🔥 It supports long-term deflation
- 💰 It boosts vault value and community capacity
- 🔁 It opens **arbitrage opportunities** — if Puddin trades below vault value, redemptions can balance the price for a profit.

> **Note:** Users may receive a mix of LSUs from various validators depending on what is held in the vault at the time of redemption. While some LSUs may be less liquid on the open market, they are never stuck — any LSU can always be unstaked through the Radix network using the standard unbonding process (typically 7–14 days).

#### 💳 Transfers (Currently Disabled)

- The original model included a 0.5% fee on transfers.
- This mechanism has been **removed** to allow for DEX compatibility and ease of use.

---

### 🧮 Portion Badges – Soul-Bound Calorie Trackers

- Minted by **burning Puddin** ("burn calories")
- Non-transferable
- Track:
  - `puddin_burned`
  - `mint_sources` (tracks LSU types used)
  - `badge_tier`
- **NFT Upgrades**: Burn more Puddin to level up
- **Combining NFTs**: Merges stats, 24h lockout on new badge

---

### 🏛️ Vault System

#### Pantry (**core_vault**)
- Stores LSUs backing Puddin
- Used for redemption payouts

#### Fridge (**growth_vault**) — *Optional / Disabled for Now*
- Originally for emissions
- Disabled until alternative funding source defined

#### Party Table (**community_vault**)
- Admin-controlled vault for events, giveaways

**Vault Rules**:
- Donations accepted by Pantry (LSU) and Party Table (Puddin)
- Only Party Table is withdrawable by Admin

---

### 🛡️ Admin Apron

- Non-soulbound badge held by deployer
- Grants access only to the **community_vault**
- **Pruneable** after 1 year of inactivity (no withdrawal)
- Public method triggers vault transfer to Pantry or burns the contents

---

### ✔️ Scrypto Requirements

- Accept LSU via `mint_with_lsu(Bucket)`
- Query real-time LSU:XRD value for mint calculation
- Vault logic for redemption and burn
- Portion Badge NFT logic:
  - Track burn totals, source validators
  - Upgrade and combine badges
- Admin-only withdraw for community vault
- Prune mechanism after 12 months of inactivity

---

### 🖥️ Frontend Requirements

- **Scoop Puddin** (Mint with LSU)
- **Burn Calories** (Burn Puddin → Portion Badge)
- **Combine Badges** (Merge with auto-claim & cooldown)
- **Pantry View** (Vault status)
- **Badge Dashboard** (Burn history, LSU sources)

---

**Status**: Transfer fees removed ✨  |  LSU-based minting enabled ✨  |  Emissions paused ✨
