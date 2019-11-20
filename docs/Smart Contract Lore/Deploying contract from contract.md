In TON blockchain, you can deploy smart contracts right out of the existing contracts.

To do so, you'll have to transfer a number of starting parameters and initial contract gram amount to create a new contract.

Deploy is done through sending an internal message including relevant code and data attached to it and a `bounce` flag set to 0.

There are three ways to make that happen, depending on what is already stored in the deploying contract.

This document covers deploying a new contract with an internal message.

In the first part, we take a look at the JS code samples for all three possible ways to deploy a contract from another contract. The second part is a sample contract compiled using the TON Labs solidity compiler.

## Case 1:

The new contract initial state is stored in the deploying contract and both the public key. Constructor parameters are explicitly transferred to the `deploy`.

To start, you'll need to store the new contract initial in the deployer contract as a `StateInit` structure.

The whole contract interface and constructor parameters have to be explicitly stated. Let's take a look at the example below:

```javascript
//Set deploying contract and define the package

    await contracts.run({
        address: deployer.address,
        functionName: 'setContract',
        abi: deployer_package.abi,
        input: {
            _contract: deployee_package.imageBase64,
        },
        keyPair: keys,
    });

//Get constructor function ID
    const constuctor_id = await contracts.getFunctionId({
        abi: deployee_package.abi,
        function: 'constructor',
        input: true,
    });

//Call deploy function
    const addressResult = await contracts.run({
        address: deployer.address,
        functionName: 'deploy',
        abi: deployer_package.abi,
        input: {
//transfer public key as a Uint256 
            pubkey: `0x${keys.public}`,
//transfer initial gram amount
            gram: 300000000,
//explicitly state contructor parameters
            constuctor_id: constuctor_id.id,
            constuctor_param0: 1,
            constuctor_param1: 2,
        },
        keyPair: keys,
    });


//New contract address output
    const address = addressResult.output.value0;
    await queries.accounts.waitFor({
        id: { eq: address },
        balance: { gt: "0" }
    }, 'id balance');

    const result = await contracts.run({
        address: address,
        functionName: 'get',
        abi: deployee_package.abi,
        input: {},
        keyPair: keys,
    });

    expect(result.output).toEqual({
        value0: '0x1',
        value1: '0x2'
    });
});
```

As you can see, deploy and constructor parameters have to be the same. In this case, the only thing the deploying contract changes from `StateInit` is the public key.

## Case 2:

The new contract being deployed has additional parameters besides the public key stored in the deploying contract. Meaning that the deploying contract has to replace more data than just the public key in the new contract.

Sometimes there are more data used in creating a new contract and it's address than just key pairs and default parameters. For example, in TON blockchain, a telegram ID can influence the final address.

An example below demonstrates the sequence of functions required:

```javascript
    const code = await contracts.getCodeFromImage({
        imageBase64: deployee_package.imageBase64
    });
//Extract code from the base StateInit and sends it to the constructor
    await contracts.run({
        address: deployer.address,
        functionName: 'setCode',
        abi: deployer_package.abi,
        input: {
            _code: code.codeBase64,
        },
        keyPair: keys,
    });
//Create starting data including parameters for the new contract
    const deployData = await contracts.getDeployData({
        abi: deployee_package.abi,
        initParams: {
            param1: 1,
            param2: 2
        },
        publicKeyHex: keys.public,
    });
//Get constructor function ID
    const constuctor_id = await contracts.getFunctionId({
        abi: deployee_package.abi,
        function: 'constructor',
        input: true,
    });
//Create new contract address from StateInit
    const addressResult = await contracts.run({
//Additional Data inputs
        address: deployer.address,
        functionName: 'deploy2',
        abi: deployer_package.abi,
        input: {
            data: deployData.dataBase64,
//transfer initial gram amount
            gram: 300000000,
//Data and parameters transferred to deploy
            constuctor_id: constuctor_id.id,
            constuctor_param0: 1,
            constuctor_param1: 2,

        },
        keyPair: keys,
    });
//New contract address output
    const address = addressResult.output.value0;
    await queries.accounts.waitFor(
        {
            id: { eq: address },
            balance: { gt: "0" }
        },
        'id balance'
    );

    const result = await contracts.run({
        address: address,
        functionName: 'get',
        abi: deployee_package.abi,
        input: {},
        keyPair: keys,
    });

    expect(result.output).toEqual({
        value0: '0x1',
        value1: '0x2'
    });
});
```

## Case 3:

This is the case where new contract data is defined by a separate payload meaning deploying contract does not replace anything or set parameters, but only executes `deploy`.

Note: `getDeployData` returns contract address within the workchain. To define the full address of the new contract, a workchain prefix is attached to it. In this case, workchain prefix is 0. You can learn more about addresses [here](https://test.ton.org/HOWTO.txt). TON Labs toolchain uses Raw addresses.

Example:

```javascript
//This creates the complete new contract image and data from pre-defined DeployData stored
//in the deploying contract
const deployData: TONContractGetDeployDataResult = await contracts.getDeployData({
        abi: deployee_package.abi,
        imageBase64: deployee_package.imageBase64,
        publicKeyHex: keys.public,
    });

    const address = "0:" + deployData.accountId || "";
//Creates a message body with serialized ABI constructor parameters
    const runBody = await contracts.createRunBody({
        abi: deployee_package.abi,
        function: "constructor",
        params: {
            _param1: 1,
            _param2: 2
        },
        internal: true,
    });


    const addressResult = await contracts.run({
        address: deployer.address,
        functionName: 'deploy3',
        abi: deployer_package.abi,
        input: {
            contr: deployData.imageBase64,
            addr: address,
//Transfer initial state gram amount
            grams: 300000000,
//Define Payload
            payload: runBody.bodyBase64,
        },
        keyPair: keys,
    });

//New contract address output
    expect(addressResult.output.value0).toEqual(address);
    await queries.accounts.waitFor(
        {
            id: { eq: address },
            balance: { gt: "0" }
        },
        'id balance'
    );

    const result = await contracts.run({
        address: address,
        functionName: 'get',
        abi: deployee_package.abi,
        input: {},
        keyPair: keys,
    });

    expect(result.output).toEqual({
        value0: '0x1',
        value1: '0x2'
    });
});
```

## Compiled contract

You can see all of the examples compiled in the sample contract below.

The example is based on TON Labs Solidity compiler.

The compiler uses a special type called `TvmCell` to represent an arbitrary tree of cells.

Functions with `tvm_*` prefix are compiler intrinsics (predefined built-in functions). They should have empty bodies.

Below is an example of the  deploying contract and the deployed contract.

Deploying:

```javascript
pragma solidity ^0.5.0;

pragma experimental ABIEncoderV2;

contract ContractCreator {

	struct TvmCell { uint _; }

	function tvm_logstr(bytes32) private pure {}
	function tvm_insert_pubkey(TvmCell memory cellTree, uint256 pubkey) private pure returns(TvmCell memory /*contract*/)  {}
	function tvm_hashcu(TvmCell memory cellTree) pure private returns (uint256) { }
	function tvm_deploy_contract(TvmCell memory my_contract, address addr, uint128 gram,
								uint constuctor_id,
								uint32 constuctor_param0,
								uint constuctor_param1) pure private { }
	function tvm_deploy_contract(TvmCell memory my_contract, address addr, uint128 grams,
								TvmCell memory payload) pure private { }
	function tvm_build_state_init(TvmCell memory /*code*/, TvmCell memory /*data*/) private pure returns (TvmCell memory /*cell*/) { }
	function tvm_accept() private pure {}
	function tvm_make_address(int8 wid, uint256 addr) private pure returns (address payable) {}

	uint m_counter = 0;
	address m_addr;

	modifier alwaysAccept {
		tvm_accept(); _;
	}


	// First variant of contract deploying

	TvmCell m_contract; // StateInit - it's a tree of cells stored code and data

	function setContract(TvmCell memory _contract) public alwaysAccept {
		tvm_logstr("setContr_0");
		m_contract = _contract;
		tvm_logstr("setContr_1");
		m_counter++;
	}

	function deploy(uint256 pubkey, uint128 gram,
					uint constuctor_id, uint32 constuctor_param0, uint constuctor_param1) public alwaysAccept returns (address) {
		tvm_logstr("deploy1_0");
		TvmCell memory contr = tvm_insert_pubkey(m_contract, pubkey);
		address addr = tvm_make_address(0, tvm_hashcu(contr));
		tvm_deploy_contract(contr, addr, gram, constuctor_id, constuctor_param0, constuctor_param1); //create internal msg
		tvm_logstr("deploy1_1");
		m_counter++;
		m_addr = addr;
		return addr;
	}

	// Second variant of contract deploying

	TvmCell m_code;

	function setCode(TvmCell memory _code) public alwaysAccept {
		tvm_logstr("setCode_0");
		m_code = _code;
		tvm_logstr("setCode_1");
		m_counter++;
	}

	function deploy2(TvmCell memory data, uint128 gram, uint constuctor_id,
		             uint32 constuctor_param0, uint constuctor_param1) public alwaysAccept returns (address) {
		tvm_logstr("deploy2_0");
//m_code is previously stated setCode and data is direclty transferred fo the deploy
		TvmCell memory contr = tvm_build_state_init(m_code, data);
		address addr = tvm_make_address(0, tvm_hashcu(contr));
		tvm_deploy_contract(contr, addr, gram, constuctor_id, constuctor_param0, constuctor_param1); //create internal msg
		tvm_logstr("deploy2_1");
		m_counter++;
		m_addr = addr;
		return addr;
	}

	// Third variant of contract deploying

	function deploy3(TvmCell memory contr, address addr, uint128 grams, TvmCell memory payload) public alwaysAccept returns (address) {
		// payload - is body of message
		tvm_logstr("deploy3_0");
		//address addr = address(tvm_hashcu(contr));
		tvm_deploy_contract(contr, addr, grams, payload); //create internal msg
		tvm_logstr("deploy3_1");
		m_counter++;
		m_addr = addr;
		return addr;
	}
}
```

Deployed:

```javascript
pragma solidity ^0.5.0;
pragma experimental ABIEncoderV2;

contract Simple {

    uint32 public param1;
    uint public param2;

    function tvm_accept() private pure {}

    constructor(uint32 _param1, uint _param2) public {
        param1 = _param1;
        param2 = _param2;
    }

    function get() public view returns (uint32, uint) {
        tvm_accept();
        return (param1, param2);
    }
}
```