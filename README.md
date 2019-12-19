# The ZRC-3 Metatransaction-enabled Fungible Token Contract

This reference implementation extends [FungibleToken.scilla](https://github.com/Zilliqa/ZRC/blob/GreyEcologist-Fungible-Contract/reference/FungibleToken.scilla) according to the reference found [here](https://github.com/starling-foundries/ZRC/blob/master/zrcs/zrc-3.md).

## Design
This contract is based largely on the recommendations of [EIP-965](https://github.com/ethereum/EIPs/issues/965). It accomplishes similar goals, but the `teller` role completely replaces `operator`, as the `operator` pattern is rife for abuse. A check is a signed transaction containing these parameters:
```
{
	"recipient": "0x00000000...",
       "amount": 100,
     "contract": "0x12395345...",
          "fee": 2,
 "accountNonce": 12,
    "signature": "asdlj3io2j..."
  
}
```
This transaction authorizes a teller to submit this check on their behalf. 