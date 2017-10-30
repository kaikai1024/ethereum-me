# ERC: Permission Contract

*[Permission Contract](https://github.com/ethereum/EIPs/issues/745)*

## 0. 什么是权限合约

> Permissions are contracts that allow controlled actions from other addresses.

允许来在其他地址的受控操作的合约. 
依赖[ERC-725](https://github.com/ethereum/EIPs/issues/725): 
identity可以持有对identity操作的keys, 此合约可以针对每个要求控制identity的dapp制定规则,限制授权地址对identity的操作.

## 1. 为什么需要权限合约(motivation)

> The need of different types of permissioning for ERC725, that are unique for each use-case, are not scope of ERC725.

对于ERC725,也就是identity的用例,每个用例都是独特的,所以需要针对每种情况制定不同类型的权限

## 2. 实现思路

权限合约应该包括管理此合约的接口以及管理identity执行规则的接口.*权限合约需要被identity持有者信任*

类似于在identity合约方法上加了一层使用上的限制.

## 2.0. Permission

* Native Call Permission
    - 允许对某些地址的本地调用(native call)
* ERC20 Approve-Call Permission
    - 通过批准(approve)和调用(call)允许安全的转移ERC20定义的代币(token)

*research ERC20?*

## 2.1. Permission Manager

允许用户通过调用和权限合约进行交互.

## 2.2. rule

* `Period Length`
    - 以秒为单位的长度的值
    - 之后`Period Limit`将会被重置
* `Period Limit`
    - wei/token为单位的值
    - 这段时间内可以被花费的值
* `Call Max Value`
    - wei/token为单位的值
    - 每次调用某个函数所能包含的最大值

## 2.3. interface

* authorize
    - 授权某种规则于目标地址的方法调用
* unauthorize
    - 未授权(为什么不可以默认未授权?)
* execute
    - 执行目标地址的方法
* isAuthorized
    - 查询是否授权

*execute包含执行的data,不知方法作用,文档中暂没有更进一步的方法用例及说明*

详细定义如下:

```
pragma solidity ^0.4.17;

contract ERC745 {

    event Authorized(address _destination, uint _maxValue, bytes4 _method, uint expiration);
    event Unauthorized(address _destination, bytes4 _method);
    event Executed(address _destination, uint _value, bytes4 _method);

    /**
     * @notice Authorizes call method `_method` to `_destination` with maximum value of `_maxValue` with expiration of `_expiration`;
     */
    function authorize(address _destination, uint _maxValue, bytes4 _method, uint _expiration) public;
    
    /**
     * @notice Unauthorizes call method `_method` to `_destination`.
     */
    function unauthorize(address _destination, bytes4 _method) public;
    
    /**
     * @dev executes into `_destination`.
     */
    function execute(address _destination, uint _value, bytes _data) public returns (bool result);
    
    /**
     * @dev reads authorization
     */
    function isAuthorized(address _trustedCaller, address _destination, uint256 _value, bytes _data) public constant returns(bool);

}
```

## 3.0 相关

[authority pattern](https://github.com/ethpm/escape/blob/master/contracts/Authority.sol)

[Standard Functions for Preauthorized Actions](https://github.com/ethereum/EIPs/issues/662)

[Token standard](https://github.com/ethereum/EIPs/issues/20)

