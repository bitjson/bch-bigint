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

The limit on the maximum length of Bitcoin Cash VM numbers (A.K.A. `nMaxNumSize`) is removed for following operations: `OP_NUM2BIN` (`0x80`), `OP_BIN2NUM` (`0x81`), `OP_1ADD` (`0x8b`), `OP_1SUB` (`0x8c`), `OP_NEGATE` (`0x8f`), `OP_ABS` (`0x90`), `OP_NOT` (`0x91`), `OP_0NOTEQUAL` (`0x92`), `OP_ADD` (`0x93`), `OP_SUB` (`0x94`), `OP_MUL` (`0x95`), `OP_DIV` (`0x96`), `OP_MOD` (`0x97`), `OP_BOOLAND` (`0x9a`), `OP_BOOLOR` (`0x9b`), `OP_NUMEQUAL` (`0x9c`), `OP_NUMEQUALVERIFY` (`0x9d`), `OP_NUMNOTEQUAL` (`0x9e`), `OP_LESSTHAN` (`0x9f`), `OP_GREATERTHAN` (`0xa0`), `OP_LESSTHANOREQUAL` (`0xa1`), `OP_GREATERTHANOREQUAL` (`0xa2`), `OP_MIN` (`0xa3`), `OP_MAX` (`0xa4`), and `OP_WITHIN` (`0xa5`).

The numbers will still be limited, but by the maximum stack item size (A.K.A. `MAX_SCRIPT_ELEMENT_SIZE`).
The effective limits will then be a calculated value from `MAX_SCRIPT_ELEMENT_SIZE`:

- `MAX_SCRIPTNUM = 2^(MAX_SCRIPT_ELEMENT_SIZE * 8 - 1) - 1`, and
- `MIN_SCRIPTNUM = -MAX_SCRIPTNUM`,

rather than independently set limits (as they were before this upgrade).
This is equivalent because script numbers are LE integers with highest bit used as sign bit so a stack item of `MAX_SCRIPT_ELEMENT_SIZE` size and with all bits set (`0xffff...ff...ffff`) will be equal to `MIN_SCRIPTNUM`, and an item of same maximum size but with all bits except the highest bit set (`0xffff...ff...ff7f`) will be equal to `MAX_SCRIPTNUM`.

Any operation that would result in a stack item bigger than `MAX_SCRIPT_ELEMENT_SIZE` must fail, e.g. `{stack: a} OP_1ADD` must fail for `a = MAX_SCRIPTNUM`.

By coupling the numerical limit with `MAX_SCRIPT_ELEMENT_SIZE` we can guarantee that **any stack item** can be cast to a minimally-encoded script number, e.g. `{stack: a} OP_BIN2NUM` will be a valid operation for any `a`, since it will not be possible to have a stack item that would be interpreted as value outside the `[MIN_SCRIPTNUM, MAX_SCRIPTNUM]` range.
This is because any operation, including data push operations, that would attempt to place an item bigger than MAX_SCRIPT_ELEMENT_SIZE on the stack will fail immediately.

Once cast with `OP_BIN2NUM`, any resulting value can be safely used as input to any arithmetic opcode, e.g. `{stack: a} OP_BIN2NUM OP_1 OP_MUL` will be valid operation for any `a`.

The [`CHIP: Targeted Virtual Machine Limits`](https://github.com/bitjson/bch-vm-limits) will set `MAX_SCRIPT_ELEMENT_SIZE` to 10,000 bytes, therefore valid range for results of arithmetic operations will effectively be the inclusive range: `[-2^(MAX_SCRIPT_ELEMENT_SIZE * 8 - 1) + 1, 2^(MAX_SCRIPT_ELEMENT_SIZE * 8 - 1) - 1]`, which resolves to `[-2^79999 + 1, 2^79999 - 1]`.

Other opcodes consuming a numerical value (such as `OP_PICK`, `OP_CHECKMULTISIG`, `OP_CHECKLOCKTIMEVERIFY`, `OP_UTXOVALUE`, etc.) are not affected and existing limits on their input arguments will not be changed.

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

## Activation Costs

We define activation cost as any cost that would arise ex-ante, in anticipation of the activation by affected stakeholders, to ensure that their services will continue to function without interruption once the activation block has been mined.
In case of this proposal, activation cost is contained to nodes and libraries that implement the Script VM, and will amount to:

- Some fixed amount of node developer man-hours, in order to release updated node software that will be able to correctly calculate the algorithmic limit.
- Some fixed amount of work by stakeholders running node software, in order to update the software in anticipation of algorithm activation.
- Some fixed amount of effort by others involved in reviewing and testing the new node software version.
- Some fixed amount of effort by library maintainers in order to update their VM implementations to correctly execute arithmetic operations that would be failing pre-upgrade.

This is the same kind of cost that any upgrade to the Script VM system has had or will have.
Nothing is required of stakeholders who are neither running nodes nor involved in node development or smart contract development.
Smart contract developers are the direct beneficiaries of this upgrade, so we expect them to be happy to bear the activation costs.

## Ongoing Costs

We define ongoing cost as any cost that would arise post upgrade.
Thanks to the opcode cost budgeting sytem introduced in [CHIP: Targeted Virtual Machine Limits](https://github.com/bitjson/bch-vm-limits), valiated by benchmarks done, we can be assured that costs of validating transactions WILL NOT be negatively impacted.

## Risk Assessment

If we define risk as [probability times severity](https://en.wikipedia.org/wiki/Risk#Expected_values) then we could make an estimate using the [risk matrix](https://en.wikipedia.org/wiki/Risk_matrix) framework, considering both severity and likelihood in our consideration of risk.

Generic consensus upgrade risks apply.
Probability of bugs and network splits is entirely controlled by node developers and quality of their work.
Generic risks are mitigated by node software development quality control, review, and testing processes, leaving the probability of a critical bug very low, and with that the generic risk is kept low.

Below we will examine some risks specific to this upgrade.

### Big Integer Implementation Risks

As we [mentioned above](#rationale), Satoshi Nakamoto himself removed the BigInt support from Bitcoin's codebase, because of instability in the OpenSSL's big number library, and the risk of different nodes coming to different conclusions about result of some Script operation and it causing a "hard" network split.

Satoshi Nakamoto was a solo developer trying to bootstrap a never before seen project, so we can argue that it was a logical choice to prioritize other things, and *temporarily* remove the risk by simply removing big integer support entirely.
We are now in a much better position, where we can give the problem the attention it deserves, and remove such risks while accessing the benefits of having big integers.

It was not the only risky dependency - signature cryptography (with secp256k1 elliptic curve) itself also required an external dependency, and later, when more developers got involved, they implemented [secp256k1](https://github.com/bitcoin/bitcoin/pull/4312) by themselves.

Thankfully, basic arithmetic operations on big integer numbers are a much simpler problem, and a problem where many industries needed a solution, so reliable big number libraries now exist for most programming languages.
Initial implementation for BCHN intends to use [The GNU Multiple Precision Arithmetic Library](https://gmplib.org/) (AKA `gmp`), which is a time-tested and performant library, of which we only need a subset of operations for BCH Script arithmetic opcodes.
The same library was used by libsecp256k1, until 2015 when it was [fully removed](https://github.com/bitcoin/bitcoin/commit/75a880390191aeaf7d2fa326f194349a891db022#diff-242b05bb9c57125ed0fbecbda608e3fa6fd3d7cae41e1cba0dcfd706252a7fc0R413) as a dependency.

There exist big integer libraries for languages other than C++ and used in Bitcoin Cash node implementations:

- Golang's [`math/big`](https://pkg.go.dev/math/big) package, expected to be used with [BCHD](https://github.com/OPReturnCode/bchd), considering it is already internally used for [cryptography operations](https://github.com/OPReturnCode/bchd/blob/67d41ee8838026571863491e88288d6d2d5cb717/bchec/privkey.go#L11).
- Java's [`java.math,BigInteger`](https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/math/BigInteger.html) library, expected to be used with [Bitcoin Verde](https://github.com/SoftwareVerde/bitcoin-verde), considering it is already internally used for [uint256 operations](https://github.com/SoftwareVerde/bitcoin-verde/blob/e9d140cac8b93a9db572bda906db9decfac1d7ae/src/main/java/com/softwareverde/bitcoin/block/header/difficulty/work/ChainWork.java#L8). It doesn't need to use it for cryptography operatins, because native secp256k1 implementation exists for Java and [is used by Bitcoin Verde](https://github.com/SoftwareVerde/bitcoin-verde/blob/e9d140cac8b93a9db572bda906db9decfac1d7ae/src/main/java/org/bitcoin/NativeSecp256k1.java#L5).

Similarly, there exist big integer libraries for languages in which Bitcoin Cash smart contract libraries are written:

- JavaScript's [`BigInt`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/BigInt) library, expected to be used with Bitcoin Cash JavaScript libraries such as [libauth](https://github.com/bitauth/libauth) and [CashScript](https://github.com/CashScript/cashscript).
- Python [natively supports unlimited integer precision](https://docs.python.org/3/library/stdtypes.html#numeric-types-int-float-complex), expected to be used with [python-bitcoincash](https://github.com/dagurval/python-bitcoincash).

All these implementations must produce exact same results for exact same inputs, else node implementations would risk a "hard" network split and smart contract libraries would risk faulty transaction validation breaking user-facing applications.

This risk is mitigated by having a [comprehensive set of test vectors](#implementations), which cover all the arithmetic opcodes and around all the edges, and that ensures low implementation risk as any implementation errors would be caught by the test suite, and before deployment.

### Risks to Existing Smart Contracts

This upgrade would make some previously invalid contract bytecode become valid.
If arithmetic opcode's failure mode was intentionally used by some contract, then such contract may start behaving in unintended ways should the opcodes cease to be failing.

This risk existed when Bitcoin Cash upgraded arithmetic opcodes from int32 to int64, and was the reason for [this notice](https://gitlab.com/GeneralProtocols/research/chips/-/blob/master/CHIP-2021-02-Bigger-Script-Integers.md):

> As always, the security of a contract is the responsibility of the entity locking funds in that contract; funds can always be locked in insecure contracts (e.g. `OP_DROP OP_1`). This notice is provided to warn contract authors and explicitly codify a network policy: the possible existence of poorly-designed contracts will not preclude future upgrades from further expanding the range of Script Numbers.
>
> To ensure a contract will always fail when arithmetic results overflow or underflow 8-byte Script Numbers (in the rare case that such a behavior is desirable), that behavior must be either 1) explicitly validated or 2) introduced to the contract prior to the activation of any future upgrade which expands the range of Script Numbers.

What is the likelihood of a contract writer not being aware of the above notice, and then devising such a contract where it would actually result in unintended results rather than not affecting active instances or just expanding the working-as-intended scope of the contract?
The Bitcoin Cash smart contract development ecosystem is still small enough, so most builders know each other, and smart contract applications with any number of users quickly become common knowledge.
As part of the CHIP process they will all be contacted for [final confirmation](#stakeholder-responses--statements).

Even before that, we can already examine how this upgrade would impact some of those:

- If we examine the [Cauldron contract](https://docs.riftenlabs.com/cauldron/whitepaper/#appendix-cauldron-contract) we can see that it would continue to function as intended even with increased precison, rather than fail and require the liquidity pools to be split across smaller ones.
Because of how it was written it automatically benefits from this upgrade.
- If we examine the Fex Cash contract, we can see a different approach: they worked around the precision issue by [explicitly limiting](https://github.com/fex-cash/fex/blob/main/covenants/amm/burn_lp_token.cash#L7) size of input integers.
In this case, the contract would work the same regardless of whether we upgraded integers or not, but would not automatically benefit from increased precision.
- If we examine the AnyHedge contract we can see it does not even use multiplication, and [`nominalUnitsXSatsPerBch`](https://gitlab.com/GeneralProtocols/anyhedge/contracts/-/blob/development/contracts/v0.12/contract.cash#L27) is set at creation of a contract instance and anything higher than current limits would make it unredeemable, while post-upgrade it would continue to work as intended, and it would work with a wider range of contract parameters.
- If we examine the [BCH Guru contract](https://ide.bitauth.com/import-template/eJztWFtv28YS_isGcR7dZO-XvMmyGguRJdeiclwkgbGX2VqtLelYVNAg8H8_Q0qiREm-FWhRFPWDSQ5nZr-Z-WZ2qe_Zf-bhBu5c9i67KYrZ_N3bt-MIb_y4cIvi5k2Y3r0tb2BSjIMrxtPJDwXczW5dAT98JW-Wtm9-nU8n2XEWYR7ux7NSC92hYOLuAO9O2mdH7xf3i6P86khzQSyRSRgBiSVpnfRWeSWASkjAHBGEMhRoaiRN1ARiqPY2AIAQJKLXEksxhnn27vvDcTYPMHH342n5mM2hKG7hDjWui99LwWOQlor4XNy7ydyFpcL3bDyZLQp09el7Nl0Us-l4UnQnEdAXOa4l-cbmzM1v0J0TiWkiePRAaAjJKmnAxxhT8l6qpCAkI7S0VifjSXIQozdJaxJCgOgRyBz-t4BJgP7izsN9td5icjsNv40nv5x8KyBMIwL_lM1vp0X25eF4DyD9WwDEGlQJxzUTlvx6leiHhy_H2Ve4n1d5ZsdZaViMy1qsErtO-1d3u4ChK6bzmzFKKCFkqd1YJtPKWSqUSdES4blhNHoajY6JGSqY9EY5mrykyB_DjXGhLPb0N6jKjFSGX6b339BRtM4zAZFKp41QIfBogRDhnAzeamUUSZRHFyi4SCTqIBGTUIw5zCI4dOvuposJRswQKwZaknK6uA8weHVY6_IehqokdcANF8YpB0FhQZWWPEYgnDJOLTc0mYi9pblFjJpEIoKIEdEGTVNJ_1Qsnc6cH9-Oi9LtZDopOwFb_W5clL2DMoxfIUSyTDP4JBIkTrXlVhOJWdBJSYvd6pRPyVLHIUio0vzwUJLzZfHusCV7-AtrVNHznzklttLqodj04N91aNRT7d8h8bqmweo-1jP_vGHx9HD8E0pUnS7KTFdni61OwqeZm8_LI8innTPHl3p4bOkfbyo2uLi-GA3PTn7OO8NrJY7I78566hKyLzjjUqCGesYDjZZhGpm11AtwwflomARGJMXEGm-4Z1R56rG6kitKwSulhQDpMUAniApJCBM8UBoT9VIrIiwF8NIpZS1ojycvooF-njQwUYWYaEgOO1y5yC3xhFi1ulojy0rv2FRxSM8c80JRFUlSzHEuU9DB4wBBoQaOp7oUpIQQlCQpgAWpELbxghJpPBVeWaCAnEERYzbopPHsB1IQjjPGOBdY8AJZpplVlksqKFYPqQfSqHAoDmlWccAKv15edXg8DpYIM5wqQ2JUkavkTJluPLeKAAGbyAdtOEnCIQLKXUgScRgcfz4pmxK2XMQqci2jMdhXwKQh2lK0ooIrjqNUI0uVjSZiXqOi1hlKrbQEB5XarYcpIRlXoiUMi-wcWaog-Gw9TeebWbB96mrwdL3fbRi6rbpF0YbX9bZcekd3q6bMv81KBzM2v2Ek26b8o1xnEgN5_UDeyQZn6MUyiAQ7A_cipWPUSDRm0FI54QVxnHhrQzlxQgoei6IowTmnQpQ2xBDNrk-OPgmHVC4aIxUyKI_0wmkotFGMeG2w2ZA9RnmIgLTETxf8JMEvGC0U4FQlOvKDXlkkNnqCjU1TEmCE5JbaEIwqwYJgOmqfgJrEApFIdSECtSqFpFLEnOxiLQnKxZLUnyfVS3lUanTbH5asKJ_6o_POT6NWr5J0f_w8OcI_vNUb1Uclovx3Oej1akn7rNP-cNrKW8Pu-4-dy-6PPy9f1QqXg7y8qC1Br1deh2ctJjfiCtNBD_lVt38xytuDUT-vhay-W8dz0Lay7PZPO1e1iLzMssrVYJRfDLr9PL86aw3P6nf0iXePe2xYNTHRl2Eye-k3eyXie5JnS6T23KrDbl9XebnnpEI7vOh1K0r0uxc1p2rh6eWgkp50-wxzUZueji6apXl_2Wnlncv8rNWvX_z5MOhwdPKyWsm9hD25wmGkz5oM_9uqRGzPtNcZDp9PzlprcLmaCU8p73OAskOp3E1aVa9RfjXIBx86_fbg_Lybn3e2mnkT5h9N3z79K-3RSXlpnQy3JtFBi3oo1RbNtq3Rt86bY2h7ucYDLtEf5M2srqft7ogZ5Xu-VyoDLERDcIhzmyJtp6pGtedjr3UOwDvkCcvb0O_0T7dNHout3Jzag9PO04EdjGpV6M1w7Q07fySFdXVOux9fn87nYtnp26cCamjQvwI8fQa8eBn4ZrlLROsObN5sb2mNgh3safrsGG2ebggJwSdqqx49PW3uRb1B-0PePe8s_TS5sz-LPrZ6o86BZcozJBjCV9Pg5UNsb1fZP0qM8uaq-1O7WefnkvMoMQeHNvUXnEx2Aa5G5utQHWBcvfzGw3Kr2FyWdCkptv5Qef6TotJqflMcPLYNHjmy7SZlX3N1TKv5vA48q77_F7PZ9L6AWH5BnbTPrhlh_JpIhFQ-DS867Wz7Ryvy8H_DLmXP), we can see it'd be impacted in similar way as AnyHedge.

These all have active contracts that handle non-trivial amounts of money, and their developers are well aware of the notice, and their contracts wouldn't be broken by this upgrade anyway.
We can expect similar findings with any others, because it would be hard to actually implement a contract that would be broken by this upgrade.

With this in mind we can estimate this risk to be extremely low, because both likelihood and severity of some exotic contract breaking are low.

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
