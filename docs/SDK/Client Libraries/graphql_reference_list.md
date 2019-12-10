## General Info

TON Labs utilizes GraphQL as a middleware between databases and other tools.

GraphQL is a query language, manipulation and a runtime designed to build client applications by providing an intuitive and flexible syntax and system for describing their data requirements and interactions.

There are two types of operations in TON LABS GraphQL implemetation:

- query – a read‐only fetch.
- subscription – a long‐lived request that fetches data in response to source events.

Every operation can contain a number of fields. Below you can see an example of a query for 5 random accounts with some additional data requests called fields.

```sql
query {
  accounts(
    limit: 5
  ) {
    id,
    last_trans_lt,
    last_paid
  }
}
```

Here you can see a request for accounts with the fields `id`, `last_trans_lt` `and last_paid` forming a selection set.

A selection set is primarily composed of fields. A field describes one discrete piece of information available to a request within a selection set.

Fields are conceptually functions which return values, and occasionally accept arguments which alter their behavior. In the example above, the argument is 5 - the number of accounts requested.

It modifies the behavior of the field `limit` making it specifically return 5 random accounts.

There maybe multiple arguments for one field.

Arguments may be provided in any syntactic order and maintain identical semantic meaning.

Also check the schema at https://github.com/tonlabs/ton-q-server/blob/master/server/db.schema.v2.js

## TON Labs Schema

A GraphQL service’s collective type system capabilities are referred to as that service’s “schema”. A schema is defined in terms of the types and directives it supports as well as the root operation types for each kind of operation.

TON Labs schema is designed to be flat with no wraps.

It includes the following 8 query types:

1. ExtBlkRef - External block references
2. MsgEnvelope - TON message envelope requests
3. InMsg - internal message status requests
4. OutMsg - outbound message status requests
5. Message- message parameters
6. Block - requests on block status and external references.
7. Account - requests on the status and of the account
8. Transaction - request on the status and parameters of the transactions.

## Account type

Recall that a smart contract and an account are the same thing in the context of the TON Blockchain, and that these terms can be used interchangeably, at least as long as only small (or “usual”) smart contracts are considered. A large smart-contract may employ several accounts lying in different shardchains of the same workchain for load balancing purposes.

An account is identified by its full address and is completely described by its state. In other words, there is nothing else in an account apart from its address and state.

Can be queried by following fields:

| Field          | Return                                                       | Comments                                                     |
| -------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| acc_type       | uninitialized: 0, active: 1, frozen: 2,                      | Returns the current status of the account according to original TON blockchain specification. |
| last_paid      | Timestamp as a uint256                                       | Contains either the unixtime of the most recent storage payment collected (usually this is the unixtime of the most recent transaction), or the unixtime when the account was created (again, by a transaction |
| due_payment    | Due payments for the account as a uint256. Returns a line starting with 0x. | If present, accumulates the storage payments that could not be exacted from the balance of the account, represented by a strictly positive amount of nanograms; it can be present only for uninitial- ized or frozen accounts that have a balance of zero Grams (but may have non-zero balances in other cryptocurrencies). When due_payment becomes larger than the value of a configurable parameter of the blockchain, the ac- count is destroyed altogether, and its balance, if any, is transferred to the zero account. |
| last_trans_lt  | uint64 of the last transaction It                            |                                                              |
| balance        | uint128 gram balance of the account                          |                                                              |
| balance_other  | Array of other currency balances, currency name as uint32 and balance as uint256 |                                                              |
| split_depth    | uint8 number of the split depth for large contracts          | Is present and non-zero only in instances of large smart contracts. |
| tick           | Boolean true/false                                           | May be present only in the masterchain—and within the masterchain, only in some fundamental smart contracts required for the whole system to function |
| code           |                                                              | If present, contains smart-contract code encoded with in base64 |
| data           |                                                              | If present, contains smart-contract data encoded with in base64 |
| datalibrary: , |                                                              | If present, contains library code used in smart-contract     |
|                |                                                              |                                                              |
| proof          | Merkle proof that account is a part of shard state it cut from as a bag of cells with Merkle proof struct encoded as base64. |                                                              |
| boc            | Bag of cells with the account struct encoded as base64.      |                                                              |

## Message type

Message layout queries. A message consists of its header followed by its body or payload. The body is essentially arbitrary, to be interpreted by the destination smart contract. It can be queried with the following fields.

| Field        | Returns                                                      | Comments                                                     |
| ------------ | ------------------------------------------------------------ | ------------------------------------------------------------ |
| msg_type     | internal: 0, extIn: 1, extOut: 2                             | Returns the type of message                                  |
| status       | unknown: 0,queued: 1, processing: 2, preliminary: 3, proposed: 4, finalized: 5, refused: 6,transiting: 7 | Returns internal processing status according to the numbers shown. |
| block_id     | String block Identifier                                      | Returns the block identifier of the block where the message was last seen. |
| body         | Bag of cells with the message body encoded as base64.        |                                                              |
| split_depth  | Uint8 split depth                                            | This is only used for special contracts in masterchain to deploy messages |
| tick         | Boolean true/false for tick                                  | This is only used for special contracts in masterchain to deploy messages |
| tock         | Boolean true/false for tock                                  | This is only used for special contracts in masterchain to deploy messages |
| code         | Bag of cells encoded as base64.                              | Represents contract code in deploy messages.                 |
| data         | Bag of cells encoded as base64.                              | Represents initial data for a contract in deploy messages    |
| library      | Bag of cells encoded as base64.                              | Represents contract library in deploy messages               |
| src          | Returns source address string                                |                                                              |
| dst          | Reutrns destination address string                           |                                                              |
| created_lt   | uint 64 number                                               | Logical creation time automatically set by the generating transaction. |
| created_at   | uint 32 number                                               | Creation unixtime automatically set by the generating transaction. The creation unixtime equals the creation unixtime of the block containing the generating transaction. |
| ihr_disabled | Boolean true/false                                           | IHR is disabled for the message.                             |
| ihr_fee      | uint128 gram amount fee of the message                       | This value is subtracted from the value attached to the message and awarded to the validators of the destination shardchain if they include the message by the IHR mechanism. |
| fwd_fee      | uint128 gram forwarding fee of the message                   | Original total forwarding fee paid for using the HR mechanism; it is automatically computed from some configuration parameters and the size of the message at the time the message is generated. |
| import_fee   | uint128 gram importing fee of the message                    |                                                              |
| bounce       | Boolean true/false                                           | Bounce flag. If the transaction has been aborted, and the inbound message has its bounce flag set, then it is “bounced” by automatically generating an outbound message (with the bounce flag clear) to its original sender. |
| bounced      | Boolean true/false                                           | Bounced flag. If the transaction has been aborted, and the inbound message has its bounce flag set, then it is “bounced” by automatically generating an outbound message (with the bounce flag clear) to its original sender. |
| value        | Internal message value in grams                              | May or may not be present                                    |
| value_other  | Value of the message in other currency as name and amount of other crypto currency | May or may not be present.                                   |
| proof        |                                                              | Merkle proof that message is a part of a block it cut from. It is a bag of cells with Merkle proof struct encoded as base64. |
| boc          |                                                              | A bag of cells with the message structure encoded as base64. |

## MsgEnvelope type

Message envelopes are used for attaching routing information, such as the current (transit) address and the next-hop address, to inbound, transit, and outbound messages.

| Field             | Returns                           | Comments                                                     |
| ----------------- | --------------------------------- | ------------------------------------------------------------ |
| msg_id            | Message id string.                |                                                              |
| next_addr         | A line with intermediate address. | Message next-hop next-hop address                            |
| cur_add           | A line with intermediate address. | Message current (or transit) address                         |
| fwd_fee_remaining | Remaining forwarding fee in grams | Explicitly represents the maximum amount of message forwarding fees that can be deducted from the message value during the remaining HR steps; it cannot exceed the value of fwd_fee indicated in the message itself. |



## InMsg type

A type to specify the parameter of the inbound message. You can query the source of the message, the reason for it's being imported into this block, and some information about its “fate”—its processing by a transaction or forwarding inside the block.

| Field            | Returns                                                      | Comments                                                     |
| ---------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| msg_type         | Message type: external: 0, ihr: 1, immediately: 2, final: 3, transit: 4, discardedFinal: 5, discardedTransit: 6, |                                                              |
| msg:             | Message ID                                                   |                                                              |
| transaction:     | Transaction ID                                               |                                                              |
| ihr_fee:         | uint128 gram amount fee of the message                       | This value is subtracted from the value attached to the message and awarded to the validators of the destination shardchain if they include the message by the IHR mechanism. |
| in_msg:          |                                                              | Contains message envelope.                                   |
| fwd_fee:         | Forwarding message fee in grams                              |                                                              |
| out_msg:         |                                                              | More research required                                       |
| transit_fee:     | transit fee in grams                                         |                                                              |
| transaction_id:  | uint64 transaction ID                                        |                                                              |
| proof_delivered: |                                                              |                                                              |

## ShardDescr type

ShardHashes is represented by a dictionary with 32-bit workchain_ids as keys, and “shard binary trees”, represented by TL-B type BinTree ShardDescr, as values. Each leaf of this shard binary tree contains a value of type ShardDescr, which describes a single shard by indicating the sequence number seq_no, the logical time lt, and the hash hash of the latest (signed) block of the corresponding shardchain.

| Field                 | Returns                                                      | Comments                                                     |
| --------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| seq_no:               | uint32 sequence number                                       |                                                              |
| reg_mc_seqno:         | Representation hash of shard block's root cell.              | Returns last known master block at the time of shard generation. The shard block configuration is derived from that block. |
| start_lt:             | Logical time of the shardchain start.                        |                                                              |
| end_lt:               | Logical time of the shardchain end.                          |                                                              |
| root_hash:            | Representation hash of shard block's root cell.              | Returns last known master block at the time of shard generation. The shard block configuration is derived from that block. |
| file_hash:            | Shard block file hash.                                       |                                                              |
| before_split:         | Boolean true/false.                                          | TON Blockchain supports dynamic sharding, so the shard configuration may change from block to block because of shard merge and split events. Therefore, we cannot simply say that each shardchain corresponds to a fixed set of accountchains.    A shardchain block and its state may each be classified into two distinct parts. The parts with the ISP-dictated form of will be called the split parts of the block and its state, while the remainder will be called the non-split parts. The masterchain cannot be split or merged. |
| before_merge:         | Boolean true/false.                                          |                                                              |
| want_split:           | Boolean true/false.                                          |                                                              |
| want_merge:           | Boolean true/false.                                          |                                                              |
| nx_cc_updated:        | Boolean true/false.                                          |                                                              |
| flags:                |                                                              |                                                              |
| next_catchain_seqno:  |                                                              |                                                              |
| next_validator_shard: |                                                              |                                                              |
| min_ref_mc_seqno:     |                                                              |                                                              |
| gen_utime:            | Generation time in uint32                                    |                                                              |
| split_type:           | none: 0,   split: 2,    merge: 3,                            |                                                              |
| split:                |                                                              |                                                              |
| fees_collected:       | Amount of fees collected int his shard in grams.             |                                                              |
| fees_collected_other: | Amount of fees collected int his shard in  other currencies. |                                                              |
| funds_created:        | Amount of funds created in this shard in grams.              |                                                              |
| funds_created_other:  | Amount of funds created in this shard in other currencies.   |                                                              |

## Block type

| **FIELD**                      | **DESCRIPTION**                                              | **COMMENT**                                                  |
| ------------------------------ | ------------------------------------------------------------ | ------------------------------------------------------------ |
| status:                        | unknown: 0,    proposed: 1,    finalized: 2,    refused: 3,  | Returns block processing status                              |
| global_id:                     | uint32 global block ID                                       |                                                              |
| want_split:                    | boolean true/false                                           |                                                              |
| seq_no:                        |                                                              |                                                              |
| after_merge:                   | boolean true/false                                           |                                                              |
| gen_utime:                     | uint 32 time stamp                                           |                                                              |
| gen_catchain_seqno:            |                                                              |                                                              |
| flags:                         |                                                              |                                                              |
| master_ref:                    |                                                              |                                                              |
| prev_ref:                      |                                                              | External block reference for previous block.                 |
| prev_alt_ref:                  |                                                              | External block reference for previous block in case of shard merge. |
| prev_vert_ref:                 |                                                              | External block reference for previous block in case of vertical blocks. |
| prev_vert_alt_ref:             |                                                              |                                                              |
| version:                       | uin32 block version identifier                               |                                                              |
| gen_validator_list_hash_short: |                                                              |                                                              |
| before_split:                  | boolean true/false                                           |                                                              |
| after_split:                   | boolean true/false                                           |                                                              |
| want_merge:                    | boolean true/false                                           |                                                              |
| vert_seq_no:                   |                                                              |                                                              |
| start_lt:                      | uint64 timestamp                                             | Logical creation time automatically set by the block formation start. Logical time is a component of the TON Blockchain that also plays an important role in message delivery is the logical time, usually denoted by Lt. It is a non-negative 64-bit integer, assigned to certain events. For more details, see the TON blockchain specification |
| end_lt:                        | uint64 timestamp                                             | Logical creation time automatically set by the block formation end. |
| workchain_id:                  | uint32 workchain identifier                                  |                                                              |
| shard:                         |                                                              |                                                              |
| min_ref_mc_seqno:              |                                                              | Returns last known master block at the time of shard generation. |
| value_flow:                    |                                                              |                                                              |
| to_next_blk:                   | Amount of grams amount to the next block.                    |                                                              |
| to_next_blk_other:             | Amount of other cryptocurrencies to the next block.          |                                                              |
| exported:                      | Amount of grams exported.                                    |                                                              |
| exported_other:                | Amount of other cryptocurrencies exported.                   |                                                              |
| imported:                      | Amount of grams imported.                                    |                                                              |
| imported_other:                | Amount of other cryptocurrencies imported.                   |                                                              |
| from_prev_blk:                 | Amoun of grams transferred from previous block.              |                                                              |
| from_prev_blk_other:           | Amount of other cryptocurrencies transferred from previous block. |                                                              |
| minted:                        | Amount of grams minted in this block.                        |                                                              |
| minted_other:                  | Amount of other cryptocurrencies minted in this block.       |                                                              |
| fees_imported:                 | Amount of import fees in grams                               |                                                              |
| fees_imported_other:           | Amount of import fees in other currrencies.                  |                                                              |
| in_msg_descr:                  | Array of InMsg decribed messages.                            |                                                              |
| rand_seed:                     |                                                              | Need more research.                                          |
| out_msg_descr:                 |                                                              |                                                              |
|                                |                                                              |                                                              |

## Transaction Type

The table below shows how our GraphQL scheme matches fields of TON transaction TLB schemes. In most cases, the specification is quoted to describe the fields. Meaning of fields is also sometime self-explanatory. 

For more details, check the specification at https://test.ton.org/tblkch.pdf.

| **FIELD**                                                    | **DESCRIPTION** | **COMMENT**                                                  |
| ------------------------------------------------------------ | --------------- | ------------------------------------------------------------ |
| **tr_type**                                                  |                 | Transaction type according to the original blockchain specification, clause 4.2.4. The following value mapping is used:ordinary: 0storage: 1tick: 2tock: 3splitPrepare: 4splitInstall: 5mergePrepare: 6mergeInstall: 7 |
| **status**                                                   |                 | Transaction processing status with the following mapping: unknown: 0preliminary: 1proposed: 2finalized: 3refused: 4 |
| block_id                                                     |                 |                                                              |
| account_addr                                                 | uint256         | Address of an account for the transaction (Tip: check the notion of an Account collection in the specification) |
| **lt**                                                       |                 | Transaction logical time. LT and the account address define the transaction on the blockchain |
| **prev_trans_hash**                                          |                 | hash of the previous transaction for the account             |
| **prev_trans_lt**                                            |                 | logical time of a previous transaction for the account       |
| now                                                          |                 |                                                              |
| **outmsg_cnt**                                               |                 | The number of generated outbound messages (one of the common transaction parameters defined by the specification) |
| **orig_status**                                              |                 | The initial state of account. Note that in this case the query may return 0, if the account was not active before the transaction and 1 if it was already active |
| **end_status**                                               |                 | The end state of an account after a transaction, 1 is returned to indicate a finalized transaction at an active account |
| **in_msg**                                                   |                 | Dictionary of transaction inbound message ID's as specified in the specification |
| **in_message**                                               |                 | Dictionary of transaction inbound messages as specified in the specification |
| **out_msgs**                                                 |                 | Dictionary of transaction outbound message ID's as specified in the specification |
| **out_message**                                              |                 | Dictionary of transaction outbound messages as specified in the specification |
| **total_fees**                                               |                 | Total amount of fees that entails account state change and used in Merkle update |
| **total_fees_other**                                         |                 | Same as above, but reserved for other coins that may appear in the blockchain |
| **old_hash****new_hash**                                     | uint256         | Hashes of the account state before and after the transaction |
| credit_first                                                 |                 |                                                              |
| **STORAGE**   (phase)                                        |                 | The storage phase is present in ordinary, merge, split, storage and tock transactions, so a common representation for this phase includes three fields. The first defines the amount , the second can be empty, the third specifies the account status change |
| **storage_fees_collected**       	**storage_fees_due**      	**status_change** |                 | Fields show amounts related to storage fees and account status change (e.g. it may be frozen or remain active (unchanged)) |
| **CREDIT (phase)**                                           |                 | The account is credited with the value of the inbound message received. The credit phase can result in the collection of some due payments |
| **due_fees_collected**                                       |                 | The sum of `due_fees_collected `and credit must equal the value of the message received, plus its `ihr_fee` if the message has not been received via Instant Hypercube Routing, IHR (otherwise the `ihr_fee` is awarded to the validators). |
| credit                                                       |                 |                                                              |
| credit_other                                                 |                 |                                                              |
| **COMPUTE (phase)**                                          |                 | The code of the smart contract is invoked inside an instance of TVM with adequate parameters, including a copy of the inbound message and of the persistent data, and terminates with an exit code, the new persistent data, and an action list (which includes, for instance, outbound messages to be sent). The processing phase may lead to the creation of a new account (uninitialized or active), or to the activation of a previously uninitialized or frozen account. The gas payment, equal to the product of the gas price and the gas consumed, is exacted from the account balance. If there is no reason to skip the computing phase, TVM is invoked and the results of the computation are logged. Possible parameters are covered below. |
| **compute_type**                                             |                 | 0: skipped, then only `skipped_reason` is defined.1: not skipped, then other fields for the phase are filled |
| **skipped_reason**                                           |                 | Reason for skipping the compute phase. According to the specification, the phase can be skipped due to the absence of funds to buy gas, absence of state of an account or a message, failure to provide a valid state in the message |
| **success**                                                  |                 | This flag is set if and only if `exit_code` is either 0 or 1. |
| **msg_state_used**                                           |                 | This parameter reflects whether the state passed in the message has been used. If it is set, the `account_activated` flag is used (see below) |
| **account_activated**                                        |                 | The flag reflects whether this has resulted in the activation of a previously frozen, uninitialized or non-existent account. |
| **gas_fees**                                                 |                 | This parameter reflects the total gas fees collected by the validators for executing this transaction. It must be equal to the product of `gas_used` and `gas_price` from the current block header. |
| **gas_used**                                                 |                 | See above                                                    |
| **gas_limit**                                                |                 | This parameter reflects the gas limit for this instance of TVM. It equals the lesser of either the Grams credited in the credit phase from the value of the inbound message divided by the current gas price, or the global per-transaction gas limit. |
| **gas_credit**                                               |                 | This parameter may be non-zero only for external inbound messages. It is the lesser of either the amount of gas that can be paid from the account balance or the maximum gas credit |
| mode                                                         |                 |                                                              |
| **exit_code**		**exit_arg**                            |                 | These parameters represent the status values returned by TVM; for a successful transaction, `exit_code` has to be 0 or 1 |
| **vm_steps**                                                 |                 | the total number of steps performed by TVM (usually equal to two plus the number of instructions executed, including implicit RETs) |
| **vm_init_state_hash**	**vm_final_state_hash**            |                 | These parameters are the representation hashes of the original and resulting states of TVM |
| **ACTION (phase)**                                           |                 | If the smart contract has terminated successfully (with exit code 0 or 1), the actions from the list are performed. If it is impossible to perform all of them—for example, because of insufficient funds to transfer with an outbound message—then the transaction is aborted and the account state is rolled back. The transaction is also aborted if the smart contract did not terminate successfully, or if it was not possible to invoke the smart contract at all because it is uninitialized or frozen. |
| success                                                      |                 |                                                              |
| valid                                                        |                 |                                                              |
| **no_funds**                                                 |                 | The flag indicates absence of funds required to create an outbound message |
| **status_change**                                            |                 | Account status change according to the list of statuses provided by the specification |
| total_fwd_fees                                               |                 | Amount in Grams                                              |
| total_action_fees                                            |                 | Amount in Grams                                              |
| result_code                                                  |                 |                                                              |
| result_arg                                                   |                 |                                                              |
| tot_actions                                                  |                 |                                                              |
| spec_actions                                                 |                 |                                                              |
| skipped_actions                                              |                 |                                                              |
| msgs_created                                                 |                 |                                                              |
| **action_list_hash**                                         |                 | Hash of the action list created during the compuation phase  |
| total_msg_size_cells                                         |                 |                                                              |
| total_msg_size_bits                                          |                 |                                                              |
| **BOUNCE (phase)**                                           |                 | If the transaction has been aborted, and the inbound message has its bounce flag set, then it is “bounced” by automatically generating an outbound message (with the bounce flag clear) to its original sender. Almost all value of the original inbound message (minus gas payments and forwarding fees) is transferred to the generated message, which otherwise has an empty body. |
| bounce_type                                                  |                 | 0: Negfunds1: Nofunds2: Ok                                   |
| msg_size_cells                                               |                 |                                                              |
| msg_size_bits                                                |                 |                                                              |
| req_fwd_fees                                                 |                 |                                                              |
| msg_fees                                                     |                 |                                                              |
| fwd_fees                                                     |                 | Amount to be bounced back                                    |
| **aborted**                                                  |                 | The flag is set either if there is no action phase or if the action phase was unsuccessful. The bounce phase occurs only if the `aborted` flag is set and the inbound message was bounceable. |
| destroyed                                                    |                 |                                                              |
| tt                                                           |                 |                                                              |
| split_info                                                   |                 | The fields below cover split prepare and install transactions and merge prepare and install transactions, the fields correspond to the relevant schemes covered by the blockchain specification. |
| cur_shard_pfx_len                                            |                 | length of the current shard prefix                           |
| acc_split_depth                                              |                 |                                                              |
| this_addr                                                    |                 |                                                              |
| sibling_addr                                                 |                 |                                                              |
| prepare_transaction                                          |                 |                                                              |
| installed                                                    |                 |                                                              |
| proof                                                        |                 |                                                              |
| boc                                                          |                 |                                                              |


 