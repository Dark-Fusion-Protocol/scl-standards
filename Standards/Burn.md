# Burn

---

# Create Burn

## Commands

```[UTXO_BURNER_1, UTXO_BURNER_2, ... , UTXO_BURNER_N]``` is the list of the UTXOs getting burned.  
```BURN_AMOUNT``` is the amount of tokens getting burned.  
```CHANGE_UTXO``` is the UTXO the change gets bound to.  

### Single Burn
```
<TXID, 
    {CONTRACT_ID_1:BURN[
        [UTXO_BURNER_1, UTXO_BURNER_2, ... , UTXO_BURNER_N],
        BURN_AMOUNT,
        CHANGE_UTXO]}>
```
### Batched Burn
```
<TXID, 
    {CONTRACT_ID_1:BURN[
        [UTXO_BURNER_1, UTXO_BURNER_2, ... , UTXO_BURNER_N],
        BURN_AMOUNT,
        CHANGE_UTXO]},
    {CONTRACT_ID_2:BURN[
        [UTXO_BURNER_1, UTXO_BURNER_2, ... , UTXO_BURNER_N],
        BURN_AMOUNT,
        CHANGE_UTXO]},
    ...,
    {CONTRACT_ID_N:BURN[
        [UTXO_BURNER_1, UTXO_BURNER_2, ... , UTXO_BURNER_N],
        BURN_AMOUNT,
        CHANGE_UTXO]}>
```

## Examples

### Single Burn
> [!NOTE]
> Coming soon!
### Batched Burn
> [!NOTE]
> Coming soon!

## Implementations

### Rust
```rust
pub fn burn(&mut self, txid: &String, payload: &String, burner_utxos: &Vec<String>, burn_amount: &u64, change_utxo: &String) -> Result<i32, String> {
    let mut owners_amount = 0;
    for burner_utxo in burner_utxos.iter() {
        if let Some(&amount) = self.owners.get(burner_utxo) {
            owners_amount += amount;
        }
    }
    
    if owners_amount == 0 {
        return Err("burn: owner has no tokens to burn".to_string());
    }

    if owners_amount >= *burn_amount {
        for burner_utxo in burner_utxos {
            if let Some(&_amount) = self.owners.get(burner_utxo) {
                self.owners.remove(burner_utxo);
            }
        }

        self.owners.insert(change_utxo.to_string(), owners_amount - *burn_amount);
        self.supply -= *burn_amount;
        self.payloads.insert(txid.to_string(), payload.to_string());
    } else {
        return Err("burn: trying to brun more than is owned".to_string());
    }
    Ok(0)
}
```

---