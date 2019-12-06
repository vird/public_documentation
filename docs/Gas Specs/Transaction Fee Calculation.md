Transaction fees consist of a few types of different fees connected to the execution of a single transaction. Transactions itself are complex processes, and fees are paid relative to different stages of executing them.

In this section, we explain how the fees are calculated.

We shall define `transaction_fee` is a sum of all fees for a single transaction.

`transaction_fee = trans.total_fees + outbound_internal_messages_fee`;

where

```
outbound_internal_messages_fee = N * (out_int_msg.header.fwd_fee + out_int_msg.header.ihr_fee)
```

where N is the number for outbound internal messages included in the transaction.

Here, `trans.total_fees` is a sum of all fees that are not dependent on the number of outbound internal messages in the transaction meaning it excludes the forward fees for outbound internal messages. Total outbound message fee might be absent if no outbound messages are sent in the process of completing the transaction.

`trans.total_fees` is calculated according to the following formula:

```
trans.total_fees = trans.storage_fees + trans.gas_fees + inbound_fwd_fees + trans.total_action_fees
```

1. `trans.storage_fees` - storage costs since the moment of the last transaction. It has no simple formula for calculation but we can simply describe it like:

```
account.cells * prices.cell_price + account.bits * prices.bit_price;
```

1. `trans.gas_fees` include all gas fees associated with the transaction. You can find more info in the Gas Calculation Basics doc.
2. `trans.total_action_fees` - fees for performing 'send message' actions.

Every outbound message sent by contract has forward fee. The forward fee for outbound internal message is splited to `int_msg_mine_fee` and `int_msg_remain_fee` :

```
msg_forward_fee = int_msg_mine_fee + int_msg_remain_fee
```

`int_msg_mine_fee` is a part of transaction `total_fees`. `int_msg_remain_fee` is placed in header of outbound internal message and will go to validators of shard to which message destination address is belong.

`trans.total_action_fees = N*out_ext_msg_fwd_fee + M*int_msg_mine_fee`,

where:

- `out_ext_msg_fwd_fee` - implicit forward fee for outbound external messages. N - number of such outbound messages.
- `int_msg_mine_fee` - 'mine' part of forward fee for outbound internal messages. M - number of such outbound messages.

1. `in_fwd_fee` - forward fee for inbound external message. It is not directly shown in the transaction data, yet it is calculated as a part of the `total_fees`.

Every forward fee is calculated according to the following formula: `fwd_fee = (lump_price + ceil((bit_price * msg.bits + cell_price * msg.cells)/2^16))`

`msg.bits` and `msg.cells` are calculated from `msg.init` and `msg.body` represented as tree of cells. Root cell of each field is not counted. `lump_price`, `bit_price`, `cell_price` are contains in global config parameter.

Formula for `trans.total_fees` is a complete formula, but not all of the fees are present in every transaction. For example, the `trans.gas_fees` can be skipped if the TVM compute phase is not initialized in a transaction. The action fee might be absent as well if no actions are performed during the transaction.

`trans.total_fwd_fees` is a separate way to calculate total forwarding fees.

```
trans.total_fwd_fees = trans.total_action_fees + N * (int_msg_remain_fee + out_int_msg.header.ihr_fee);
```

`out_int_msg.header.ihr_fee` - this fee is zero if `out_int_msg.header.ihr_disabled` flag is set to 1. Otherwise it is calculated according to:

```
(msg_forward_fee * ihr_factor) >> 16;` where `msg_forward_fee = int_msg_mine_fee + int_msg_remain_fee;
```

`ihr_factor` is part of global config parameter with message prices.