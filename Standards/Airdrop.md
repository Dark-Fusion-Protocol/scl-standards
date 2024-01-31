# Airdrop

> [!NOTE]
> These airdrops are specific to a contract that does airdrops.

---

# Claim Airdrop

## Commands

```UTXO_RECEIVER_1``` is the UTXO the airdrop will be bound to.  

### Single Airdrop
```
<TXID, 
	{CONTRACT_ID_1:AIRDROP[
        UTXO_RECEIVER_1]}>
```
### Batched Airdrop
```
<TXID, 
	{CONTRACT_ID_1:AIRDROP[
        UTXO_RECEIVER_1]},
	{CONTRACT_ID_2:AIRDROP[
        UTXO_RECEIVER_1]},
	...,
	{CONTRACT_ID_N:AIRDROP[
        UTXO_RECEIVER_1]}>
```

## Examples

### Single Airdrop
```
<04affa741549ea4f9f2f331a36f83906b558862bf2e70bef833ef1f8e14bc3f2,
    {d4ef02ac55ac30b973caf8e468028734a71665f931e95295fcd02d07b287cd00:AIRDROP[
        TXID:0]}>
```
### Batched Airdrop
```
<410cf1c90521ccb0dcae689d4c4f0465410a29e4cd00ac05c27696b08b8bbef3,
    {d4ef02ac55ac30b973caf8e468028734a71665f931e95295fcd02d07b287cd00:AIRDROP[
        TXID:0]},
    {6a6787f011052aca7955a2c3c90e836f94f80d0918ba5b9b57e85ec3676857d7:AIRDROP[
        TXID:1]},
    {edd71b97449e993e07bcc451fd0bf76c90d9b7ac0a031930ed2a79fef5cd4a22:AIRDROP[
        TXID:2]},
    {05874c81d5f3b83880d112178d09dbe2845148d256d4183936229547d8497505:AIRDROP[
        TXID:3]},
    {eca67b99f1a32f1be4ca66bfb38d453aed73e660b805256fd98a303fd69e50fe:AIRDROP[
        TXID:4]},
    {021725919e91ee5422a9fd7f3e21c88e7abe802978e9584abd817244a5a11560:AIRDROP[
        TXID:5]},
    {0eebc6baa4463e8f0f9f8cf31440ed12e9f312671b334de97e338380ee1f5cc1:AIRDROP[
        TXID:6]},
    {84ca055f9c7aa69d63241d3853b0dc701feeaba4def4acb8d2db5358dfc9cdf0:AIRDROP[
        TXID:7]},
    {38d40dfbcc39069c5b9dd7225ecfa9ef2140ebd2dca962328c30f922d74f776e:AIRDROP[
        TXID:8]},
    {c0e011fd493874be949e801363df02b59c345a599710d0d8c76db91975c33ad1:AIRDROP[
        TXID:9]},
    {cd0c4866d59e422d2e34b5e9653020b35752c5ae649c0907c02732b7e02e214e:AIRDROP[
        TXID:10]},
    {4571c9572ed3500b9fe680a504dfb6c32397221427eefd5d8337ab5fe7087aab:AIRDROP[
        TXID:11]},
    {b7e8d09296164c1b8cc641902f18b4535211e18426c0ca20ddcbfaf1a250c70c:AIRDROP[
        TXID:12]},
    {d4b57d034b5c5093860385b36a14334649768fad07a039930bd1860d14a63da0:AIRDROP[
        TXID:13]},
    {8d3c6ef59956d6205cb61666500797e3c9876388f68002b613094ded7f83bfb2:AIRDROP[
        TXID:14]},
    {78be7725c4afa49c07e121b54c3691094e0dbd07bbf63f4f6cb6bd913c42be73:AIRDROP[
        TXID:15]},
    {bcf68a4029f4b58edce3678883b692d1fcbf0da2866a3ce5451e355cd7f098d2:AIRDROP[
        TXID:16]},
    {d376e2c28288642deb311cd445b0ed3f6a6813e0377b0de8814a4aaf2e0025ba:AIRDROP[
        TXID:17]},
    {75f0d0ab533400eaf9f96d2c417073f71d77d775bfd1b7929bdcea897f08c88e:AIRDROP[
        TXID:18]},
    {676a24bed8a03c8ec0a6124c183012c4eceb896d0600ddf7609f7030f57a56df:AIRDROP[
        TXID:19]},
    {ce6bf8edf41f43cc3676c0f431aa07db45cf8d4efb1ec0d5c785e092400cca85:AIRDROP[
        TXID:20]},
    {166bdeea01c49f6eeef2e65b4770afb79a2e5dc8617b9e087fe31c19c1029635:AIRDROP[
        TXID:21]},
    {3adfaad7f2e137974df201bd1f83272e834b484b77178bce5ef1a957ea0dd171:AIRDROP[
        TXID:22]},
    {50b7fc619f858f5bf7d12b392a2b26489ef11d4d62c3d182d0754f3df9bbebf4:AIRDROP[
        TXID:23]},
    {65d61c39c2516beb16e61c782009bf59c64ee45321b4056336ee435f9f25dfe9:AIRDROP[
        TXID:24]},
    {db719fdf3009f68daff1b4b8a0f3f698dde602c9ef73e8292afe8f71f0c46c8d:AIRDROP[
        TXID:25]}>
```

## Implementations

### Rust
```rust
pub fn airdop(&mut self, txid: &String, payload: &String, receiver: &String, pending: bool) -> Result<u64, String> {
    let current_airdrops = match self.current_airdrops.clone() {
        Some(current_airdrops) => current_airdrops,
        None =>  return  Err("airdop: no airdrops".to_string()),
    };

    let airdrop_amount = match self.airdrop_amount.clone() {
        Some(airdrop_amount) => airdrop_amount,
        None =>  return  Err("airdop: no airdrops".to_string()),
    };

    let total_airdrops = match self.total_airdrops.clone() {
        Some(total_airdrops) => total_airdrops,
        None =>  return  Err("airdop: no airdrops".to_string()),
    };

    if current_airdrops >= total_airdrops {
        return Err("airdop: contract has reached max supply".to_string());
    }

    let mut owner_amount = airdrop_amount;

    if current_airdrops + 1 == total_airdrops {
        let mut last_airdrop_split = match self.last_airdrop_split.clone() {
            Some(last_airdrop_split) => last_airdrop_split,
            None =>  Vec::new()
        };

        last_airdrop_split.push(receiver.to_string());
        self.last_airdrop_split = Some(last_airdrop_split);
        self.payloads.insert(txid.to_string(), payload.to_string());
        return Ok(owner_amount);
    }

    let mut p_c = match self.pending_claims.clone() {
            Some(p_c) => p_c,
            None => HashMap::new(),
    };

    if pending {
       p_c.insert(receiver.to_string(), airdrop_amount);
       if let Some(owned) = self.owners.get(receiver){
            owner_amount += owned;    
       }
    } else {
        p_c.remove(receiver);
        match self.owners.get(receiver) {
            Some(&e) => {
                self.owners.insert(receiver.to_string(), &e + airdrop_amount);
                owner_amount += e;               
            }
            None => {
                self.owners.insert(receiver.to_string(), airdrop_amount);
            }
        }
    }
  
    self.current_airdrops = Some(current_airdrops + 1);
    self.pending_claims = Some(p_c);
    self.payloads.insert(txid.to_string(), payload.to_string());
    self.supply += airdrop_amount;
    return Ok(owner_amount);
}
```

---