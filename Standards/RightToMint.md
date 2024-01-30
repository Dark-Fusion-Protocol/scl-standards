# Right to Mint

Right to mint can only be awarded during a minting event.

---

# Create Right to Mint

## Commands

```RIGHTS_UTXO``` is the UTXO that has the right to mint.  
```UTXO_RECEIVER``` is the UTXO which binds the mint amount.  

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
<005974de174aa8319ee829fafe25b5e43848ce3965970b228a7b64c8a5e9b522,
    {adc110d77942e3e4ca9f2c585249a8d3b82d5b5a82563a0a01e9c591837d99d5:RIGHTTOMINT[
        adc110d77942e3e4ca9f2c585249a8d3b82d5b5a82563a0a01e9c591837d99d5:0,
        TXID:0]}
```

## Implementations

### Rust
```rust
pub fn right_to_mint(&mut self, txid: &String, payload: &String, rtm: &String, reciever: &String)-> Result<(String, u64, bool), String>{
    let mut right_to_mint = match self.right_to_mint.clone() {
        Some(right_to_mint) => right_to_mint,
        None => return Err("right to mint | no rights to mint for contract".to_string()),
    };

    let drips = match self.drips.clone() {
        Some(drips) => drips,
        None => HashMap::new(),
    };
    
    let rights_amount = match right_to_mint.get(rtm){
        Some(rights_amount) => rights_amount,
        None => return Err("right to mint | rights not found".to_string()),
    };

    let mut new_owner: (String, u64, bool) = (reciever.to_string(), 0, false);
    match drips.get(reciever){
        Some(_) => new_owner.2 = true,
        None => new_owner.2 = false,
    };
    match self.owners.get(reciever) {
        Some(&e) => {
            self.owners.insert(reciever.clone(), &e + rights_amount);
            new_owner.1 = &e + rights_amount;
        }
        None => {
            self.owners.insert(reciever.clone(), *rights_amount);
            new_owner.1 = *rights_amount;
        }
    }

    self.supply += *rights_amount;
    right_to_mint.remove(rtm);
    self.right_to_mint = Some(right_to_mint);
    self.payloads.insert(txid.to_string(), payload.to_string());
    return Ok(new_owner)
}
```

---
