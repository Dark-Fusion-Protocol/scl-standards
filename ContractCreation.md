# Contract Creation

The final formalisation for method payloads is:
```
<TXID, PAYLOAD> 

<TXID, {CONTRACT_ID_1:COMMAND}{CONTRACT_ID_2:COMMAND}...{CONTRACT_ID_N:COMMAND}>
```

> [!NOTE]
> Mint commands create the contract ID used in the PAYLOAD portion and therefore need to be unique.
```
<MINT, MINT_PAYLOAD>

<MINT, {CONTRACT_TYPE:[PARAM_1, PARAM_2,..., PARAM_N]}>
```

> [!NOTE]
> TXID is self referential to the outputs of the actual transaction. The MINT word becomes the contract ID after minting.