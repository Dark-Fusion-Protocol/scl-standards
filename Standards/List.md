# List

---

# Create Listing

A listing can be created with this command.

> [!IMPORTANT]
> You cannot create multiple listings with the same contract ID in one command. Only the first listing will be processed.

## Commands

> [!NOTE]
> An order ID is created and the order ID is always the first UTXO that is spent.

```[UTXO_SENDER_1, UTXO_SENDER_2, ... ,UTXO_SENDER_N]``` is the list of UTXOs containing the tokens for the listing.  
```CHANGE_UTXO``` is the UTXO the change gets bound to.  
```LISTING_UTXO``` is the UTXO the listing gets bound to.  
```LISTING_AMOUNT``` is the quantity of the token being listed.  
```SELL_PRICE``` is price per token of the token being lsited.  
```PAY_ADDRESS``` is the address the BTC payment must go to (usually the lister's address).  

### Single List
```
<TXID, 
    {CONTRACT_ID_1:LIST[
        [UTXO_SENDER_1, UTXO_SENDER_2, ... ,UTXO_SENDER_N],
        CHANGE_UTXO,
        LISTING_UTXO,
        LISTING_AMOUNT,
        SELL_PRICE,
        PAY_ADDRESS]}>
```
### Batched List
```
<TXID, 
    {CONTRACT_ID_1:LIST[
        [UTXO_SENDER_1, UTXO_SENDER_2, ... ,UTXO_SENDER_N],
        CHANGE_UTXO,
        LISTING_UTXO,
        LISTING_AMOUNT,
        SELL_PRICE,
        PAY_ADDRESS]},
    {CONTRACT_ID_2:LIST[
        [UTXO_SENDER_1, UTXO_SENDER_2, ... ,UTXO_SENDER_N],
        CHANGE_UTXO,
        LISTING_UTXO,
        LISTING_AMOUNT,
        SELL_PRICE,
        PAY_ADDRESS]},
    ...,
    {CONTRACT_ID_N:LIST
        [UTXO_SENDER_1, UTXO_SENDER_2, ... ,UTXO_SENDER_N],
        CHANGE_UTXO,
        LISTING_UTXO,
        LISTING_AMOUNT,
        SELL_PRICE,
        PAY_ADDRESS]}>
```

## Examples

### Single List
```
<8380c54d943ec72857b3022da9906e6f7da5bb8220ab03ed7b5b0a724144c5f2,
    {3b1b20518485ec89ce9acf5bb23c5ccdb0ac26d0661e377014e894d295eec29e:LIST[
        [9859b9e8d3d1a343819f22876c74fc3b304438083f005378f7e64085c7c5bf27:0,55c9d410e1c0fa7ff62efbad05361da919f7c77b7ef14590e6db5d4a1954a698:0],
        TXID:0,
        TXID:1,
        100000000,
        10000,
        tb1ql2qy8ecqdtfdd0np54c75vzlymkqykfc4uaa9h]}>
```
### Batched List
```
<07de2d16fb4a0dc653f1667f52ebc05da24ada755bc68cd7c44bfefef4b95325,
    {0eebc6baa4463e8f0f9f8cf31440ed12e9f312671b334de97e338380ee1f5cc1:LIST[
        [a3e71f146c17db792d12e24a25957a0fe7baebfc40b0bf3a9bb1d52f8ce9844b:6],
        TXID:0,
        TXID:1,
        9462500000000,
        2,
        tb1q3aryc8esjula2m87k2gr3fmme8mjnmwulx8lm4]},
    {6a6787f011052aca7955a2c3c90e836f94f80d0918ba5b9b57e85ec3676857d7:LIST[
        [494afb28cd5de10ab76394ccaf18f6f2ee7e4fe0d8c19ad04fc4b40e63f7be92:1],
        TXID:2,
        TXID:3,
        50000000000,
        2000,
        tb1q3aryc8esjula2m87k2gr3fmme8mjnmwulx8lm4]}>
```

## Implementations

### Rust
```rust
pub fn list(&mut self, txid: &String, payload: &String, sender_utxos: &Vec<String>, new_listing: Listing, current_block_height:u64) -> Result<(String,u64,bool), String> {
    let mut owners_amount: u64 = 0;
    for sender_utxo in sender_utxos.clone() {
        if self.owners.contains_key(&sender_utxo) {
            owners_amount += self.owners[&sender_utxo];
        }
    }
    if owners_amount == 0 {
        return Err("list: owner amount is zero".to_string());
    }

    if sender_utxos.len() == 0 {
        return Err("list: no senders".to_string());
    }

    let mut new_owner = (new_listing.change_utxo.to_string(),0,false);

    if new_listing.list_amt <= owners_amount {
        let mut drips = match self.drips.clone() {
            Some(drips) => drips,
            None => HashMap::new(),
        };

        for sender_utxo in sender_utxos.clone() {
            if self.owners.contains_key(&sender_utxo) {
                self.owners.remove(&sender_utxo);
            }

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

                drips.insert(new_listing.change_utxo.clone(),new_drips);
                drips.remove(&sender_utxo);
                new_owner.2 = true;
            }
        }

        let change_amt: u64 = owners_amount - new_listing.list_amt;
        if change_amt > 0 {
            if self.owners.contains_key(&new_listing.change_utxo) {
                let new_amount = self.owners[&new_listing.change_utxo] + change_amt;
                new_owner.1 = new_amount.clone();
                self.owners.insert(new_listing.change_utxo.to_string(), new_amount);
            } else {
                new_owner.1 = change_amt.clone();
                self.owners.insert(new_listing.change_utxo.to_string(), change_amt);
            }
        }
        
        let order_id: String = sender_utxos[0].to_string();
        let mut listing = match self.listings.clone() {
            Some(listings) => listings,
            None => HashMap::new(),
        };

        listing.insert(order_id, new_listing.clone());
        self.listings = Some(listing);
        self.drips = Some(drips);
        self.payloads.insert(txid.to_string(), payload.to_string());
    }
    return Ok(new_owner);
}
```

---

# Cancel Listing

A listing can be cancelled with this command.

## Commands

```LISTING_UTXO``` is the UTXO the listing is bound to.  

```
<TXID, 
    {CONTRACT_ID_1:CANCELLISTING[
        LISTING_UTXO]}>
```

## Examples

```
<c6dee0b8c41255ddd4faba2916b37797a5c0632ba56fa597c72471fa8fde39f3,
    {05874c81d5f3b83880d112178d09dbe2845148d256d4183936229547d8497505:CANCELLISTING[
        c894c5c8ea21bfe8c9aa0d9295db235ad74ec307ed4a493c06dd0393288608f7:1]}>
```

## Implementations

### Rust
```rust
pub fn cancel_listing(&mut self, txid: &String, listing_utxo: &String,  payload: String) -> Result<i32, String> {
    let mut bids_available = match self.bids.clone() {
        Some(bids_available) => bids_available,
        None => HashMap::new(),
    };

    let fulfillments = match self.fulfillments.clone() {
        Some(fulfillments) => fulfillments,
        None => HashMap::new(),
    };

    let mut listings = match self.listings.clone() {
        Some(listings) => listings,
        None => return Err("cancel_listing: no listings for contract".to_string()),
    };

    let mut canceled_listing = Listing::default();
    let mut order_id = String::new();
    for (key, value) in listings.clone() {
        if value.list_utxo == listing_utxo.to_string() {
            canceled_listing = value.clone();
            order_id = key.clone();
            break;
        }
    }

    for (_, value) in fulfillments {
        if value == order_id{
            return Err("cancel_listing: order has been fulfilled".to_string());
        }
    }

    let recievers_utxo: String = format!("{}:0",txid);
    if self.owners.contains_key(&recievers_utxo) {
        let new_amount = self.owners[&recievers_utxo] + canceled_listing.list_amt;
        self.owners.insert(recievers_utxo.to_string(), new_amount);
    } else {
        self.owners.insert(recievers_utxo.to_string(), canceled_listing.list_amt);
    }

    listings.remove(&order_id);

    for (key, value) in bids_available.clone().iter_mut() {
        if value.order_id == order_id.to_string() {
            bids_available.remove(key);
        }
    }

    self.bids = Some(bids_available.clone());
    self.listings = Some(listings.clone());

    self.payloads.insert(txid.to_string(), payload);
    return Ok(0);
}
```

---