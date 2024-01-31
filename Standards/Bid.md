# Bid

---

# Create Bid

A bid can be created with this command.

## Commands

```ACCEPT_TX``` is an unsigned raw transaction in hex spending the listing UTXO, sending it to the bidder.  
```FULFIL_TX``` is a signed raw transaction in hex that fulfills the trade, paying the BTC to the lister and paying the tokens to the bidder.  
```ORDER_ID_1``` is the UTXO the listing is bound to.  
```PURCHASE_AMOUNT_1``` is the quantity of the token being bid on.  
```PURCHASE_PRICE_1``` is price per token of the token being bid on.  
```RES_UTXO_1``` is the UTXO where the tokens are bound to.  

### Single Bid
```
<TXID, 
    ACCEPT_TX,
    FULFIL_TX, 
    {CONTRACT_ID_1:BID[
        [ORDER_ID_1, PURCHASE_AMOUNT_1, PURCHASE_PRICE_1, RES_UTXO_1],
        [ORDER_ID_2, PURCHASE_AMOUNT_2, PURCHASE_PRICE_2, RES_UTXO_2],
        ...,
        [ORDER_ID_N, PURCHASE_AMOUNT_N, PURCHASE_PRICE_N, RES_UTXO_N]]}>
```
### Batched Bid
```
<TXID,
    ACCEPT_TX,
    FULFIL_TX,  
    {CONTRACT_ID_1:BID[
        [ORDER_ID_1, PURCHASE_AMOUNT_1, PURCHASE_PRICE_1, RES_UTXO_1],
        [ORDER_ID_2, PURCHASE_AMOUNT_2, PURCHASE_PRICE_2, RES_UTXO_2],
        ...,
        [ORDER_ID_N, PURCHASE_AMOUNT_N, PURCHASE_PRICE_N, RES_UTXO_N]]},
    {CONTRACT_ID_2:BID[
        [ORDER_ID_1, PURCHASE_AMOUNT_1, PURCHASE_PRICE_1, RES_UTXO_1],
        [ORDER_ID_2, PURCHASE_AMOUNT_2, PURCHASE_PRICE_2, RES_UTXO_2],
        ...,
        [ORDER_ID_N, PURCHASE_AMOUNT_N, PURCHASE_PRICE_N, RES_UTXO_N]]},
    ...,
    {CONTRACT_ID_N:BID[
        [ORDER_ID_1, PURCHASE_AMOUNT_1, PURCHASE_PRICE_1, RES_UTXO_1],
        [ORDER_ID_2, PURCHASE_AMOUNT_2, PURCHASE_PRICE_2, RES_UTXO_2],
        ...,
        [ORDER_ID_N, PURCHASE_AMOUNT_N, PURCHASE_PRICE_N, RES_UTXO_N]]}>
```

## Examples

### Single Bid
```
<456d3dcf23a55636a0f0683c2c8f5e9b53a787857b7ffc11be64c288a564c15d,
    {3b1b20518485ec89ce9acf5bb23c5ccdb0ac26d0661e377014e894d295eec29e:BID[
        [9859b9e8d3d1a343819f22876c74fc3b304438083f005378f7e64085c7c5bf27:0,
        100000000,
        30000,
        TXID:0]]}>
```
### Batched Bid
```
<2947627925e5b3df4c5597c23242f145d129f89bd1ca9b13b68113303f229431,
    02000000010665f30a264ee20cebc431d850802bdf74b02b687a351afa3d00a680121620320100000000ffffffff0226020000000000001600148f464c1f30973fd56cfeb29038a77bc9f729eddc0000000000000000226a20ad6937747b0d5d015f4259c904b4112a7ec821c07ba6cee3960b781173148fcc00000000,
    020000000001023194223f301381b6139bcad19bf829d145f14232c297554cdfb3e525796247290000000000ffffffffdfbc66bfe18c9ad005a2541fb8f4be3e1ded2c62a2cd968c3547f8c8a2b3de8a0000000000ffffffff0426020000000000001600148f464c1f30973fd56cfeb29038a77bc9f729eddcd007000000000000160014192e37f6b6bff95093f11400553355882450ea3a2602000000000000160014192e37f6b6bff95093f11400553355882450ea3a0000000000000000226a2001aebd6d901368e26497230a932132f74fd828c797547067e244ac93db70eef8024730440220279c8fc5057f891129dc82f009605f7e8ac0a19fbcadf216ff24b472f8f5991b022034848bb306668e032a0018e19b4a1dc5231992527c8b9a8b4e2adebd5c5d05a701210230b19ebf19c082e890a32f125d729dcb806995c662dc99c3e9b3be8651d310bf02483045022100dd6d197b843af3fe01b493a521e7d04d53598f5d4a9e6eaaa8cb30c694301cc002202001d69a40e196c732d3027ba425f29d744241961e394b4434c6263f0a35887d01210230b19ebf19c082e890a32f125d729dcb806995c662dc99c3e9b3be8651d310bf00000000,
    {38d40dfbcc39069c5b9dd7225ecfa9ef2140ebd2dca962328c30f922d74f776e:BID[
        [159b63cb83892ee7c5d85b997ec4dc67b4e71b79467f20043a51b3731e06ad42:8,
        100000000,
        2000,
        TXID:0]]},
    {75f0d0ab533400eaf9f96d2c417073f71d77d775bfd1b7929bdcea897f08c88e:BID[
        [9ad0031f5c4fcd208c3329b7b63b48cb8f263b1ab40d2e529857ae73c2e2f2f6:2,
        2500000000,
        100,
        TXID:1]]}>
```

## Implementations

### Rust
```rust
pub fn bid(&mut self, txid: &String, payload: &String, bids: Vec<Bid>, bidding_ids: &Vec<String>, current_block_height: i32) -> Result<i32, String> {
    let mut listings_available = match self.listings.clone() {
        Some(listings_available) => listings_available,
        None => return Err("bid: no listings for contract".to_string()),
    };

    let mut bids_available = match self.bids.clone() {
        Some(bids_available) => bids_available,
        None => HashMap::new(),
    };

    for (i, _) in bids.iter().enumerate() {
        if listings_available.clone().contains_key(&bids[i].order_id) {
            if bids[i].bid_amount > listings_available[&bids[i].order_id].list_amt {
                continue;
            }

            if bids[i].bid_amount/ 10u64.pow(self.decimals as u32)  * bids[i].bid_price  >=  listings_available[&bids[i].order_id].list_amt / 10u64.pow(self.decimals as u32)  * listings_available[&bids[i].order_id].price{
                let mut listing = listings_available[&bids[i].order_id].clone();
                listing.valid_bid_block = Some(current_block_height);
                listings_available.insert(bids[i].order_id.to_string(), listing);
                self.listings = Some(listings_available.clone());
            }
            
            bids_available.insert(bidding_ids[i].to_string(), bids[i].clone());
            self.bids = Some(bids_available.clone());
        }
    }

    self.payloads.insert(txid.to_string(), payload.to_string());
    return Ok(0);
}
```

---

# Cancel Bid

A bid can be cancelled with this command.

## Command

```BID_UTXO``` is the UTXO the Bid is bound to.  

```
<TXID, 
    {CONTRACT_ID_1:CANCELBID[
        BID_UTXO]}>
```

## Examples
```
<243afbb1aa0c3e04f8b165416d3ee02ce175f22dae1b997d7ba06d12e88e388f,
    {9ad0031f5c4fcd208c3329b7b63b48cb8f263b1ab40d2e529857ae73c2e2f2f6:CANCELBID[
        2947627925e5b3df4c5597c23242f145d129f89bd1ca9b13b68113303f229431:1]}>
```

## Implementations

### Rust
```rust
pub fn cancel_bid(&mut self, txid: &String, bidding_utxo: &String, payload: String) -> Result<i32, String> {
    let mut bids_available = match self.bids.clone() {
        Some(bids_available) => bids_available,
        None => return Err("cancel_bid: no bids for contract".to_string()),
    };

    let fulfillments = match self.fulfillments.clone() {
        Some(fulfillments) => fulfillments,
        None => HashMap::new(),
    };

    match self.listings.clone() {
        Some(listings) => listings,
        None => return Err("cancel_bid: no listings for contract".to_string()),
    };

    let mut bid_id = String::new();
    for (key, value) in bids_available.clone() {
        if value.reseved_utxo == bidding_utxo.to_string() {
            bid_id = key.clone();
            break;
        }
    }

    if fulfillments.contains_key(&bid_id) {
        return Err("cancel_bid: order has been fulfilled".to_string());
    }

    bids_available.remove(&bid_id);

    self.bids = Some(bids_available.clone());
    self.payloads.insert(txid.to_string(), payload);
    return Ok(0);
}
```