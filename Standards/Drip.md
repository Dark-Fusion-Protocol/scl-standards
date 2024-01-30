# Drip

Drips work in a similar fashion to a transfer but they include a block duration, which specifies how many blocks the drip must continue for.

The amount dripped per block is proportional to *total spend*/*drip duration*.

---

# Create Drip

## Commands

```[UTXO_SENDER_1, UTXO_SENDER_2, ... , UTXO_SENDER_N]``` is the list of the UTXOs getting spent for the drip.  
```AMOUNT``` is the amount of tokens that get dripped onto the UTXO.  
```DRIP_DURATION``` is the amount of blocks the drip will be active for.  
```[UTXO_RECEIVER_1(AMOUNT, DRIP_DURATION), UTXO_RECEIVER_2(AMOUNT, DRIP_DURATION), ... , UTXO_RECEIVER_N(AMOUNT, DRIP_DURATION)]``` is the UTXOs that will get dripped on.  
```UTXO_CHANGE``` is the UTXO the change is bound to.  

### Single Drip
```
<TXID,
    {CONTRACT_ID_1:DRIP[
        [UTXO_SENDER_1, UTXO_SENDER_2, ... , UTXO_SENDER_N],
        [UTXO_RECEIVER_1(AMOUNT, DRIP_DURATION), UTXO_RECEIVER_2(AMOUNT, DRIP_DURATION), ... , UTXO_RECEIVER_N(AMOUNT, DRIP_DURATION)],
        UTXO_CHANGE]}>
```
### Batched Drip
```
<TXID,
    {CONTRACT_ID_1:DRIP[
        [UTXO_SENDER_1, UTXO_SENDER_2, ... , UTXO_SENDER_N],
        [UTXO_RECEIVER_1(AMOUNT, DRIP_DURATION), UTXO_RECEIVER_2(AMOUNT, DRIP_DURATION), ... , UTXO_RECEIVER_N(AMOUNT, DRIP_DURATION)],
        UTXO_CHANGE]}>
    {CONTRACT_ID_2:DRIP[
        [UTXO_SENDER_1, UTXO_SENDER_2, ... , UTXO_SENDER_N],
        [UTXO_RECEIVER_1(AMOUNT, DRIP_DURATION), UTXO_RECEIVER_2(AMOUNT, DRIP_DURATION), ... , UTXO_RECEIVER_N(AMOUNT, DRIP_DURATION)],
        UTXO_CHANGE]},
    ...,
    {CONTRACT_ID_N:DRIP[
        [UTXO_SENDER_1, UTXO_SENDER_2, ... , UTXO_SENDER_N],
        [UTXO_RECEIVER_1(AMOUNT, DRIP_DURATION), UTXO_RECEIVER_2(AMOUNT, DRIP_DURATION), ... , UTXO_RECEIVER_N(AMOUNT, DRIP_DURATION)],
        UTXO_CHANGE]}>
```

## Examples

### Single Drip
```

```
### Batched Drip
```

```

## Implementations

### Rust
```rust
pub fn drip(&mut self, current_block_height: u64)-> Result<Vec<(String, u64, bool)>,String>{
    let mut drips = match self.drips.clone() {
        Some(drips) => drips,
        None => return Ok(Vec::new()),
    };
    
    let mut new_owners: Vec<(String, u64, bool)> = Vec::new();
    for (utxo, drips_on_utxo) in drips.clone() {
        let mut updated_drips: Vec<Drip> = Vec::new();
        let mut new_owner = (utxo.to_string(),0, true);
        for mut drip in drips_on_utxo{
            let mut drip_amount = (current_block_height - drip.last_block_dripped) * drip.drip_amount;
            if current_block_height == drip.block_end && ((drip.block_end  - drip.start_block) + 1) * drip.drip_amount < drip.amount  {
                drip_amount += drip.amount - (drip.block_end  - drip.start_block + 1) * drip.drip_amount;
            }

            match self.owners.get(&utxo) {
                Some(&e) => {
                    self.owners.insert(utxo.clone(), &e + drip_amount);
                    new_owner.1 = &e + drip_amount;
                }
                None => {
                    self.owners.insert(utxo.clone(), drip_amount);
                    new_owner.1 = drip_amount;
                }
            }

            self.supply += drip_amount;
            drip.last_block_dripped = current_block_height;
            if current_block_height != drip.block_end {
                updated_drips.push(drip);
            }else{
                new_owner.2 = false;
            }
        }

        if new_owner.1 != 0 {
            new_owners.push(new_owner);
        }
    
        if updated_drips.len() >= 1 {
            drips.insert(utxo, updated_drips);
        }else{
            drips.remove(&utxo);
        }
    }
    self.drips = Some(drips);
    return Ok(new_owners)
}
```

---