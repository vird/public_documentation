Follow guidelines in this section to create your own environment based on Node.js with the Node SE tools and to develop your first test contract. 

> Note that this guide is also good for React Native and WebAssembly. The only difference is the source library used.

First and foremost, create a local folder for all your test projects. Note that in this documentation the sample folder is called **ton-dev**. If your folder has a different name, make sure to edit the code accordingly.

## Creating a contract

1. Create "hello" folder and place the "hello.sol" contract source code into it:

```javascript
pragma solidity ^0.5.4;

contract HelloTON {
    uint32 timestamp;

    modifier alwaysAccept {
      tvm_accept();
      _;
    }

    function tvm_accept() private pure {}

    constructor() public {
        timestamp = uint32(now);
    }

    function touch() external alwaysAccept {
        timestamp = uint32(now);
    }

    function sayHello() external view returns (uint32) {
        return timestamp;
    }
}
```

> ​    **Note**:  If you do not use the tvm_accept method, use runLocal method to call all functions that do not change the contract state.



<iframe class="no-border max-full-width full-height flex-grow" src="https://www.youtube.com/embed/TyMOi1kLz20?autohide=1&amp;showinfo=0&amp;rel=0&amp;fs=0" style="max-width: 100%; height: 346px; -webkit-box-flex: 1; flex-grow: 1; border: none;"></iframe>
2. Call `cd hello` to navigate to the new folder.

3. Run TON labs Sol2TVM compiler:

```shell
tondev sol hello -l js -L deploy
```

`-l js` option is used to generate JavaScript client helper code for the compiled contract.

`-L deploy is` used to include an `imageBase64` field into the generated JavaScript contract client code.

L deploy with js flag generates a .js file with a contract wrapper class. This wrapper contains methods for deploying and calling the contract in a friendly format. There is the **deploy** method for constructor message (for more details check the [API description](SDK/SDK API/Contract Management)). Two class methods are implemented for each ABI function; one is named as the function itself, the other as `function_nameLocal`. The class has the package static variable that stores two fields: ABI and imageBase64.

##### Sample 1

```javascript
const hello = new HelloContract(client);
await hello.deploy();
console.log(hello.address, hello.keys);
const response = await hello.sayHello();
```

##### Sample 2

```javascript
const existingKeys = { secret: ..., public: ...};
const existingAddress = '...';
const hello = new HelloContract(client, existingAddress, existingKeys);
const response = await hello.sayHello();
```

To generate the contractPackage.js file manually, call `tondev gen -h` command of the CLI.

## Creating an app

Before being able to play with a smart contract on the blockchain, we need a blockchain infrastructure for contract testing and debugging.

Make sure that you started a local node instance according to the guidelines provided in the [**Installation**](/SDK/Installation/) section.

```shell
tondev start
```



Let's start with Node.js to show how to build a test application, deploy and run its smart contract.

> **Note:** To create an application according to this procedure, you have to install Node.js. It is recommended to have the latest version.

1. Let's initialize a Node.js application:

```shell
~/ton-dev/hello$ touch index.js
~/ton-dev/hello$ npm init
```

 Add a section to package.json and execute (run from the in the project folder):

```shell

npm install --save ton-client-node-js@latest
```

2. Open the "index.js" file in your preferred editor and enter the code below. Note that in servers you specify the local node address. 

```javascript
const client = new TONClient();
    client.config.setData({
        servers: ['<http://0.0.0.0>']
    });
```

> **Important**: for Windows use <http://127.0.0.1/> or [http://localhost](http://localhost/).

If you use TON Labs testnet, specify `'<https://testnet.ton.dev>'`:

```javascript
  const client = new TONClient();
    client.config.setData({
        servers: ['<https://https://testnet.ton.dev>']
    });
```

> In JS Client you can simultaneously use several nodes. Create a separate TONClient object for each connection.

3. Open the "index.js" file in your preferred editor and enter the code below.

```javascript
const { TONClient } = require('ton-client-node-js');

async function main(client) {
}

(async () => {
    try {
        const client = new TONClient();
        client.config.setData({
            servers: ['NodeSE/Testnet URL']
        });
        await client.setup();
        await main(client);
        console.log('Hello TON Done');
    process.exit(0);
    } catch (error) {
        console.error(error);
    }
})();
```

4. Run the client app as follows:

```shell
~/ton-dev/hello$ node index.js
Hello TON Done
~/ton-dev/hello$

```

## Deployment

Before a contract is deployed, it has to be defined in your node.js application. The necessary elements are: a compatible TVM code and an ABI structure. Both elements were obtained at the compilation stage before in the `helloPackage.js`file.

> **Tip**: For more details on the ABI, see the Specification.

For deployment, you also have to take the following steps:

1. Generate the key pair. Each time a contract is deployed, you can generate keys with a built-in `ton.crypto.ed25519Keypair` crypto module.

2. The contract is almost ready for deployment, but **in TON blockchain you must deposit GRAMs to the address of the deployed contract before the actual deploy**. **Otherwise deploy will fail.** 

You can send Grams from another contract or use our giver. To learn how to use giver, check the relevant section in the document covering the [Contracts](SDK/Client Libraries/Library Modules/Contracts/) module. There is a detailed usage example. 

```javascript
...

async function main(client) {
    const helloKeys = await client.crypto.ed25519Keypair();
    const helloAddress = (await client.contracts.deploy({
        package: HelloContract.package,
        constructorParams: {},
        keyPair: helloKeys,
    })).address;
    console.log(`Hello contract was deployed at address: ${helloAddress}`);
}

...
```

Now run to check how it works.

```shell
~/ton-dev/hello$ node index.js
Hello contract was deployed at address: 516c7a2bc72c5728526eb73064da07a2876d964c3da5ed2488e1aba3da20be3f
~/ton-dev/hello$
```

## Running a contract

Running your contract on blockchain is also quite simple

```javascript
...

async function main(client) {
    const helloAddress = (await client.contracts.deploy({
        package: HelloContract.package,
        constructorParams: {},
        keyPair: helloKeys,
    })).address;
    console.log(`Hello contract was deployed at address: ${helloAddress}`);

    const response = await client.contracts.run({
        address: helloAddress,
        abi: HelloContract.package.abi,
        functionName: 'touch',
        input: {},
        keyPair: helloKeys,
    });
    console.log('Hello contract was responded to touch:', response.transaction.id);}
}

...
```

Now run the app:

```shell
~/ton-dev/hello$ node index.js
Hello contract was deployed at address: 516c7a2bc72c5728526eb73064da07a2876d964c3da5ed2488e1aba3da20be3f
Hello contract was responded to touch: b930dc72640d2ff9a9b2acedf3935706570f81f32d9171d41c55a0ae75adc909
Hello TON Done
~/ton-dev/hello$
```

Alternatively, you can run a contract in the TVM instance included into a client library without interaction with TVM node:

```javascript
...

async function main(client) {
    const helloAddress = (await client.contracts.deploy({
        package: HelloContract.package,
        constructorParams: {},
        keyPair: helloKeys,
    })).address;
    console.log(`Hello contract was deployed at address: ${helloAddress}`);

    const response = await client.contracts.run({
        address: helloAddress,
        abi: HelloContract.package.abi,
        functionName: 'touch',
        input: {},
        keyPair: helloKeys,
    });
    console.log('Hello contract was responded to touch:', response.transaction.id);}

    const localResponse = await client.contracts.runLocal({
        address: helloAddress,
        abi: HelloContract.package.abi,
        functionName: 'sayHello',
        input: {},
        keyPair: helloKeys,
    });
    console.log('Hello contract was ran on a client TVM and also responded to sayHello:', localResponse);
}

...
```

Run:

```shell
~/ton-dev/hello$ node index.js
Hello contract was deployed at address: 516c7a2bc72c5728526eb73064da07a2876d964c3da5ed2488e1aba3da20be3f
Hello contract was responded to touch: b930dc72640d2ff9a9b2acedf3935706570f81f32d9171d41c55a0ae75adc909
Hello contract was ran on a client TVM and also responded to sayHello: { output: { value0: '0x5d6fba2e' } }
Hello TON Done
~/ton-dev/hello$
```



> Find more information about deploying and running in the [**Contracts**](SDK/Client Libraries/Library Modules/Contracts/) section.

## Querying blockchain

Each node server is equipped with a database that tracks the relevant blockchain.

This database is accessible through a GraphQL based protocol for querying blockchain.

The Client library contains the Query Module designed to perform GraphQL queries over a blockchain. The simplest way to query a blockchain is using the following `query` method:

```javascript
async function queries(client) {
    const transactions = await client.queries.transactions.query({}, 'id now status');
    console.log('All Transactions: ', transactions);    
}
```

Then all transactions in the relevant blockchain are displayed (the first 50, to be precise). 50  is the default number used when no limit is specified or when it exceeds 50.

For each transaction, we have three result fields: **id**, **now** and **status**.

We have several options to filter the results:

```javascript
const transactions = await client.queries.transactions.query({
    now: { eq: 1567601735 }
}, 'id now status');
console.log('Filtered Transactions: ', transactions);
```

The example gets all transactions with **now** equals to 1567601735. 

> Find more information about a filtering in the [**Queries**](SDK/Client Libraries/Library Modules/Queries) section.