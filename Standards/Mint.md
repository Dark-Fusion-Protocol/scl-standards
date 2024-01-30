# Mint

---

## Commands

### Mint SCL01

```TICKER``` is the symbol representing the token.  
```MAX_SUPPLY``` is the max supply of the token.  
```DECIMALS``` is how many decimal places the token gets represented as.  
```RECEIVER_UTXO``` is the UTXO the tokens get bound to.  

```
<MINT,
    {SCL01:[TICKER, MAX_SUPPLY, DECIMALS, RECEIVER_UTXO]}>
```

### Mint SCL02

```TICKER``` is the symbol representing the token.  
```MAX_SUPPLY``` is the max supply of the token.  
```AIRDROP_AMOUNT``` is the quantity of tokens per airdrop.  
```DECIMALS``` is how many decimal places the token gets represented as.  

```
<MINT,
    {SCL02:[TICKER, MAX_SUPPLY, AIRDROP_AMOUNT, DECIMALS]}>
```

### Mint SCL03

```TICKER``` is the symbol representing the token.  
```DECIMALS``` is how many decimal places the token gets represented as.  
```[TXID:1(AMOUNT), TXID:2(AMOUNT),..., TXID:N(AMOUNT)]``` is a list of UTXOs the AMOUNT gets bound to.  

```
<MINT,
    {SCL03:[TICKER, DECIMALS, [TXID:1(AMOUNT), TXID:2(AMOUNT),..., TXID:N(AMOUNT)]]}>
```

## Examples

### Mint SCL01

```
<5e73fe53cf33648755899f9f8252d969d7d36808f7339f8fe67033b1430f9987,
    {SCL01:[SMOL,31415900000000,8,TXID:0]}>
```

### Mint SCL02

```
<0eebc6baa4463e8f0f9f8cf31440ed12e9f312671b334de97e338380ee1f5cc1,
    {SCL02:[ELON,42069420690,420690,8}>
```

### Mint SCL03

```
<adc110d77942e3e4ca9f2c585249a8d3b82d5b5a82563a0a01e9c591837d99d5,
    {SCL03:[DFG,8,[TXID:0(10000000000000000)]]}
```

## Implementations

### Rust
```rust

```

---