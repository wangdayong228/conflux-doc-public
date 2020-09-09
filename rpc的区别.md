

因为Conflux实现结构与Ethereum不同，所以概念上及实现上也有很大的区别，本文主要介绍针对Conflux RPC使用与以太坊RPC使用的区别

## 概念介绍
### Epoch 

以太坊区块链中只有主链上的交易是有效的，所以可以认为以太坊区块链账本是一条单链，从前往后每个区块都有一个编号，叫做区块号（ block number），conflux 开发了一种全新的账本结构: 树图，实现了高吞吐，低延迟。

在树图区块结构中，如果只看父边他是一个 Tree，如果父边引用边都看则是一个 Graph。正是这种结构使得 Conflux 网络可以并发出块，即多个区块可以都在某个区块之后生成。因此在 Conflux 是没有 block number 的概念。 但为了实现全序，Conflux 通过 GHAST 规则从创世区块开始，在其所有子区块中选择最重子树 block 为 pivot block，所有的 pivot block 链到一块也形成一条链 定义为 pivot chain，如果只看 pivot chain 其跟普通的区块链结构一致，在这条链上基于每个 pivot block 定义一个Epoch，因此你可以把 conflux 中的 Epoch 理解为跟 block number 对应的概念，只不过 conflux 中的每个 epoch 中可能会有多个 block。


### Storage Limit

在现实世界中，发送转账交易需要给银行付手续费，在比特币中发送交易需要给矿工付手续费，在以太坊中同样如此。具体来讲，以太坊网络的交易最终是由矿工 运行的 EVM 执行的，gas 是用来衡量一笔交易执行的工作量（可以理解为工作的工时），交易发送者，发送交易时可以指定愿意给每个工作量付的价格即 gasPrice。 因此最终一笔交易的手续费为 gas * gasPrice。 在发送一笔交易时指定的 gas 则是一个限制值，即发送方最大愿意为一笔交易支付 gas 这么多的工时，如果交易需要的工作量超过 gas，则不会再付钱，交易不会被执行。

在 Dapp 系统中，交易执行除了需要矿工进行计算付出计算资源外，还需要矿工存储合约的状态，因此需要付出存储资源。在 Conflux 系统中发送交易时，还需要为状态存储 抵押一部分费用，因此在 conflux 中发送交易时会比以太坊多一个 storageLimit 参数，用于设置愿意为某笔交易存储所抵押的费用上限。在合约释放掉使用的存储空间之后，抵押的费用 也会得到返还。

## RPC 相关参数介绍

### Epoch Number 可选项

当发送RPC请求时，有些RPC需要携带Epoch Number参数， Epoch Number 表示要请求在哪个Epoch的状态。 以下是参数 Epoch Number 的可选项:

格式	|意义
--|--
16进制字符串 | epoch number 整形数值（如"0x1000" 表示 Epoch number 4096）
字符串 "earliest"| 创世块所在的 Epoch高度
字符串 "latest_checkpoint"| 当前checkpoint的第一个Epoch高度
字符串 "latest_state"|  最近执行的区块所在的Epoch高度
字符串 "latest_mined"| 最新挖出的区块所在的Epoch高度

注意，出于性能优化的考虑，最新的Epoch是不可执行的，所以在这些Epoch里没有可用的状态。对于大多数与状态查询有关的 RPCs，建议使用"latest_state"。

### 数据的16进制编码规则

发送RPC请求时，有两种关键数据类型通过JSON传输，无格式的字节数组和数值。 两者都以十六进制编码传输，但格式稍有不同：

#### 1. 对数值编码
当编码数值（整数，数字）时：编码为前缀为"0x"的十六进制，使用最紧凑的表现形式（例外：零表示为"0x0"）。 示例如下：

正确示例：
- 0x41 （十进制数字 65）
- 0x400 （十进制数字 1024）

错误示例：
- 错误: 0x（需至少有一位数字 - 零为 "0x0"）
- 错误: 0x0400（不允许以零开头）
- 错误: ff（必须以 0x 为前缀）

#### 2. 对无格式数据编码
当编码无格式数据（字节数组，帐户地址，哈希值，字节码数组）时也是使用前缀为"0x"的十六进制编码，与数值编码的区别是：每个字节必须使用两位十六进制数字。 示例如下：

正确示例：
- 0x41 (长度为 1，表示字符串"A")
- 0x004200 (长度为 3，表示字符串"\0B\0")
- 0x (长度为0，表示字符串"")

错误示例
- 错误: 0xf0f0f (必须为偶数位数)
- 错误: 004200 (必须以0x开头)

### 设置 GasLimit & StorageLimit & EpochHeight
发送交易时，建议先使用 RPC `cfx_estimateGasAndCollateral` 返回预估的gas使用数量和存储抵押数量；然而由于实际执行消耗与预估结果有差异，为了防止交易执行失败，建议在实际发送交易时，设置gaslimit与storage_limit的值为`预估值 * 4/3`。

参数EpochHeight表示这笔交易将会在 Epoch为 `[EpochHeight-100000 , EpochHeight+100000]` 的区间内执行，当超出这个区间这笔交易将被丢弃，建议设置当前Epoch值即可。


# Conflux VS Ethereum JSONRPC

以下是对 Conflux 与 Ethereum 的JSON RPC的使用比较，Conflux RPC 详细介绍请参见[JSON RPC开发者文档](https://developer.conflux-chain.org/docs/conflux-doc/docs/json_rpc)

## Public RPC
Conflux Public RPC |	对应 Ethereum RPC |	RPC 参数| 需要 epoch number 参数？
--|--|--|--
cfx_gasPrice|	eth_gasPrice	||N
cfx_epochNumber|	eth_blockNumber|	epoch_number|Y
cfx_getBalance|	eth_getBalance|	address,epoch_number|Y
cfx_getAdmin|	-|	address,epoch_number|Y
cfx_getSponsorInfo|	-|	address,epoch_number|Y
cfx_getStakingBalance|	-|	address,epoch_number|Y
cfx_getCollateralForStorage|	-|	address,epoch_number|Y
cfx_getCode|	eth_getCode|	address,epoch_number|Y
cfx_getStorageAt|	eth_getStorageAt|	address,pos,epoch_number|Y
cfx_getStorageRoot|	-|	address,epoch_number|Y
cfx_getBlockByHash|	eth_getBlockByHash|	block_hash,include_txs|N
cfx_getBlockByHashWithPivotAssumption|	-|	block_hash, pivot_hash, epoch_number,|Y
cfx_getBlockByEpochNumber|	eth_getBlockByNumber|	epoch_number, include_txs|Y
cfx_getBestBlockHash|	-|	|N
cfx_getNextNonce|	-|	address, epoch_number|Y
cfx_sendRawTransaction|	eth_sendRawTransaction|	raw_tx|N
cfx_call|	eth_call|	tx, epoch_number|Y 
cfx_getLogs|	eth_getLogs|	filter|N
cfx_getTransactionByHash|	eth_getTransactionByHash|	tx_hash|N
cfx_estimateGasAndCollateral|	eth_estimateGas|	request, epoch_number|Y
cfx_checkBalanceAgainstTransaction|	-|	account_addr, contract_addr, gas_limit,gas_price, storage_limit, epoch|Y
cfx_getBlocksByEpoch|	-|	epoch_number|Y
cfx_getSkippedBlocksByEpoch|	-|	epoch_number|Y
cfx_getTransactionReceipt|	eth_getTransactionReceipt|	tx_hash|N
cfx_getAccount|	-|	address, epoch_number|Y
cfx_getInterestRate|	-|	epoch_number|Y
cfx_getAccumulateInterestRate|	-|	epoch_number|Y
cfx_getConfirmationRiskByHash|	-|	block_hash||N
cfx_getStatus|	-|	|N
cfx_getBlockRewardInfo|	-|	epoch_number|Y
cfx_clientVersion|	web3_clientVersion|	|N


## Local RPC
Conflux 除了公共rpc外，还有一部分rpc出于安全及性能方面的考虑限制只能在本地访问

Conflux local rpc | 对应 Ethereum rpc
--|--
txpool_status	|-	
tx_inspect	|-	
txpool_inspect	|-	
txpool_content	|eth_newPendingTransactionFilter	
getTransactionsFromPool	|-	
clear_tx_pool	|-	
net_throttling	|-	
net_node	|-	
net_disconnect_node	|-	
net_sessions	|-	
current_sync_phase	|-	
consensus_graph_state	|-	
sync_graph_state	|-	
cfx_sendTransaction	|eth_sendTransaction	
accounts	|eth_accounts	
new_account	|-	
unlock_account	|-	
lock_account	|-	
sign	|eth_sign	

## 暂无对应关系的以太坊RPC

以下是以太坊中有的，Conflux暂时还没有对应关系的RPC

- web3_sha3
- net_version
- net_peerCount
- net_listening
- eth_protocolVersion
- eth_coinbase
- eth_mining
- eth_hashrate
- eth_getTransactionCount
- eth_getBlockTransactionCountByHash
- eth_getBlockTransactionCountByNumber
- eth_getUncleCountByBlockHash
- eth_getUncleCountByBlockNumber
- eth_signTransaction
- eth_getTransactionByBlockHashAndIndex
- eth_getTransactionByBlockNumberAndIndex
- eth_getUncleByBlockHashAndIndex
- eth_getUncleByBlockNumberAndIndex
- eth_getCompilers
- eth_compileLLL
- eth_compileSolidity
- eth_compileSerpent
- eth_newFilter
- eth_newBlockFilter
- eth_uninstallFilter
- eth_getFilterChanges
- eth_getWork
- eth_submitWork
- eth_submitHashrate
- db_putString
- db_getString
- db_putHex
- db_getHex
- shh_post
- shh_version
- shh_newIdentity
- shh_hasIdentity
- shh_newGroup
- shh_addToGroup
- shh_newFilter
- shh_uninstallFilter
- shh_getFilterChanges
- shh_getMessages
- eth_syncing
