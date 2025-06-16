# 🍮 Puddin – Developer Spec

## 🧠 Overview
Puddin is a community-powered meme coin on **Radix Babylon** (Scrypto).  
It blends warmth, self-control, and sharing through food metaphors.  
Puddin utilizes a vault-backed token and soul-bound NFTs called **Portion Badges** to track burned tokens and reward NFT holders.

---

## ⚙ Core Mechanics

### 🧹 Minting
Users mint **Puddin** by depositing **1 XRD** into the Pantry (`core_vault`).

Mint rate:
- If **P = 0** or **T = 0** → mint **100 Puddin/XRD**.  
  (P = Puddin supply, T = amount of XRD in the Pantry/Treasury)
- Else → `P / T`.

Early adopters get sweeter portions.

---

### ↻ Redemption
Redeem **Puddin** for XRD at rate `T / P`.

- Users receive **95%** of value.
- Remaining **5%** is retained.
- Redeemed Puddin is **burned** ("calories consumed").

---

### 💸 Transfers
All transfers incur a fee between **0.1%** and **1%**. However, to begin with, the fee is set to **0.5%**. This amount can change by community vote (likely to be offchain):
- **50%** is **burned**
- **50%** is split:
  - **85%** → `growth_vault` (Fridge)
  - **15%** → `community_vault` (Party Table)

**Fee exemptions:** minting, redemption, and vault I/O.

---

## 🧏‍♀️ Portion Badges – Soul-Bound Calorie Trackers

Minted via **burning Puddin**.

Tracks per badge:
- `puddin_burned`
- `last_active`
- `emissions_claimed`

Emissions:
- Earned from the **Fridge** (`growth_vault`)
- **Pause after 12 months** of inactivity (per NFT)
- **Auto-burned** if unclaimed for **another 12 months**
- **Combining badges**:
  - Auto-claims emissions
  - Merges stats
  - Locks claiming for **24 hours**

---

## 🏦 Vault System

### Types:
- `core_vault` (Pantry) 🛡 – Holds XRD for redemption
- `growth_vault` (Fridge) 🌊 – Funds emissions
- `community_vault` (Party Table) 🎉 – Used for events

### Permissions:
- All vaults accept **donations**
- Only `community_vault` allows **admin withdrawals**
- `core_vault` accepts **XRD**
- `community_vault` accepts **Puddin**

---

## 🛡 Admin Apron (Badge)
- Non-soulbound, transferable badge
- Grants access only to `community_vault`
- Can be **pruned after 12 months** of inactivity
- Used for **airdrops and community events**

---

## 🧹 Party Cleanup (Inactivity Pruning)
- If no withdrawals from `community_vault` for **1 year**, anyone can call prune.
- Funds are either:
  - **Transferred to the Fridge**
  - Or **burned**

---

## 🖥 Frontend Requirements

Required UI Buttons:
- **Scoop Puddin** – Mint with XRD
- **Burn Calories** – Burn Puddin to mint/upgrade Portion Badge
- **Claim Your Portion** – Daily claim (per badge, 24h cooldown)
- **Combine Badges** – Merge badges with auto-claim

Optional UI:
- **Pantry View** – Vault status
- **Badge Dashboard** – Burn/claim/emission history

---

## ✅ Scrypto Design Requirements

- Mint, redeem, transfer with fee logic
- Track Portion Badges, activity, emissions
- Admin badge enforcement on `community_vault`
- Badge upgrades and merges with auto-claim
- Pruning and emission expiry logic
- 24h claim cooldown per badge
