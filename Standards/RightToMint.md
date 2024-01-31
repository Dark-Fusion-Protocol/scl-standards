# Right to Mint

Right to mint can only be awarded during a minting event.

---

# Create Right to Mint

## Commands

```RIGHTS_UTXO``` is the UTXO that has the right to mint.  
```UTXO_RECEIVER``` is the UTXO which binds the mint amount.  
```CHANGE_UTXO``` is the UTXO the change is bound to.  
```MINT_AMOUNT``` is the amount of tokens that get minted.

```
<TXID,
    {CONTRACT_ID_1:RIGHTTOMINT[
        RIGHTS_UTXO,
        UTXO_RECEIVER,
        CHANGE_UTXO,
        MINT_AMOUNT]}>
```

## Examples

```
<782f9b6e1329a400bb0f6dc3b678a1b3a0c3a8186b44d5cd7b82b19478264548,
    {3b1b20518485ec89ce9acf5bb23c5ccdb0ac26d0661e377014e894d295eec29e:RIGHTTOMINT[
        3b1b20518485ec89ce9acf5bb23c5ccdb0ac26d0661e377014e894d295eec29e:0,
        TXID:0,
        TXID:1,
        5000000000000000]}>
```

## Implementations

### Rust
```rust
pub fn right_to_mint(&mut self, txid: &String, payload: &String, rtm: &String, reciever: &String, change_utxo: &String, mint_amount: &u64)-> Result<(String, u64, bool), String>{
    let mut right_to_mint = match self.right_to_mint.clone() {
        Some(right_to_mint) => right_to_mint,
        None => return Err("right_to_mint: no rights to mint for contract".to_string()),
    };

    let drips = match self.drips.clone() {
        Some(drips) => drips,
        None => HashMap::new(),
    };
    
    let rights_amount = match right_to_mint.get(rtm){
        Some(rights_amount) => rights_amount.clone(),
        None => return Err("right_to_mint: rights not found".to_string()),
    };

    let change: u64 = rights_amount - mint_amount;
    let mut amount_to_mint = rights_amount;
    if change  > 0 {
        amount_to_mint =  *mint_amount;
        right_to_mint.insert(change_utxo.to_string(), change);
    }

    let mut new_owner: (String, u64, bool) = (reciever.to_string(), 0, false);
    match drips.get(reciever){
        Some(_) => new_owner.2 = true,
        None => new_owner.2 = false,
    };
    match self.owners.get(reciever) {
        Some(&e) => {
            self.owners.insert(reciever.clone(), &e + amount_to_mint);
            new_owner.1 = &e + amount_to_mint;
        }
        None => {
            self.owners.insert(reciever.clone(), amount_to_mint);
            new_owner.1 = amount_to_mint;
        }
    }

    self.supply += amount_to_mint;
    right_to_mint.remove(rtm);
    self.right_to_mint = Some(right_to_mint);
    self.payloads.insert(txid.to_string(), payload.to_string());
    return Ok(new_owner)
}
```

---
