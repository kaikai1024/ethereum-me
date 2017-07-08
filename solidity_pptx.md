# smart contract - solidity
## bitcoin script && smart contract 
* UTXO in Bitcoin can be owned not just by a public key, but also by a more complicated script expressed in a simple stack-based programming language
* mart contracts, cryptographic "boxes" that contain value and only unlock it if certain conditions are met, can also be built on top of the platform, with vastly more power than that offered by Bitcoin scripting because of the added powers of Turing-completeness, value-awareness, blockchain-awareness and state.
## programming language
* LLL
* serpent
* Viper
* <font color=red>solidity</font>
## solidity
* contract-oriented
* high-level language
* statically typed
* supports inheritance
* libraries
* complex user-defined types
## a simple smart constract
```
pragma solidity ^0.4.0;

contract SimpleStorage {
    uint storedData;

    function set(uint x) {
        storedData = x;
    }

    function get() constant returns (uint) {
        return storedData;
    }
}
```
* pragma
* contract
* uint
* function
## subcurrency example
```
pragma solidity ^0.4.0;

contract Coin {
    // The keyword "public" makes those variables
    // readable from outside.
    address public minter;
    mapping (address => uint) public balances;

    // Events allow light clients to react on
    // changes efficiently.
    event Sent(address from, address to, uint amount);

    // This is the constructor whose code is
    // run only when the contract is created.
    function Coin() {
        minter = msg.sender;
    }

    function mint(address receiver, uint amount) {
        if (msg.sender != minter) return;
        balances[receiver] += amount;
    }

    function send(address receiver, uint amount) {
        if (balances[msg.sender] < amount) return;
        balances[msg.sender] -= amount;
        balances[receiver] += amount;
        Sent(msg.sender, receiver, amount);
    }
}
```
* address
* public
* mapping
* event
* msg
## blockchain basics
* transactions
* blocks
## ethereum virtual mathine
* runtime environment
* sandboxed, isolated, limit
### account
* EOA
* contract address
### transaction
**from -> to**
*to*
* contains code
* zero-account
### gas
gas_prise * gas
### storage, memory, stack
### instraction set
### message calls
### delegatecall callcode and libraries
### logs
### create
### self-destruct
## struct of a contract
* state variables - getter
* functions - visibility -fallback - constant
* function modifiers
* event
* struct types
* enum types
* inherientance
## solidity assembly && security
