# ERC: Identity

*[Identity](https://github.com/ethereum/EIPs/issues/725)* /
*[Claim Holder](https://github.com/ethereum/EIPs/issues/735)*

## 0. 什么是identity

> The following describes standard functions for a unique identity for humans and machines. 

如上引用, identity(身份)代表一个人或者机器的身份,在区块链上的概念就可以理解为外部账户地址及合约账户地址. 
Identity可以包含多个外部账户地址及合约地址(keys).是一个`self sovereign identity`,也就是只有持有者可以控制其identity.
在链上表现为智能合约.

## 1. 为什么需要identity(motivation)

> This standardised identity interface will allow Dapps, smart contracts and thirdparties to check the validity of a person, object or machine

在链上与Dapp,智能合约及第三方机构(在链上通常表现为智能合约)交互时,包括转账,登录,访问等,必须向其证明自己的身份,链上表现为即是identity的有效性.
同时需要一种方式提供给对方,使其能够验证真实性.

## 2. Identity实现思路

`Identity`的操作及实现需要包括:
* 验证一个identity
* 管理identity

其要解决的问题是如何自证以及别人如何验证.也就是之间信任的问题.
这里引入了一个`claim`(声明)的概念, `claim`是`claim`的发行者(issuer)拥有的一些关于identity持有者的信息.
`Identity`相关的一些重要定义如下:
* `keys`(不太准确): 外部账户或合约地址的公钥
* `claim issuer`: 是另一个智能合约或者外部帐户,其发行关于`identity`的`claim`.可以是`identity`合约自身.
* `signature`: 是一份证明,证明`claim issuer`发行了这个`identity`的`claimType`类型的`claim`.是由`issuer`对所拥有的`identity`地址的签名.
               (若是identity智能合约,其需要持有签名私钥对应的公钥,否则此claim无效)
* `claim type`; 数字,代表`cliam`的类型(未定):更类似于identity持有者的属性(the nature of properties),例子如下:
    - 1: 生物特征数据(Biometric data): 表明是个人而不是企业.
    - 2: 永久地址(Permanent address): 表明你在某个地方,有个物理地址或者参照点(reference point)

自证的问题本质上还是通过签名来解决,引入了`claim`以后,相当于多了一个第三方叫做`claim issuer`,
identity信任它,会保存所有拥有自己相关信息的的`claim`列表(根据issuer和type计算claim的索引),
验证者只需要从`claim`列表中,选择自己信任的`claim issuer`进行验证,所以信任转移到了`claim issuer`.
任何人都可以对`claim`进行验证(因为identity部署在链上,而claim存储在identity中).并可以在`claim issuer`中跳转直到找到自己信任的issuer.
换句话说,可以由`calim issuer`组成一个复杂的信任网.

管理identity即是对key的CRUD的操作.

## 3. 应用场景

### 3.0. 银行开户

`user U`要使用`bank B`的智能合约来创建一个银行账户,合约提供的接口为`openAccount`.由于需要进行`KYC`,流程如下:
1. `user U`使用自己的identity来调用接口`openAccount`
2. `bank B`收到来在`user U`identity的请求,`bank B`选择他们信任的`claim type`的`claim issuer`,比如类型2的`Deutsche Post`(德国邮政局),
    `bank B`知道其identity的合约地址`IssuerAddress`
3. `bank B`根据keccak256(IssuerAddress+1)可以计算出`claim`在`user U`identity中的索引`index`
4. `bank B`根据计算的`index`获得`claim`,验证其中的`signature`
5. `bank B`验证issuer依然持有签名signature的pub key
6. `bank B`验证identity的地址

以上,银行没有得到用户的生物特征或者住址,而是因为信任isser达到了验证用户的目的.信任可以通过issuser传递的另一个前提是issuer验证了用户的identity(?)

## 4. 为什么标准化

* 标准化之后其他智能合约可以和现实世界身份进行交互,自动检查并验证用户身份
* `claim`标准化,那么其他`identity`就可以相互之间发行`claim`
* 可以在各种UI接口中使用,比如`Mist`,`MetaMask`,`Browsers`等.

## 5. 具体实现规范(specification)

### 5.0. key的管理

* addKey.
    - 只能由key类型为1,或者dentity本身操作
    - 其中key的类型如下:
        * 1: `Management keys`: 管理identity
        * 2: `Action keys`: 以此身份名义执行操作(signing, logins, transactions, etc.)
        * 3: `Claim signer keys`: 对需要撤销的其他identity签署claims
        * 4: `Encryption keys`: 加密数据,比图claim中的数据
* removeKey
    - 只能由key类型为1,或者dentity本身操作
* replaceKey
    - 只能由key类型为1,或者dentity本身操作
* getKeyType
    - 如果由identity持有,则返回key的类型,否则返回0
* getKeysByType
    - 返回identity持有的key列表.

### 5.1. identity用法

* execute
    - 操作其他合约或者本身,转账
    - 需要调用`approve`方法来批准
    - 可以作为一些操作唯一的访问器(accessors),包括`addKey`, `removeKey`, `replaceKey`, `removeClaim`.
    - 由于需要和approve交互,所以定义了`Transaction`数据结构,由其成员的hash作为存储的索引,并交于`approve`,结构定义如下:
        ```
        struct Transaction {
            address to;
            uint value;
            bytes data;
            uint nonce;
        }
        ```
* approve
    - 批准一次执行操作或添加`claim`操作
    - 如果执行identity合约本身,则需要key类型为1的n/m批准
    - 如果执行其他合约,则需要key类型为2的n/m批准

### 5.2. identity验证

#### 5.2.0. Claim数据结构

```
struct Claim {
    uint256 claimType;
    address issuer; // msg.sender
    uint256 signatureType; // The type of signature
    bytes signature; // this.address + claimType + data
    bytes claim;
    string uri;
}
```

余下的一些定义说明如下(见2.):
* `signatureType`: 签名(算法)类型
* `claim`: claim数据的hash(?)
* `uri`: claim的位置,比如heetp链接,swarm hashes, IPFS hashes.

#### 5.2.1. Claim的管理

* getClaim
    - 通过ID返回claim
* gelClaimsByType
    - 通过claimType获取ID数组
* addClaim
    - `issuer`请求`claim`的`ADDITION`或者`CHANGE`操作
    - signature: `keccak256(address this_identities_address + uint256 _claimType + bytes _claim)`
    - claim id: `keccak256(address issuer_address + uint256 _claimType)`
* removeClaim
    - 只能够被claim issuer或者claim持有者删除

#### 5.2.3. Central claim registry vs. In-Contract claims

*[discussion about two side](https://github.com/ethereum/wiki/wiki/ERC-735:-Claim-Holder-Registry-vs.-in-contract)*

以上是关于claim是中心机构注册还是智能合约形式的优缺点的讨论.

### 5.3. 用法流程示例

#### 5.3.0. 链下(Off-chain)

```
                                          +----------+
                                          | identity |--+
                                          +----------+  |
                                               ^        |
                                 3. getClaim() |        |  4. check claim
+-----+   1. random string                 +------+     |
| key | <--------------------------------- | user | <---+
+-----+                                    +------+
   |                                           ^
   | 2. signed(random string+identity address) |
   ---------------------------------------------
```

#### 5.3.1. 链上(On-chain)

```
         
                                           +--------+
                                           | issuer | 
                                           +--------+
                                                ^       
                                                | 4. check if claim issuer still holds key from signature      
+----------+   1. superCoolFunc()           +---------+ 
| identity | -----------------------------> | bankOr? | 3. check claim(check claim signature get key)
+----------+                                +---------+
     ^                                          |
     |        2. getCliam()                     |
     --------------------------------------------

```

*以上key表示identity地址*

#### 5.3.2. check claims

* 检查使用了哪种签名类型
* 构造签名哈希:`keccak256(address this_identities_address + uint256 _claimType + bytes _claim)`
* 从`signature`中恢复地址k
* 检查`issuer`的identity合约是否依然持有地址k
* done

## 6. 约束(Constraints)

每个issuer的claim只能是每种类型(claim type)的一种类型.

## 7. 代码示范

[Identity code](https://github.com/status-im/contracts/tree/master/contracts)

## 8. 资料

[Identity介绍的幻灯片](https://www.slideshare.net/FabianVogelsteller/erc-725-identity)
