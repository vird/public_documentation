# Smart-contract development workflow

## Step by step

### Compile from source to assembly

```
solc --tvm MyContract.sol > MyContract.code
```

### Generate contract interface

```
solc --tvm_abi MyContract.sol > MyContract.abi.json
```

### Link assembly with runtime

```
tvm_linker compile MyContract.code --lib <path to>/stdlib_sol.tvm --abi-json MyContract.abi.json
```

### Generate constructor message 

```
tvm_linker message <MyContractAddress> --init -w 0 [--setkey <path_to_keyfile>]
```

### Pay for contract deployment

```
TODO
```

### Deploy the contract

```
TODO: use our tools
lite-client -C ton-lite-client-test1.config.json
```

### Check the contract status

```
TODO: use our tools
getaccount 0:<MyContractAddress>
```

## Just take me there
Use script/SDK to do all the above in one command

## Check out samples
[Sample contracts in Solidity](https://github.com/tonlabs/samples/tree/master/solidity)
[Sample contracts in C](https://github.com/tonlabs/samples/tree/master/c)


## Into the fray
[I'm pro. Show me the real stuff!](Smart Contract Lore/Advanced)
