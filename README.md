#TRC20 Token

###Simple Summary

A standard interface for tokens in TON (like ERC20 in Etherium).

###Abstract

The following standard allows for the implementation of a standard API for tokens within smart contracts. This standard provides basic functionality to transfer tokens, as well as allow tokens to be approved so they can be spent by another on-chain third party.

###Motivation

A standard interface allows any tokens on TON to be re-used by other applications: from wallets to decentralized exchanges.


TON supports build-in extra currencies.

According to the current documentation, extra currencies are distinguished by ID and total supply.

There is no mention that the extra currency is based on its own smart contract. In this case, TRC20 makes sense, because many consumers would like to have additional token functionality.

TON also supports the creation of new workchains. But according to the available documentation, creating your own workchain is time-consuming, in contrast to creating a new TRC20 token.

###Specification

###Get-methods:

###get_name

Returns the name of the token - e.g. "MyToken".

OPTIONAL - This method can be used to improve usability, but interfaces and other contracts MUST NOT expect these values to be present.

`slice get_name()`

###get_symbol

Returns the symbol of the token. E.g. “HIX”.

OPTIONAL - This method can be used to improve usability, but interfaces and other contracts MUST NOT expect these values to be present.

`slice get_symbol()`

###get_decimals

Returns the number of decimals the token uses - e.g. 8, means to divide the token amount by 100000000 to get its user representation.

OPTIONAL - This method can be used to improve usability, but interfaces and other contracts MUST NOT expect these values to be present.

`int get_decimals()`

###get_total_supply

Returns the total token supply.

`int get_total_supply()`

###balance_of

Returns the account balance of another account with address 'owner'.

`int balance_of(slice owner)`

###allowance

Returns the amount which 'spender' is still allowed to withdraw from 'owner'.

`int allowance(slice owner, slice spender)`

###Internal messages 

Following the [TON smartcontracts guidelines](https://test.ton.org/smguidelines.txt) all internal messages should start with 32 uint operation_id and 64 uint query_id.

If the query is invalid, the smart contract should throw an exception.

After successful completion of the query, the smart contract should respond with a message 0x80000000

`<b 0x80000000 32 u, query_id 64 u, query_op 32 u, b>`

If operation is unsupported, the smart contract should respond with a message 0xffffffff.

###transfer 

`op = 1`

`<b 1 32 u, query_id 64 u, to_addr addr, value Gram, b>`

Transfers 'value' amount of tokens to address 'to_addr'. The function SHOULD throw if the message caller’s account balance does not have enough tokens to spend.

Note Transfers of 0 values MUST be treated as normal transfers.


###transfer_from

`op = 2`

`<b 2 32 u, query_id 64 u, from_addr addr, to_addr addr, value Gram, b>`

Transfers 'value' amount of tokens from address 'from_addr' to address 'to_addr'.

The transfer_from method is used for a withdraw workflow, allowing contracts to transfer tokens on your behalf. This can be used for example to allow a contract to transfer tokens on your behalf and/or to charge fees in sub-currencies. The function SHOULD throw unless the 'from' account has deliberately authorized the sender of the message via some mechanism.

Note Transfers of 0 values MUST be treated as normal transfers.

###approve

`op = 3`

`<b 3 32 u, query_id 64 u, spender_addr addr, value Gram, b>`

Allows 'spender' to withdraw from your account multiple times, up to the 'value' amount. If this function is called again it overwrites the current allowance with 'value'.

NOTE: To prevent attack vectors like the one [described here](https://docs.google.com/document/d/1YLPtQxZu1UAvO9cZ1O2RPXBbT0mooh4DYKjA_jp-RLM/), clients SHOULD make sure to create user interfaces in such a way that they set the allowance first to 0 before setting it to another value for the same spender. THOUGH The contract itself shouldn’t enforce it.


###Implementation

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

This implementation works with addresses in the following format: 8bit int wc , 256bit uint addr.

For convenience, you can create queries by sending simple messages with text comment to token smart contract.
In this case, the comment text should contain the hex of the message body (produced by 'csr.' fift command).

