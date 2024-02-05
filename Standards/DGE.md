# DGE

---

# Create DGE

## Commands

```[UTXO_SPENDER_1, UTXO_SPENDER_2, ... , UTXO_SPENDER_N]``` is a list of the UTXOs getting spent.  
```POOL_AMOUNT``` is total amount of tokens you are putting up for DGE.  
```SATS_RATE``` is the amount of sats exchanged per token. eg 10 000 sats rate means tokens will trade 1 for 0.00010000 BTC (10k sats).  
```MAX_DROP``` is the upper limit (in tokens) per drop.  
```DRIP_DURATION``` is how long the drip should last.  
```DONATION_ADDRESS``` is the address the donations will go to.  
```CHANGE_UTXO``` is where the DGE creators change goes.  
```SINGLE DROP``` is true or false and indicates if multiple claims can be made from one address or not.  

```
<TXID,
    {CONTRACT_ID_1:DGE[
        [UTXO_SPENDER_1, UTXO_SPENDER_2, ... , UTXO_SPENDER_N],
        POOL_AMOUNT,
        SATS_RATE,
        MAX_DROP,
        DRIP_DURATION,
        DONATION_ADDRESS,
        CHANGE_UTXO,
        SINGLE_DROP]}>
```

## Examples

```
<8c6be55f904d5711b6a3edb7921981fcdbe4e9b67c98ac06e0c0b9b121dc070c,
    {3b1b20518485ec89ce9acf5bb23c5ccdb0ac26d0661e377014e894d295eec29e:DGE[
        [782f9b6e1329a400bb0f6dc3b678a1b3a0c3a8186b44d5cd7b82b19478264548:0],
        700000000000000,
        5351,
        70000000000,
        12960,
        tb1qlh458zyuv4kc9g4pawvczss0tz09ht0u28e7u3,
        TXID:1,
        true]}>
```

## Implementations

### Rust
```rust
pub fn create_dge(&mut self, txid: &String, payload: &String, sender_utxos: &Vec<String>, dge: DGE, change_utxo: &String, current_block_height: u64) -> Result<(String, u64, bool), String> {
    let mut owners_amount: u64 = 0;
    for sender_utxo in sender_utxos.clone() {
        if self.owners.contains_key(&sender_utxo) {
            owners_amount += self.owners[&sender_utxo];
        }
    }
    if owners_amount == 0 {
        return Err("create_dge: owner amount is zero".to_string());
    }

    if dge.pool_amount > owners_amount {
        return Err("create_dge: pool amount is more than the owned amount".to_string());
    }

    let mut drips = match self.drips.clone() {
        Some(drips) => drips,
        None => HashMap::new(),
    };

    let mut new_owner = (change_utxo.to_string(),0, false);
    let mut dges = match self.dges.clone() {
        Some(dges) => dges,
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

    let change_amt: u64 = owners_amount - dge.pool_amount;
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

    dges.insert(sender_utxos[0].clone(), dge.clone());

    self.payloads.insert(txid.to_string(), payload.to_string());
    self.supply -= dge.pool_amount;
    self.dges = Some(dges);
    return Ok(new_owner);
}
```

---

# Claim DGE

## Commands

Use this command to claim from the DGE.

```DGE_ID``` is the first spender UTXO from the DGE call.  
```TXID:0``` is the UTXO the claimant must bind to.  

```
<TXID,
    {CONTRACT_ID:CLAIM_DGE[
        DGE_ID,
        TXID:0]}>
```

## Examples

```
<45dde82c8282be5c59c65456d7e2b6111fd66426fd59bb87d53bb94bc1e56dba,
    {adc110d77942e3e4ca9f2c585249a8d3b82d5b5a82563a0a01e9c591837d99d5:CLAIM_DGE[
        df3b9efc0f30f822888d17e0f26d089c82f3d2c05bd1e7778ab09b93b134d6ce:1,
        TXID:0]}
```

## Implementations

### Rust
```rust
pub fn claim_dge(&mut self, txid: &String, payload: &String, claim_id: &String, reciever_utxo: &String, donater: &String, donation: u64, current_block_height: u64) -> Result<(String, u64), String> {
    let mut dges = match self.dges.clone() {
        Some(dges) => dges,
        None => return Err("claim_dge: contract has reached no claimable dges".to_string()),
    };

    let mut dge: DGE =  match dges.get(claim_id) {
        Some(dge) => dge.clone(),
        None => return Err("claim_dge: dge claim id not found".to_string()),
    };

    if donation as u128 > (dge.max_drop as u128 * dge.sats_rate as u128) / 10u64.pow(self.decimals as u32) as u128 {
        return Err("claim_dge: donation over maximum limit".to_string())
    }

    let mut new_owner = (reciever_utxo.to_string(), 0);
    let mut token_amount = donation* 10u64.pow(self.decimals as u32)/ (dge.sats_rate);
    if token_amount == 0 {
        return Err("claim_dge: token allocation is zero".to_string())
    }

    if token_amount + dge.current_amount_dropped >= dge.pool_amount {
       token_amount = dge.pool_amount - dge.current_amount_dropped;  
    }

    let mut drips = match self.drips.clone() {
        Some(drips) => drips,
        None => HashMap::new(),
    };

    let drip_amount = token_amount / dge.drip_duration;
    let drip = Drip{
         block_end: current_block_height + dge.drip_duration -1,
         drip_amount: drip_amount.clone(),
         amount: token_amount.clone(),
         start_block: current_block_height.clone(),
         last_block_dripped:current_block_height.clone()
    };
    
    let mut new_drips = Vec::new();
    new_drips.push(drip);
    drips.insert(reciever_utxo.clone(), new_drips);
    match self.owners.get(reciever_utxo) {
        Some(&existing_amount) => {
            self.owners.insert(reciever_utxo.to_string(), &existing_amount + drip_amount);
            new_owner.1 = &existing_amount + drip_amount;
        }
        None => {
            self.owners.insert(reciever_utxo.to_string(), drip_amount);
            new_owner.1 = drip_amount;
        }
    }
    
    self.supply += drip_amount;
    self.drips = Some(drips);
    dge.current_amount_dropped += token_amount;
    if dge.single_drop {
        dge.donaters.insert(donater.to_string(), donation);
    }
    
    dges.insert(claim_id.to_string(), dge);
    self.dges = Some(dges);
    self.payloads.insert(txid.to_string(), payload.to_string());
    return Ok(new_owner);
}
```

---
