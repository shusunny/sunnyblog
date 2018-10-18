---
layout: post
title: "如何仿照比特币创造自己的山寨币（一）"
author: "sun"
categories: Blockchain
tags: [bitcoin, C++]
---

最近在做一些比特币的工作，记录自己的工作的同时，也希望顺便科普一下比特币和各种各样的山寨币。

首先，本文介绍的方法仅供技术讨论，不适用于任何商业行为，本人对任何用于商业行为可能出现的情况概不负责。
其次是通过创造电子货币从而获得大量利润的方法很难实现，货币的升值需要大量的信用进行支持。
再次文章中包含大量的代码，并且需要一定的C++知识和ubuntu命令行操作基础。如果你强行想用windows进行编译，我只能在这里祝你好运了。

现在让我们进入正题

## 如何创造像比特币一样的属于自己的电子币

> 本文参考[How to make an altcoin](https://bitcointalk.org/index.php?topic=1030365.0)

### 得到源代码

首先，进行ubuntu的命令行模式，创建我们的工作目录，你可以放在任何地方。

```ruby
mkdir src
cd src
```

下一步我们需要bitcoin的代码。可以到这里https://bitcoin.org/en/version-history 选择版本和对应的操作系统进行下载。其实哪个版本不太重要，因为其中的代码大同小异。但请注意一定要下载源代码，不是QT客户端。

当然也可以使用git命令下载源码，但是git源得到的文件可能和旧版本的文件不太一样，但主要的参数还是差不多的。

`git clone https://github.com/bitcoin/bitcoin.git`

如果下载的是tar.gz文件，我们需要对其进行解压。我们这里得到的是0.10.2的版本，请根据自己的版本号进行调整。

```ruby
cd ~/Downloads
tar -zxvf bitcoin-0.10.2.tar.gz
cp -r bitcoin-0.10.2 ~/src/bitcoin
cp -r bitcoin-0.10.2 ~/src/newcoin
cd ~/src
```

这样，我们就在工作目录下得到了两个文件夹，bitcoin和他的山寨备份newcoin。你也可以把newcoin变成任何你想要的名字。

### 创建环境并编译比特币
比特币的编译需要一定的环境要求，我们先创建环境以包证我们的山寨币也可以编译。

```ruby
su
echo 'deb-src ftp://ftp.us.debian.org/debian/ sid main contrib non-free' >> /etc/apt/sources.list
apt-get update
apt-get build-dep bitcoin
apt-get –install-recommends install libbitcoin-dev
exit
```
上面的代码稍稍有些作弊地使用了debian的源，如果出现版本号不一致的情况并导致下面的编译无法进行，
请参考http://fengmm521.blog.163.com/blog/static/2509135820142154251413/ 自行安装相应的库。

下面我们进行编译

```ruby
cd bitcoin
aclocal
automake --add-missing
./configure --with-incompatible-bdb
make
```

视电脑的运算速度大概需要几分钟到半小时不等的时间，编译成功后，我们可以用以下代码查询是否成功编译出了主要文件，如果其中出现了问题，一般都是由于库没有安装完全或版本不一致的情况，请参考上面分享的文章进行排查。

```ruby
ls src/bitcoind
ls src/bitcoin-cli
ls src/qt/bitcoin-qt
```

如果存在，那么恭喜你，你已经可以用bitcoin-qt文件进行比特币的操作并挖矿了。使用命令cd src/qt;sudo ./bitcoin-qt

而不甘心于此的高玩可以进行继续进行下面的工作。

### 重新命名
即然是新的电子币，那么里面的所有文件现在者需要一个新的名字了，我们仍以newcoin为例

```ruby
cd ~/src/newcoin
find . -type f -print0 | xargs -0 sed -i 's/bitcoin/newcoin/g'
find . -type f -print0 | xargs -0 sed -i 's/Bitcoin/Newcoin/g'
find . -type f -print0 | xargs -0 sed -i 's/BitCoin/Newcoin/g'
find . -type f -print0 | xargs -0 sed -i 's/BITCOIN/NEWCOIN/g'
find . -type f -print0 | xargs -0 sed -i 's/BTC/NCN/g'
find . -type f -print0 | xargs -0 sed -i 's/btc/NCN/g'
find . -type f -print0 | xargs -0 sed -i 's/Btc/NCN/g'
find . -exec rename 's/bitcoin/newcoin/' {} ";"
find . -exec rename 's/btc/NCN/' {} ";"
```

前8行代码将文件中的bitcoin进行替换，后两行将文件名替换为我们的新名字。

这里注意，由于比特币客户端多语言环境的问题，上面的命令可能会造成一些拼写和其他语言没有被更改的问题。需要做其他语言的替换请进入代码内部进行修改。我们这里不再详细说明。

下面我们更改客户端使用的端口号

```ruby
find . -type f -print0 | xargs -0 sed -i 's/8332/9443/' {} ";"
find . -type f -print0 | xargs -0 sed -i 's/8333/9444/' {} ";"
```

这样就将原来的8332及种子的8333端口号变成了9443和9444，以免发生冲突。

下面需要设计自己的logo，从而让我们的newcoin焕然一新。qt客户端的图片文件主要存放在src/qt/res中。将其中的图片文件按照原来的像素大小一一替换掉即可。

现在我们编译我们自己的newcoin

```ruby
aclocal
automake --add-missing
./configure --with-incompatible-bdb
make
```

如果编译成功即可以找到newcoin-qt,newcoind,newcoin-cli，说明我们上面的改动没有改变基础文件，非常成功。如果编译出现问题，你已经成功编译了比特币客户端了，请参照bitcoin客户端的编译方法进行逐一排查。

这里先不要运行，因为我们虽然变了名字，但内核还是比特币，属于换汤不换药，所以我们需要将参数改掉以后才能正式创造我们的newcoin。
我们将在下一篇文章中详细介绍有关内核参数的更改。
