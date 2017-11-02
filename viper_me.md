## viper

* security
* language and compiler simplicity
* auditability: it should be maximally difficult to write misleading code.

### struct of a contract

Contracts in Viper are contained within files, with each file being one smart-contract.

state variables are values which are permanently stored in contract storage. `storedData: num`

functions are the executable units of code within a contract.

```
@payable
function bid():
```

### types
#### value types

they are always copied when they are used as function arguments or in assignments.

* `num`: equivalent to `int128` 
* `decimal`
* `timestam`
* `timedelta`
* `wei_value`
* `currency_value`
* `address`
    - `balance`
    - `send`
* `bytes32`
* `bytes32 <= maxlen`
* `type[length]`
* `struct.argument`

#### mapping

* `_ValueType[_KeyType]`
    - Mapping are only allowed as state variables.
    - it is possible to make mappings `public ` and have Viper create a getter.
