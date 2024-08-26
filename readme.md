# CHIP-2024-07-BigInt: High-Precision Arithmetic for Bitcoin Cash

        Title: High-Precision Arithmetic for Bitcoin Cash
        Type: Standards
        Layer: Consensus
        Maintainer: Jason Dreyzehner
        Status: Draft
        Initial Publication Date: 2024-07-24
        Latest Revision Date: 2024-07-24
        Version: 1.0.0

## Summary

This proposal increases the maximum length of numbers in the Bitcoin Cash Virtual Machine from 8 bytes to 258 bytes.

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

The maximum length of Bitcoin Cash VM numbers (A.K.A. `nMaxNumSize`) is increased from `8` to `258`.

### Notice of Possible Future Expansion

While unusual, it is possible to design contracts which rely on the rejection of otherwise-valid VM Numbers which are larger than 258 bytes. Contract authors are advised that future upgrades may further expand the supported range of BCH VM Numbers beyond 258 bytes.

**This proposal interprets such failure-reliant constructions as intentional** – they are designed to fail unless/until a possible future network upgrade in which larger VM Numbers are enabled (i.e. a contract branch which can only be successfully evaluated in the event of such an upgrade).

<details>

<summary>Notes</summary>

As always, the security of a contract is the responsibility of the entity locking funds in that contract; funds can always be locked in insecure contracts (e.g. `OP_DROP OP_1`). This notice is provided to warn contract authors and explicitly codify a network policy: the possible existence of poorly-designed contracts will not preclude future upgrades from further expanding the range of VM Numbers.

To ensure a contract will always fail when arithmetic results overflow or underflow 258-byte VM Numbers (in the rare case that such a behavior is desirable), that behavior must be either 1) explicitly validated or 2) introduced to the contract prior to the activation of any future upgrade which expands the range of VM Numbers.

This notice also appeared in [CHIP-2021-03: Bigger Script Integers](https://gitlab.com/GeneralProtocols/research/chips/-/blob/master/CHIP-2021-02-Bigger-Script-Integers.md#notice-of-possible-future-expansion).

</details>

## Rationale

### Selection of 258 Byte Limit

This proposal increases the maximum length of VM numbers from 8 bytes to 258 bytes. The 258 byte limit is selected to significantly exceed typical requirements, enabling practically all financial and classical cryptographic use cases without emulated precision.

In particular, this limit is two bytes beyond `256` (2<sup>8</sup>), enabling simpler bitwise manipulation of 256-byte results using VM operations.

Notably, this limit is considered to be a conservative choice in part because it matches the initial limit selected by Satoshi Nakamoto in [`reverted makefile.unix wx-config -- version 0.3.6`](https://gitlab.com/bitcoin-cash-node/bitcoin-cash-node/-/commit/757f0769d8360ea043f469f3a35f6ec204740446)
(July 29, 2010), one of a series of emergency patches deployed in 2010 to address potential Denial-Of-Service attacks against the network. This limit was later reduced to 4 bytes in [`misc changes`](https://gitlab.com/bitcoin-cash-node/bitcoin-cash-node/-/commit/4bd188c4383d6e614e18f79dc337fbabe8464c8) (August 15, 2010) as Satoshi removed the VM's dependency on OpenSSL's big number implementation, following [instability he'd discovered in the right shift operation](https://gitlab.com/bitcoin-cash-node/bitcoin-cash-node/-/commit/c4319e678f693d5fbc49bd357ded1c8f951476e9). By July of 2010, Satoshi had identified that such instability created a risk of network splits, and he had begun to limit reliance on dependencies in consensus code.

While this proposal could safely support higher limits, enabling simple manipulation of 512-byte, 1024-byte, or higher precision results, the `258` byte limit is considered more conservative, as it can be raised by future proposals without introducing a new number format or breaking existing applications (see [Notice of Possible Future Expansion](#notice-of-possible-future-expansion)), while limit reductions would be much more disruptive or impossible on today's network.

## Stakeholder Responses & Statements

[TODO: after initial public feedback]

## Test Vectors

[TODO: after initial public feedback]

## Implementations

Please see the following reference implementations for additional examples and test vectors:

### Node Implementations

- Bitcoin Cash Node (BCHN): [Merge Request 1876 - Implement CHIP-2024-07-BigInt: High-Precision Arithmetic for Bitcoin Cash (on top of VM Limits)](https://gitlab.com/bitcoin-cash-node/bitcoin-cash-node/-/merge_requests/1876)

## Activation Costs

We define activation cost as any cost that would arise ex-ante, in anticipation of the activation by affected stakeholders, to ensure that their services will continue to function without interruption once the activation block has been mined.
In case of this proposal, activation cost is contained to nodes and libraries that implement the Script VM, and will amount to:

- Some fixed amount of node developer man-hours, in order to release updated node software that will be able to correctly calculate the algorithmic limit.
- Some fixed amount of work by stakeholders running node software, in order to update the software in anticipation of algorithm activation.
- Some fixed amount of effort by others involved in reviewing and testing the new node software version.
- Some fixed amount of effort by library maintainers in order to update their VM implementations to correctly execute arithmetic operations that would be failing pre-upgrade.

This is the same kind of cost that any upgrade to the Script VM system has had or will have.
Nothing is required of stakeholders who are neither running nodes nor involved in node development or smart contract development. Smart contract developers are the direct beneficiaries of this upgrade, so we expect them to be happy to bear the activation costs.

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

Satoshi Nakamoto was a solo developer trying to bootstart a never before seen project, so we can argue that it was a logical choice to prioritize other things, and *temporarily* remove the risk by simply removing big integer support entirely.
We are now in a much better position, where we can give the problem the attention it deserves, and remove such risks while still unlocking the benefits.

It was not the only risky dependency - signature cryptography (with secp256k1 elliptic curve) itself also required an external dependency, and later, when more developers got involved, they implemented [secp256k1](https://github.com/bitcoin-core/secp256k1) by themselves.

Thankfully, basic arithmetic operations on big integer numbers are a much simpler problem, and a problem where many industries needed a solution, so reliable big number libraries now exist for most programming languages.
Initial implementation for BCHN intends to use [The GNU Multiple Precision Arithmetic Library](https://gmplib.org/) (AKA `gmp`), which is a time-tested and performant library, of which we only need a subset of operations for BCH Script arithmetic opcodes.
The same library was used by secp256k1, until 2021 when it was [fully removed](https://github.com/bitcoin-core/secp256k1/commit/1f233b3fa05eb29a744487e0682d925055fb0d4c) as a dependency.

There exist big integer libraries for other laguages, used in Bitcoin Cash node implementations:

- Golang's [`math/big`](https://pkg.go.dev/math/big) package, expected to be used with [BCHD](https://github.com/OPReturnCode/bchd), considering it is already internally used for [cryptography operations](https://github.com/OPReturnCode/bchd/blob/67d41ee8838026571863491e88288d6d2d5cb717/bchec/privkey.go#L11).
- Java's [`java.math,BigInteger`](https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/math/BigInteger.html) library, expected to be used with [Bitcoin Verde](https://github.com/SoftwareVerde/bitcoin-verde), considering it is already internally used for [uint256 operations](https://github.com/SoftwareVerde/bitcoin-verde/blob/e9d140cac8b93a9db572bda906db9decfac1d7ae/src/main/java/com/softwareverde/bitcoin/block/header/difficulty/work/ChainWork.java#L8). It doesn't need to use it for cryptography operatins, because native secp256k1 implementation exists for Java and [is used by Bitcoin Verde](https://github.com/SoftwareVerde/bitcoin-verde/blob/e9d140cac8b93a9db572bda906db9decfac1d7ae/src/main/java/org/bitcoin/NativeSecp256k1.java#L5).

Similarly, there exist big integer libraries for languages in which Bitcoin Cash smart contract libraries are written:

- JavaScript's [`BigInt`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/BigInt) library, expected to be used with Bitcoin Cash JavaScript libraries such as [libauth](https://github.com/bitauth/libauth) and [CashScript](https://github.com/CashScript/cashscript).
- Python [natively supports unlimited integer precision](https://docs.python.org/3/library/stdtypes.html#numeric-types-int-float-complex), expected to be used with [python-bitcoincash](https://github.com/dagurval/python-bitcoincash).

All these implementations must produce exact same results for exact same inputs, else node implementations would risk a "hard" network split and smart contract libraries would risk faulty transaction validation breaking user-facing applications.

This risk is mitigated by having a [comprehensive set of test vectors](#test-vectors), which cover all the arithmetic opcodes and around all the edges, and that ensures low implemnatiton risk as any implementation errors would be caught by the test suite, and before deployment.

### Risks to Existing Smart Contracts

This upgrade would make some previously invalid contract bytecode become valid.
If arithmetic opcode's failure mode was intentionally used by some contract, then such contract may start behaving in unintended ways should the opcodes cease to be failing.

This risk existed when Bitcoin Cash upgraded arithmetic opcodes from int32 to int64, and was the reason for [this notice](https://gitlab.com/GeneralProtocols/research/chips/-/blob/master/CHIP-2021-02-Bigger-Script-Integers.md):

> As always, the security of a contract is the responsibility of the entity locking funds in that contract; funds can always be locked in insecure contracts (e.g. `OP_DROP OP_1`). This notice is provided to warn contract authors and explicitly codify a network policy: the possible existence of poorly-designed contracts will not preclude future upgrades from further expanding the range of Script Numbers.
>
> To ensure a contract will always fail when arithmetic results overflow or underflow 8-byte Script Numbers (in the rare case that such a behavior is desirable), that behavior must be either 1) explicitly validated or 2) introduced to the contract prior to the activation of any future upgrade which expands the range of Script Numbers.

What is the likelihood of a contract writer not being aware of the above notice, and then devising such a contract where it would actually result in unintended results rather than not affecting active instances or just expanding the working-as-intended scope of the contract? The Bitcoin Cash smart contract development ecosystem is still small enough, so most builders know each other, and dapps with any number of users quickly become common knowledge.
As part of the CHIP process they will all be contacted for [final confirmation](#stakeholder-responses-statements).

Even before that, we can already examine how this upgrade would impact some of those:

- If we examine the [Cauldron contract](https://docs.riftenlabs.com/cauldron/whitepaper/#appendix-cauldron-contract) we can see that it would continue to function as intended even with increased precison, rather than fail and require the liquidity pools to be split across smaller ones.
Because of how it was written it automatically benefits from this upgrade.
- If we examine the fex.cash contrat, we can see a different approach: they worked around the precision issue by [explicitly limiting](https://github.com/fex-cash/fex/blob/main/covenants/amm/burn_lp_token.cash#L7) size of input integers.
In this case, the contract would work the same regardless of whether we upgraded integers or not, but would not automatically benefit from increased precision.
- If we examine the AnyHedge contract we can see it does not even use multiplication, and [`nominalUnitsXSatsPerBch`](https://gitlab.com/GeneralProtocols/anyhedge/contracts/-/blob/development/contracts/v0.12/contract.cash#L27) is set at creation of a contract instance and anything higher than current limits would make it unredeemable, while post-upgrade it would continue to work as intended.
- If we examine the bch.guru contract, we can see it'd be impacted the same as AnyHedge.

These all have active contracts that handle non-trivial amounts of money, and their developers are well aware of the notice, and their contracts wouldn't be broken by this upgrade anyway.
We can expect similar findings with all others, because it would be hard to actually implement a contract that would be broken by this upgrade.

With this in mind we can estimate this risk to be extremely low, because both likelihood and severity of some exotic contract breaking are low.

## Feedback & Reviews

- [BigInt CHIP Issues](https://github.com/bitjson/bch-bigint/issues)
- [`CHIP 2024-07 BigInt` - Bitcoin Cash Research](https://bitcoincashresearch.org/t/chip-2024-07-bigint-high-precision-arithmetic-for-bitcoin-cash/1356)

## Changelog

This section summarizes the evolution of this document.

- **v1.0.0 – 2024-07-24** (current)
  - Initial publication

## Copyright

This document is placed in the public domain.
