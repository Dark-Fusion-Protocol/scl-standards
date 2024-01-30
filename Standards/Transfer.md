# Transfer

The **Transfer** standard allows the sending of SCL tokens. This can be done either through a single transfer command or through a batched transfer command (where multiple tokens can be sent at the cost of one transaction fee).

---

# Create Transfer

## Commands

```[UTXO_SENDER_1, UTXO_SENDER_2, ... , UTXO_SENDER_N]``` is the list of UTXOs which send the tokens.  
```[UTXO_RECEIVER_1(AMOUNT), UTXO_RECEIVER_2(AMOUNT), ... , CHANGE_UTXO(AMOUNT)]``` is the list of UTXOs which receive the tokens. The CHANGE_UTXO is the last UTXO.  

### Single Transfer
```
<TXID,
    {CONTRACT_ID_1:TRANSFER[
        [UTXO_SENDER_1, UTXO_SENDER_2, ... , UTXO_SENDER_N],
        [UTXO_RECEIVER_1(AMOUNT), UTXO_RECEIVER_2(AMOUNT), ... , CHANGE_UTXO(AMOUNT)]]}>
```
### Batched Transfer
```
<TXID,
    {CONTRACT_ID_1:TRANSFER[
        [UTXO_SENDER_1, UTXO_SENDER_2, ... , UTXO_SENDER_N],
        [UTXO_RECEIVER_1(AMOUNT), UTXO_RECEIVER_2(AMOUNT), ... , CHANGE_UTXO(AMOUNT)]]}>
    {CONTRACT_ID_2:TRANSFER[
        [UTXO_SENDER_1, UTXO_SENDER_2, ... , UTXO_SENDER_N],
        [UTXO_RECEIVER_1(AMOUNT), UTXO_RECEIVER_2(AMOUNT), ... , CHANGE_UTXO(AMOUNT)]]},
    ...,
    {CONTRACT_ID_N:TRANSFER[
        [UTXO_SENDER_1, UTXO_SENDER_2, ... , UTXO_SENDER_N],
        [UTXO_RECEIVER_1(AMOUNT), UTXO_RECEIVER_2(AMOUNT), ... , CHANGE_UTXO(AMOUNT)]]}>
```
x
## Examples

### Single Transfer
```
<df8125d5302213844221fc97660e7b538f55155a48577f33b09a86d2808ca848,
    {50b7fc619f858f5bf7d12b392a2b26489ef11d4d62c3d182d0754f3df9bbebf4:TRANSFER[
        [494afb28cd5de10ab76394ccaf18f6f2ee7e4fe0d8c19ad04fc4b40e63f7be92:23,a3e71f146c17db792d12e24a25957a0fe7baebfc40b0bf3a9bb1d52f8ce9844b:23],
        [TXID:0(42000000000),TXID:2(818000000000)]]}
```
### Batched Transfer
> [!NOTE]
> Coming soon!

## Implementations

### Rust
```rust
pub fn transfer(&mut self, txid: &String, payload: &String, sender_utxos: &Vec<String>, receivers: &Vec<(String, u64)>, current_block_height: u64) -> Result<(Vec<bool>, u64), String> {
    let mut owners_amount: u64 = 0;
    for sender_utxo in sender_utxos.clone() {
        if self.owners.contains_key(&sender_utxo) {
            owners_amount += self.owners[&sender_utxo];
        }
    }
    if owners_amount == 0 {
        return Err("transfer | owner amount is zero".to_string());
    }

    let mut total_value: u64 = 0;
    for entry in receivers.clone() {
        total_value += entry.1;
    }

    let mut drips = match self.drips.clone() {
        Some(drips) => drips,
        None => HashMap::new(),
    };

    if total_value <= owners_amount {
        let mut new_drips: Vec<Drip> = Vec::new();
        for sender_utxo in sender_utxos.clone() {
            if self.owners.contains_key(&sender_utxo.clone()) {
                self.owners.remove(&sender_utxo);
                if let Some(old_drips) = drips.get(&sender_utxo) {
                    for drip in old_drips {
                        let new_drip = Drip {
                            block_end: drip.block_end.clone(),
                            drip_amount: drip.drip_amount.clone(),
                            amount: drip.amount.clone() - (current_block_height - drip.start_block) * drip.drip_amount,
                            start_block: current_block_height,
                            last_block_dripped: current_block_height
                        };

                        new_drips.push(new_drip.clone());
                    }
                    //remove the old drip from the vector
                    drips.remove(&sender_utxo);
                }
            }
        }

        let last_index = receivers.len() - 1;
        drips.insert(receivers[last_index].0.clone(),new_drips);

        let mut recievers_drips_present: Vec<bool> = Vec::new(); 
        let mut drip_ret = 0;
        for entry in receivers.clone() {
            match self.owners.get(&entry.0) {
                Some(&e) => self.owners.insert(entry.0.clone(), &e + entry.1),
                None => self.owners.insert(entry.0.clone(), entry.1)
            };

            if drips.contains_key(&entry.0) {
                let blocks_dripped = owners_amount - total_value;
                match self.owners.get(&entry.0) {
                    Some(&e) => {
                        self.owners.insert(entry.0.clone(), &e + blocks_dripped);
                        drip_ret = &e + blocks_dripped;
                    },
                    None => {
                        self.owners.insert(entry.0.clone(), entry.1);
                        drip_ret = entry.1;
                    }
                };

                recievers_drips_present.push(true);
            }else{
                recievers_drips_present.push(false);
            }
        }

        self.payloads.insert(txid.to_string(), payload.to_string());
        self.drips = Some(drips);
        return Ok((recievers_drips_present, drip_ret));
    } else{
        return Err("transfer | owner amount is less than recievers total".to_string());
    }
}
```

---