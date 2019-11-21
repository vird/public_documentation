## Contract Development Flow



This is an ironical guide to smart contract development created by a writer for developers. Use it as a roadmap and keep smiling while on the way ; ). 

## def develop_code(null):

- Check the current [ABI Spec]()

<!--Contract life-cycle is basically a constant message exchange. Messages are used to address contract public functions and trigger transactions that change the contract state and to get contract data without changing its state. The message format in TON is based on the specification and rules for interaction with a specific contract are defined by its ABI. So before you start, check the ABI spec.-->

- Check the general info on how [Gas is calculated in TON]() in general and for [transaction]()
- Check how you can [manage gas in your contract]()
- Check how you can [modify code of a deployed contract]()
- Check our documents on [Solidity language feature implementation](Compilers/Solidity Compiler/Solidity Support Status)
- Create your code now in Solidity, C, C++ 

## def install_tools(preferred delivery):

<!--To start you need to build your compilation and deployment toolchain there are several options-->

 **if** preferred delivery = source code from open source:

​     **then** get the needed components: 

- ​     [Solidity Compiler](https://github.com/tonlabs/TON-Solidity-Compiler)
- ​     [LLVM Compiler]( https://github.com/tonlabs/TON-Compiler)
- ​     [tvm-linker utility](https://github.com/tonlabs/TVM-linker) 

​     Follow guidelines in Readme's to build a toolchain

​	 **result** = working toolchain

 **elif** preferred delivery = TON Labs SDK/Node SE from containers:

​    install the SDK/Node SE according to the [guidelines](SDK/Installation).

​    **result** =  working toolchain with a [CLI](SDK/TONDEV CLI)

 **else** preferred delivery ==  SDK/Node SE from sources:

​     get sources:

- ​     [the SDK core]( https://github.com/tonlabs/TON-SDK )
- ​     [TVM]( https://github.com/tonlabs/ton-labs-vm)
- ​     [JS Library]( https://github.com/tonlabs/ton-client-js )
- ​     [Rust Library](https://github.com/tonlabs/ton-client-rs) 

​     Follow guidelines in Readme's to create a project 

   **result** = working toolchain with a CLI 

**return** **result**

## def compilation(smart contract source):

  smart contract source = **Solidity**

 <!--you need to compile the source code and there are several options depending on compiler preferences and source language-->

​    **if** compiler toolchain built from <u>source code</u>:

​       call: `tvm_linker compile [--lib <lib_file>] [--abi-json <abi_file>] [--genkey | --setkey <keyfile>] [-w <workchain_id>] [--debug] <source>`

**result** = .tvc file

 <!--Note that if you use this compilation option, you get the key pair and the future contract address after compilation. You can still change the address at contract deployment and initialization via the SDK. -->

​        **elif** preferred compiler delivery == SDK/Node SE from containers:

​           then call the CLI command: `tondev sol`

​           result = .tvc file

<!--With this option the future address is unknown-->

​        **else** preferred compiler delivery ==  SDK /Node SE from sources:

*​      then [TODO]*

  smart contract source = C/C++:

​       **if** preferred compiler delivery= <u>source code</u>:

​       call: `tvm_linker compile [--lib <lib_file>] [--abi-json <abi_file>] [--genkey | --setkey <keyfile>] [-w <workchain_id>] [--debug] <source>`

​	  **result** = .tvc file

 <!--Note that if you use this compilation option, you get the key pair and the future contract address after compilation. You can still change the address at contract deployment and initialization via the SDK.-->  

​    elif preferred compiler delivery == SDK from pre-build container:
​    [TODO]
​    result = .tvc file with this option the future address unknown
​ 
else preferred compiler delivery option == SDK from sources:
​         [TODO]  
​    smart contract source = Fift/FunC
​    [TODO]

 return result

def get_boc(.tvc):
 The option available for .tvc. files generated by the tvm_linker. You need .boc files to work directly with TON testnet and lite client. 
[linker command]
    result = .boc file
  return result

def get address(.tvc):
 you need this function to get a future contract address when you used the SDK locally from containers or with Node SE.

[create deploy message link]
    result = address
  return result 

def positive balance(address):
In TON you cannot deploy to an address with zero balance due to storage fees. See more in Gas Specs.
   Use TON Labs Giver to top up the address [add code]
 or transfer tokens to the addreess from an active contract you have
    result = positive balance
  return result

def deployment(compiled code):
 you can use one of the options depending on your preferences and on the compilation option you used before.
  compiled code = .tvc built from open source compilers
​   then use tvm_linker deployment command from the project folder.
​   result = contract deployed to address  
  compiled code = .tvc built by the SDK:
​   create the deploy message in JS OR Rust
​   result = contract deployed to address
  compiled code = .boc:
​    Install TON Lite Client and follow its guidelines
  return result


