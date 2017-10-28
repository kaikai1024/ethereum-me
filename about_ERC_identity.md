# ERC: Identity
*[Identity](https://github.com/ethereum/EIPs/issues/725)* /
*[Claim Holder](https://github.com/ethereum/EIPs/issues/735)*

## 0. 什么是Identity

> The following describes standard functions for a unique identity for humans and machines. 

如上引用,Identity(身份)代表一个人或者机器的身份,在区块链上的概念就可以理解为外部账户地址及合约账户地址.Identity可以包含多个外部账户地址及合约地址(keys).
是一个`self sovereign identity`,也就是只有持有者可以控制其Identity.
在链上表现为智能合约.

## 1. 为什么需要Identity(motivation)

> This standardised identity interface will allow Dapps, smart contracts and thirdparties to check the validity of a person, object or machine

在链上与Dapp,智能合约及第三方机构(在链上通常表现为智能合约)交互时,包括转账,登录,访问等,必须向其证明自己的身份,链上表现为即是Identity的有效性.
同时需要一种方式提供给对方,使其能够验证真实性.

## 2. Identity实现思路

`Identity`的操作及实现需要包括:
* 验证一个identity
* 管理identity

其要解决的问题是如何自证以及别人如何验证.也就是之间信任的问题.
这里引入了一个`claim`(声明)的概念, `claim`是`claim`的发行者(issuer)拥有的一些关于Identity持有者的信息.
`Identity`相关的一些重要定义如下:
* `keys`(不太准确): 外部账户或合约地址的公钥
* `claim issuer`: 是另一个智能合约或者外部帐户,其发行关于`Identity`的`claim`.可以是`Identity`合约自身.
* `signature`: 是一份证明,证明`claim issuer`发行了这个`Identity`的`claimType`类型的`claim`.是由`issuer`对所拥有的`identity`地址的签名.
        (若是Identity智能合约,其需要持有签名私钥对应的公钥,否则此claim无效)
* `claim type`; 数字,代表`cliam`的类型(未定):更类似于Identity持有者的属性(the nature of properties),例子如下:
    - 1: 生物特征数据(Biometric data): 表明是个人而不是企业.
    - 2: 永久地址(Permanent address): 表明你在某个地方,有个物理地址或者参照点(reference point)

自证的问题本质上还是通过签名来解决,引入了`claim`以后,相当于多了一个第三方叫做`claim issuer`,
Identity信任它,会保存所有拥有自己相关信息的的`claim`列表(根据issuer和type计算claim的索引),
验证者只需要从`claim`列表中,选择自己信任的`claim issuer`进行验证,所以信任转移到了`claim issuer`.
任何人都可以对`claim`进行验证(因为Identity部署在链上,而claim存储在Identity中).并可以在`claim issuer`中跳转直到找到自己信任的issuer.
换句话说,可以由`calim issuer`组成一个复杂的信任网.

管理identity即是对key的CRUD的操作.

## 3. 应用场景

### 3.0. 银行开户

`user U`要使用`bank B`的智能合约来创建一个银行账户,合约提供的接口为`openAccount`.由于需要进行`KYC`,流程如下:
1. `user U`使用自己的Identity来调用接口`openAccount`
2. `bank B`收到来在`user U`Identity的请求,`bank B`选择他们信任的`claim type`的`claim issuer`,比如类型2的`Deutsche Post`(德国邮政局),
    `bank B`知道其Identity的合约地址`IssuerAddress`
3. `bank B`根据keccak256(IssuerAddress+1)可以计算出`claim`在`user U`Identity中的索引`index`
4. `bank B`根据计算的`index`获得`claim`,验证其中的`signature`
5. `bank B`验证issuer依然持有签名signature的pub key
6. `bank B`验证identity的地址

以上,银行没有得到用户的生物特征或者住址,而是因为信任isser达到了验证用户的目的.信任可以通过issuser传递的另一个前提是issuer验证了用户的identity(?)

## 4. 为什么标准化

* 标准化之后其他智能合约可以和现实世界身份进行交互,自动检查并验证用户身份
* `claim`标准化,那么其他`identity`就可以相互之间发行`claim`
* 可以在各种UI接口中使用,比如`Mist`,`MetaMask`,`Browsers`等.

## 5. 具体实现规范(specification)

### 5.0. Identity

#### 5.0.0 key的管理

* addKey,其中key的类型如下:
    - 1: `Management keys`: 管理identity
    - 2: `Action keys`: 以此身份名义执行操作(signing, logins, transactions, etc.)
    - 3: `Claim signer keys`: 对需要撤销的其他identity签署claims
    - 4: `Encryption keys`: 加密数据,比图claim中的数据
* removeKey
* replaceKey
* getKeyType

#### 5.0.1 identity用法
#### 5.0.2 identity验证
### 5.1 event

