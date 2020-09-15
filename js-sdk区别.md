# Web3.js vs js-conflux-sdk
`js-conflux-sdk` 最新版本为`1.x`, 与`0.x`有很大差别, 但由于其还在测试阶段，本文只对`0.13.4`与`Web3`进行对照，待`1.x`版本稳定后, 再针对`1.x`与Web3进行对照。
`Web3` 和 `js-conflux-sdk` 都是最顶层模块，他们包含了其它子模块以及在顶层暴露了一些子模块中的方法方便开发者快捷使用，我们这里只比较各模块，而不再对这些快捷方法做特别说明。

## 模块对比
`Web3` 和 `js-conflux-sdk` 有对应关系的主要有以下模块：

Ethereum中的模块| Conflux中的模块| 功能
--|--|--
Eth | Conflux| 用于管理连接节点，与节点交互，发送rpc请求，包含读取状态、发送交易等
Contract + abi| Contract| 用于操作智能合约，包含创建合约实例、调动合约方法，以及提供一些根据abi编解码的功能
accounts|account + Message + Transaction|用于管理账户，包含创建、删除账户，及使用账户对消息或交易进行签名等操作
utils|util|工具类，提供一些通用函数，例如hex格式转换，单位转换，私钥转公钥及地址，生成随机私钥，判断数据类型，sha3等，方便dapp开发及其他js包使用


### 各模块方法对比
#### Conflux模块 Vs Eth模块 

以下是Conflux模块包含的方法及与Eth模块的对应关系，该模块主要封装了JSONRPC与节点交互，还有一部分JSONRPC暂未实现，将来会实现所有RPC。具体RPC介绍请参见 [Conflux JSONRPC介绍](https://developer.conflux-chain.org/docs/conflux-doc/docs/json_rpc) 及 [Conflux RPC与Ethereum RPC的区别]()
这里的Epoch Number指用于划分一组Conflux区块为一个Epoch，Conflux区块链是以Epoch为顺序组织的。

Conflux Conflux模块 | Ethereum Web3-Eth模块 | 对应RPC|--
--|--|--|--|--
setProvider	|setProvider|--|--
getStatus	|getChainId|cfx_getStatus
getGasPrice	|getGasPrice|cfx_gasPrice
getEpochNumber	|getBlockNumber|cfx_epochNumber
getBalance	|getBalance|cfx_getBalance
getNextNonce	|getBlockTransactionCount|cfx_getNextNonce
getBlockByEpochNumber	|getBlock|cfx_getBlockByEpochNumber
getBlocksByEpochNumber	|-|cfx_getBlocksByEpoch
getBlockByHash	|getBlock|cfx_getBlockByHash
getBlockByHashWithPivotAssumption	|-|cfx_getBlockByHashWithPivotAssumption
getTransactionByHash	|getTransaction|cfx_getTransactionByHash
getTransactionReceipt	|getTransactionReceipt|cfx_getTransactionReceipt
sendTransaction	|sendTransaction|cfx_sendTransaction
sendRawTransaction	|sendSignedTransaction|cfx_sendRawTransaction
getCode	|getCode|cfx_getCode
call	|call|cfx_call
estimateGasAndCollateral	|estimateGas|cfx_estimateGasAndCollateral
getLogs	| getPastLogs(contract模块)|cfx_getLogs
getBestBlockHash	|-|cfx_getBestBlockHash
getConfirmationRiskByHash	|-|cfx_getConfirmationRiskByHash
close	|-|-


#### Contract 模块对比

Conflux Contract 模块 | Eth Contract + abi 模块 
--|--
contract.mymethod|	methods.myMethod.call
contract.mymethod.call|	methods.myMethod.call
contract.mymethod.decodeData|	decodeParameters
contract.mymethod.decodeOutputs|-	
contract.mymethod.encodeData|	methods.myMethod.encodeABI
contract.mymethod.send|	methods.myMethod.send
contract.mymethod.estimateGasAndCollateral|	methods.myMethod.estimateGas
contract.myEvent.getLogs|	getPastLogs
contract.myEvent.encodeTopics|-	
contract.myEvent.decodeLog|	decodeLog(ABI模块)
contract.abi.decodeData|	decodeParameters
contract.abi.decodeLog|	decodeLog(ABI模块)

#### accounts 模块对比
Conflux Account模块|Ethereum Accounts模块
--|--
random	|create
decrypt	|decrypt
encrypt	|encrypt
signTransaction	|signTransaction
signMessage	|sign

Conflux Message模块	|Ethereum Accounts模块
--|--
sign	|sign
recover	|recover
hash (getter)	|hashMessage
from (getter)	|-
sign	|sign

Conflux Transaction模块	|Ethereum Accounts模块
--|--
hash (getter)	|-
from (getter)	|-
sign	|signTransaction
recover	|recoverTransaction
encode	|-
serialize	|-


#### utils 模块对比
Conflux util 模块| Ethereum utils模块
--|--
format.any (setter)	|-
format.hex (setter)	|toHex, numberToHex|
format.uInt (setter)|-	
format.bigInt (setter)	|toBN
format.bigUInt (setter)|-
format.hexUInt (setter)|-	
format.riskNumber (setter)|-
format.epochNumber (setter)|-
format.address (setter)	|bytesToHex
format.publicKey (setter)|	bytesToHex
format.privateKey (setter)|	bytesToHex
format.signature (setter)|	bytesToHex
format.blockHash (setter)|	bytesToHex
format.txHash (setter)	|bytesToHex
format.buffer (setter)	|toHex + hexToBytes
format.boolean (setter)|-
sha3|	sha3
checksumAddress|	toChecksumAddress
randomBuffer|-
randomPrivateKey|	randomHex
privateKeyToPublicKey|	privateKeyToAccount(Accounts模块)
publicKeyToAddress|	privateKeyToAccount(Accounts模块)
privateKeyToAddress|	privateKeyToAccount(Accounts模块)
ecdsaSign|	sign(Accounts模块)
ecdsaRecover|	recover(Accounts模块)
encrypt|encrypt(Accounts模块)
decrypt|decrypt(Accounts模块)
unit.fromCFXToGDrip|-
unit.fromCFXToDrip|	toWei
unit.fromGDripToCFX|-	
unit.fromGDripToDrip|-	
unit.fromDripToCFX|fromWei
unit.fromDripToGDrip|-	





## 使用对比

### 初始化
#### 初始化Web3实例
```javascript
var Web3 = require('web3');
// "Web3.providers.givenProvider" will be set if in an Ethereum supported browser.
var web3 = new Web3(Web3.givenProvider || 'ws://some.local-or-remote.node:8546');
// or
var web3 = new Web3(new Web3.providers.HttpProvider('http://localhost:8545'));

```
#### 初始化Conflux实例
Conflux实例化与web3类似，但当前版本还不支持 `webscoket provider`
```javascript
const { Conflux } = require('js-conflux-sdk');
const cfx = new Conflux({url:'http://testnet-jsonrpc.conflux-chain.org:12537'});
```
### 读状态
#### Ethereum读状态
```javascript
web3.eth.getBalance("0x107d73d8a49eeb85d32cf465507dd71d507100c1").then(console.log);
> "1000000000000"
```
#### Conflux读状态
```javascript
await cfx.getBalance("0x107d73d8a49eeb85d32cf465507dd71d507100c1");
// or
cfx.getBalance("0x107d73d8a49eeb85d32cf465507dd71d507100c1").then(console.log)
> "1000000000000"
```

### 发送交易
#### web3 发送交易
web3发送交易后通过`event`方式在以下各个阶段达成时通知：
- `transactionHash` 交易已发送
- `receipt` 交易已执行
- `confirmation` 交易已确认
- `error` 交易执行失败
```javascript
// compiled solidity source code using https://remix.ethereum.org
var code = "603d80600c6000396000f3007c01000000000000000000000000000000000000000000000000000000006000350463c6888fa18114602d57005b6007600435028060005260206000f3";

// using the callback
web3.eth.sendTransaction({
    from: '0xde0B295669a9FD93d5F28D9Ec85E40f4cb697BAe',
    data: code // deploying a contracrt
}, function(error, hash){
    ...
});

// using the promise
web3.eth.sendTransaction({
    from: '0xde0B295669a9FD93d5F28D9Ec85E40f4cb697BAe',
    to: '0x11f4d0A3c12e86B4b5F39B213F7E19D048276DAe',
    value: '1000000000000000'
})
.then(function(receipt){
    ...
});

// using the event emitter
web3.eth.sendTransaction({
    from: '0xde0B295669a9FD93d5F28D9Ec85E40f4cb697BAe',
    to: '0x11f4d0A3c12e86B4b5F39B213F7E19D048276DAe',
    value: '1000000000000000'
})
.on('transactionHash', function(hash){
    ...
})
.on('receipt', function(receipt){
    ...
})
.on('confirmation', function(confirmationNumber, receipt){ ... })
.on('error', console.error); // If a out of gas error, the second parameter is the receipt.
```

#### Conflux发送交易

js-conflux-sdk 由于当前版本没有钱包系统，所以发送交易时需要直接指定`Account`; `sendTransaction`返回的是一个 `Promise.<PendingTransaction>`对象, 可以直接`await`获取`transaction hash`；

也可以通过该对象方法在不同状态下返回`transaction`或`transaction receipt`, 方法列举如下：
- `get`在发送完成后返回`transaction`,
- `mined`在打包完成后返回`transaction`,
- `executed`在执行完成后返回`transaction receipt`,
- `confirmed`在`transaction risk < threshold`时返回`transaction receipt`

```javascript
const account = cfx.Account(your_private_key);

const promise = cfx.sendTransaction({ // Not await here, just get promise
      from: account,
      to: "0x11f4d0A3c12e86B4b5F39B213F7E19D048276DAe",
      value: Drip.fromCFX(0.007),
    });
> await promise; // transaction
   "0x91fbdfb33f3a585f932c627abbe268c7e3aedffc1633f9338f9779c64702c688"
> await promise.get(); // get transaction
   {
    "blockHash": null,
    "transactionIndex": null,
    "hash": "0x91fbdfb33f3a585f932c627abbe268c7e3aedffc1633f9338f9779c64702c688",
    ...
   }
> await promise.mined(); // wait till transaction mined
   {
    "blockHash": "0xe9b22ce311003e26c7330ac54eea9f8afea0ffcd4905828f27c9e2c02f3a00f7",
    "transactionIndex": 0,
    "hash": "0x91fbdfb33f3a585f932c627abbe268c7e3aedffc1633f9338f9779c64702c688",
    ...
   }
> await promise.executed(); // wait till transaction executed in right status. and return it's receipt.
   {
    "blockHash": "0xe9b22ce311003e26c7330ac54eea9f8afea0ffcd4905828f27c9e2c02f3a00f7",
    "index": 0,
    "transactionHash": "0x91fbdfb33f3a585f932c627abbe268c7e3aedffc1633f9338f9779c64702c688",
    "outcomeStatus": 0,
    ...
   }
> await promise.confirmed(); // wait till transaction risk coefficient '<' threshold.
   {
    "blockHash": "0xe9b22ce311003e26c7330ac54eea9f8afea0ffcd4905828f27c9e2c02f3a00f7",
    "index": 0,
    "transactionHash": "0x91fbdfb33f3a585f932c627abbe268c7e3aedffc1633f9338f9779c64702c688",
    "outcomeStatus": 0,
    ...
   }
```
##### 需要注意的地方：
发送交易时，建议先使用 `estimateGasAndCollateral` 返回预估的gas使用数量和存储抵押数量；然而由于实际执行消耗与预估结果有差异，为了防止交易执行失败，建议在实际发送交易时，设置`gaslimit`与`storage_limit`的值为`预估值 * 4/3`。

参数EpochHeight表示这笔交易将会在 `Epoch` 为 `[EpochHeight-100000 , EpochHeight+100000]` 的区间内执行，当超出这个区间这笔交易将被丢弃，建议设置当前Epoch值即可。

### 部署合约
部署合约实质就是发送一笔`data`为合约`bytecode`，`to`为`null`的交易

#### web3部署合约
web3可以通过直接发送交易的方式部署，也可以通过contract实例部署
```javascript
// send transaction directly
await web3.eth.sendTransaction({
    from: '0xde0B295669a9FD93d5F28D9Ec85E40f4cb697BAe',
    data: contract_bytecode,
})
> {
  "status": true,
  "contractAddress": "0x11f4d0A3c12e86B4b5F39B213F7E19D048276DAe",
  ...
}

// or use contract instance
var myContract = new web3.eth.Contract(contract_abi, '0xde0B295669a9FD93d5F28D9Ec85E40f4cb697BAe', {
    from: '0x1234567890123456789012345678901234567891', // default from address
    ...
});
myContract.deploy({
    data: '0x12345...',
    arguments: [123, 'My String']
})
.send({
    from: '0x1234567890123456789012345678901234567891',
    gas: 1500000,
    gasPrice: '30000000000000'
}, function(error, transactionHash){ ... })
.on('error', function(error){ ... })
.on('transactionHash', function(transactionHash){ ... })
.on('receipt', function(receipt){
   console.log(receipt.contractAddress) // contains the new contract address
})
.on('confirmation', function(confirmationNumber, receipt){ ... })
.then(function(newContractInstance){
    console.log(newContractInstance.options.address) // instance with the new contract address
});
```
#### conflux部署合约
conflux当前只能通过发送交易的方式部署，`transaction receipt`的`contractCreated`字段就是部署后的合约地址
```javascript
const account = cfx.Account(your_private_key);

await cfx.sendTransaction({ // Not await here, just get promise
      from: account,
      data: contract_bytecode,
    }).executed();
>  {
   "outcomeStatus": 0,
   "contractCreated": "0x11f4d0A3c12e86B4b5F39B213F7E19D048276DAe",
   ...
   }
```

### 调用合约
调用合约分两种:
- 一种是读状态，不需要发送交易，读状态web3跟conflux都使用call方法。
- 一种是修改合约状态，需要发送交易，修改合约状态web3使用`send`,`conflux`使用`sendTransaction`
#### web3调用合约
```javascript
// create contract instance
var myContract = new web3.eth.Contract(contract_abi, contract_address);

// calling a method
myContract.methods.myMethod(123).call({from: '0xde0B295669a9FD93d5F28D9Ec85E40f4cb697BAe'}, function(error, result){
    ...
});

// or sending and using a promise
myContract.methods.myMethod(123).send({from: '0xde0B295669a9FD93d5F28D9Ec85E40f4cb697BAe'})
.then(function(receipt){
    // receipt can also be a new contract instance, when coming from a "contract.deploy({...}).send()"
    ...
});
```
#### conflux调用合约
conflux 调用合约与web3类似，需要注意的地方是调用合约发送交易前建议根据estimateGasAndCollateral得到的gasUsed与storageCollateralized计算gas（同以太坊的gasLimit）与storageLimit; 由于实际执行交易时使用的gas与storageCollateralized与预估可能会有差别，为了防止失败，建议设置 `gas = estimated.gasUsed * 4 / 3`,`storageLimit = estimated.storageCollateralized * 4 /3`
```javascript
// create contract instance
var myContract = cfx.Contract({
    abi: contract_abi,
    address: contract_address
});

// call
await contract.myCallMethod(123)
await contract.myCallMethod().call(123)

// send
// set gasLimit and sotrageLimit large 1/3 than estimated value
const estimated = await contract.mySendMethod(123).estimateGasAndCollateral({from: '0x1e0B295669a9FD93d5F28D9Ec85E40f4cb697BAe'});
let gasLimit = JSBI.multiply(estimated.gasUsed, JSBI.BigInt(4))
gasLimit = JSBI.divide(gasLimit, JSBI.BigInt(3))
let storageLimit = JSBI.multiply(estimated.storageCollateralized, JSBI.BigInt(4))
storageLimit = JSBI.divide(storageLimit, JSBI.BigInt(3))

// send transaction
await contract.mySendMethod(123).sendTransaction({from: "addres",gasLimit:estimated.gas*4/3}).executed();
```

## eth 中的其它模块
除了以上介绍的模块，web3中还包含了以下模块；这些模块conflux中暂时还没有对应模块，将来可能支持类似功能
web3模块|作用
--|--
personal|使用所请求节点的账户进行签名、锁定、解锁等操作
shh|用于使用whisper协议传播消息
bzz|用于与swarm交互，swarm是一个分布式文件存储系统
net|获取节点信息
subscribe|订阅链上产生的新时间，包含`logs`,`pendingTransactions`,`newBlockHeaders`.`syncing`等
ens|用于操作以太坊的域名服务
Iban|以太坊地址格式与`IBAN(国际银行帐号)` `BBAN(基本银行账号)`地址格式的转换


## 文章引用
1. [js-conflux-sdk 用户手册](https://developer.conflux-chain.org/docs/js-conflux-sdkjavascript_sdk)
2. [Web3 v1.2.11 用户手册](https://web3js.readthedocs.io/en/v1.2.11/web3-eth-contract.html)
3. [Conflux JSONRPC介绍](https://developer.conflux-chain.org/docs/conflux-doc/docs/json_rpc) 
4. [Conflux RPC与Ethereum RPC的区别]()
<!-- 
废话：
从以下几个方面比较
1. sdk功能分几个模块，
js-conflu-sdk (version)：Conflux(rpc)，account，contract，util；Message, Transaction, format(包含在util中)
web3js(version 1.2.11): web3.eth, web3.eth.net, web3.eth.personal, web3.shh, web3.bzz, web3.utils

web3.eth 模块包含了属性 subscribe, Contract, Iban, personal, accounts, ens, abi, net; 这些实例分别对应一个子模块，这些模块后面会分别单独介绍。
这里主要介绍除这些子模块外的功能。





 -->
