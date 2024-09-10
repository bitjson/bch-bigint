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

## Rationale

### Removal of Number Length Limit

With the activation of the [Arithmetic Operation Cost Limit](https://github.com/bitjson/bch-vm-limits#arithmetic-operation-cost), the additional limitation on VM number length has no impact on worst-case transaction validation performance (see [Tests & Benchmarks](#tests--benchmarks)). This proposal removes the limit, allowing valid VM numbers to extend to the stack element size limit (A.K.A. `MAX_SCRIPT_ELEMENT_SIZE`) of 10,000 bytes. See [`CHIP: Targeted Virtual Machine Limits`](https://github.com/bitjson/bch-vm-limits).

Alternatively, this proposal could raise the limit to a higher constant value like `258`, the constant selected by Satoshi Nakamoto in [`reverted makefile.unix wx-config -- version 0.3.6`](https://gitlab.com/bitcoin-cash-node/bitcoin-cash-node/-/commit/757f0769d8360ea043f469f3a35f6ec204740446) (July 29, 2010). However, because the limit is no longer necessary for mitigating worst-case transaction validation cost, selection of any particular constant would be arbitrary.

### Interaction with Existing Operations

With one exception regarding overflow, the behavior of every operation remains unchanged. The overflow exception is described in [Removal of Overflow Behavior](#removal-of-overflow-behavior).

The following list is the complete set of operations that interact with data on the stack with the assumption that the input(s) and/or output are VM numbers: `OP_NUM2BIN` (`0x80`), `OP_BIN2NUM` (`0x81`), `OP_1ADD` (`0x8b`), `OP_1SUB` (`0x8c`), `OP_NEGATE` (`0x8f`), `OP_ABS` (`0x90`), `OP_NOT` (`0x91`), `OP_0NOTEQUAL` (`0x92`), `OP_ADD` (`0x93`), `OP_SUB` (`0x94`), `OP_MUL` (`0x95`), `OP_DIV` (`0x96`), `OP_MOD` (`0x97`), `OP_BOOLAND` (`0x9a`), `OP_BOOLOR` (`0x9b`), `OP_NUMEQUAL` (`0x9c`), `OP_NUMEQUALVERIFY` (`0x9d`), `OP_NUMNOTEQUAL` (`0x9e`), `OP_LESSTHAN` (`0x9f`), `OP_GREATERTHAN` (`0xa0`), `OP_LESSTHANOREQUAL` (`0xa1`), `OP_GREATERTHANOREQUAL` (`0xa2`), `OP_MIN` (`0xa3`), `OP_MAX` (`0xa4`), and `OP_WITHIN` (`0xa5`).

### Simplification of VM Implmentations and Contracts

By fully-removing the limit, overall protocol complexity is reduced, simplifying both future VM implementations and contract development. For VM implementations, eliminating out-of-range cases significantly reduces the combinatorial set of possible inputs and outputs for the complete set of numeric operations listed in [Interaction with Existing Operations](#interaction-with-existing-operations). For contract authors, eliminating the possibility of out-of-range errors prevents a class of potential vulnerabilities arising from a contract system's failure to validate that intermediate arithmetic results never exceed an (uncommonly-encountered) maximum number length limit.

### Non-Inclusion of Implementation-Specific Technical Details

This proposal specifies only the necessary change to the consensus protocol: removal of the number length limit.

The software changes required to support this consensus change differ significantly from implementation to implementation. VM implementations which already internally use arbitrary-precision arithmetic for VM number manipulation may only need to disable code enforcing the previous limit, while other implementations may be required to integrate an arbitrary-precision arithmetic library or language primitive. In all cases, implementations should verify their functional behavior and performance characteristics against the [Tests & Benchmarks](#tests--benchmarks).

### Non-Inclusion of VM Number Format or Operation Descriptions

As described in [Interaction with Existing Operations](#interaction-with-existing-operations), this proposal does not modify any other properties of the VM. Notably, the precise format and behavior of VM numbers across all VM operations is part of network consensus and does not change as a result of this proposal. For the avoidance of doubt, see the [Tests & Benchmarks](#tests--benchmarks).

### Removal of Overflow Behavior

As a practical consequence of removing the limit, numerical operation overflow is no longer a meaningful concept. A number that becomes too large would be too large due to the cost system established in [`CHIP: Targeted Virtual Machine Limits`](https://github.com/bitjson/bch-vm-limits), not due to a numerical boundary. These operations previously required overflow considerations: `OP_1ADD` (`0x8b`), `OP_1SUB` (`0x8c`), `OP_ADD` (`0x93`), `OP_SUB` (`0x94`), `OP_MUL` (`0x95`).

Example showing previous 64-bit integer overflow no longer overflows:

- In the current VM, multiplication of two large 64-bit numbers using `OP_MUL` (`0x95`), where the result is too large to fit in the 64-bit number specification, will result in an overflow and therefore script failure. In other words, the two inputs are valid, but the output is too large.
- In the VM upgraded with both this CHIP and the cost system of [`CHIP: Targeted Virtual Machine Limits`](https://github.com/bitjson/bch-vm-limits), the multiplication of the same two numbers no longer results in script failure per se. It might result in script failure if the overall cost of the transaction happens to be exceeded by this specific operation.

Example showing removal of overflow behavior in general:

- In the VM upgraded with both this CHIP and the cost system of [`CHIP: Targeted Virtual Machine Limits`](https://github.com/bitjson/bch-vm-limits), the maximum number is a 10,000 byte number, due to the cost system.
- Multiplying two valid numbers that result in, for example, a 10,005 byte number would result in script failure due to the result being too large to push onto the stack.
- Notably, the failure would not be due to an overflow of the number system.

## Stakeholder Responses & Statements

[Stakeholder Responses & Statements &rarr;](stakeholders.md)

## Tests & Benchmarks

[`CHIP: Targeted Virtual Machine Limits`](https://github.com/bitjson/bch-vm-limits) includes a suite of functional tests and benchmarks to verify the behavior and performance of all operations within virtual machine implementations, including high-precision arithmetic operations. See [CHIP Limits: Tests & Benchmarks](https://github.com/bitjson/bch-vm-limits/blob/master/tests-and-benchmarks.md) for details.

Correctness of big integer arithmetic operations implementation in [Bitcoin Cash Node (BCHN)](https://bitcoincashnode.org/) has been additionally verified by running [property tests](https://en.wikipedia.org/wiki/Software_testing#Property_testing) for all combinations of parameters, as per:

- [Property Tests for Big Integer Arithmetic Script Operations](https://gitlab.com/cculianu/bitcoin-cash-node/-/blob/b7e767a48c5be3265f578013b256cae6bb0b8fd3/doc/bigint-script-property-tests.md)

Running the whole test suite will result in about 1.7 million script evaluations, covering all combinations of random input parameters and their edge values.

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
