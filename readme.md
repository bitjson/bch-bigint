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

### Practical Applications

Currently the Bitcoin Cash network has a few decentralized applications that would benefit from higher precision arithmetic limits:

- [AnyHedge](https://gitlab.com/GeneralProtocols/anyhedge), a decentralized hedge solution against arbitrary assets on Bitcoin Cash, currently with approx. 25,000 BCH in total value locked (TVL),
- [Cauldron](https://gitlab.com/riftenlabs/cauldron-whitepaper), an efficient constant product market maker contract on Bitcoin Cash through micro-pools, currently with approx. 200 BCH in TVL.
- [fex.cash](https://github.com/fex-cash/fex/blob/main/whitepaper/fex_whitepaper.md), an advanced automatic market maker (AMM) contract system, current state unknown.

Both contracts would benefit from increased precision because they could calculate payouts with full precision while avoiding workarounds.

Future applications that could be created with greater ease:

- High-precision state accumulators for finance operations, e.g. adding/removing small shares to/from one big liquidity pool with accurate calculation of payout/deposit amounts, and splitting and merging such liquidity pools without leakeages due to rounding errors. These would be building blocks of more advanced decentralized (DEX) systems or decentralized autonomous organization (DAO) systems.
- Contracts processing block headers in order to verify SPV proofs, or in order to feed mining information into contracts in a trustless way. These applications require performing 256-bit unsigned arithmetics in order to verify accumulated chainwork, so would need at least 513-bit signed integer support. Applications include hashrate predicition markets and gambling/lottery applications (public entropy accumulation).
- Contracts wanting to implement custom operations on elliptic curve(s), e.g. [key tweaking](https://bitcoin.stackexchange.com/questions/110402/how-to-tweak-a-public-key-for-taproot) or amount blinding schemes ([confidential transactions](https://elementsproject.org/features/confidential-transactions)). These operations would similarly require 513-bit signed integer support (for 256-bit keys).
- Contracts wanting to verify Rivest-Shamir-Adleman (RSA) signatures, these operations would require 8193-bit signed integer support (for 4096-bit keys).
- Contracts wanting to experiment with more advanced cryptography such as implementing zero-knowledge scalable transparent argument of knowledge (ZK-STARK) proof verification.

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

[TODO: after initial public feedback]

## Feedback & Reviews

- [BigInt CHIP Issues](https://github.com/bitjson/bch-bigint/issues)
- [`CHIP 2024-07 BigInt` - Bitcoin Cash Research](https://bitcoincashresearch.org/t/chip-2024-07-bigint-high-precision-arithmetic-for-bitcoin-cash/1356)

## Changelog

This section summarizes the evolution of this document.

- **v1.0.0 – 2024-07-24** (current)
  - Initial publication

## Copyright

This document is placed in the public domain.
