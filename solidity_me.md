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
array literals are arrays that are written as an expression and are not assigned to a variable right away.
note: currently,fixed size memory arrays cannot be assigned to dynamically-sized memory arrays.
##### members
* length: the size of memory arrays is fixed(but dynamic, i.e. it can depend on runtime parameters) once they are created.
* push: dynamic storage arrays and bytes. the function returns the new length.

warning: it is not yet possible to use arrays of arrays in external functions.
warning: due to limitations of the EVM, it is not possible to return dynamic content from external function calls. 
```
contract C { function f() returns (uint[]) { ... } }
```
will return something if called from web3.js, but not if called from Solidity. ***?? what***  
the only workaround for now is to use large statically-sized arrays statically-sized arrays.
#### struct
a new way to define new types in the form of structs.  
struct types can be used inside mapping and arrays and they can itself contain mapping and arrays.  
it is not possible for a struct to contain a member of its own type,although the struct itself can be the value type of a mapping member. as the size of the struct has to be finite.

note: how in all the functions, a struct type is assigned to a local variable(of default storage data location). this does not copy the struct but only stores a reference so that assignments to members of the local cariable actually write to the state.
### mappings
mapping types are declared as mapping(_KeyType => _ValueType).  
* _KeyType: almost any type except for a mapping, a dynamically sized array. a contract, an enum and a struct.
* _ValueType: any type. including mappings.

mapping can be seen as hash tables which are virtually initialized such that every possible key exists and is mapped to a value whose byte-representation is all zeros: a type's default value.

mapping are only allowed for state variables(or as storage reference types in internal functions)

it is possible to mark mappings public and have solidity create a getter. ***?? still getter. but a little understand***  
### operators involving LValues
LValue: avariable or something that can be assigned to.  
+=, ++, -- something like that.
#### delete
delete a assigns the initial value for the type to a.  
delete has no effect on whole mappings(as the keys of mappings may be arbitrary and are generally unknown)  
note: behaves like an assignment to a, i.e. it stores a new object in a.
### conversions between elementary types
#### implicit conversions
in general, an implicit conversion between value-types is possible if it makes sense semantically and no information is lost.
#### explicit conversions
be care
### type deduction
using var is not possible for function parameters or return parameters.
warning: the type is noly reduced from the first assignment, so the loop in the following snippet is infinite, as i will have the type uint8 and any value of this type is smaller than 2000. ***?? why***
```
for (var i = 0; i < 2000; i++) { ... }
```
### units and globally avaliable variables
#### Ether units
wei, finney, szabo, ether. 2 ether == 2000 finney evaluates to true.
### time units
* 1 == 1 seconds
* 1 minutes == 60 seconds
* other: minutes, hours, days, weeks, years

due to the fact that leap seconds cannot be predicted, an exact calendat library has to be updated by an external oracle. ***??oracle. later to learn***

these suffixes cannnot be applied to variables.
```
function f(uint start, uint daysAfter) {
    if (now >= start + dayAfter * 1 days) { ... }
}
```
#### special variables and functions
##### block and transaction properities
* block.blockhash(uint blockNUmber) returns (bytes32)
* block.coinbase (address)
* block.difficulty (uint)
* block.gaslimit (uint)
* block.number (uint)
* block.timestamp (uint)
* msg.data (bytes)
* msg.gas (uint)
* msg.sender (address)
* msg.sig (bytes4)
* msg.value (uint)
* now (uint)
* tx.gasprise (uint)
* tx.origin (address)

note:
* the value of all members of msg, including msg.sender and msg.value can change for every external function call. this includes calls to library functions.
* if you want to implement access restrictions in library functions using msg.sender, you have to manually supply the value of msg.sender as an argument. ***??what***
* the block  hashes are not avaliables for all blocks for scalability reasons. you can only access the hashes of the most recent 256 blocks, all other values will be zero.
##### error handing
* assert(bool condition)
* require(bool condition)
* revert() ***?? oh***
##### mathematical and cryptographic functions
* addmod(uint x, uint y, uint k) returns (uint): (x + y) % k
* mulmod(uint x, uint y, uint k) returns (uint): (x * y) % k
* keccak256(...) returns (bytes32)
* sha3(...) returns (bytes32)
* sha256(...) returns(bytes32)
* ripemd160(...) returns(bytes32)
* ecrecover(bytes32 hash, uint8 v, bytes32 r, bytes32 s) returns (address)

tightly packed: the arguments are concatenated without padding. 

message to non-existing contracts are more expensive. ***??oh***
##### address related
* <address>.balance (uint256)
* <address>.transfer(uint256 amount)
* <address>.send(uint256 amount) returns (bool)
* <address>.call(...) returns (bool)
* <address>.callcode(...) returns (bool)
* <address>.delegatecall(...) returns (bool)

##### contract related
* this: the current contract, explicilty convertible to Address
* selfdestruct(address recipient)
### expressions and control structures
#### input parameters and output parameters
##### input parameters
the input parameters are declared the same way as variables are. 
##### output parameters
the output parameters can be declared with the same syntax after the returns keywords.

the name of output parameters can be omitted. the output values can also be specified using return statements.  
the rerurn statements are also capable of returning multiple values.
#### control structure
most of the control structures from jacascript are available except for switch and goto.  
parentheses can not be omitted for conditionals.  
note: there is no type conversion from non-boolean to boolean types as there is in C and JavaScript
##### returning multiple values
when a function has multiple output parameters, return (v0, v1, ..., vn) can return multiple values
#### function calls
##### internal function calls
these function calls are translated into simple jumps inside the EVM.  
only functions of the same contract can be called internally.
##### external function calls
via a message call and not directly via jumps.  
note: function calls on this cannot be used in the constructors, as the actual contract has not been created yet.

for an external call, all function arguments have to be copied to memory.
* .value()
* .gas()

note: the expression InfoFeed(addr) performs an explicit type conversion stating that "we know that the type of the contract at the given address is InfoFeed" and this does not execute a constructor.

function calls cause exception if the called contract does not exist(in the sense that the account does not contain code) or if the called contract itself throws an exception or goes out of gas.

warning:n interaction with another contract imposes a potertial danger, expecially if the source code of the contract is not known in advance.
##### named calls and anonymous function parameters
the argument list has to coincide by name with the list of parameters from the function declaration, but can be in arbitrary order.
##### omitted function parameter names
the names of unused parameters(especially return parameters) can be omitted. those names will still be present on the stack, but they are inaccessible.
#### creating contracts via new
the full code of the contract being created has to be known in advance, so recursive creation-dependenceies are not possible  
it is possible to forward Ether to the creation using the  .value() option, but it is not possible to limit the amount of gas.
#### order of evalution of expressions
it is only guaranteed that statements are executed in order and short-circuiting for boolean expressions is done.
#### assignment
##### destructuring assignments and returning multiple values
solidity internally allows tuple types, i.e. a list of objects of potentially different types whose size is a constant at compile-time.
##### complications for arrays and structs
assignments to memebers(or elements) of the local variable do change the state.
#### scoping and declaration
the "default value" of variables are the typical "zero-state" of whatever the type is.  
a variable declared anywhere within a function will be in scope for the entire function, regardless of where it is declared.  
 if a variable is declared, it will be initialized at the beginning of the function to its default value.
#### error handing: assert, require, revert and exceptions
solidity uses state-reverting exceptions to handle errors. such an exception will undo all changes made to the state in the current call(and all its sub-calls) and also falg an error to the caller.
* assert: should only be used for internal errors
* require: should be used to check external conditions(invalid inputs or errors in external components)
* revert: can be used to flag an error and revert the current call.
* throw alternative to revert()

catching exceptions is not yet possible.

the reason for reverting is that there is no safe way to continue execution, because an expected effect did not occur. because we want to retain the atomicity of transactions, the safest thing to do is to revert all changes and make the whole transaction(or at least call) without effect.

note: assert-style exceptions consume all gas available to the call, while revert-style exceptions will not consume any gas starting from the Metropolis release. ***?? consume all gas avaliable***
### contracts
contracts in solidity are similar to classes in object-oriented languages. they contain persistent data in state variables and functions that can modify these variables.
#### creating contracts
contracts can be created "from outside" or from solidity contracts. when a contract is created, its constructor is executed once. optional
#### visibility and getters
functions can be specified as being external, public, internal or private, where the default is public.  
state variables, external is not possible and the default is internal.
* external: external functions are part of the contract interface, which means they can be called from other contracts and via transactions.
* public: public functions are part of the cntract interface and can be either called internally or via messages.
* internal: those functions and state variables can only be accessed internally, without using this.
* private: private functions and state variables are only visible for the contract they are defined in and not in derived contracts. ***four types***

note: everything that is inside a contract is visible to all external observers. making something private only prevents other contracts from accessing and modifying the information, but it will still be visible to the whole world outside of the blockchain
##### getter functions
the compiler automatically creates getter functions for all public state variables.  
the getter functions have external visibility.
#### function modifiers
modifiers can be used to easily change the behaviour of functions.  
modifiers are inheritable properties of contracts and may be overridden by derived contracts  
return variables are assigned and control flow continues after the "_" in the preceding modifier. ***??what's the meaning***
#### constant state variables
state variables can be declared as constant. in this case, they have to be assigned from an expression which is a constant at compile time.
#### constant functions
functions can be declared constant in which case they promise not to modify the state.  
note: getter methods are marked constant.  
warning: the compile does not enforce yet that a constant method is not modifying state. 
#### fallback function
a contract can have exactly one unnamed function.  
this function cannot have arguments and cannot return anything.  
it is executed on a call to the contract if none of the other functions match the given function identifier(or if no data was supplied at all)  
furthermore, this function is executed whenever the contract receives plain Ether(without data).  
it is important to make fallback functions as cheap as possible.  
the following operations will consume more gas than the stipend provided to a fallback function:
* writing to storage
* creating a contract
* calling an external function which consumes a large amount of gas
* sending Ether

warning: if you want your contract to receive Ether, you have to implement a fallback function.
#### events
events allow the convenient usage of the EVM logging facilities.  
events are inheritable members of contracts.   
when they are called, they cause the arguments to be stored in the transaction's log - a special data structure in the blockchain.  
these logs are associated with the address of the contract and will be incorporated into the blockchain and stay there as long as a block is accessible(forever as of Frontier and homestead, but this might change with serenity)  
log and envent data is not accessible from within contracts(not even from the contract that created them) ***?? what event data***  
up to three parameters can receive the attribute indexed which will cause the respective arguments to be searched for: it is possible to filter for specific values of indexed arguments in the user interface.  
the hash of the signature of the event is one of the topic except if you declared the event with anonymous specifier.
* non-indexed arguments will be stored in the data part of the log
* indexed arguments will not be stored themselves. you can only search for the values, but it is impossible to retrieve the values themselves. ***store where***
##### low-level interface to logs
it is also possible to access the low-level interface to the logging mechanism via the functions log0,log1,log2,log3 and log4. logi takes i+1 parameter of type bytes32, where the first argument will be used for the data part of the log the the others as topics.
#### inheritance
solidity supports multiple inheritance by copying code including polymorphism.  
all function calls are virtual, which means that the most derived function is called, except when the contract name is explicitly given. ***??not understand***   
when a contract inherits from multiple contracts, only a single contract is created on the blockchain, and the code from all the base contracts is copied into the created contract.  
super: it calls the function on the next base contract in the final inheritance graph ***??final-base1-base2-mortal-owned, why base1 first? the order from the right?***
##### arguments for base constructors
derived contracts need to provide all arguments needed for the base constructors.
##### multiple inheritance and linearization
expecially, the order in which the base classes are given in the is directive is important  
a simple rule to remember is to specify the base classes in the order from "most base-like" to "most derived" ***??meaning***
##### inheriring different kinds of members of the same name
as an exception, a state variable getter can override a public function
#### abstract contracts
contract functions can lack an implementation. such contracts cannot be compiled, but they can be used as base contracts.
#### interface
interfaces are similar to abstract contracts, but they cannot hace any function implemented.
* cannot inherit other contracts or interfaces
* cannot define constructor
* cannot define variables
* cannot define structs
* cannot define enums

interfaces are basically limited to what the contract ABI can represent, and the conversion between the ABI and an interface should be possible without any information loss. ***think the ABI***
#### libraries
libraries are similar to contracts, but their purpose is that they are deployed only once at a specific address and their code is reused using the DELEGATECALL(CALLCODE until Homestead)feature of the EVM

libraries can be seen as implicit base contracts of the contracts that use them.

to realize this in the evm, code of internal library functions and all functions called form therein will be pulled into the calling contract. and a regular JUMP call will be used instead of a DELEGATECALL

as the compiler cannot know where the library will be deployed at, these addresses have to be filled into the final bytecode by a linker ***?? need to read the link part***
* no state variables
* cannot inherit nor be inherited
* cannot receive ether ***how to do it? the compiler?***
#### using for
using A for B; can be used to attach library functions(from the library A) to any type (B). ***??exactly meaning***

note: all library calls are actual EVM function calls.
### solidity assembly
solidity defines an assembly language that can also be used without solidity. ***cool***  
this assembly language can also be used as "inline assembly" inside solidity source code.
#### inline assembly
due to the fact that the evm is a stack machine, it is often hard to address the correct stack slot and provide arguments to opcodes at the correct point on the stack.
* functional-style opcodes: mul(1, add(2, 3)) instead of push1 3 push1 2 add push1 mul
* assembly-local variables: let x := add(2, 3) let y:= mload(0x40) x:= add(x, y)
* access to external variables: function f(unix x) { assembly {x := sub(x, 1) } } ***??where is external variables***
* labels: let x := 10 repeat: x := sub(x, 1) jumpi(repeat, eq(x, 0))
* loops: for { let i := 0 } lt(i, x) {i := add(i, 1)} { y := mul(2, y) } ***??not very clear***
* switch statements: switch x case 0 { y := mul(x, 2) } default { y := 0}
* function calls: function f(x) -> y { switch x case 0 {y := 1 } default { y := mul(x, f(sub(x, 1))) } }
##### example
please be aware that assembly is much more difficult to write because the compiler does not perform checks, so you should use it for complex things only if you really know what you are doing.
##### syntax
use the usual // and /* */ comments. inline assembly is marked by assembly { ... } 
* literals. 0x123, 42 or "abc"
* opcodes. mload sload dup1 sstore
* opcodes in functional style, add(1, mlod(0))
* labels. name:
* variable declarations. let x := 7
* identifiers. jump(name), 3 x add ***??***
* assignments. 3 =: x ***??***
* assignments in function style. x := add(y, 3)
* blocks where local variables are scoped inside. { let x := 3 { let y := add(x, 1) } }
##### opcodes
the order of arguments can be seen to be reversed in non-functional style.
##### literals
you can use integer constants by typing them in decimal or hexadecimal notation and an approprite PUSHi instruction will automatically be generated.
##### functional style
note: the order of arguments is reversed in functional-style as opposed to the instruction-style way.
##### access to external variables and functions
* memory variables: push the address and not the value onto the stack.
* storage variables: values in storage might not occupy a full storage slot, so their "address" is composed of a slot and a byte-offset inside that slot. x, x_slot, x_offset.
##### labels
note: labels are a low-level feature and it is possible to write efficient assembly without labels, just using assembly functions, loops and switch instructions.
##### declaring assembly-local variables
use let keyword to declare variables that are only visiable in inline assembly and actually only in the current {...}-block.  
what happens is that the let instruction will create a new stack slot that is reserved for the variable and automatically removed again when the end of the block is reached.
##### assignments
assignments are possible to assembly-local variables and to function-local variables.  
when you assign to variable that point to memory or storage, you will only change the pointer and not the data.
* functional-style: variable := value
* instruction-style: =: variable, the value is just taken from the stack top.
##### switch
it takes the value of an expression and compares it to several constants.  
there can be a fallback or default case called default.
##### loops
for-style loops have a header containing an initializing part, a condition and a post-iteration part.
##### functions
functions can be defined anywhere and are visible in the block they are declared in.  
there is no explicit return statement.
##### things to avoid
it is important to know that the assembler only counts stack height from top to bottom, not necessarily following control flow.
