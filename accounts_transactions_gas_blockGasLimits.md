# 以太坊上的账户，交易，GAS，以及区块gas limit   
                                                                    --Hudson Jamesoon

***[原文链接](http://hudsonjameson.com/2017-06-27-accounts-transactions-gas-ethereum/)***
## 什么是账户？
### EOA vs 合约账户
以太坊上有两种类型的账户：
* 外部拥有账户(Externally Owned Accounts)
* 合约账户
这个区别将会在即将到来的大都会版本升级中抽象出来。
#### 外部拥有账户(EOA)
一个外部控制的账户
* 拥有一个以太余额
* 可以发送交易(以太转账或者触发合约代码)
* 由私钥控制
* 没有相关代码
#### 合约账户
一个合约
* 拥有一个以太余额
* 拥有相关代码
* 由交易或者从其他合约收到的消息(call)触发执行代码
* 当执行的时候-执行任意复杂的操作(图灵完备)-操作它自己的持续储存，也就是说，可以拥有它自己的永久的状态-能够调用其他合约。

通过从账户触发的交易，以太坊区块链上所有的操作都将启动。每次一个合约账户收到一个交易，它的代码是按照作为交易的一部分发送的输入参数执行的。合约代码由每个作为验证新块的一部分参与到的网络中的节点上的EVM执行。
## 什么是交易和消息？
### 交易
以太坊使用的“交易”一词是指经过签名的数据包，其中数据包中存储着从区块链上一个外部拥有账户到另一个账户的消息。  ***Prob:合约账户不可以吗？目前不可以，无法完成签名***

交易包括：
* 消息的接收者，
* 一个签名，可以认证发送者并且证明他们通过区块链发送消息给接收者的意图,
* <font color=red>VALUE</font> 字段 - 从发送者转到接收者的wei的数量，
* 一个可选的数据段，可以包含发送给合约的消息，
* 一个<font color=red>GASLIMIT</font>值，代表允许交易运行计算步数的最大值，
* 一个<font color=red>GASPRICE</font>值，代表发送者愿意为gas花费的费用。一个gas单位对应一个原子指令的执行，也就是一个计算步骤。
### 消息
合约有发送给其他合约“消息”的能力。消息是一个从未序列化的虚拟对象，并且只存在以太坊运行环境中。它们可以被认为是函数调用。

一个消息包含：
* 消息的发送者(隐式的)
* 消息的接收者
* <font color=red>GASPRICE</font>字段 - 附着在消息上转给合约地址的wei的数量，
* 一个可选的数据段，合约实际的输入数据
* 一个<font color=red>GASPRICE</font>值，其限制由消息触发的代码运行所产生的gas的最大数量。

基本上，一个消息就像一个交易，除了它是由合约生成并且不是一个外部的执行者(actor)。当正在执行代码的合约执行<font color=red>CALL</font>或者<font color=red>DELEGATECALL</font>操作码时，就生成了一个消息，合约可以生成并执行消息。消息有时也会被称作“内部交易”。就像一个交易，一个消息可以导致接收账户运行其代码。如此，合约就可以和其他合约有与外部执行者(actors)完全相同的关系。很多时候人们使用交易这个术语，实际上他们指的是消息，所以有可能这个术语会通过社区共识不使用它而逐步淘汰。
## 什么是GAS？
以太坊在区块链上实现了一个叫做以太坊虚拟机(EVM)的执行环境。每一个参与网络的节点作为区块验证协议的一部分都运行EVM。他们通过(go through)正在验证的区块中列出的交易，并且在EVM中运行由交易触发的代码。网络上的每个完整节点做同样的运算并且存储同样的值。合约的执行在节点间冗余复制，这个事实自然地使其变得昂贵，这样，通常就会激励人们对那些可以在链下完成的计算不放在区块链上进行。对每一个执行的操作都会有一个明确的成本，其由若干gas单位表示。合约能够利用的每个操作都有一个相关的gas值。[这是一个过期的每个操作码(opcode)对应的gas成本的列表](https://docs.google.com/spreadsheets/d/1m89CVujrQe5LAFJ8-YAUCcNK950dUzMQPMJBxRtGCqs/edit#gid=0).
### Gas以及交易成本
每一个交易被要求包括一个gas limit(有时候也被称为<font color=red>startGas</font>)和愿意为每个gas花费的费用。矿工可以选择是否包括这些交易并获取这些费用。实际上，如今所有的交易最终都会被矿工打包，但是使用者选择发送的交易费用的数量会影响这个交易被挖到的时间。如果由交易产生的计算步数所使用的gas总量，包括原始的消息和可能被触发的任何子消息，不大于gas limit，那么交易就会被执行。如果gas总量超过gas limit，那么所有的改变都会被重置，除了交易依然是有效的而且费用依然能够被矿工获得。区块链表明尝试一个交易，但是没有提供足够的gas，所有的合约操作都被重置。所有没有被交易执行使用的过量gas会以以太偿还给发送者。因为gas成本估计只能够近似，很多使用者多付gas来保证他们的交易被接受。这是没有关系的，因为多余的gas会退还给你。
### 估计交易成本
一个交易全部的以太成本取决于两个因素：
* <font color=red>gasUsed</font>:交易消耗的总gas
* <font color=red>gasPrise</font>:交易中指定的单位gas的价格(以以太为单位)

**总成本 = gasUsed * gasPrise**
#### gasUsed
EVM中的每个操作被分配给一定数量的消耗gas的数目。gasUsed是所有操作执行的gas的总和。

对于估计gasUsed，有一个[可以使用的estimateGas的API，但是有一些注意事项](http://ethereum.stackexchange.com/q/266/)。
#### gasPrise
一个使用者构造并签名了一个交易，并且每个使用者可能指定任意的他们想要的gasPrise，可能为零。但是，以太坊客户端推出的前沿(Frontier)有一个默认的0.05e12的<font color=red>gasPrise</font>。由于矿工会优化他们的收入，如果大部分的交易都以默认的0.05e12 wei的<font color=red>gasPrise</font>提交，就会很难说服矿工接受制定较低或者零的gasPrise的交易。
#### 交易成本的例子
*得到许可的情况下，我现在借这个例子并从一个很棒的叫做MyEtherWallet的团队做了类比，[请点击这里访问他们关于gas写的很好的一个指南](https://myetherwallet.groovehq.com/knowledge_base/topics/what-is-gas)，他们还有个[很棒的实用程序页面，你可以将以太数转换成子单元(subunits)](https://www.myetherwallet.com/helpers.html)*

你可以把gas limit当作一辆车的汽油单位或者公升或者加仑的数量。可以把gas price当作是汽油单位/公升/加仑的成本。

对于一辆车，每加仑(单位)是2.50美元(价格)。对于以太坊，每个gas(单位)20 GWEI(价格)。为了填满你的“坦克”，需要……- 10 加仑2.50美元的价格 = 25美元 - 21000单位gas20GWEI的价格 = 0.00042 ETH。

因此，总共的交易费用将会是0.00042 以太。

发送代币通常会花费大约50000到100000gas，所以总共的交易费用会增加到0.001ETH -0.002ETH。
### 什么是“区块gas limit”？
区块gas limit是一个区块允许的最大的gas数，由此来决定区块中能够填充多少交易。比如，假设我们有五个交易，其中每个交易分别有10,20,30,40,50的gas limit。如果区块gas limit是100,那么最开始的四个交易可以填充进区块中。矿工决定区块中包括哪些交易。不同的矿工可以尝试在区块中包含后两个交易(50+40)，并且他们只有空间来包括地一个交易(10)。如果你尝试包括使用查过现在区块gas limit的gas，就会被网络拒绝，同时你的以太坊客户端也会给你一个“交易超过区块gas limit”的消息。  
[这个例子来自以太坊StackExchange的“ethers”](https://ethereum.stackexchange.com/questions/7359/are-gas-limit-in-transaction-and-block-gas-limit-different)

[根据ethstats.net，区块gas limit在写这篇文章时是4,712,357gas](https://ethstats.net)，这意味着一个区块中大约可以填充gas limit为21000的交易224个(平均15-20s产生***PRob:产生什么？区块？***)。[协议允许一个区块的矿工在1/1024(0.0976%)的幅度调整区块gas limit](https://www.reddit.com/r/ethereum/comments/6g6tww/there_are_hundreds_or_even_thousands_of_pending/dinzrgq/)。
#### 谁决定区块gas limit？
网络上的矿工决定区块gas limit。与区块gas limit可调协议分开的是[大多数客户端默认的挖矿策略设置的最小区块gas limit4,712,388](https://github.com/search?q=org%3Aethereum+4712388&type=Code)。矿工可以选择改变这个值，但是很多人都不会这么做。
#### 区块gas limit是怎样改变的？
以太坊上的矿工使用一个挖矿程序，比如ethminer，其连接到geth或者parity以太坊客户端。geth和parity有一些选项可以允许矿工改变。[geth的关于挖矿的命令行选项在这里](https://github.com/ethereum/go-ethereum/wiki/Command-Line-Options)，[parity的选项在这里](https://github.com/paritytech/parity/wiki/Configuring-Parity#cli-options)。
## 什么是以太坊网络的“DoS”？
最近有很多关于以太坊网络变慢、堵塞以及不能用的言论。这些言论把这种变慢描述为以太坊网络的“Dos”。当网络上一直有满区块以及很多挂起的交易，一个以太坊上的拒绝服务(DoS)攻击事件就发生了。回顾上文说矿工可以依据交易上的交易费用来选择是否包括它。如果在队列中(或者技术术语，交易池)有成百上千的交易，就会导致不寻常的数小时的交易延迟。DDoS攻击事件可以是恶意的或者非恶意的。
### 恶意DoS攻击
去年秋天，以太坊受到了一个人或者团队发起的叫做交易的垃圾邮件攻击。这次攻击在[这篇博客](https://blog.ethereum.org/2016/10/18/faq-upcoming-ethereum-hard-fork/)中描述如下:
>攻击者通过在他们的合约中反复的调用某些操作码(opcodes)来发动DoS攻击，这些操作码在客户端运行的时候计算很困难，但是添加到网络上却很容易。

在攻击过程中，矿工被要求降低区块gas limit为150万，然后在另外一种事件下设置为200万。还有一些其他事件是，在对网络进行攻击时，矿工已经被要求降低区块gas limit。
### 非恶意的DoS攻击
非恶意DoS事件仅仅是当网络上有太多挂起的交易以至于花费异常多的时间处理交易。最近代币分布(distribution)事件(或者最初的货币发行(ICOs)或者代币售卖)的普及和扩散，已经导致网络变成了交易的备份(backed up)。Infura的伙伴写了一篇[关于技术细节的博客](https://blog.infura.io/when-there-are-too-many-pending-transactions-8ec1a88bc87e?gi=569738c91a48)。
## 为什么即使区块是满的区块gas limit也没有改变？
**主要的原因：矿工没有使用自适应的gas limit功能**

以太坊协议有一个内置的矿工可以对gas limit进行投票的机制，因此无需在硬分叉上协调就可以增加容量。起初，这个机制加了一个默认的策略，其矿工对gas limit投票至少为470万，但是如果数值较高，会达到最近(1024-区块指数的移动)平均使用的gas的150%，允许随着需求增长而有机(organically)的增长，同时仍然包括反垃圾邮件目的的上限。

根据上述“恶意DoS攻击”部分，过去有很多次矿工被建议改变他们的设置来修改区块gas limit为非默认值来帮助阻挡攻击直到攻击被修复。问题是即使在攻击消退以后，一些矿池也从不把设置改回去。大约一个月以前，[矿工被要求改变gas limit以及gas price的设置来重新引入自适应gas limit的功能](https://www.reddit.com/r/ethereum/comments/6ehp60/recommendations_to_miners_to_change_gas_limit_and/)，原因是最近的代币出售很快的把区块填满并导致了区块链交易拥塞，

[ETH Gas Station](http://ethgasstation.info/index.php)是一个很好的资源，可以在上面查看[关于矿池正在投票的区块gas limit](http://ethgasstation.info/minerVotes.php)的最新信息。
### 矿工需要做什么来解决这个问题？
矿工可以在他们的geth或者Parity上调整他们的设置来重新启用gas limits。注意：下面的值取自[这个Reddit的帖子](https://www.reddit.com/r/ethereum/comments/6ehp60/recommendations_to_miners_to_change_gas_limit_and/)，如[这个Reddit帖子](https://www.reddit.com/r/ethereum/comments/6jn6lr/miners_increase_the_capacity_of_the_ethereum/)解释的，实际上值可以设置的更高。
#### Geth
##### 建议的设置
```--gasprice 4000000000 --targetgaslimit 4712388```
##### 解释
<font color=red>--targetgaslimit</font> 目标gas limit为挖的区块设置一个人工设定的目标gas floor(默认：“4712388”)  
<font color=red>--gasprice</font>  打包交易可以接受的最小的gas price(默认：20000000000)。注意：<font color=red>gasprice</font>单位为[wei](https://www.myetherwallet.com/helpers.html)。
#### Parity
##### 建议的设置
```--gas-floor-target 4712388 --gas-cap 9000000 --gasprice 4000000000```
##### 解释
<font color=red>--gas-floor-target</font> 当打包一个新的区块时，每个块gas数的目标(默认：4700000)。  
<font color=red>--gas-cap</font> 由于交易量每个区块将会提高的gas limit的一个上限(默认：6283184)。  
<font color=red>--gasprice</font> 为一个交易支付的能够被挖矿所接受的每GAS最小的Wei数。注意：<font color=red>--gasprice</font> 是一个“传统的选项”
#### 其他的挖矿选项
请访问[geth](https://github.com/ethereum/go-ethereum/wiki/Command-Line-Options)和[Parity](https://github.com/paritytech/parity/wiki/Configuring-Parity#cli-options)的CLI选项页面，就可以看到矿工可以设置的所有的选项列表，以此来优化的调整自己的设置。
## 资源和深入阅读



