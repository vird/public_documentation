# Key Generation and Storage

The key pair is generated according to Ed25519 algorithm.

TON Labs developer tools offer two options to generate the key pair. You can use the linker utility or the SDK. Note that in linker keys are generated at compilation or separately, in the SDK keys are  generated at deploy.

**Linker key generation command**:

```shell
tvm_linker compile <source> --genkey <key_file>
```

where `key_file` - a name of the file to store public and private keys. The linker generates 2 files: `key_file.pub` for public key and `key_file` for private key.

**Linker compilation command**:

```shell
tvm_linker compile <source> [--lib library [...]] [--abi-json <abi_file>] [--genkey | --setkey <keyfile>]
```

**SDK Key Pair Generation functions in Rust and JS**

```rust
pub keyPair: Ed25519KeyPair,
```

```javascript
async ed25519Keypair(): Promise<TONKeyPairData> {
        return this.requestLibrary('crypto.ed25519.keypair');
    }
```

TON Labs implementation of the deployment logic includes the so-called constructor message defined by TON specification. Constructor message parameters (in particular, keys) are used in address generation. In the current implementation you cannot deploy a contract with zero balance. So, before deploying the contract code you already have to know its address and send some (test) Grams to it. 

For more details on SDK deploy methods, refer to Contract Interaction document covering key API functions.

**Linker deploy command:**

```shell
tvm_linker message <contract-address> [--init] [--data] [-w]
```

For more details on TVM linker, see the TVM Linker CLI guide.

**SDK Deploy Function Implementations in Rust and JavaScript**

```rust
pub(crate) fn deploy(_context: &mut ClientContext, params: ParamsOfDeploy) -> ApiResult<ResultOfDeploy> {
    debug!("-> contracts.deploy({})", params.constructorParams.to_string());

    let key_pair = params.keyPair.decode()?;

    let contract_image = create_image(&params.abi, params.initParams.as_ref(), &params.imageBase64, &key_pair.public)?;
    let account_id = contract_image.account_id();
    debug!("-> -> image prepared with address: {}", account_encode(&account_id));

    debug!("-> -> deploy");
    let tr_id = deploy_contract(&params, contract_image, &key_pair)?;
    debug!("-> -> deploy transaction: {}", u256_encode(&tr_id.clone().into()));

    let tr_id_hex = tr_id.to_hex_string();

    debug!("load transaction {}", tr_id_hex);
    let tr = super::run::load_transaction(&tr_id);

    debug!("<-");
    if tr.tr().is_aborted() {
        debug!("Transaction aborted");
        super::run::get_result_from_block_transaction(tr.tr())?;
        Err(ApiError::contracts_deploy_transaction_aborted())
    } else {
        Ok(ResultOfDeploy {
            address: account_encode(&account_id)
        })
    }
}

```

```javascript
class myContract {
    constructor(client, address, keys) {
        this.client = client;
        this.address = address;
        this.keys = keys;
        this.package = pkg;
        this.abi = abi;
    }

    async deploy(constructorParams) {
        if (!this.keys) {
            this.keys = await this.client.crypto.ed25519Keypair();
        }
        this.address = (await this.client.contracts.deploy({
            package: pkg,
            constructorParams,
            keyPair: this.keys,
        })).address;
    }
```

The public key is then stored in the contract persistent memory and the private key is stored at the client (user) side (e.g., in a hardware wallet) and never leaves the storage point.

The private key is used to sign messages, the public key  (`tvm_sender_pubkey()`) is used in external messages to verify the signature.

The signing algorithm is covered in the ABI specification, SHA256 hashing is used.

# Message Validation

Messages to contracts are validated by a signature and public key. Key methods for validation are `msg.sender` that contains the sender address and `tvm_sender_pubkey()`  runtime function that passes the public key. They are used differently, depending on whether an internal or external message is validated.

## Internal Message 

To validate internal messages, the public key is not used.  Each message is identified by its address and the public key is not provided. `msg.sender` that contains the address of the sender contract is used, while `tvm_sender_pubkey()`  message is empty.

Message signing is optional. Identification is based on the sender address.

## External Message 

For external messages, `msg.sender` is always empty, while `tvm_sender_pubkey()` is mandatory for identification and validation; it contains the sender public key.

Signature is verified by the contract runtime.

## Signing Messages

Messages are signed according to the ABI. Please, see the specification, [Message Signing section](Smart Contract Lore/ABI Specification New/#message-body-signing).



