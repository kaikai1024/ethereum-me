# solidity
## introduction to smart contracts
## a simple smart contract
## blockchain basics
### EVM
the EVM is not a register machine but a stack machine,so all computations are performed on an area called the stack.  
***??*** the instruction set of the EVM is kept minimal in order to avoid incorrect implementation which could casue consensus problems. ***get.other nodes need to verification to make a consensus ?? but does node need to run the contract? yes***  
#### call vs delegatecall vs  callcode
contracts can call other contracts or send Ether to non-contract accounts by the means of message calls.
***??why non-contract accounts. actually can***  
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
### layout of a solidity source file
#### version pragma
source files can and should be annotated with a so-called version pragma to reject being compiled with furure compiler versions that might inreoduce incompatible changes.
### importing other source files
##### syntax and semantics
solidity supports import statement
##### paths
it depends on the compiler how to actually resolve the paths. it can also map to resources discovered via e.g. ipfs,http or git.
##### use in actual compilers  
***??Prob: not understand. something about complile path. need to try in code***
remapping
solc/remix
#### comments
single-line comments(//) and multi-line comments(/*...*/)are possible.  
tripple slash(///) or a double asterisk block(/** ... */) should be used directly above function declaration or statements.
### structure of a contract
each contract can contain declarations of state variables,functions,fcunction modifiers, events, struct types, and enum types. furthermore, contracts can inherit from other contracts.
#### state variables
state cariables are values which are permanently stored in contract storage.
#### functions
fucnctions calls can happen internally or externally and have different levels of visibility towards other contracts. 
#### function modifiers
function modifiers can be used to amend the semantics of functions in a delcarative way.
#### events
events are convenience interfaces with the EVM logging facilities.
#### structs types
structs are custom defined types that can group several variables.
#### enum types
enums can be used to create custom types with a finite set of values.
### types
solidity is a statically typed language,which means that the type of each cariable(state and local) needs to be specified(or at least known) at compile-time.  
types can interact with each other in expressions containing operators. ***??not understand. still not understand how interact***
#### value types
variables of these types will aways be passed by value.
##### booleans
true/false:!,&&,||,==,!=. || and && apply the common short-circuiting rules.
##### integers
int/uint: uint8 to uint256; int8 to int256
- comparisons: evaluate to bool
- bit operators: &, |, ^, ~
- arithmetic operators: +, -, unary -. unary +, *, /, %, **, <<, >> ***??unary +/-***

division always truncates, but it does not truncate if both operators are literals(or literal expressions). ***?? what's the literals***

the result of a shift operation is the type of the left operand.
#### address
holds a 20 byte value. also have members and serve as a base for all contracts.
operators:<=, <, ==, !=, >=, and >.
#### members of address
* balance and transfer  
it is possible to query the balance of an address using the property balance and to send Ether(wei) to an address using the transfer function.
note: if the address is a contract address ,its code(fallback function ***??fallback. a function not named and if not map other method then call it or receive an Eth***) will be executed together with the transfer call if that execution runs out of gas or fails in any way, the Either transfer will be reverted and the current contract sill stop with an exception.
* send  
send is the low-level conterpart of transfer. if the execution fails, the current contract will not stop with an exception, but send will return false.  
warning:in order to make safe Ether transfers , always check the return value of send, use transfer or even better:use a pattern where the recipent withdraws the money. ***??need to discuss. about the contract security*** 
* call, callcode, delegatecall  
call is provided which takes an arbitrary number of arguments of any type. these arguments are padded to 32 bytes and concatenated. one exception is the case where the first argument is encoded to exactly bour bytes. in that case,it is not padded to allow the use of function signatures here. ***??why. about encoding***

call returns a boolean. it is not possible to access the actual data returned(for this we would need to know the encoding and size in advance.) ***??encoding,***

delegatecall: the difference is that only the code of the given address is used, all other aspects(storage,balance)are taken from the current contract.  
the purpose of delegatecall is to use library code which is stored in another contract. the user has to ensure that the layout of storage in both contracts is suitable for delegatecall to be used ***??why both. about library***  
callcode was available that did nor=t provide access to the original msg.sender and msg.value values.

all three functions are very low-level functions and should only be used as a last resort as they break the type-safety of solidity.
* .gas() option is avaliable on all three methods, while the .value() option is not supported for delegratecall.
note: all contracts inherit the members of address ***??. about the meaning***, so it is possible to query the balance of the current contract using this.balance.
#### fixed-size byte arrays
bytes1 ... bytes32. byte is an alias for bytes1.
operators:
* comparisons: evaluate to bool
* bit operators:
* index access: read-only

memebers: .length: read-only
#### dynamically-sized byte array
* bytes: dynamically-sized byte array. not a value type
* string: dynamically-sized UTF-8-encoded string,not a value type

use bytes for arbitrary-length raw byte data and string for arbitrary-length string(UTF-8) data.
#### fixed point numbers
coming soon
#### address literals
hexadecimal literals that are between 39 and 41 digits long and do not pass the checksum test produce a warning and are treated as regular rational number literals
#### rational and integer literals
integer literals are formed from a sequence of numbers in the range 0-9.Octal literass do not exists in Solidity and leading zeros are invalid.  
decimal fraction literals are formed by a . with at least one number on one side.  
scientific notation is also supported, where the base can have fractions while the exponent cannot.  
number literal expressions retain arbitrary precision until they are converted to a non-literal type. this means that computations do not overflow and divisons do not truncate in number literal expressions.

any operator that can be applied to integers can also be applied to number literal expressions as long as the operands are integers.

note: all number literal expressions(i.e. the expressions that contain only number literals and operators) belongs to number literal types. so the number literal expressions 1 + 2 and 2+1 both belong to the same number literal types for the rationa number three.

warning: division on integer literals used to truncate in earlier versions, but it will now convert into a rational number, i.e. 5/2 is not equal to 2, but to 2.5.

note: number literal expressions are converted into a non-literal type as soon as they are used with non-literal expressions. ***??still prob about literal. understand but how to translate***

#### string literals
they don't imply trailing zeroes as in C; "foo" represents three bytes not four.  
string literals support escape characters.
#### hexadecimal literals
hex"001122FF",  
hexademical literals behave like string literals and have the same convertibility restrictions.
#### enums
enums are one way to create a user-defined type in solidity. they are explicitly convertible to and from all integer types but implicit conversion is not allowed.  
enums needs at least one member.
#### function types
function types come in two flavours - internal and external functions  
* internal functions can only be used inside the current contract(more specifically, inside the current code unit, which also includes internal library functions and inherited function) because they cannot be executed outside of the context of the current contract. calling an internal function is realized by jumping to its entry label.
* external function consist of an address and a function signature and they can be passed via and returned form external functions calls. 
``` 
function (<parameter types>) {internal|external} [constant] [payable] [returns (<return types>)] 
```
in contrast to the parameter types, the return types cannot be empty - if the function type should not return anything, the whole returns part has to be omitted.  
by default, function types are internal, so the internal keyword can be omitted.

internal: directly by its name, f  
external: using this.f

if a function type variable is not initialized, calling it will result in an exception. the same happens if you call a function after using delete on it. ***?? initialized? delete?***

if external function types are used outside of the context of solidity, they are treated as the function tyep, which encodes the address followed by the function identifier together ina single bytes24 type.

note that public functions of the current contract can be used both as an internal and as an external function.  
***?? the example code***  
the lambda or inline functions are planned but not yet supported.
### reference types
#### data location
every complex type, i.e. *arrays* and *structs*, has an additional annotation, the "data location", about whether it is stored in memory or in storage.

there is a third data location. "calldata", which is a non-modifiable, non-persistent area where function arguments are stored.  
function parameters(not return parameters) of external functions are forced to "calldata" and behave mostly like memory. ***??not return parameters***

data locations are important because they change how assignments behave: assignments between storage and memory and also to a state variable(even from other state variable) always create an independent copy.  
***?? example code***
##### summary
* forced data location:
    - parameters(not return) of external functions: calldata
    - state variables: storage
* default data location:
    - parameters(also return) of functions: memory
    - all other local variables: storage
#### arrays
array can have a compile-time fixed size or they can be dynamic
* for storage array, the element type can be arbitrary(i.e. also other arrays, mappings or structs).
* for memeory arrays, it cannot be a mapping and has to ban an ABI type if it is an argument of a publicly-visible function. ***??can arrayss or structs? ABI type?***

fixed size: T[k]; dynamic size: T[].  
example: an array of 5 dynamic arrays of uint is uint[][5](note that the notation is reversed when compared to some other languages). x[2][1]: the secont uint in the third dynamic array. ***??fuck this. whats the meaning***

variables fo type bytes and string are special arrays.
* a bytes is similar to bytes[], but it is packed tightly in calldata
* string is equal to bytes but does not allow length or index access(for now)

so bytes should always be perfered over byte[] because it is cheaper.  
note: string s: bytes(s).length / bytes(s)[7] = 'x'.

it is possible to make arrays public and have solidity create a getter.the numeric index will become a required parameter for the getter. ***?? getter? not understand***
##### allocating memory arrays
creating arrays with variable length in memory can be done using the new keywoed. as opposed to storage arrays, it is not possible to resize memory arrays by assigning to the .length member.
***?? the example code: uint[] memory a = new uint[][7]***
##### array literals / inline arrays
