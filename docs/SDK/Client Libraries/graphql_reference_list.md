Also check the schema at https://github.com/tonlabs/ton-q-server/blob/master/server/db.schema.v2.js

| **FIELD**                               | **DESCRIPTION**                                              |
| :-------------------------------------- | ------------------------------------------------------------ |
| **tr_type**                             | Transaction type according to the original blockchain specification, clause 4.2.4. The following value mapping is used:</br> ordinary: 0</br>storage: 1</br>tick: 2</br>tock: 3</br>splitPrepare: 4</br>splitInstall: 5</br>mergePrepare: 6</br>mergeInstall: 7 |
| **status**                              | transaction processing status with the following mapping:</br> unknown: 0</br>preliminary: 1</br>proposed: 2</br>finalized: 3</br>refused: 4 |
| **orig_status**                         | The initial state of account. Note that in this case the query may return 0, if the account was not active before the transaction and 1 if it was already active |
| **end_status**                          | The end state of an account after a transaction, 1 is returned to indicate a finalized transaction at an active account |
| **value**                               | Stands for the amount in [test]  Grams                       |
| **value_other**                         | Stands for the amount other tokens (this field is reserved for tokens that may be added in the future |
| **total_fees**                          | see above                                                    |
| **total_fees_other**                    | see above                                                    |
| **src**                                 | message sender address                                       |
| **dst**                                 | message receiver address                                     |
| **transaction.block_id**                | The name is self-explanatory, but there are particularities. In case of external inbound message, `transaction.block_id` should always match the `in_msg.block_id` |
| **transaction.in_message.block_id**     | The ID of a block the delivered the transaction with the original message. |
| **transaction.out_message[i].block_id** |                                                              |
| **in_msg.block_id**                     | The name is self-explanatory, but there are particularities. Tn case of an internal message, the `msg.block_id` will match whichever block generated the internal message as an `out_msg`. |
| out_msg                                 |                                                              |