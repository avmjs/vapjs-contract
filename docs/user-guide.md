# User Guide

All information for developers using `vapjs-contract` should consult this document.

## Install

```
npm install --save vapjs-contract
```

## Usage

```js
const HttpProvider = require('vapjs-provider-http');
const Vap = require('vapjs-query');
const VapContract = require('vapjs-contract');
const vap = new Vap(new HttpProvider('http://localhost:8545'));
const contract = new VapContract(vap);

const SimpleStore = contract(abi, bytecode, defaultTxObject);
const simpleStore = SimpleStore.at('0x000...');
const simpleStore = SimpleStore.new((error, result) => {
  // result null '0x928sdfk...'
});

simpleStore.set(45000, (error, result) => {
  // result null '0x2dfj24...'
});

simpleStore.get().catch((error) => {
  // error null
}).then(result) => {
  // result <BigNumber ...>
});

const filter = simpleStore.SetComplete((error, result) => {
  // result null <BigNumber ...> filterId
});
filter.watch().then((result) => {
  // result null FilterResult {...}
});
filter.stopWatching((error, result) => {
  // result null Boolean filterUninstalled
});
```

## API Design

### constructor

[index.js:vapjs-contract](../../../blob/master/src/index.js "Source code on GitHub")

Intakes an `vapjs-query` instance, outputs a single `contract` instance.

**Parameters**

-   `vap` **Object** a single `vapjs-query` `Vap` instance for RPC formatting, and querying.

Result `contract` **Object**.

```js
const HttpProvider = require('vapjs-provider-http');
const Vap = require('vapjs-query');
const VapContract = require('vapjs-contract');
const vap = new Vap(new HttpProvider('http://localhost:8545'));
const contract = new VapContract(vap);
```

### contract

[index.js:vapjs-contract](../../../blob/master/src/index.js "Source code on GitHub")

Intakes the contract Vapory standard ABI schema, optionally the contract bytecode and default transaction object. Outputs a `ContractFactory` instance for the contract.

**Parameters**

-   `abi` **Array** a single Vapory standard contract ABI array
-   `bytecode` **String** [optional] the contract bytecode as a single alphanumeric hex string
-   `defaultTxObject` **Object** [optional] a single default transaction object

Result `ContractFactory` **Object**.

```js
const HttpProvider = require('vapjs-provider-http');
const Vap = require('vapjs-query');
const VapContract = require('vapjs-contract');
const vap = new Vap(new HttpProvider('http://localhost:8545'));
const contract = new VapContract(vap);

// the abi
const SimpleStoreABI = JSON
.parse('[{"constant":false,"inputs":[{"name":"_value","type":"uint256"}],"name":"set","outputs":[{"name":"","type":"bool"}],"payable":false,"type":"function"},{"constant":true,"inputs":[],"name":"get","outputs":[{"name":"storeValue","type":"uint256"}],"payable":false,"type":"function"},{"inputs":[],"payable":false,"type":"constructor"},{"anonymous":false,"inputs":[{"indexed":false,"name":"_newValue","type":"uint256"},{"indexed":false,"name":"_sender","type":"address"}],"name":"SetComplete","type":"event"}]');

// bytecode
const SimpleStoreBytecode = '606060405234610000575b5b5b61010e8061001a6000396000f360606040526000357c01000000000000000000000000000000000000000000000000000000009004806360fe47b1146100435780636d4ce63c14610076575b610000565b346100005761005e6004808035906020019091905050610099565b60405180821515815260200191505060405180910390f35b3461000057610083610103565b6040518082815260200191505060405180910390f35b6000816000819055507f10e8e9bc5a1bde3dd6bb7245b52503fcb9d9b1d7c7b26743f82c51cc7cce917d60005433604051808381526020018273ffffffffffffffffffffffffffffffffffffffff1681526020019250505060405180910390a1600190505b919050565b600060005490505b9056';

vap.accounts().then((accounts) => {
  const SimpleStore = contract(SimpleStoreABI, SimpleStoreBytecode, {
    from: accounts[0],
    gas: 300000,
  });

  // create a new contract
  const simpleStore = SimpleStore.new((error, result) => {
    // result null '0x928sdfk...' (i.e. the transaction hash)
  });

  // setup an instance of that contract
  const simpleStore = SimpleStore.at('0x000...');
});
```

### ContractFactory.new

[index.js:vapjs-contract](../../../blob/master/src/index.js "Source code on GitHub")

The contract factory has two methods, 'at' and 'new' which can be used to create the contract instane. the `at` method is used to create a `Contract` instance for a contract that has already been deployed to the Vapory blockchain (testnet, livenet, local or otherwise). The `new` method is used to deploy the contract to the current chain.

**Parameters**

-   [`params`] **Various** the contract constructor input paramaters, if any have been specified, these can be of various types, lengths and requirements depending on the contract constructor.
-   `txObject` **Object** [optional] a web3 standard transaciton JSON object
-   `callback` **Function** [optional] a standard async callback which is fired when the contract has either been created or the transaction has failed.

Result a single Promise **Object** instance.


```js
const HttpProvider = require('vapjs-provider-http');
const Vap = require('vapjs-query');
const VapContract = require('vapjs-contract');
const vap = new Vap(new HttpProvider('http://localhost:8545'));
const contract = new VapContract(vap);

// the abi
const SimpleStoreABI = JSON
.parse('[{"constant":false,"inputs":[{"name":"_value","type":"uint256"}],"name":"set","outputs":[{"name":"","type":"bool"}],"payable":false,"type":"function"},{"constant":true,"inputs":[],"name":"get","outputs":[{"name":"storeValue","type":"uint256"}],"payable":false,"type":"function"},{"inputs":[],"payable":false,"type":"constructor"},{"anonymous":false,"inputs":[{"indexed":false,"name":"_newValue","type":"uint256"},{"indexed":false,"name":"_sender","type":"address"}],"name":"SetComplete","type":"event"}]');

// bytecode
const SimpleStoreBytecode = '606060405234610000575b5b5b61010e8061001a6000396000f360606040526000357c01000000000000000000000000000000000000000000000000000000009004806360fe47b1146100435780636d4ce63c14610076575b610000565b346100005761005e6004808035906020019091905050610099565b60405180821515815260200191505060405180910390f35b3461000057610083610103565b6040518082815260200191505060405180910390f35b6000816000819055507f10e8e9bc5a1bde3dd6bb7245b52503fcb9d9b1d7c7b26743f82c51cc7cce917d60005433604051808381526020018273ffffffffffffffffffffffffffffffffffffffff1681526020019250505060405180910390a1600190505b919050565b600060005490505b9056';

vap.accounts().then((accounts) => {
  const SimpleStore = contract(SimpleStoreABI, SimpleStoreBytecode, {
    from: accounts[0],
    gas: 300000,
  });

  // create a new contract
  SimpleStore.new((error, result) => {
    // result null '0x928sdfk...' (i.e. the transaction hash)
  });
});
```

### ContractFactory.at

[index.js:vapjs-contract](../../../blob/master/src/index.js "Source code on GitHub")

The contract factory has two methods, 'at' and 'new' which can be used to create the `Contract` instane.

**Parameters**

-   `address` **String** a single 20 byte alphanumeric hex string contract address

Result a single `Contract` **Object** instance.

```js
const HttpProvider = require('vapjs-provider-http');
const Vap = require('vapjs-query');
const VapContract = require('vapjs-contract');
const vap = new Vap(new HttpProvider('http://localhost:8545'));
const contract = new VapContract(vap);

// the abi
const SimpleStoreABI = JSON
.parse('[{"constant":false,"inputs":[{"name":"_value","type":"uint256"}],"name":"set","outputs":[{"name":"","type":"bool"}],"payable":false,"type":"function"},{"constant":true,"inputs":[],"name":"get","outputs":[{"name":"storeValue","type":"uint256"}],"payable":false,"type":"function"},{"inputs":[],"payable":false,"type":"constructor"},{"anonymous":false,"inputs":[{"indexed":false,"name":"_newValue","type":"uint256"},{"indexed":false,"name":"_sender","type":"address"}],"name":"SetComplete","type":"event"}]');

// bytecode
const SimpleStoreBytecode = '606060405234610000575b5b5b61010e8061001a6000396000f360606040526000357c01000000000000000000000000000000000000000000000000000000009004806360fe47b1146100435780636d4ce63c14610076575b610000565b346100005761005e6004808035906020019091905050610099565b60405180821515815260200191505060405180910390f35b3461000057610083610103565b6040518082815260200191505060405180910390f35b6000816000819055507f10e8e9bc5a1bde3dd6bb7245b52503fcb9d9b1d7c7b26743f82c51cc7cce917d60005433604051808381526020018273ffffffffffffffffffffffffffffffffffffffff1681526020019250505060405180910390a1600190505b919050565b600060005490505b9056';

vap.accounts().then((accounts) => {
  const SimpleStore = contract(SimpleStoreABI, SimpleStoreBytecode, {
    from: accounts[0],
    gas: 300000,
  });

  // setup an instance of that contract
  const simpleStore = SimpleStore.at('0x000...');

  // use a method that comes with the contract
  simpleStore.set(45).then((txHash) => {
    console.log(txHash);
  });
});
```

### Contract

[index.js:vapjs-contract](../../../blob/master/src/index.js "Source code on GitHub")

The contract instance is meant to simulate a deployed Vapory contract interface as a javascript object. All specified call methods are attached to this object (as specified by the contract ABI schema array).

In the example below, the SimpleStore contract has methods `set`, `get`, `constructor` and `SetComplete`.

The `get` method is flagged as `constant`, which means it will not make changes to the blockchain. It is purely for getting information from the chain.

However, the `set` method is not constant, which means it can be transacted with and change the blockchain.

The `constructor` method is only used when deploying the contract, i.e. when `.new` is used.

In this contract, the `SetComplete` event is fired when the `set` method has set a new value successfully.

You will notice the `simpleStore` instance makes all these methods available to it.

```js
const HttpProvider = require('vapjs-provider-http');
const Vap = require('vapjs-query');
const VapContract = require('vapjs-contract');
const vap = new Vap(new HttpProvider('http://localhost:8545'));
const contract = new VapContract(vap);

// the abi
const SimpleStoreABI = JSON
.parse('[{"constant":false,"inputs":[{"name":"_value","type":"uint256"}],"name":"set","outputs":[{"name":"","type":"bool"}],"payable":false,"type":"function"},{"constant":true,"inputs":[],"name":"get","outputs":[{"name":"storeValue","type":"uint256"}],"payable":false,"type":"function"},{"inputs":[],"payable":false,"type":"constructor"},{"anonymous":false,"inputs":[{"indexed":false,"name":"_newValue","type":"uint256"},{"indexed":false,"name":"_sender","type":"address"}],"name":"SetComplete","type":"event"}]');

// bytecode
const SimpleStoreBytecode = '606060405234610000575b5b5b61010e8061001a6000396000f360606040526000357c01000000000000000000000000000000000000000000000000000000009004806360fe47b1146100435780636d4ce63c14610076575b610000565b346100005761005e6004808035906020019091905050610099565b60405180821515815260200191505060405180910390f35b3461000057610083610103565b6040518082815260200191505060405180910390f35b6000816000819055507f10e8e9bc5a1bde3dd6bb7245b52503fcb9d9b1d7c7b26743f82c51cc7cce917d60005433604051808381526020018273ffffffffffffffffffffffffffffffffffffffff1681526020019250505060405180910390a1600190505b919050565b600060005490505b9056';

vap.accounts().then((accounts) => {
  const SimpleStore = contract(SimpleStoreABI, SimpleStoreBytecode, {
    from: accounts[0],
    gas: 300000,
  });

  // setup an instance of that contract
  const simpleStore = SimpleStore.at('0x000...');

  simpleStore.set(45000, (error, result) => {
    // result null '0x2dfj24...'
  });

  simpleStore.get().catch((error) => {
    // error null
  }).then(result) => {
    // result <BigNumber ...>
  });

  const filter = simpleStore.SetComplete().new((error, result) => {
    // result null <BigNumber ...> filterId
  });
  filter.watch().then((result) => {
    // result null FilterResult {...} (will only fire once)
  });
  filter.uninstall((error, result) => {
    // result null Boolean filterUninstalled
  });
});
```

## Promises And callbacks

All `Contract` object prototype methods support both promises and callbacks. `EventFilter` objects also have complete promise support.

## Why BN.js?

`vapjs` has made a policy of using `BN.js` across all of its repositories. Here are some of the reasons why:

  1. lighter than alternatives (BigNumber.js)
  2. faster than most alternatives, see [benchmarks](https://github.com/indutny/bn.js/issues/89)
  3. used by the Vapory foundation across all [`vaporyjs`](https://github.com/vaporycojs) repositories
  4. is already used by a critical JS dependency of many vapory packages, see package [`elliptic`](https://github.com/indutny/elliptic)
  5. purposefully **does not support decimals or floats numbers** (for greater precision), remember, the Vapory blockchain cannot and will not support float values or decimal numbers.

## Browser Builds

`vapjs` provides production distributions for all of its modules that are ready for use in the browser right away. Simply include either `dist/vapjs-contract.js` or `dist/vapjs-contract.min.js` directly into an HTML file to start using this module. Note, an `VapContract` object is made available globally.

```html
<script type="text/javascript" src="vapjs-contract.min.js"></script>
<script type="text/javascript">
new VapContract(...);
</script>
```

Note, even though `vapjs` should have transformed and polyfilled most of the requirements to run this module across most modern browsers. You may want to look at an additional polyfill for extra support.

Use a polyfill service such as `Polyfill.io` to ensure complete cross-browser support:
https://polyfill.io/

## Latest Webpack Figures

```
Hash: 7a3fde336c393e27e46b                  
Version: webpack 2.1.0-beta.15
Time: 891ms
                Asset    Size  Chunks             Chunk Names
    vapjs-contract.js  204 kB       0  [emitted]  main
vapjs-contract.js.map  249 kB       0  [emitted]  main
    + 17 hidden modules

Version: webpack 2.1.0-beta.15
Time: 3276ms
                Asset     Size  Chunks             Chunk Names
vapjs-contract.min.js  89.4 kB       0  [emitted]  main
    + 17 hidden modules
```

## Other Awesome Modules, Tools and Frameworks

### Foundation
 - [web3.js](https://github.com/vaporyco/web3.js) -- the original Vapory JS swiss army knife **Vapory Foundation**
 - [vaporyjs](https://github.com/vaporycojs) -- critical vapory javascript infrastructure **Vapory Foundation**
 - [browser-solidity](https://vapory.github.io/browser-solidity) -- an in browser Solidity IDE **Vapory Foundation**

### Nodes
  - [geth](https://github.com/vaporyco/go-vapory) Go-Vapory
  - [parity](https://github.com/ethcore/parity) Rust-Vapory build in Rust
  - [testrpc](https://github.com/vaporycojs/testrpc) Testing Node (vaporyjs-vm)

### Testing
 - [wafr](https://github.com/silentcicero/wafr) -- a super simple Solidity testing framework
 - [truffle](https://github.com/ConsenSys/truffle) -- a solidity/js dApp framework
 - [embark](https://github.com/iurimatias/embark-framework) -- a solidity/js dApp framework
 - [dapple](https://github.com/nexusdev/dapple) -- a solidity dApp framework
 - [chaitherium](https://github.com/SafeMarket/chaithereum) -- a JS web3 unit testing framework
 - [contest](https://github.com/DigixGlobal/contest) -- a JS testing framework for contracts

### Wallets
 - [ethers-wallet](https://github.com/ethers-io/ethers-wallet) -- an amazingly small Vapory wallet
 - [metamask](https://metamask.io/) -- turns your browser into an Vapory enabled browser =D

## Our Relationship with Vapory & VaporyJS

 We would like to mention that we are not in any way affiliated with the Vapory Foundation or `vaporyjs`. However, we love the work they do and work with them often to make Vapory great! Our aim is to support the Vapory ecosystem with a policy of diversity, modularity, simplicity, transparency, clarity, optimization and extensibility.

 Many of our modules use code from `web3.js` and the `vaporyjs-` repositories. We thank the authors where we can in the relevant repositories. We use their code carefully, and make sure all test coverage is ported over and where possible, expanded on.
