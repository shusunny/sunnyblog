---
layout: post
title: "如何仿照比特币创造自己的山寨币（二）"
author: "sun"
categories: Cryptocurrencie
tags: [bitcoin, C++]
---

上次我们把比特币中的所有文件名及内容都改成了我们的新币newcoin。但是这还远远不够。
why? 因为所有的内核参数还是比特币的，所以即使运行了客户端，也会同步之前的比特币数据。
现在是时候进入代码的内部，探寻比特币的奥秘，并做一些真正属于我们的改动。

### 修改区块链参数
首先我们需要进入chainparams.cpp

```cpp
cd src
sudo gedit chainparams.cpp
```

然后我们在代码中搜索mapCheckpoints，在第55行，它大概是这个样子的：

```cpp
static Checkpoints::MapCheckpoints mapCheckpoints =
      boost::assign::map_list_of
      ( 11111, uint256("0x00223155*****"));
```

这里会显示一大串数据，分别是不同区块的hash值，是比特币客户端用来验证之前区块值是否正确。
而我们需要把11111那一行的数值改为

```cpp
( 0, uint256("0x001"));
```

这里，0代表的是第几个区块。0后面的uint256表示的就是初始块的hash值。
可是我们的新币还没有正式运行呢？怎么得到它的hash值呢？我们下面会讲如果挖创始块。这里我们姑且做一个大胆并且绝对错误的推断，让其等于001。

同样的，我们还需要搜索并更改`mapCheckpointsTestnet` 和`mapCheckpointsRegtest` 同一位置的数据。可能他们只有一行，并非大串数据。他们表示的是测试网络的区块值。
没关系，大胆的把他们全部变成`( 0, uint256("0x001"))`;。

现在我们返回`mapCheckpoints`,在刚才的数据地图的下面，我们可以看到

```cpp
1397080064, // * UNIX timestamp of last checkpoint block
36544669,   // * total number of transactions between genesis and last checkpoint
            //   (the tx=... number in the SetBestChain debug.log lines)
60000.0     // * estimated number of transactions per day after checkpoint
```

这里通过英文注释我们可以看到，第一行代表上个检查点的时间，第二行是创始块直到上次检查时的交易数量。第三行表示平均每天的交易量。我们运行

```c
date +%s
```

将上面命令行得到的值替换到上述bitcoin代码的第一行，第二行交易数量随便填，估且填为10吧。第三行看情况来，先填成10000吧。

将同样的时间和交易数量数据像上面那样填进test和regtest的信息里。平均交易量填一半就好。

在`mapCheckpointsRegtest`下是CMainParams函数。在函数中有四行指针代码表示pchMessageStart。一般作为用来对比特币的交易进行验证的协议(protocol)。其中每个数组的值都是0-255中的数值，以16进制的形式表示。他们一旦改变，任何人都无法将我们的客户端连进比特币的了。

我们可以在http://numbermonk.com/?all=1 的表中从0-255找到我们喜欢的数字填进去。如果可以的话，再找8个填进`CTestNetParams`和`CRegTestParams`。

下面的代码是**vAlertPubKey**。所以我们需要更改alert和genesis coinbase的key值。我们用命令行生成一些

```cpp
openssl ecparam -genkey -name secp256k1 -out alertkey.pem
openssl ec -in alertkey.pem -text > alertkey.hex
openssl ecparam -genkey -name secp256k1 -out testnetalert.pem
openssl ec -in testnetalert.pem -text > testnetalert.hex
openssl ecparam -genkey -name secp256k1 -out genesiscoinbase.pem
openssl ec -in testnetalert.pem -text > genesiscoinbase.hex
```

这样我们就得到了一系列alertkey和genesiscoinbase key。我们首先打开alertkey 
```cpp
cat alertkey.hex
```

我们把‘pub’和‘ASN1 OID: secp256k1′中间的5行数值去掉冒号和空格填到`vAlertPubKey = ParseHex(“…”)`;里去。然后我们再用`cat testnetalert.hex`把其中的数据填入到CTestNetParams函数中去。

下面我们更改时间标签**pszTimestamp**。在这一行

```cpp
const char* pszTimestamp = "The Times 03/Jan/2009 Chancellor on brink of second bailout for banks";
```
我们去外国网站上找到一条今天的新闻（但不要太长，最好小于90个字母）填进去。

接下来我们将创始块的**pubkey**填进去。打开`cat genesiscoinbase.hex`，在

```cpp
txNew.vout[0].scriptPubKey = CScript() << ParseHex("...") << OP_CHECKSIG;
```

中插入我们的key值。同样的去掉冒号和空格。

接下来，我们需要将

```cpp
assert(hashGenesisBlock == uint256("0x000000000019d6689c085ae165831e934ff763ae46a2a6c172b3f1b60a8ce26f"));
assert(genesis.hashMerkleRoot == uint256("0x4a5e1e4baab89f3a32518a88c31bc87f618f76673e2cc77ab2127b7afdeda33b"));
```
这两行用//注释掉，以免在我们找初始块的时候报错影响程序运行。所有的文件改完别忘了保存一下。

下面的vSeeds表示的种子文件的位置，我们还没有种子文件的域名，先不要进行改动。

### 挖取创始块
现在我们开挖，打开main.cpp

```
sudo gedit main.cpp
```
我们在这一行后面

```
CBlock &block = const_cast<CBlock&>(Params().GenesisBlock());
```
以及

```
// Start new block file
unsigned int nBlockSize = ::GetSerializeSize(block, SER_DISK, CLIENT_VERSION);
```
这一行前面，加入我们挖创始块的代码

```cpp
uint256 bnTarget;
bool fNegative;
bool fOverflow;
uint256 hashGenesisBlock;
block.nBits    = 0x1f00ffff;
bnTarget.SetCompact(block.nBits, &fNegative, &fOverflow);
LogPrintf("ProofOfWorkLimit %s\n", Params().ProofOfWorkLimit().ToString());
if (fNegative || bnTarget == 0 || fOverflow || bnTarget > Params().ProofOfWorkLimit()) {
	error("InitBlockIndex CheckProofOfWork() : nBits below minimum work");
}else {
	block.nTime    = GetTime();//1231006505;
	LogPrintf("block.nTime %d\n", block.nTime);
	LogPrintf("bnTarget %s\n", bnTarget.ToString());
	block.nNonce = 0;
	while (true) {
		if (block.nNonce%1000000000 == 0)
			LogPrintf("block.nNonce--- %d\n", block.nNonce);
		hashGenesisBlock = block.GetHash();
		if (hashGenesisBlock <= bnTarget)
			break;
		block.nNonce++;
	}
	LogPrintf("block.nNonce******** %d\n", block.nNonce);
	LogPrintf("hashGenesisBlock******** %s\n", hashGenesisBlock.ToString());
	LogPrintf("hashMerkleRoot******** %s\n", block.hashMerkleRoot.ToString());
}
```
这段代码是会在我们程序运行的时候不断计算，并在debug.log中导出我们所需要的GenesisBlock的Hash值。保存并返回newcoin文件夹。重新编译

```
cd ..
make
```
如果我们上一篇中环境安装完全的话，应该会秒通过。然后我们运行我们的程序

```
cd src/qt
sudo ./newcoin-qt
```
然后程序会通知我们将文件目录放在哪里，用默认的设置就好。这时程序可能会跑一段时间崩溃掉。没关系，我们用

```
tail ~/.newcoin/debug.log
```
进入到debug文件中去。看看我们是否导出了以下数值

```cpp
2016-04-11 21:16:20 ProofOfWorkLimit 00ffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffff
2016-04-11 21:16:20 block.nTime 1460410772
2016-04-11 21:16:20 bnTarget 0000ffff00000000000000000000000000000000000000000000000000000000
2016-04-11 21:16:20 block.nNonce--- 0
2016-04-11 21:16:20 block.nNonce******** 43973
2016-04-11 21:16:20 hashGenesisBlock******** 0000f51a735d0f7da96f10797ad76e00e53614422987f2b6056aaf61857e165e
2016-04-11 21:16:20 hashMerkleRoot******** d5c218da6f5c257709c0d59177ae4527ea5049f15baf26260a071c2e212154ac
2016-04-11 21:16:20 Pre-allocating up to position 0x1000000 in blk00000.dat
2016-04-11 21:16:20 Pre-allocating up to position 0x100000 in rev00000.dat
```
如果有，那么恭喜你，你已经挖到了创始块，那么让我们返回到chainparams.cpp中，将hashGenesisBlock,hashMerkleRoot填入到刚才的区块地图中，即uint256(“0x001”)，将001改为我们前面得到的hashGenesisBlock值，并将其填入已经被我们注释掉的assert中，并把注释去掉。
同样的，将上面log中block.nTime和block.nNonce的值填回到CMainParams的genesis.nTime和genesis.nNonce中去。他们在下面这行代码的上边。

```
hashGenesisBlock = genesis.GetHash();
```
保存。现在我们已经得到了我们的初始块，**别忘了将main.cpp中我们刚才填加的代码删掉**，这样我们就完全构建出了一套属于我们的电子币系统。并且不会出现像比特币那样挖不着的情况。（毕竟是我们自己的，想挖多少就挖多少。哈哈哈）

你也可以将上面的程序额外运行两次，并把得到的hash值和各参数填入到Testnet和Regtest中去。

下面我们返回newcoin文件夹中，**重新编译，并删除原来的文件目录(非常重要)**

```
cd ..
sudo make
rm -rf ~/.newcoin
```
然后运行起来。如果一切顺利的话，程序会运行起来，然后我们就可以得到下面的程序
![newcoin](https://github.com/shusunny/sunnyblog/blob/gh-pages/assets/img/newcoin.png "newcoin")

不用羡慕上面的数值，既然所有的参数都是我们自己改的，当然可以在main.cpp里面修改区块数据，从而在短时间内挖到最多的币。
