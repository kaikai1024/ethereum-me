# solidity
## introduction to smart contracts
## a simple smart contract
## blockchain basics
### EVM
the EVM is not a register machine but a stack machine,so all computations are performed on an area called the stack.  
***??*** the instruction set of the EVM is kept minimal in order to avoid incorrect implementation which could casue consensus problems. ***get.other nodes need to verification to make a consensus ?? but does node need to run the contract? yes***  
#### call vs delegatecall vs  callcode
contracts can call other contracts or send Ether to non-contract accounts by the means of message calls.
***??why non-contract accounts***  
- every transaction consists of a top-level message call which in turn can create further message calls.
- there exists a special variant of a message call, named delegatecall which is identical to a message call apart from the fact that the code at the target address is executed in the context of the calling contract and msg.sender and msg.value do not change their values.  
This means that a contract can dunamicallu load code from a different address at runtime. Storage,current address and balance still refer to the calling contract, only the code is taken from the called address.

#### log
contracts cannot access log data after it has been created, but they can be efficiently accessed from outside the blockchain.
#### create
contracts can even create other contracts using a special opcode(i.e. they do not simply call the zero address). the only difference between these create calls and normak message calls is that the payload data is executed and the result stored as code and the caller/creator receives the address of the new contract on the stack.
#### self-destruct
the only possibility the code is removed from the blockchain is when a contract at that address performs the selfdestruct operation. even if a contract's code does not contain a call to selfdestruct, it can still perform that operation using delefatecall or callcode.
## installing solidity
remix/node.js/docker/binary packages/source
## solidity in depth
### version pragma
source files can and should be annotated with a so-called version pragma to reject being compiled with furure compiler versions that might inreoduce incompatible changes.
### importing other source files
#### syntax and semantics

