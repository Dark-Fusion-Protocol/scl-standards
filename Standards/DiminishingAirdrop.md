# Diminishing Airdrop

---

# Create Diminishing Airdrop

## Commands

```[UTXO_SPENDER_1, UTXO_SPENDER_2, ... , UTXO_SPENDER_N]``` is a list of the UTXOs getting spent.  
```POOL_AMOUNT``` is the total amount available to be airdropped.  
```STEPDOWN_AMOUNT``` is the amount that the Airdrop diminishes by per step.  
```STEPDOWN_PERIOD``` is the amount of claims before the airdrop diminishes.
```MAX_AIRDROP``` is the first airdrop amount that will be received at the beginning of the STEPDOWN_PERIOD.  
```MIN_AIRDROP``` is the last airdrop amount that will be received at the end of the STEPDOWN_PERIOD.  
```CHANGE_UTXO``` is the UTXO the change gets bound to.  
```SINGLE DROP``` is true or false and indicates if multiple claims can be made from one address or not.  

### Single Diminishing Airdrop
```
<TXID,
    {CONTRACT_ID_1:DIMAIRDROP[
        [UTXO_SPENDER_1, UTXO_SPENDER_2, ... , UTXO_SPENDER_N],
        POOL_AMOUNT,
        STEPDOWN_AMOUNT,
        STEPDOWN_PERIOD,
        MAX_AIRDROP,
        MIN_AIRDROP,
        CHANGE_UTXO,
        SINGLE_DROP]}>
```
### Batched Diminishing Airdrop
```
<TXID,
    {CONTRACT_ID_1:DIMAIRDROP[
        [UTXO_SPENDER_1, UTXO_SPENDER_2, ... , UTXO_SPENDER_N],
        POOL_AMOUNT,
        STEPDOWN_AMOUNT,
        STEPDOWN_PERIOD,
        MAX_AIRDROP,
        MIN_AIRDROP,
        CHANGE_UTXO]}>
    {CONTRACT_ID_2:DIMAIRDROP[
        [UTXO_SPENDER_1, UTXO_SPENDER_2, ... , UTXO_SPENDER_N],
        POOL_AMOUNT,
        STEPDOWN_AMOUNT,
        STEPDOWN_PERIOD,
        MAX_AIRDROP,
        MIN_AIRDROP,
        CHANGE_UTXO]},
    ...,
    {CONTRACT_ID_N:DIMAIRDROP[
        [UTXO_SPENDER_1, UTXO_SPENDER_2, ... , UTXO_SPENDER_N],
        POOL_AMOUNT,
        STEPDOWN_AMOUNT,
        STEPDOWN_PERIOD,
        MAX_AIRDROP,
        MIN_AIRDROP,
        CHANGE_UTXO]}>
```

## Examples

### Single Diminishing Airdrop
```
<8d8f887752e0e3b5fbfc04958a54c107211aa1cd19062cf73bc8e8436fbc0c74,
    {3b1b20518485ec89ce9acf5bb23c5ccdb0ac26d0661e377014e894d295eec29e:DIMAIRDROP[
        [8c6be55f904d5711b6a3edb7921981fcdbe4e9b67c98ac06e0c0b9b121dc070c:1],
        40000000000000,
        10,
        160,
        250,
        50,
        TXID:1,
        true]}>
```
### Batched Diminishing Airdrop
> [!NOTE]
> Coming soon!

## Implementations

### Rust
```rust
pub fn create_dim_airdrop(&mut self, txid: &String, payload: &String, sender_utxos: &Vec<String>, pool_amount: &u64, step_down_amount: &u64, step_period_amount: &u64, max_airdrop: &u64, min_airdrop: &u64, change_utxo: &String, single_drop: &bool, current_block_height: u64) -> Result<(String, u64, bool), String> {
    let mut owners_amount: u64 = 0;
    for sender_utxo in sender_utxos.clone() {
        if self.owners.contains_key(&sender_utxo) {
            owners_amount += self.owners[&sender_utxo];
        }
    }

    if owners_amount == 0 {
        return Err("create_dim_airdrop: owner amount is zero".to_string());
    }

    if pool_amount > &owners_amount {
        return Err("create_dim_airdrop: pool amount is more than the owned amount".to_string());
    }

    let mut drips = match self.drips.clone() {
        Some(drips) => drips,
        None => HashMap::new(),
    };

    let mut new_owner = (change_utxo.to_string(),0, false);
    let mut diminishing_airdrops = match self.diminishing_airdrops.clone() {
        Some(diminishing_airdrops) => diminishing_airdrops,
        None => HashMap::new(),
    };

    for sender_utxo in sender_utxos.clone() {
        if self.owners.contains_key(&sender_utxo.clone()) {
            self.owners.remove(&sender_utxo);
            if let Some(old_drips) = drips.get(&sender_utxo) {
                let mut new_drips: Vec<Drip> = Vec::new();
                for drip in old_drips {
                    let new_drip = Drip {
                        block_end: drip.block_end.clone(),
                        drip_amount: drip.drip_amount.clone(),
                        amount: drip.amount.clone() - (current_block_height - drip.start_block) * drip.drip_amount,
                        start_block: current_block_height,
                        last_block_dripped:current_block_height.clone()
                    };
                    new_drips.push(new_drip.clone());
                }

                drips.insert(change_utxo.clone(),new_drips);
                // Remove the old drip from the vector
                drips.remove(&sender_utxo);
                new_owner.2 = true;
            }
        }
    }

    let change_amt: u64 = owners_amount - pool_amount;
    if change_amt > 0 {
        if self.owners.contains_key(change_utxo) {
            let new_amount = self.owners[change_utxo] + change_amt;
            new_owner.1 = new_amount.clone();
            self.owners.insert(change_utxo.to_string(), new_amount);
        } else {
            new_owner.1 = change_amt.clone();
            self.owners.insert(change_utxo.to_string(), change_amt);
        }
    }

    let dim_airdrop = DimAirdrop {
        pool_amount: *pool_amount,
        step_down_amount: *step_down_amount,
        step_period_amount: *step_period_amount,
        max_airdrop: *max_airdrop,
        min_airdrop: *min_airdrop,
        current_airdrop: *max_airdrop,
        current_in_period: 0,
        amount_airdropped: 0,
        last_airdrop_split: None,
        claimers: HashMap::new(),
        single_drop: *single_drop,
    };

    diminishing_airdrops.insert(sender_utxos[0].clone(), dim_airdrop);

    self.payloads.insert(txid.to_string(), payload.to_string());
    self.diminishing_airdrops = Some(diminishing_airdrops);
    self.supply -= pool_amount;
    self.drips = Some(drips);
    return Ok(new_owner);
}
```

---

# Claim Diminishing Airdrop

## Commands

```AIRDROP_ID``` is the first spender UTXO from the DIMAIRDROP call.  
```TXID:0``` is the UTXO the claimant must bind to.  

```
<TXID,
    {CONTRACT_ID:CLAIM_DIMAIRDROP[
        AIRDROP_ID,
        TXID:0]}>
```

## Examples

```
<55c9d410e1c0fa7ff62efbad05361da919f7c77b7ef14590e6db5d4a1954a698,
    {3b1b20518485ec89ce9acf5bb23c5ccdb0ac26d0661e377014e894d295eec29e:CLAIM_DIMAIRDROP[
        8c6be55f904d5711b6a3edb7921981fcdbe4e9b67c98ac06e0c0b9b121dc070c:1,
        TXID:0]}
```

## Implementations

### Rust
```rust
pub fn claim_dim_airdrop(&mut self, txid: &String, payload: &String, claim_id: &String, reciever_utxo: &String, pending: bool, donater_pub_address: &String) -> Result<(String, u64, bool), String> {
    let mut diminishing_airdrops = match self.diminishing_airdrops.clone() {
        Some(diminishing_airdrops) => diminishing_airdrops,
        None => return Err("claim_dim_airdrop: contract has reached no claimable diminishing airdrops".to_string()),
    };

    let mut dim_airdrop: DimAirdrop =  match diminishing_airdrops.get(claim_id) {
        Some(dim_airdrop) => dim_airdrop.clone(),
        None => return Err("claim_dim_airdrop: diminishing airdrop claim id not found".to_string()),
    };

    let mut new_owner = (reciever_utxo.to_string(), 0, false);
    if dim_airdrop.step_period_amount == dim_airdrop.current_in_period {
        dim_airdrop.current_in_period = 0;
        if dim_airdrop.current_airdrop > dim_airdrop.min_airdrop {
            dim_airdrop.current_airdrop -= dim_airdrop.step_down_amount; 
        }
    }

    let mut airdrop_amount = dim_airdrop.current_airdrop;
    if dim_airdrop.amount_airdropped + dim_airdrop.current_airdrop >= dim_airdrop.pool_amount {
        airdrop_amount = dim_airdrop.pool_amount  - dim_airdrop.amount_airdropped;
    }

    let drips = match self.drips.clone() {
        Some(drips) => drips,
        None => HashMap::new(),
    };

    let mut pending_claims = match self.pending_claims.clone() {
        Some(pending_claims) => pending_claims,
        None => HashMap::new(),
    };

    if pending {
        pending_claims.insert(reciever_utxo.to_string(), airdrop_amount);
        let mut new_amount = airdrop_amount;
        if self.owners.contains_key(reciever_utxo) {
            new_amount = self.owners[reciever_utxo] + airdrop_amount;
        }

        new_owner.1 = new_amount;
     } else {
        pending_claims.remove(reciever_utxo);
         if self.owners.contains_key(reciever_utxo) {
            let new_amount = self.owners[reciever_utxo] + airdrop_amount;
            new_owner.1 = new_amount.clone();
            self.owners.insert(reciever_utxo.to_string(), new_amount);
        } else {
            new_owner.1 = airdrop_amount;
            self.owners.insert(reciever_utxo.to_string(), airdrop_amount);
        }
     }

    if drips.contains_key(reciever_utxo) {
        new_owner.2 = true;
    }

    if dim_airdrop.single_drop {
        dim_airdrop.claimers.insert(donater_pub_address.to_string(), airdrop_amount);
    }

    dim_airdrop.amount_airdropped += airdrop_amount;
    dim_airdrop.current_in_period += 1;
    if dim_airdrop.amount_airdropped == dim_airdrop.pool_amount  {
        diminishing_airdrops.remove(claim_id);      
    }else{
        diminishing_airdrops.insert(claim_id.to_string(), dim_airdrop);
    }
    
    self.supply += airdrop_amount;
    self.diminishing_airdrops = Some(diminishing_airdrops);
    self.pending_claims = Some(pending_claims);
    self.payloads.insert(txid.to_string(), payload.to_string());
    return Ok(new_owner);
}
```

---