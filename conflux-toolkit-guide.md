# Conflux toolkit 使用指南
## 一、批量地址空投
### 1. flags说明
运行`conflux-toolkit transfer -h` 查看transfer子命令所需flags

- batch: 表示每一批次发送的地址数，默认100，该值不能设置大于2000
- from: 发送者的地址，需要先导入私钥或者创建账户
- number: 发送的CFX数量（权重为1时），单位为CFX
- price: gas price, 单位为drip
- receivers: 包含接收者列表的文件
- url: 节点url，默认是主网节点
```
transfer subcommand
Usage:
  conflux-toolkit transfer [flags]

Flags:
      --batch uint         send tx number per batch (default 100)
      --from string        From address in HEX format or address index number
  -h, --help               help for transfer
      --number uint        send value in CFX (default 1)
      --price string       Gas price in drip (default "1")
      --receivers string   receiver list file path
      --url string         Conflux RPC URL (default "http://main.confluxrpc.org")
```
### 2. 执行步骤
#### 2.1 创建空投列表文件
空投列表包含两列，第1列为接收方地址，第2列为空投权重，列之间需要用空格、逗号或tab分割；
权重表示发送的倍数， 即发送数量为 `number*权重`

如创建空投列表文件 `~/transferlist.csv` ，内容如下
```
0x18e79e2c653d8e63a132aa197343b71b21ba0f78 5
0x12794484b0a6b0f1fcee72b29d322e93e9de5aac 5
0x1c0cc2082540bc9a9853f73ce47df7be55535c34 5
0x113c5708495a737f0abd52b879306e252eda15b8 5
0x19a6a8f7f746ded355609c04d4f3f931c1dacd0b 5
0x1f80c9b80ba9b23b7b638412e8eb51582dae402a 5
0x11782b012f293e1d0edf44c31a563f06b56835f2 4
```
#### 2.2 导入私钥或创建新账户
创建新账户
```
conflux-toolkit account import --key 0x......
```

导入私钥
```
conflux-toolkit account create
```

#### 2.3 开始空投
如空投到列表`~/transferlist.csv`, 发送者`0x11c5aeed5256b825bfc83e674327fc58dbd3e111`, 单次空投数量是5CFX, 每批次发送1000个地址, 命令如下
```
conflux-toolkit transfer --receivers "~/transferlist.csv" --from 0x11c5aeed5256b825bfc83e674327fc58dbd3e111 --number 5 --url http://test.confluxrpc.org --batch 1000
```