# SDK and Integration

**Q**: Would you be able to point me to where BOC messages are created or signed inside the JavaScript or Rust SDK? In the JS SDK, I just see that all calls end up hitting `this.requestLibrary` with some library specified, and that looks like it ends up calling a `TONClientLibrary` module, but it's not clear where these libraries actually exist (they seem to be on the node itself).

**A**: BOC messages are created and signed inside the Rust SDK library. The library is located at the client side. For the Node.js SDK client, the native Node.js addon is used. For React Native, there is a native library for target platform where the client application is running. For the Web client, there is the Web Assembly module downloaded and executed at the client device. The Rust SDK library contains core SDK functions used on all platforms.

The SDK will be available as Open Source and more platform and language rappers are to be developed. We are open to customer and community suggestions and contributions to the repository, once it is open.

------

**Q**: Don’t see any code in the SDK's that is actually generating keys locally or constructing .boc messages. It seems they are just calling your node to do that for them, so it seems like more of a client to the node. Please correct me if that’s not right.

**A**: See the previous answer. Keys are generated in the Rust core library running on a client device. The message BOC construction code is also located there. The SDK's interaction with the node is limited to sending external blockchain messages calling contracts and querying results from the GraphQL server. All crypto operations are performed locally.

------

**Q**: Is there an API call to the full node that returns a given account’s balance along with the block height at which that balance was calculated, in a single API call? Note: This IS NOT the same thing as historical balance lookup. (Example: GET /balance/account1 -> {balance:10, height: 123902}) The current account state provided by the lite-client does not include this information. Can we be given this information if it is not available today?

**A**: It is possible to get this information by the series of graphQL queries:

- query an account by its ID (address) and remember `last_trans_lt` (last transaction logical time)

```SQL
 query {
   accounts(filter: {id: {eq: "0:0000000000000000000000000000000000000000000000000000000000000000"}}) {
     id,
     last_trans_lt
   }
 }
```

- result:

```html
{
   "data": {
     "accounts": [
       {
         "id": "0:0000000000000000000000000000000000000000000000000000000000000000",
         "last_trans_lt": "0x1d83f62c2"
       }
     ]
   }
 }
```

- query the transaction with your account ID and the logical time; remember the corresponding `block_id`

```sql
query {
   transactions(filter: {
     account_addr: {eq: "0:8900d4821d8a2ed48c70c65fa60a4bb6e0741049a5c830b1fbb1b418895c1a5f"},
     lt: {eq: "0x243c63dc1"}
   }) {
     id,
     block_id
   }
}
```

- result:

```HTML
{
  "data": {
    "transactions": [
      {
        "id": "eb2ec458adfbc72b62180501d0892e79b6fb0a0cd1a1fa125616a4d6a2d851a1",
        "block_id": "922b64c98eb3d695b859ede7da5169c96f996ec4c8da7b755b72fad76ab4cd69"
      }
    ]
  }
}
```

- query a block by an ID with the required data:

```sql
query {
   blocks(filter: {
     id: {eq: "a82e433173c96ca4770283709b781095f63dba19b7a590025cd31b1d705ff896"}
   }) {
    id
    seq_no
   }
 }
```

- result:

```html
{
  "data": {
    "blocks": [
      {
        "id": "a82e433173c96ca4770283709b781095f63dba19b7a590025cd31b1d705ff896",
        "seq_no": 9182
      }
    ]
  }
}
```

------

**Q**: I made a query for transaction data:

```sql
query {
  transactions(filter:{in_msg:{ne:null}}) {
    id
    block_id
    in_message {
      id
      value
    }
    out_messages {
      id
      value
    }
  }
}
```

And it returned `"out_messages":[]`in multiple instances. What does it mean?

**A**: An empty array means that there were no events in transactions (calls to other contracts).

------

**Q**: Does your node currently (or will you) provide an API to submit pre-signed messages (e.g. a pre-signed bag-of-cells message)? Can we have this endpoint (not a CLI interface) if it does not exist today?

**A**: To send pre-generated messages, use the `processMessage` function of the SDK JS client. Also, there will be a function to construct unsigned messages. You will be able to sign generated message yourself and then to send it using the SDK. 

For example you can generate message to call the contract by SDK function. That function returns a serialized message and its hash to be signed. You sign it using your own implementation of the signature algorithm (or using some hardware security module). Then you call another SDK function that adds your signature to the message generated at first step. Finally, the resulting signed message can be sent by `processMessage` function.

------

**Q**: How does one derive the message (or transaction) hash from a signed bag-of-cells message to a contract? Can you provide details on how we can compute this on our own? (Ideally we can compute this so we can look it up after submitting to the network.)

**A**: The SDK returns hash of the generated message as a message ID. We will provide a function which will return hash for a specific message in the next few weeks. To calculate hash manually, deserialize the bag of cells into a tree of cells and calculate the representation hash of the root cell as described in paragraph 3.1.5 of the original [TVM specification](/TON Blockchain/TON Specifications/TON Virtual Machine). It's quite an effort, though.

------

**Q**: It seems that messages are not replay protected on their own. Is this the case? Is there any default replay protection provided by the network or must every contract implement their own sequence number or similar method to replay protection?

**A**: There is no default replay attack protection in TON. A contract developer is supposed to implement it manually. There are only "best practices" by TON developers [here](https://test.ton.org/smc-guidelines.txt). TON Labs compilers will have a default inline implementation of replay protection.

------

**Q**: Can you provide more explicit details or provide an example of how an account’s address is derived from the compiled contract and initial state and specifically how that hash is computed and over what components? Source code is also helpful here if you can provide it.

**A**: A blockchain account contains the `State Init` field. It stores the contract code and persisted data. `State init` is also tree of cells. At contract deployment this field is transferred alongside the deploying message (so called constructor message) and is set as the contract initial state. The representation hash (see paragraph 3.1.5 of the [TVM specification](/TON Blockchain/TON Specifications/TON Virtual Machine)) of the contract initial state is the account address. The node checks that the state init hash of the deployment message is equal to the destination address. The SDK will have a function for computing account address by its initial state. The SDK source code will be available soon.

------

**Q**: Is (or will be) the solidity-to-tvm and tvm-linker source code available to users? If not, can agreements be signed to make it available to partners?

**A**: Yes, both Sol2Tvm compiler and TVM-Linker will be available as open source.

------

**Q**: With a Solidity contract, how does one properly specify an address parameter? In solidity, addresses are uint256 but this can only capture the address portion and not the workchain_id. How would one specify the full address parameter correctly here? (e.g. “0:D702CC0414858D83A5538A3C1C86231872F006453AF7C825A59F9DD12D636AF” is invalid in solidity for an address param)

**A**: Now we only use the `address` field of *MsgAddressInt* structure as Solidity `address` type. We assume that worckchain_id is 0. But in future the `address` type will represent the whole *MsgAddressInt* structure. Now workchain_id can be defined separately as int8.

------

**Q**: When deploying a contract init message that was created using a solidity contract, the Sol2TVM compiler and tvm_linker, I’m seeing the init succeed initially but then also get replayed dozens of times after. How does one replay protect a contract init message for a solidity-compiled contract? (example message that was replayed after the initialization was already successful previously: <https://test.ton.org/testnet/transaction?account=EQDXAswEFIWNg6VTijwchiMYcvAGRTr3yCWln53RLWNq8CvQ<=195287000001&hash=7F09EC8EEBBAF510D7873CD3DEC494B2DD28A790A7A46E83D0A183F3E672B509>)

**A**: The sample message at <https://test.ton.org/testnet/transaction?account=EQDXAswEFIWNg6VTijwchiMYcvAGRTr3yCWln53RLWNq8CvQ<=195287000001&hash=7F09EC8EEBBAF510D7873CD3DEC494B2DD28A790A7A46E83D0A183F3E672B509> has no init (it is `init:nothing`), and a transaction with this message is aborted. But in general, the StateInit data attached to a message is used by the node only once: when a contract is in the Uninit state. Replays of such message can succeed, but the contract code will not be changed. An Init msg created with sol2tvm compiler and tvm_linker contains an encoded constructor call. Now you can call constructor multiple times, but in future releases it will be changed.

------

**Q**: Can I convert an address into base64 with workchain ID encoded?

**A**: Yes, there is an SDK option for it, check the sample code:

```javascript
const address1 = {
        address: '0:c7202c3af71b07cfe1d92626b3d660feb5bb808c2825b0ac6f15c11a101d3117',
        convertTo: 'Base64',
        base64Params: {    url: false,
            test: true,
            bounce: false,},
    }
    const address2  = {
        address: '0QCVRQAmm2wSJwyrIoIrOxf8XHV5fkA-s_RAxTA8Jc_80-2Q',
        convertTo: 'Base64',
        base64Params: {    url: false,
            test: true,
            bounce: false,},
    }
    

    const bounceable1 = (await client.contracts.convertAddress(address1)).address; 
    const bounceable2 = (await client.contracts.convertAddress(address2)).address;
```

