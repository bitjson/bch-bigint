# CHIP-2024-07-BigInt: High-Precision Arithmetic for Bitcoin Cash

        Title: High-Precision Arithmetic for Bitcoin Cash
        Type: Standards
        Layer: Consensus
        Maintainer: Jason Dreyzehner
        Status: Draft
        Initial Publication Date: 2024-07-24
        Latest Revision Date: 2024-08-28
        Version: 1.1.0

## Summary

This proposal removes the limit on the maximum length of numbers in the Bitcoin Cash Virtual Machine. The number length limit is superseded by the [Arithmetic Operation Cost Limit](https://github.com/bitjson/bch-vm-limits#arithmetic-operation-cost).

## Deployment

Deployment of this specification is proposed for the May 2025 upgrade.

- Activation is proposed for `1731672000` MTP, (`2024-11-15T12:00:00.000Z`) on `chipnet`.
- Activation is proposed for `1747310400` MTP, (`2025-05-15T12:00:00.000Z`) on the BCH network (`mainnet`), `testnet3`, `testnet4`, and `scalenet`.

This proposal requires [`CHIP: Targeted Virtual Machine Limits`](https://github.com/bitjson/bch-vm-limits).

## Motivation

Many financial and cryptographic applications require higher-precision arithmetic than is currently available to Bitcoin Cash contracts.

While Bitcoin Cash's 2022 upgrade [increased the maximum number length from 4 bytes to 8 bytes](https://gitlab.com/GeneralProtocols/research/chips/-/blob/master/CHIP-2021-02-Bigger-Script-Integers.md), the design of the Bitcoin Cash Virtual Machine's anti-Denial-Of-Service limits prevented further increase.

Following [`CHIP: Targeted Virtual Machine Limits`](https://github.com/bitjson/bch-vm-limits), the Virtual Machine (VM) can correctly account for arithmetic operation costs, so higher-precision arithmetic operations can be enabled without increasing the processing or memory requirements of the VM.

## Benefits

By allowing contracts to efficiently operate on larger numbers, this proposal improves contract security and reduces transaction sizes.

### Safer Contracts

This proposal obviates the need for higher-precision math emulation. As a result, existing applications can be simplified, making them easier to develop and review. Additionally, by making arithmetic overflows less common, contracts are less likely to include vulnerabilities or faults which can expose users to attacks and losses.

### Reduced Transaction Sizes

Because this proposal allows existing contracts to remove higher-precision math emulation, transactions employing these contracts are reduced in size. This reduces transaction fees for contract users, and it reduces storage and bandwidth costs for validators.

## Technical Specification

The limit on the maximum length of Bitcoin Cash VM numbers (A.K.A. `nMaxNumSize`) is removed.
The numbers will still be limited, by the maximum stack item size (A.K.A. `MAX_SCRIPT_ELEMENT_SIZE`).

For reference, here we will specify the whole set of affected operations.

### Locktime Opcodes

The upgrade DOES NOT change the behavior of OP_CHECKLOCKTIMEVERIFY (0xb1) and OP_CHECKSEQUENCEVERIFY (0xb2).

### Script Number Encoding

The numbers are encoded as variable-length byte arrays in little endian byte order (least significant byte first).

Sign is encoded with a sign bit, and sign bit is defined as the highest order bit, e.g. `ff7f` encodes 32767, `ffff` encodes -32767.
This means that some positive values must have `00` appended so to move the sign bit to the next byte, and allow encoding positive values which would need to use the sign bit (`0x80`), e.g. `ff8000` encodes 33023 and `ff8080` encodes -33023.

Encoding of value 0 is special, it is encoded as an empty stack item of 0 length and NOT with 0-byte (`00`) or "negative" 0-byte (`80`).

Numbers must be minimally encoded, e.g. `0200` could be decoded to number 2 but it is not valid and encoding it as `02` is mandatory.

This upgrade affects two non-arithmetic opcodes that are used to convert an arbitrary byte array to a minimally encoded script number and back.

For reference, we will specify them below as well.

#### OP_NUM2BIN (0x80)

Pop two items from stack.  
The top-most value is read as a desired length of the output's stack item, and the other one as binary value to be converted.  
The length must be a minimally encoded script number, otherwise the script fails immediately.  
The binary value may be any length and does not need to start out as a minimally encoded script number, however.
If the requested length is larger than `MAX_SCRIPT_ELEMENT_SIZE`, fail immediately.
The value is then transformed to a minimally-encoded script number (meaning trailing zeroes may be popped off).
At this point the value's size may shrink.  
Then, if the new (possibly reduced) length of the value is larger than the requested length, fail immediately.  
Otherwise, taking account of the sign bit, pad the value with 0-bytes until the desired length is reached and then push the result to stack.

Executing the operation on value 0 and length 0 is valid and will return 0 as an empty stack item.  
When positive values are padded the operation will simply add 0-bytes on the higher end, e.g. executing it on `7b` (123) and `05` (5) will return `7b00000000`.  
When negative values are padded the operation will similarly add 0-bytes on the higher end but it must also move the sign bit to highest byte, e.g. executing it on `fb` (-123) and `05` (5) will return `7b00000080`.  
The value to be converted can be a padded number, so this opcode can be used to change padding, too, e.g. executing it on `7b00000000` (123) and `03` (3) will return `7b0000`.  
Similarly, for negative numbers, executing it on `7b00000080` (-123) and `03` (3) will return `7b0080`.  
If called on other encodings of 0 (like `00`, `80`, `0000`, `0080`) etc. it WILL NOT preserve the sign bit, and it will return a "positive" encoding of 0, e.g. executing it on `0080` (-0) and `03` (3) will return `000000` (0).

#### OP_BIN2NUM (0x81)

Pop one item from stack.  
Decode the stack item to a numerical value using script number encoding scheme.  
Push the value on stack as a minimally-encoded script number.  

For example, byte sequence `0080` is not a valid script number encoding, but the opcode would convert it to value 0 and return an empty stack item which is the only valid encoding for value 0.
Similarly, byte sequence `ff0080` is not a valid script number encoding, but the opcode would convert it to value -255 and return `ff80` which is the only valid encoding for value -255.

This operation may reduce the size of the stack item but it will never increase it.

Before this upgrade, the operation would fail if it would have to return a value outside the int64 range.
After this upgrade, the operation will never fail because it can not increase the size of the stack item so any byte sequence on stack will be decodeable.

### Arithmetic Operations

**General requirements**

If any of the input stack items is not a minimally-encoded script number then the operation must fail, e.g. trying to add `0100` and `01` must fail rather than return `02`.

The operation must fail if any resulting stack item would exceed `MAX_SCRIPT_ELEMENT_SIZE`.  
Before this upgrade, the operation would fail if any resulting stack item would exceed `nMaxNumSize` which was set to 8 bytes.
With this upgrade, that requirement has been removed and `MAX_SCRIPT_ELEMENT_SIZE` will be the new limit.

Any result must be returned as a minimally encoded script number, e.g. number 1 is to be returned as `01` rather than `0100`, number -1 is to be returned as `81` rather than `0180`, and number 0 is to be returned as empty stack item rather than `00`.

#### Unary Operations

Any unary operation must fail if executed on empty stack (stack depth of 0).

##### OP_1ADD (0x8b)

Pop one item from stack, add 1 to the value, push the result on stack.
This may increase the size of the stack item, e.g. if executed on `7f` (127) the result must be `ff00` (128).
This may reduce the size of the stack item, e.g. if executed on `8080` (-128) the result must be `ff` (-127).

##### OP_1SUB (0x8c)

Pop one item from stack, subtract 1 from the value, push the result on stack.
This may increase the size of the stack item, e.g. if executed on `ff` (-127) the result must be `8080` (-128).
This may reduce the size of the stack item, e.g. if executed on `ff00` (128) the result must be `7f` (127).

##### OP_NEGATE (0x8f)

Pop one item from stack, if it is nonzero, flip the sign, if it is zero, leave the item unchanged. In either case, push the result on stack.
This will always result in the input operand and the result having the same byte size.

##### OP_ABS (0x90)

Pop one item from stack, if value is negative flip the sign else do nothing, push the result on stack.
This will result in same size of the result.

##### OP_NOT (0x91)

Pop one item from stack.
If value is 0 change it to 1, else (for any other value) change it to 0.
Push the result on the stack.
This may reduce the size of the stack item to 0, e.g. if executed on `aabbdd` the result will be an empty stack item (0), size of which is 0.
Or, it may increase the size of the stack item from 0 to 1, e.g. if executed on an empty stack item (0) it will yield `01`.

##### OP_0NOTEQUAL (0x92)

Pop one item from stack.
If value is not 0 change it to 1.
If it is 0, leave it unchanged.
Push the result on stack.
This may reduce the size of the stack item to 1, e.g. if executed on `aabbdd` the result will be `01` (1), size of which is 1.
If executed for an empty stack item (0), it will leave the top stack item unchanged.

#### Binary Operations

Any binary operation must fail if executed on stack depth of 1 or less.

##### OP_ADD (0x93)

Pop two items from stack, add the values together, push the result on stack.

##### OP_SUB (0x94)

Pop two items from stack, subtract the top-most value from the other one, push the result on stack.

##### OP_MUL (0x95)

Pop two items from stack, multiply the values together, push the result on stack.

##### OP_DIV (0x96)

Pop two items from stack.
If the top-most item (divisor) is 0, fail immediately.
Divide the 2nd-to-top value (dividend) with the top-most value (divisor).
Push the **quotient** to stack, e.g.

- 3/2 will return 1,
- -3/2 will return -1,
- 3/-2 will return -1, and
- -3/-2 will return 1.

##### OP_MOD (0x97)

Pop two items from stack.
If the top-most item (divisor) is 0, fail immediately.
Divide the 2nd-to-top value (dividend) with the top-most value (divisor).
Push the **remainder** to stack.
The sign of the result will match the sign of the dividend, e.g.

- 7/3 will return 1,
- -7/3 will return -1,
- 7/-3 will return 1, and
- -7/-3 will return -1.

##### OP_BOOLAND (0x9a)

Pop two items from stack.
If both numbers are non-zero then return 1, else return 0.

##### OP_BOOLOR (0x9b)

Pop two items from stack.
If at least one of the numbers is non-zero then return 1, else return 0.

##### OP_NUMEQUAL (0x9c)

Pop two items from stack.
If the two numbers are equal then return 1, else return 0.

##### OP_NUMEQUALVERIFY (0x9d)

Pop two items from stack.
If the two numbers are equal then continue evaluation with nothing returned to stack, else fail.

##### OP_NUMNOTEQUAL (0x9e)

Pop two items from stack.
If the two numbers are not equal then return 1, else return 0.

##### OP_LESSTHAN (0x9f)

Pop two items from stack.
If the 2nd-to-top value is less than the top-most value then return 1, else return 0.

##### OP_GREATERTHAN (0xa0)

Pop two items from stack.
If the 2nd-to-top value is greater than the top-most value then return 1, else return 0.

##### OP_LESSTHANOREQUAL (0xa1)

Pop two items from stack.
If 2nd-to-top value is less than or equal to the top-most value then return 1, else return 0.

##### OP_GREATERTHANOREQUAL (0xa2)

Pop two items from stack.
If 2nd-to-top value is greater than or equal to the top-most value then return 1, else return 0.

##### OP_MIN (0xa3)

Pop two items from stack.
Return the lesser of the 2 values (if they're equal just return any one value).

##### OP_MAX (0xa4)

Pop two items from stack.
Return the greater of the 2 values (if they're equal just return any one value).

### Ternary Operations

Any ternary operation must fail if executed on stack depth of 2 or less.

##### OP_WITHIN (0xa5)

Pop three items from stack.  
The top-most item defines the "right" and non-inclusive boundary of a range.  
The 2nd-to-top item defines the "left" and inclusive boundary of a range.  
The 3rd-to-top item is the "value" to be evaluated.  
If "value" being evaluated is in the range (greater or equal to the "left" value, and less than the "right" value) then return 1, else return 0.

Consider the following examples (shorthand for `<value> <left> <right> OP_WITHIN`):

- evaluating `1 5 8 OP` will return 0,
- evaluating `5 5 8 OP` will return 1,
- evaluating `7 5 8 OP` will return 1, and
- evaluating `8 5 8 OP` will return 0.

Uses where the "left" value is greater or equal to the "right" value are valid and will always return 0, e.g.

- evaluating `X 8 5 OP` will return 0 for any value of X.

## Rationale

### Removal of Number Length Limit

With the activation of the [Arithmetic Operation Cost Limit](https://github.com/bitjson/bch-vm-limits#arithmetic-operation-cost), the additional limitation on VM number length has no impact on worst-case transaction validation performance (see [Tests & Benchmarks](#tests--benchmarks)). This proposal removes the limit, allowing valid VM numbers to extend to the stack element size limit (A.K.A. `MAX_SCRIPT_ELEMENT_SIZE`) of 10,000 bytes. See [`CHIP: Targeted Virtual Machine Limits`](https://github.com/bitjson/bch-vm-limits).

Alternatively, this proposal could raise the limit to a higher constant value like `258`, the constant selected by Satoshi Nakamoto in [`reverted makefile.unix wx-config -- version 0.3.6`](https://gitlab.com/bitcoin-cash-node/bitcoin-cash-node/-/commit/757f0769d8360ea043f469f3a35f6ec204740446) (July 29, 2010). However, because the limit is no longer necessary for mitigating worst-case transaction validation cost, selection of any particular constant would be arbitrary.

By fully-removing the limit, overall protocol complexity is reduced, simplifying both future VM implementations and contract development. For VM implementations, eliminating out-of-range cases significantly reduces the combinatorial set of possible inputs and outputs for the numeric operations: `OP_1ADD` (`0x8b`), `OP_1SUB` (`0x8c`), `OP_NEGATE` (`0x8f`), `OP_ABS` (`0x90`), `OP_NOT` (`0x91`), `OP_0NOTEQUAL` (`0x92`), `OP_ADD` (`0x93`), `OP_SUB` (`0x94`), `OP_MUL` (`0x95`), `OP_DIV` (`0x96`), `OP_MOD` (`0x97`), `OP_BOOLAND` (`0x9a`), `OP_BOOLOR` (`0x9b`), `OP_NUMEQUAL` (`0x9c`), `OP_NUMEQUALVERIFY` (`0x9d`), `OP_NUMNOTEQUAL` (`0x9e`), `OP_LESSTHAN` (`0x9f`), `OP_GREATERTHAN` (`0xa0`), `OP_LESSTHANOREQUAL` (`0xa1`), `OP_GREATERTHANOREQUAL` (`0xa2`), `OP_MIN` (`0xa3`), `OP_MAX` (`0xa4`), and `OP_WITHIN` (`0xa5`). For contract authors, eliminating the possibility of out-of-range errors prevents a class of potential vulnerabilities arising from a contract system's failure to validate that intermediate arithmetic results never exceed an (uncommonly-encountered) maximum number length limit.

### Non-Inclusion of Implementation-Specific Technical Details

This proposal specifies only the necessary change to the consensus protocol: removal of the number length limit.

The software changes required to support this consensus change differ significantly from implementation to implementation. VM implementations which already internally use arbitrary-precision arithmetic for VM number manipulation may only need to disable code enforcing the previous limit, while other implementations may be required to integrate an arbitrary-precision arithmetic library or language primitive. In all cases, implementations should verify their functional behavior and performance characteristics against the [Tests & Benchmarks](#tests--benchmarks).

### Non-Inclusion of VM Number Format or Operation Descriptions

Beyond dropping the unnecessary limit on VM number length, this proposal does not modify any other properties of the VM. Notably, the precise format and behavior of VM numbers across all VM operations – especially `OP_1ADD` (`0x8b`), `OP_1SUB` (`0x8c`), `OP_NEGATE` (`0x8f`), `OP_ABS` (`0x90`), `OP_NOT` (`0x91`), `OP_0NOTEQUAL` (`0x92`), `OP_ADD` (`0x93`), `OP_SUB` (`0x94`), `OP_MUL` (`0x95`), `OP_DIV` (`0x96`), `OP_MOD` (`0x97`), `OP_BOOLAND` (`0x9a`), `OP_BOOLOR` (`0x9b`), `OP_NUMEQUAL` (`0x9c`), `OP_NUMEQUALVERIFY` (`0x9d`), `OP_NUMNOTEQUAL` (`0x9e`), `OP_LESSTHAN` (`0x9f`), `OP_GREATERTHAN` (`0xa0`), `OP_LESSTHANOREQUAL` (`0xa1`), `OP_GREATERTHANOREQUAL` (`0xa2`), `OP_MIN` (`0xa3`), `OP_MAX` (`0xa4`), and `OP_WITHIN` (`0xa5`) – are part of network consensus and do not otherwise change as a result of this proposal. For the avoidance of doubt, see the [Tests & Benchmarks](#tests--benchmarks).

## Stakeholder Responses & Statements

[Stakeholder Responses & Statements &rarr;](stakeholders.md)

## Tests & Benchmarks

[`CHIP: Targeted Virtual Machine Limits`](https://github.com/bitjson/bch-vm-limits) includes a suite of functional tests and benchmarks to verify the behavior and performance of all operations within virtual machine implementations, including high-precision arithmetic operations. See [CHIP Limits: Tests & Benchmarks](https://github.com/bitjson/bch-vm-limits/blob/master/tests-and-benchmarks.md) for details.

## Implementations

Please see the following reference implementations for additional examples and test vectors:

- C++:
  - [Bitcoin Cash Node (BCHN)](https://bitcoincashnode.org/) – A professional, miner-friendly node that solves practical problems for Bitcoin Cash. [Merge Request !1876](https://gitlab.com/bitcoin-cash-node/bitcoin-cash-node/-/merge_requests/1876).
- JavaScript/TypeScript
  - [Libauth](https://github.com/bitauth/libauth) – An ultra-lightweight, zero-dependency JavaScript library for Bitcoin Cash. [Pull Request #139](https://github.com/bitauth/libauth/pull/139).
  - [Bitauth IDE](https://github.com/bitauth/bitauth-ide) – An online IDE for bitcoin (cash) contracts. [Pull Request #101](https://github.com/bitauth/bitauth-ide/pull/101).

## Feedback & Reviews

- [BigInt CHIP Issues](https://github.com/bitjson/bch-bigint/issues)
- [`CHIP 2024-07 BigInt` - Bitcoin Cash Research](https://bitcoincashresearch.org/t/chip-2024-07-bigint-high-precision-arithmetic-for-bitcoin-cash/1356)

## Changelog

This section summarizes the evolution of this document.

- **v1.1.0 – 2024-08-28**
  - Remove the VM number length limit ([#1](https://github.com/bitjson/bch-bigint/issues/1))
- **v1.0.0 – 2024-07-24** ([`b114c957`](https://github.com/bitjson/bch-bigint/commit/b114c95729e670f4b0780d4fd14590c35d281d77))
  - Initial publication

## Copyright

This document is placed in the public domain.
