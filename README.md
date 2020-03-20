# TRC20 Token

### Simple Summary

A standard interface for tokens in TON (like ERC20 in Etherium).

### Abstract

The following standard allows for the implementation of a standard API for tokens within smart contracts. This standard provides basic functionality to transfer tokens, as well as allow tokens to be approved so they can be spent by another on-chain third party.

### Motivation

A standard interface allows any tokens on TON to be re-used by other applications: from wallets to decentralized exchanges.


TON supports build-in extra currencies.

According to the current documentation, extra currencies are distinguished by ID and total supply.

There is no mention that the extra currency is based on its own smart contract. In this case, TRC20 makes sense, because many consumers would like to have additional token functionality.

TON also supports the creation of new workchains. But according to the available documentation, creating your own workchain is time-consuming, in contrast to creating a new TRC20 token.

### Differences from ERC20

TRC20 saves the ERC20 interface. 

At the same time TRC20 corrects ERC20 weaknesses:

* Race-prone approve function.

* The token recipient is not notified about the transfer.

* You can not add additional data to the transfer.

* If you send a token to a smart contract that does not support tokens, then you will lose them.

* The token sending style is different from the blockchain transaction sending style.

TRC20 corrects all these flaws, more in the following paragraphs.

### Specification

### Get-methods:

### get_name

Returns the name of the token - e.g. "MyToken".

`slice get_name()`

### get_symbol

Returns the symbol of the token. E.g. “HIX”.

`slice get_symbol()`

### get_decimals

Returns the number of decimals the token uses - e.g. 8, means to divide the token amount by 100000000 to get its user representation.

`int get_decimals()`

### get_total_supply

Returns the total token supply.

`int get_total_supply()`

### balance_of

Returns the account balance of another account with address 'owner'.

`int balance_of(slice owner)`

### allowance

Returns the amount which 'spender' is still allowed to withdraw from 'owner'.

`int allowance(slice owner, slice spender)`

### Internal messages 

Following the [TON smartcontracts guidelines](https://test.ton.org/smguidelines.txt) all internal messages should start with 32 uint operation_id and 64 uint query_id.

If the query is invalid, the smart contract should throw an exception.

After successful completion of the query, the smart contract should send event message to token recipient.

If operation is unsupported, the smart contract should respond with a message 0xffffffff.

### transfer 

`op = 1`

`<b 1 32 u, query_id 64 u, to_addr addr, value Gram, .. payload  b>`

Transfers 'value' amount of tokens to address 'to_addr'. The function SHOULD throw if the message caller’s account balance does not have enough tokens to spend.

Note Transfers of 0 values MUST be treated as normal transfers.

'payload' is optional extra data.

After completion an event message is sent to 'to_addr'.

`<b 0x20000001 32 u, query_id 64 u, sender addr, value Gram, .. payload b>`

### transfer_from

`op = 2`

`<b 2 32 u, query_id 64 u, from_addr addr, to_addr addr, value Gram, .. payload b>`

Transfers 'value' amount of tokens from address 'from_addr' to address 'to_addr'.

The transfer_from method is used for a withdraw workflow, allowing contracts to transfer tokens on your behalf. This can be used for example to allow a contract to transfer tokens on your behalf and/or to charge fees in sub-currencies. The function SHOULD throw unless the 'from' account has deliberately authorized the sender of the message via some mechanism.

Note Transfers of 0 values MUST be treated as normal transfers.

'payload' is optional extra data.

After completion an event message is sent to 'to_addr'.

`<b 0x20000002 32 u, query_id 64 u, sender addr, value Gram, .. payload b>`

### approve

`op = 3`

`<b 3 32 u, query_id 64 u, spender_addr addr, value Gram, .. payload b>`

Allows 'spender' to withdraw from your account multiple times, up to the 'value' amount. If this function is called again it overwrites the current allowance with 'value'.

NOTE: To prevent attack vectors like the one [described here](https://docs.google.com/document/d/1YLPtQxZu1UAvO9cZ1O2RPXBbT0mooh4DYKjA_jp-RLM/), contract REQUIRES set the allowance first to 0 before setting it to another value for the same spender.

'payload' is optional extra data.

After completion an event message is sent to 'spender_addr'.

`<b 0x20000003 32 u, query_id 64 u, sender addr, new_allowance Gram, .. payload b>`

### increase/decrease allowance

`op = 4`

`<b 4 32 u, query_id 64 u, is_decrease 1 u, spender_addr addr, delta_value Gram, .. payload b>`

Atomically increase/decrease the allowance by 'delta_value', granted to 'spender' by the caller. This is an alternative to 'approve' method.  

'payload' is optional extra data.

After completion an event message is sent to 'spender_addr'.

`<b 0x20000003 32 u, query_id 64 u, sender addr, new_allowance Gram, .. payload b>`

### Rollback

If the transfer recipient sends a 0xffffffff response (the operation is not supported) or if a bounced message arrives (an error occurred on the recipient side): 

* If message was a transfer: transfer must be returned (equivalent of transfer from receiver to sender).

* If message was a transfer_from: sent tokens must be approved in favor sender (equivalent receiver approves tokens in favor sender).

NOTE: the response must contain the body of the original message.

This prevents the loss of tokens sent to a contract that does not support them.

### Implementation

This repository contains basic TRC20 token smart contract and scripts.

---

token.fc - TRC20 basic token implementation.

compilation: `func -P -o token.fif stdlib.fc msg_hex_comment.fc token.fc`

---

new.fif - Creates a new basic TRC20 token in specified workchain.

`fift -s new.fif 0 DOGE 🐶 18 100000000000000000000000000 owner.addr`

---

transfer.fif - Creates a transfer message body.

`fift -s transfer.fif to.addr 0.1`

---

approve.fif - Creates an approve message body.

`fift -s approve.fif spender.addr 77.7`

---

transfer_from.fif - Creates a transfer_from message body.

`fift -s transfer_from.fif owner.addr spender.addr 77.7`

---

increase-allowance.fif - Creates an increase allowance message body.

`fift -s increase-allowance.fif owner.addr spender.addr 77.7`

---

decrease-allowance.fif - Creates a decrease allowance message body.

`fift -s decrease-allowance.fif owner.addr spender.addr 77.7`

---

This implementation works with addresses in the following format: 8bit int wc , 256bit uint addr.

For convenience, you can create queries by sending simple messages with text comment to token smart contract.
In this case, the comment text should contain the hex of the message body (produced by 'csr.' fift command).

## Discussion

Zero Address

Some ERC20 implementations (e.g. OpenZeppelin) require that the destination address not be 0, because "As @zmitton suggested, keeping the require(to != address(0)) in place will prevent bad software that cause empty arg or improper address parsing from sending the funds into the void" [(link)](https://github.com/OpenZeppelin/openzeppelin-contracts/issues/1493).

I believe that bad software should be fixed, instead of adding checks to the contract.

## References

ERC-20

ERC-223

ERC-777

EIP-677 

EIP-827

EIP-1376