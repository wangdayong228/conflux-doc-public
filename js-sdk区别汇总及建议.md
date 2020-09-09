[toc]
# Conflux模块
ETH|CFX|备注
--|--|--
setProvider|	Conflux.prototype.setProvider|web3支持http, ipc, ws, tcp
--	|Conflux.prototype.close|
getChainId|	Conflux.prototype.getStatus|
getGasPrice|	Conflux.prototype.getGasPrice|
getBlockNumber|	Conflux.prototype.getEpochNumber|
-	|Conflux.prototype.getLogs|
getBalance|	Conflux.prototype.getBalance|
getBlockTransactionCount|	Conflux.prototype.getNextNonce|
-	|Conflux.prototype.getConfirmationRiskByHash|
getBlock|	Conflux.prototype.getBlockByEpochNumber|
-	|Conflux.prototype.getBlocksByEpochNumber|
-	|Conflux.prototype.getBestBlockHash|
getBlock|	Conflux.prototype.getBlockByHash|
-	|Conflux.prototype.getBlockByHashWithPivotAssumption|
getTransaction|	Conflux.prototype.getTransactionByHash|
getTransactionReceipt|	Conflux.prototype.getTransactionReceipt|
sendTransaction|	Conflux.prototype.sendTransaction| web3支持事件响应
sendSignedTransaction|	Conflux.prototype.sendRawTransaction|
getCode|	Conflux.prototype.getCode|
call|	Conflux.prototype.call|
estimateGas|	Conflux.prototype.estimateGasAndCollateral|
providers	| |列出支持的provier类型，建议支持
givenProvider	||
currentProvider	||获取当前provider实例， 建议支持
BatchRequest	||批量请求，建议支持
extend	||
defaultAccount	||建议支持
defaultBlock	||
defaultHardfork	||
defaultChain	||
defaultCommon	||
transactionBlockTimeout	||webscodket provider下使用
transactionConfirmationBlocks	||建议支持
transactionPollingTimeout	||http provider下使用，等待transaction被mine的超时块数
handleRevert	||没用
maxListenersWarningThreshold	||
getProtocolVersion	||建议支持，需要rpc支持
isSyncing	||建议支持，需要rpc支持
getCoinbase	||建议支持，需要rpc支持
isMining	||建议支持，需要rpc支持
getHashrate	||建议支持，需要rpc支持
getAccounts	||建议支持
getStorageAt	||建议支持，需要rpc支持
getBlockUncleCount	||
getUncle	||
getPendingTransactions	||建议支持，需要rpc支持
getTransactionFromBlock	||
getTransactionCount	|getNextNonce
signTransaction	|Transaction模块支持|
getWork	||
submitWork	||
requestAccounts	||
getNodeInfo	||建议支持，需要rpc支持
getProof	||

eth默认会在以下方法的返回值中携带错误原因
- web3.eth.call();
- web3.eth.sendTransaction();
- contract.methods.myMethod(…).send(…);
- contract.methods.myMethod(…).call(…)

## 缺失的RPC建议补充

# Contract模块
ETH|	CFX	| 备注
--|--|--
methods.myMethod.call|	contract.mymethod	|
methods.myMethod.call|	contract.mymethod.call	|
decodeParameters	|contract.mymethod.decodeData|	
-	|contract.mymethod.decodeOutputs	|
methods.myMethod.encodeABI|	contract.mymethod.encodeData	|
methods.myMethod.send|	contract.mymethod.send	|
methods.myMethod.estimateGas|	contract.mymethod.estimateGasAndCollateral|	
getPastLogs	|contract.myEvent.getLogs	|
-	|contract.myEvent.encodeTopics	|
decodeLog	|contract.myEvent.decodeLog	|
-	|contract.myEvent.call？| 	作用是什么，是否可以去掉
decodeParameters|	contract.abi.decodeData	|
decodeLog|	contract.abi.decodeLog	|

Web3在call或send失败后会在返回结果中携带错误原因，建议支持。

## new contract options	
ETH|CFX
--|--
address|	address	
jsonInterface|	abi
-	|bytecode
	

## = Properties =	
ETH|备注	
--|--
defaultAccount		|
defaultBlock		|
defaultHardfork		|
defaultChain		|
defaultCommon		|
transactionBlockTimeout		|与Conflux模块中同名属性作用相同
transactionConfirmationBlocks		|与Conflux模块中同名属性作用相同
transactionPollingTimeout		|与Conflux模块中同名属性作用相同
handleRevert		|表示在send或call失败时是否返回revert原因（默认false）

## = Methods =		
ETH|备注
--|--
clone|		
deploy|		建议支持
methods|		建议支持

## = Events =		
ETH|功能|备注
--|--|--
once|		订阅一次事件 |建议支持
events|		订阅事件 |建议支持
events.allEvents| 获取过去该合约的事件并订阅新事件    |建议支持
getPastEvents|	获取过去该合约的事件，可以根据过滤条件筛选, 比getPastLogs更精细	|建议支持

## web3.eth.abi		

ETH|备注
--|--
encodeFunctionSignature		|建议支持
encodeEventSignature		|建议支持
encodeParameter		|建议支持
encodeParameters		|建议支持
encodeFunctionCall		|建议支持
decodeParameter		|建议支持

# Accounts模块
ETH	|CFX|备注
--|--|--
create|	Account.random	
decrypt|	Account.decrypt	
encrypt|	Account.prototype.encrypt	
signTransaction|	Account.prototype.signTransaction	
sign|	Account.prototype.signMessage	
-|	Account.prototype.toString	
sign|	Message.sign	
recover|	Message.recover	
-|	Message.prototype.constructor	
-|	Message.prototype.from (getter)	
sign|	Message.prototype.sign	
-|	Transaction.prototype.constructor	
-|	Transaction.prototype.hash (getter)	
-|	Transaction.prototype.from (getter)	
signTransaction|	Transaction.prototype.sign	
recoverTransaction|	Transaction.prototype.recover	
-|	Transaction.prototype.encode|	rlp编码为16进制
-|	Transaction.prototype.serialize|	这个建议命名为encodeToBytes
privateKeyToAccount|	utils模块支持	
hashMessage|	Message.prototype.hash (getter)	

## wallet模块 
钱包模块，管理多个账户，建议支持
ETH|备注
--|--
wallet.create		|建议支持
wallet.add		|建议支持
wallet.remove		|建议支持
wallet.clear		|建议支持
wallet.encrypt		|建议支持
wallet.decrypt		|建议支持
wallet.save		|建议支持
wallet.load		|建议支持

# Utils模块

ETH|	CFX|	备注
--|--|--
-|	format.any (setter)	|有点奇怪
toHex|	format.hex (setter)	
-|	format.uInt (setter)	
toBN|	format.bigInt (setter)	
-|	format.bigUInt (setter)	
-|	format.hexUInt (setter)	
-|	format.riskNumber (setter)	
-|	format.epochNumber (setter)	
bytesToHex|	format.address (setter)	|建议该方法改为isAddress，另增加bytesToHex
bytesToHex|	format.publicKey (setter)	|建议该方法改为isPublicKey，另增加bytesToHex
bytesToHex|	format.privateKey (setter)	|建议该方法改为isPrivateKey，另增加bytesToHex
bytesToHex|	format.signature (setter)	|建议该方法改为isSignature，另增加bytesToHex
bytesToHex|	format.blockHash (setter)	|建议该方法改为isBlockHash，另增加bytesToHex
bytesToHex|	format.txHash (setter)	|建议该方法改为isTxHash，另增加bytesToHex
toHex| + hexToBytes	format.buffer (setter)	
-|	format.boolean (setter)	
sha3|	sha3	
toChecksumAddress|	checksumAddress	
-|	randomBuffer	
randomHex|	randomPrivateKey	
privateKeyToAccount|(Accounts模块)	privateKeyToPublicKey	
privateKeyToAccount|(Accounts模块)	publicKeyToAddress	
privateKeyToAccount|(Accounts模块)	privateKeyToAddress	
sign（Accounts模块）|	ecdsaSign	
recover（Accounts模块）|	ecdsaRecover	
encrypt|(Accounts模块)	encrypt	
decrypt|(Accounts模块)	decrypt	
-|	unit.fromCFXToGDrip	|javascript bignumber类型为JSBI
toWei|	unit.fromCFXToDrip	| `web3.utils.toWei('1', 'finney');` toWei第二个参数unit表示数据源单位; 即该方法支持所有单位到wei的转换; 建议支持
-|	unit.fromGDripToCFX	
-|	unit.fromGDripToDrip	
fromWei|	unit.fromDripToCFX	| `web3.utils.fromWei('1', 'finney');` fromWei第二个参数表示要转换成什么单位; 即该方法支持wei到所有单位的转换; 建议支持
|	unit.fromDripToGDrip	
unitMap| | 所有ETH单位与其表示以wei为单位的数量的对照表 建议支持
toTwosComplement||	负数转16进制 bignumber
soliditySha3Raw|	
soliditySha3||	使用solidity sha3算法计算哈希值	建议支持
sha3Raw|		
padRight|		
padLeft|		
isHexStrict||与isHex区别为`isHexStrict`要求必须有`0x`前缀
isHex|		|建议支持
isBN|		|建议增加isJSBN
isBigNumber|		|
isAddress|		|建议支持
hexToUtf8|		|建议支持
hexToNumberString|		
hexToNumber|		|建议支持
hexToBytes|		|建议支持
hexToAscii|		|建议支持
checkAddressChecksum|		|建议支持
BN|		|`new BN(1234).toString();` 建议支持JSBN
Bloom Filters||建议支持
utf8ToHex|		|建议支持
asciiToHex|		|建议支持
bytesToHex|		|建议支持
numberToHex|hex支持|		
bytesToHex|		|建议支持

# ETH 中的其它模块
除了以上介绍的模块，web3中还包含了以下模块；这些模块conflux中暂时还没有对应模块，将来可能支持类似功能
web3模块|作用|备注
--|--|--
personal|使用所请求节点的账户进行签名、锁定、解锁等操作|建议支持
shh|用于使用whisper协议传播消息|--
net|获取节点信息|建议支持 需要rpc支持
subscribe|订阅链上产生的新时间，包含`logs`,`pendingTransactions`,`newBlockHeaders`.`syncing`等|基于`websocket provider`支持
ens|用于操作以太坊的域名服务|建议将来支持
Iban|以太坊地址格式与`IBAN(国际银行帐号)` `BBAN(基本银行账号)`地址格式的转换|建议将来支持