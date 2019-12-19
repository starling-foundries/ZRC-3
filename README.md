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
This transaction authorizes a teller to submit this check on their behalf. Note: the addresses must all be converted to 0x format before signing to be valid.


## Teller responsibilities
The teller provides a public endpoint for recieving metatransactions, checking it's target contract and looking up nonces within that contract. It may optionally report a minimum fee, but it cannot charge a fee that the token owner did not approve in the signed metatransaction. The anticipated malevolent behavior - a teller censoring transactions - is mitigated by the possibility of enabling multiple tellers. Tellers are an abstraction over signing accounts, so one functional 'authority' might choose to run multiple tellers with different fee structures or properties. 
It should be noted that at large volume, the reliance on a signing account (single `_sender`) will effectively de-shard the contract. to avoid this, a radix sort of transactions could be applied to enable multiple 'teller' accounts to coordinate as one authority and re-shard the transaction stream, shared over something like redis. Note that the metatransaction does not include a hash, this is by design - to force the teller to validate that the sender/signer pair corresponds to the data recieved to avoid both malicious and malformed transactions from being processed by the chain.  


## Nonce behavior
The nonce is separate from your overall zilliqa account nonce, it is tracked in-contract and per-token to ensure that tranfers are both atomic and ordered whether or not they were processed directly or via a teller. In order to make the nonce simple and consistent, the normal transfer remains backwards compatible with the ERC20, ZRC2 signature - a non-check transaction doesn't need to include it's nonce. Instead, this class of transaction gains precedence, meaning that normal transfers clear immediately and always get the `transaction_nonce = current_nonce + 1` priority. 

This encourages tellers to resolve checks ASAP, and forces check recipients to wait until their check clears to be sure, but it allows for a user to supercede a misbehaving, nonresponsive or censoring teller without losing funds.

## Modifying for subscriptions

To enable subscriptions or perhaps periodic donations, you only need to add two fields to the metatransaction: block_enabled, block_expired. These fields allow you to process an off-chain hashed timelock transaction, but you would have to either register the nonce or keep the whole unprocessed transaction in an on-chain queue to work well with the nonce pattern. The nonce pattern could be removed if this was the teller's only purpose and no regular metatransactions will be processed.