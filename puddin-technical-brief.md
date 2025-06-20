# Puddin - Technical Development Brief

## Project Overview
Puddin is a deflationary meme token on Radix backed by LSU (Liquid Staking Units). Users mint Puddin by depositing LSU, can redeem it back for LSU (with fees), or burn it to create soul-bound NFT "Portion Badges" that earn XRD emissions.

**Contract Budget**: $2500-3500  
**Timeline**: MVP focused, no unnecessary features  
**Platform**: Radix Babylon (Scrypto)

---

## Core Components

### 1. Token & Resources
- **Puddin Token**: Fungible token, mintable/burnable by component
- **Portion Badge**: Non-transferable NFT with mutable data
- **Admin Apron**: Admin badge (transferable) for community vault access

### 2. Vaults
- **core_vault**: Holds Puddin Validator LSU (backs all Puddin)
- **growth_vault**: Holds XRD emissions for badge holders (the "Fridge")
- **community_vault**: Holds Puddin for community use (admin accessible)

---

## State Variables Required

```rust
struct PuddinComponent {
    // Resource managers
    puddin_token_manager: ResourceManager,
    portion_badge_manager: ResourceManager,
    admin_badge_manager: ResourceManager,
    
    // Vaults
    core_vault: Vault,              // LSU storage
    growth_vault: Vault,            // XRD emissions
    community_vault: Vault,         // Puddin community funds
    
    // Tracking
    total_puddin_burned_for_badges: Decimal,
    cumulative_rewards_per_puddin: Decimal,
    whitelisted_validators: HashSet<ComponentAddress>,  // Multiple validators allowed
    
    // Dynamic mint pricing
    current_mint_price: Decimal,    // XRD per Puddin
    
    // Admin activity
    last_admin_activity: Instant,
}

struct PortionBadgeData {
    puddin_burned: Decimal,
    last_claimed_rewards_per_puddin: Decimal,
    creation_epoch: u64,
    last_updated: Instant,
}
```

---

## Core Functions Specification

### 1. Constructor
```rust
pub fn instantiate_puddin(
    initial_validator_address: ComponentAddress
) -> (ComponentAddress, Bucket) {
    // Create Puddin token
    // Create Portion Badge NFT resource (non-transferable)
    // Create Admin Apron badge
    // Initialize vaults
    // Initialize whitelisted_validators with initial_validator_address
    // Return component address and admin badge
}
```

### 2. Mint Puddin
```rust
pub fn mint_with_lsu(
    &mut self, 
    lsu_bucket: Bucket, 
    max_price_per_puddin: Option<Decimal>
) -> Bucket {
    // Get LSU validator address from LSU resource metadata
    // Verify LSU is from a whitelisted validator
    assert!(
        self.whitelisted_validators.contains(&validator_address),
        "LSU must be from a whitelisted validator"
    );
    
    // Get LSU:XRD value from LSU resource manager
    
    // Check slippage tolerance if provided
    if let Some(max_price) = max_price_per_puddin {
        assert!(
            self.current_mint_price <= max_price, 
            "Mint price {} exceeds max price {}", 
            self.current_mint_price, 
            max_price
        );
    }
    
    // Calculate Puddin to mint using current mint price:
    //   puddin_amount = lsu_xrd_value / self.current_mint_price
    // Deposit LSU to core_vault
    // Update mint price after vault change
    self.update_mint_price();
    // Mint and return Puddin
}

fn update_mint_price(&mut self) {
    let vault_xrd_value = self.get_vault_value_in_xrd();
    let total_supply = self.puddin_token_manager.total_supply();
    
    if total_supply > Decimal::zero() {
        self.current_mint_price = vault_xrd_value / total_supply;
    } else {
        self.current_mint_price = Decimal::from("0.01"); // Initial: 100 Puddin per XRD
    }
}
```

### 3. Redeem Puddin
```rust
pub fn redeem_puddin(&mut self, puddin_bucket: Bucket) -> Bucket {
    // Calculate backing rate: core_vault_value / total_puddin_supply
    // Apply 2.5% fee:
    //   - 1.25% stays in core_vault
    //   - 1.25% goes to community_vault as Puddin
    // Burn the Puddin
    // Withdraw LSU from core_vault
    // Update mint price after supply change
    self.update_mint_price();
    // Return LSU
}
```

### 4. Burn for Badge
```rust
pub fn burn_for_badge(&mut self, puddin_bucket: Bucket) -> Bucket {
    // Apply burn bonus: if amount >= 100000, add 10% to burn credit
    // Update total_puddin_burned_for_badges
    // Create or update Portion Badge NFT with:
    //   - puddin_burned = amount + bonus
    //   - last_claimed_rewards_per_puddin = current cumulative_rewards_per_puddin
    // Burn the Puddin
    // Update mint price after supply change
    self.update_mint_price();
    // Return badge
}
```

### 5. Claim Emissions
```rust
pub fn claim_emissions(&mut self, badge_proof: Proof) -> Bucket {
    // Validate badge proof
    // Calculate: claimable = badge.puddin_burned * 
    //   (cumulative_rewards_per_puddin - badge.last_claimed_rewards_per_puddin)
    // Update badge.last_claimed_rewards_per_puddin to current
    // Withdraw XRD from growth_vault
    // Return XRD
}
```

### 6. Combine Badges
```rust
pub fn combine_badges(
    &mut self, 
    badge1_bucket: Bucket, 
    badge2_bucket: Bucket
) -> Bucket {
    // Claim any pending emissions for both badges
    // Sum puddin_burned amounts
    // Take maximum last_claimed_rewards_per_puddin
    // Burn old badges
    // Mint new badge with combined stats
    // Return new badge
}
```

### 7. Deposit Validator Commission
```rust
pub fn deposit_validator_commission(&mut self, xrd_bucket: Bucket) {
    // Verify bucket contains XRD
    // Calculate new rewards per puddin:
    //   new_rewards = xrd_amount / total_puddin_burned_for_badges
    // Update cumulative_rewards_per_puddin += new_rewards
    // Deposit to growth_vault
}
```

### 8. Admin Functions
```rust
pub fn withdraw_community_vault(
    &mut self, 
    amount: Decimal, 
    admin_proof: Proof
) -> Bucket {
    // Verify admin badge
    // Update last_admin_activity
    // Withdraw from community_vault
}

pub fn add_validator(
    &mut self, 
    validator_address: ComponentAddress, 
    admin_proof: Proof
) {
    // Verify admin badge
    // Add validator to whitelisted_validators
    // Update last_admin_activity
}

pub fn remove_validator(
    &mut self, 
    validator_address: ComponentAddress, 
    admin_proof: Proof
) {
    // Verify admin badge
    // Ensure at least one validator remains
    assert!(
        self.whitelisted_validators.len() > 1,
        "Cannot remove last validator"
    );
    // Remove validator from whitelisted_validators
    // Update last_admin_activity
}

pub fn keep_alive(&mut self, admin_proof: Proof) {
    // Verify admin badge
    // Update last_admin_activity
}
```

### 9. Public Read Methods
```rust
pub fn get_stats(&self) -> PuddinStats {
    // Return:
    // - total_supply
    // - total_burned_for_badges
    // - backing_rate
    // - cumulative_rewards_per_puddin
    // - vault_balances
    // - current_mint_price
}

pub fn get_current_mint_price(&self) -> Decimal {
    // Return current mint price
    self.current_mint_price
}

pub fn get_whitelisted_validators(&self) -> Vec<ComponentAddress> {
    // Return list of whitelisted validator addresses
    self.whitelisted_validators.iter().cloned().collect()
}

pub fn get_top_burners(&self) -> Vec<(NonFungibleLocalId, Decimal)> {
    // Query top 10 badges by puddin_burned
    // Return list of badge IDs and amounts
}

pub fn calculate_claimable(&self, badge_id: NonFungibleLocalId) -> Decimal {
    // Read badge data
    // Calculate and return claimable amount
}
```

---

## Key Calculations

### Mint Price (Instant Updates)
```
// Updates after every mint, redemption, or badge burn
current_mint_price = core_vault_xrd_value / total_puddin_supply
puddin_to_mint = lsu_xrd_value / current_mint_price

// Initial price when supply is 0: 0.01 XRD per Puddin (100:1)
// Suggested default slippage: 1-3% for normal conditions
```

### Redemption
```
backing_rate = core_vault_lsu_value / total_puddin_supply
lsu_to_return = puddin_amount * backing_rate * 0.975  // After 2.5% fee
```

### Emission Claims
```
claimable = badge.puddin_burned * 
    (current_cumulative_rewards - badge.last_claimed_rewards)
```

### Burn Bonus
```
if puddin_amount >= 100000:
    burn_credit = puddin_amount * 1.1
else:
    burn_credit = puddin_amount

// Example: Burn 150,000 Puddin → Badge shows 165,000 puddin_burned
// This 165,000 is used for all emission calculations
```

---

## Validation Requirements

1. **LSU Validation**: Only accept LSU from whitelisted validators
2. **Badge Validation**: Ensure badges are legitimate before claims
3. **Amount Validation**: No negative amounts, check sufficient balances
4. **Division by Zero**: Handle case where total_burned_for_badges is 0
5. **Initial Mint Price**: Set to 0.01 XRD per Puddin (100:1) when total supply is 0
6. **Validator Management**: Cannot remove last validator from whitelist

---

## Testing Requirements

### Unit Tests
1. Mint with valid/invalid LSU
2. Mint with slippage tolerance (both passing and failing cases)
3. Redeem with fee calculations
4. Burn with and without bonus threshold
5. Emission claims with various scenarios
6. Badge combining with pending rewards
7. Admin functions and access control

### Integration Tests
1. Full user flow: Mint → Burn → Claim → Combine
2. Multiple users claiming from same emission pool
3. Edge cases: empty vaults, first user, etc.

---

## Deliverables

1. **Smart Contract**: Complete Scrypto component
2. **Unit Tests**: Comprehensive test coverage
3. **Deployment Script**: Transaction manifest for deployment
4. **Documentation**:
   - Function signatures and parameters
   - Example transaction manifests for each function
   - Slippage parameter usage examples
   - Component address and resource addresses after deployment
5. **Read Method Examples**: How to query stats and balances

---

## Important Notes

- **No Complex Features**: Skip pause/unpause, recycle mechanism, tiers
- **Keep It Simple**: Optimize for gas efficiency and clarity
- **Soul-Bound Badges**: Ensure NFTs are truly non-transferable
- **Error Messages**: Clear revert messages for debugging
- **Instant Price Updates**: Mint price updates after every mint, redemption, or burn to prevent sandwich attacks
- **Slippage Protection**: Optional max price parameter prevents failed transactions from price movements

---

## Post-Deployment Support

- One round of bug fixes if critical issues found
- Transaction manifest examples for all functions
- Basic guidance on frontend integration points

---

## Payment Terms

- 50% upon acceptance of proposal
- 50% upon successful testnet deployment and testing