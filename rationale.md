# Rationale

This section documents design decisions made in this specification.

<details>

<summary><strong>Table of Contents</strong></summary>

- [Extraction from `CHIP: Targeted Virtual Machine Limits`](#extraction-from-chip-targeted-virtual-machine-limits)
- [Removal of Number Length Limit](#removal-of-number-length-limit)
  - [Alternative: Preserve 8-Byte Limit](#alternative-preserve-8-byte-limit)
    - [Scenario: Never Increase](#scenario-never-increase)
    - [Scenario: Delay Activation](#scenario-delay-activation)
  - [Alternative: Select a Higher Constant Limit](#alternative-select-a-higher-constant-limit)
  - [Full Removal Simplifies Protocol and Contract Review](#full-removal-simplifies-protocol-and-contract-review)
  - [Removal of Overflow Behavior](#removal-of-overflow-behavior)
    - [Consideration of Potential Impacts](#consideration-of-potential-impacts)
- [Omission of Implementation-Specific Technical Documentation](#omission-of-implementation-specific-technical-documentation)
- [Omission of VM Number Format or Operation Documentation](#omission-of-vm-number-format-or-operation-documentation)

</details>

## Extraction from `CHIP: Targeted Virtual Machine Limits`

This proposal is extracted from [`CHIP: Targeted Virtual Machine Limits`](https://github.com/bitjson/bch-vm-limits) following [cross-implementation verification](risk-assessment.md#consideration-of-possible-future-changes) that the number length limit (A.K.A. `nMaxNumSize`) can be simultaneously removed. Though the number length limit is within the scope of the Bitcoin Cash Virtual Machine (VM) limits, `CHIP: Targeted Virtual Machine Limits` did not initially address the number length limit in its published scope.

To maximize stakeholder review, the CHIP maintainer chose to extract this change into an independent proposal.

## Removal of Number Length Limit

With the activation of the [Arithmetic Operation Cost Limit](https://github.com/bitjson/bch-vm-limits#arithmetic-operation-cost), the additional limitation on VM number length has no impact on worst-case transaction validation performance (see [CHIP Limits: Tests & Benchmarks](https://github.com/bitjson/bch-vm-limits/blob/master/tests-and-benchmarks.md)
). This proposal removes the limit, allowing valid VM numbers to extend to the stack element size limit (A.K.A. `MAX_SCRIPT_ELEMENT_SIZE`) of 10,000 bytes. See [`CHIP: Targeted Virtual Machine Limits`](https://github.com/bitjson/bch-vm-limits).

### Alternative: Preserve 8-Byte Limit

Alternatively, [`CHIP: Targeted Virtual Machine Limits`](https://github.com/bitjson/bch-vm-limits) could be deployed without increasing the VM number length limit from it's current value of `8` bytes. This alternative would be reasonable if 1) the maximum length of Bitcoin Cash VM numbers should never be increased, or 2) delaying activation would reduce overall risks, e.g. by allowing an additional year for review.

#### Scenario: Never Increase

In the scenario that Bitcoin Cash would be better off without support for high-precision math, the 8-byte limit would be reasonable to preserve. This might be the case if 1) the VM limit system could not otherwise prevent excessive arithmetic usage or if 2) emulated-precision math were considered always superior to native-precision math.

First, [`CHIP: Targeted Virtual Machine Limits`](https://github.com/bitjson/bch-vm-limits) establishes [Arithmetic Operation Cost](https://github.com/bitjson/bch-vm-limits#arithmetic-operation-cost) to comprehensively limit excessive usage of arithmetic operations, particularly operations with <code>O(n<sup>2</sup>)</code> worst-case performance in naive implementations. Performance requirements are further reduced by the [inclusion of numeric encoding in arithmetic operation costs](https://github.com/bitjson/bch-vm-limits/blob/master/rationale.md#inclusion-of-numeric-encoding-in-operation-costs), eliminating the need even for performance-critical implementations to include potential optimizations.

Second, when compared to native-precision math, emulated-precision math is more wasteful in every applicable measure: greater overall network bandwidth usage, larger individual transaction sizes, greater overall block storage requirements, and greater net computation costs of equivalent arithmetic operations across all validating nodes on the network.

It is notable that Bitcoin (Cash) was originally designed and launched in 2009 with native, arbitrary-precision math. Arithmetic precision was only later limited by a series of emergency patches deployed in 2010 to address potential Denial-Of-Service attacks against the network<sup>1</sup>. The precise, consensus-critical, binary representation of Bitcoin Cash VM numbers (at the time, `CBigNum`) was based on OpenSSL's Multi-Precision Integer (MPI) format, with the leading `length` omitted (as Bitcoin Cash VM stack items must already encode a length) and the remaining bytes reversed (little endian rather than OpenSSL MPI's big endian ordering).

By using a binary representation exposed by a big-integer arithmetic library, the design of the Bitcoin Cash VM optimizes for maximum theoretical performance of native, high-precision math: with a sufficiently optimized implementation, arithmetic operands and results do not need to be decoded or re-encoded when passed between binary and arithmetic operations, e.g. an `OP_MUL` result can be immediately `OP_HASH256`ed without decoding, and the result can be immediately (`OP_BIN2NUM`ed to remove potentially-invalid padding and) used by further arithmetic operations without a potentially-expensive numeric re-encoding step. In practice, this significantly reduces the worst-case overhead of high-precision math even when compared to a hypothetical, native-precision system using a less-amenable binary representation for VM numbers. When compared to emulated-precision math, the difference is even more extreme.

<details>

<summary>Notes</summary>

1. From unlimited precision, precision was first limited on July 29, 2010 to `258` bytes ([`reverted makefile.unix wx-config -- version 0.3.6`](https://gitlab.com/bitcoin-cash-node/bitcoin-cash-node/-/commit/757f0769d8360ea043f469f3a35f6ec204740446), then on August 15, 2010 to `4` bytes ([`misc changes`](https://gitlab.com/bitcoin-cash-node/bitcoin-cash-node/-/commit/4bd188c4383d6e614e18f79dc337fbabe8464c8)) immediately prior to the [patch which resolved CVE-2010-5139](https://gitlab.com/bitcoin-cash-node/bitcoin-cash-node/-/commit/d4c6b90ca3f9b47adb1b2724a0c3514f80635c84). Later [`CHIP-2021-03: Bigger Script Integers`](https://gitlab.com/GeneralProtocols/research/chips/-/blob/master/CHIP-2021-02-Bigger-Script-Integers.md) (activated May 15, 2022) raised the currently-active limit to `8` bytes.

</details>

#### Scenario: Delay Activation

In another scenario, it could be deemed prudent to delay removal of the number length limit to reduce other risks, e.g. consensus risks resulting from defects in specific implementations.

On this topic, it is critical to note that [performance benchmarking for `CHIP: Targeted Virtual Machine Limits`](https://github.com/bitjson/bch-vm-limits/blob/master/tests-and-benchmarks.md) specifically includes worst-case scenarios **made possible only by the combination of both proposals**. For example, Test ID `nevxwn` approaches the worst-case validation cost for 2025 nonstandard cases, with worst-case signature checking requiring less than `2x` the validation time (vs. approximately `11x` the [baseline benchmark](https://github.com/bitjson/bch-vm-limits/blob/master/tests-and-benchmarks.md#baseline-benchmark))<sup>1</sup>. By testing the proposals together, implementers and reviewers can ensure an overall higher quality of review of both the proposed protocol specifications and of each specific implementation.

Alternatively, if production-quality review of this proposal had been de-prioritized until after the deployment of [`CHIP: Targeted Virtual Machine Limits`](https://github.com/bitjson/bch-vm-limits) – i.e. by delaying activation of this proposal until a later upgrade, important protocol-level cost considerations might have been omitted from the final specification of `CHIP: Targeted Virtual Machine Limits` (See [Rationale: Inclusion of Numeric Encoding in Arithmetic Operation Costs](https://github.com/bitjson/bch-vm-limits/blob/master/rationale.md#inclusion-of-numeric-encoding-in-operation-costs)), increasing [combined protocol complexity risks](https://github.com/bitjson/bch-vm-limits/blob/master/risk-assessment.md#protocol-complexity-risks). Testing the proposals together improved the overall quality of both protocol upgrades.

Additionally, combined testing in preparation for a simultaneous activation also reduced overall testing and verification costs for specific implementations. It is critical to note that both proposals require thorough performance review and testing to identify any relevant performance issues in specific implementations.

Isolating only [`CHIP: Targeted Virtual Machine Limits`](https://github.com/bitjson/bch-vm-limits) for testing would primarily waste testing efforts, as nearly all worst-case performance scenarios (with the exception of hashing and signature checking) are entirely altered by the inclusion or exclusion of this proposal. In practice, this would squander limited resources – particularly of less-popular (for mining) implementations – by spreading out a single, comprehensive testing cycling into a multi-year affair: the results of the first cycle (with the aforementioned exceptions) would be mostly irrelevant to the results of the second cycle, and a significant portion of the combined benefits of both proposals would not become available to contract authors until after the activation of this (second) proposal. Beyond wasted work, this may also limit excitement about either upgrade, reducing the ability for alternative implementations to raise additional resources for a combined implementation and testing effort.

Finally, it must considered that the existence of a well-formed proposal to enable high-precision math on Bitcoin Cash constitutes a ["credible threat"](https://en.wikipedia.org/wiki/Game_theory) – regardless of whether this proposal is activated in 2025 or in a future year, competitors to Bitcoin Cash can already ["update their priors"](https://en.wikipedia.org/wiki/Bayesian_inference). The possible existence of a future Bitcoin Cash with high-precision math threatens a larger sphere of potential competitors – both legacy financial institutions and other cryptocurrencies – who are incentivized to disrupt or undermine ecosystem consensus surrounding the upgrade. Given additional delay, these efforts are more likely to be successful, increasing overall network consensus and split risks (e.g. the 1MB block size upgrade).

<details>

<summary>Notes</summary>

1. See for example [`2025 Nonstandard, BCHN, i9-10980HK (x86-64)`](https://github.com/bitjson/bch-vm-limits/blob/master/tests-and-benchmarks.md#2025-nonstandard-bchn-i9-10980hk-x86-64), where `c7ykrf`'s signature checking (`2-of-20 multisig with checkBits of zero (ECDSA, key 13 and key 13) (nonP2SH)`) has a `RelCostPerByte` of `11.625265` compared to `8.878946` in the case of `nevxwn` (`Within BCH_2025_05 nonP2SH/nonstandard, single-input limits, maximize control stack and stack usage checking (OP_NOTIF, OP_1ADD)`). This highlights an important class of scenarios in which the combined result of increased operation density (magnifying the worst-case impact of high per-operation overhead) and arithmetic operation implementation details might have produced an unexpected, new worst-case scenario. By testing the proposals together, implementers and reviewers can ensure an overall higher quality of review of both protocol specifications and specific implementations.

</details>

### Alternative: Select a Higher Constant Limit

Alternatively, this proposal could raise the limit to a higher constant value like `258`, the constant selected by Satoshi Nakamoto in [`reverted makefile.unix wx-config -- version 0.3.6`](https://gitlab.com/bitcoin-cash-node/bitcoin-cash-node/-/commit/757f0769d8360ea043f469f3a35f6ec204740446) (July 29, 2010). However, because the limit is no longer necessary for mitigating worst-case transaction validation cost, selection of any particular constant would be arbitrary.

### Full Removal Simplifies Protocol and Contract Review

By fully-removing the limit, overall protocol complexity is reduced, simplifying both future VM implementations and contract development. For VM implementations, eliminating out-of-range cases significantly reduces the combinatorial set of possible inputs and outputs for the numeric operations. For contract authors, eliminating the possibility of out-of-range errors prevents a class of potential vulnerabilities arising from a contract system's failure to validate that intermediate arithmetic results never exceed an (uncommonly-encountered) maximum number length limit.

Finally, full removal eliminates a potential source of future protocol ambiguity from the Bitcoin Cash VM. See [Removal of Overflow Behavior: Consideration of Potential Impacts](#consideration-of-potential-impacts).

### Removal of Overflow Behavior

As a practical consequence of removing the number length limit, arithmetic operation overflow is no longer relevant; the applicable, [10,000-byte stack element length limit](https://github.com/bitjson/bch-vm-limits#increased-stack-element-length-limit) (A.K.A. `MAX_SCRIPT_ELEMENT_SIZE`) only limits numeric operands and numeric results in precisely the same way that it limits the pushed-length of all other stack items: if any VM operation attempts to push an item longer than the limit to the stack, evaluation immediately fails. See [Rationale: Full Removal Simplifies Protocol and Contract Review](#full-removal-simplifies-protocol-and-contract-review).

In practice, this removal of overflow behavior impacts only the following operations: `OP_1ADD` (`0x8b`), `OP_1SUB` (`0x8c`), `OP_ADD` (`0x93`), `OP_SUB` (`0x94`), `OP_MUL` (`0x95`).

Previously, these operations were required, by consensus, to fail when provided operand(s) such that the encoded result required more than 8 bytes to represent (e.g. performing `OP_1ADD` on the maximum VM number). Following activation, these cases will instead return a correct result and continue evaluation<sup>1</sup>.

#### Consideration of Potential Impacts

Note that this change of behavior could hypothetically create faults in certain, specifically-crafted contracts in which 1) an attacker can introduce numeric operands which are immediately – without range sanitization – used in an arithmetic computation, and 2) the result is immediately – again without range sanitization – somehow used as a valid non-numeric value over the remainder of the evaluation, and 3) creates some benefit to the attacker or negative impact on the contract in question (which would typically require the re-interpretation of the invalid result as a number, rather than e.g. as simply a hash preimage, nonce, or other non-numeric value). Though unlikely, some contributors have also reviewed on-chain and publicly-known contract system designs to further reduce risks, [finding zero plausibly-impacted cases](https://github.com/bitjson/bch-bigint/pull/3).

At a higher level, it is critical to note that Bitcoin Cash already increased the number length limit from `4` to `8` via [`CHIP-2021-03: Bigger Script Integers`](https://gitlab.com/GeneralProtocols/research/chips/-/blob/master/CHIP-2021-02-Bigger-Script-Integers.md) (activated May 15, 2022), with similar impact on similarly-hypothetical contracts, establishing a clear precedent that number length limits may be increased in future upgrades in the same way as various other limits.

Finally, to reduce consensus and protocol complexity risks, `CHIP-2021-03: Bigger Script Integers` also included [a specific notice about the issue](https://gitlab.com/GeneralProtocols/research/chips/-/blob/master/CHIP-2021-02-Bigger-Script-Integers.md#notice-of-possible-future-expansion), reproduced below.

<details>

<summary>Notice of Possible Future Expansion</summary>

> #### Notice of Possible Future Expansion
>
> While unusual, it is possible to design contracts which rely on the rejection of otherwise-valid Script Numbers which are larger than 8 bytes. Contract authors are advised that future upgrades may further expand the supported range of BCH VM Script Numbers beyond 8 bytes.
>
> **This proposal interprets such failure-reliant constructions as intentional** – they are designed to fail unless/until a possible future network upgrade in which larger Script Numbers are enabled (i.e. a contract branch which can only be successfully evaluated in the event of such an upgrade).
>
> As always, the security of a contract is the responsibility of the entity locking funds in that contract; funds can always be locked in insecure contracts (e.g. `OP_DROP OP_1`). This notice is provided to warn contract authors and explicitly codify a network policy: the possible existence of poorly-designed contracts will not preclude future upgrades from further expanding the range of Script Numbers.
>
> To ensure a contract will always fail when arithmetic results overflow or underflow 8-byte Script Numbers (in the rare case that such a behavior is desirable), that behavior must be either 1) explicitly validated or 2) introduced to the contract prior to the activation of any future upgrade which expands the range of Script Numbers.

</details>

`CHIP-2021-03: Bigger Script Integers` goes on to provide [further clarification in the Rationale](https://gitlab.com/GeneralProtocols/research/chips/-/blob/master/CHIP-2021-02-Bigger-Script-Integers.md#inclusion-of-future-expansion-notice), reproduced below.

<details>

<summary>Inclusion of Future Expansion Notice</summary>

> ### Inclusion of Future Expansion Notice
>
> The [Notice of Possible Future Expansion](#notice-of-possible-future-expansion) is included in this specification to avert future controversy by documenting the proposal's intent with respect to future (not-yet specified) upgrades: **the BCH VM is not guaranteed to forever limit Script Numbers to 8 bytes**.
>
> If this were not clarified, any future Script Number upgrade proposals could be more easily mischaracterize by publicizing deposits made to contracts that are [intentionally designed to rely on the 8-byte overflow behavior](https://bitcoincashresearch.org/t/improve-utility-of-script-calculations-larger-numbers-op-mul/39/12). With this notice, such misdirection might be more easily identified as disingenuous.

</details>

Furthermore, it must be noted that any contract author purporting to have 1) unintentionally developed a contract matching the precise, aforementioned fault path, and 2) been unaware of Bitcoin Cash's established precedent of raising number length limits (and explicit, consensus acceptance of a notice highlighting the precedent), must necessarily have used Bitcoin Cash's 2022-increased number length limits to accidentally develop the faulty contract in question (as a contract designed to be exploitable under the older `4` byte limit is not necessarily impacted by this behavior change).

In essence, such a contract would need to have been developed with post-2022 BCH contract development and wallet tooling, with significant knowledge of modern BCH protocol changes, without the use of a high-level compiler, and with unusual bytecode-level optimizations – all while maintaining a plausible explanation for security-critical reliance on a particular setting of the number length limit. Given the current size of the Bitcoin Cash contract development ecosystem, this proposal considers the likelihood of any such case to be sufficiently low.

Finally, note that fully eliminating this potential source of future protocol ambiguity from the Bitcoin Cash VM is an additional, meaningful benefit of fully removing the number length limit. See [Rationale: Removal of Number Length Limit](#removal-of-number-length-limit).

<details>

<summary>Notes</summary>

1. Note also that the [Arithmetic Operation Cost Limit](https://github.com/bitjson/bch-vm-limits#arithmetic-operation-cost) also applies, but by design, arithmetic operations in existing contracts are afforded a [higher per-byte limit](risk-assessment.md#increased-or-equivalent-contract-capabilities) than they could have possibly used prior to the upgrade.

</details>

## Omission of Implementation-Specific Technical Documentation

This proposal specifies only the necessary change to the consensus protocol: removal of the number length limit.

The software changes required to support this consensus change differ significantly from implementation to implementation. VM implementations which already internally use arbitrary-precision arithmetic for VM number manipulation may only need to disable code enforcing the previous limit, while other implementations may be required to integrate an arbitrary-precision arithmetic library or language primitive. In all cases, implementations should verify their functional behavior and performance characteristics against the [CHIP Limits: Tests & Benchmarks](https://github.com/bitjson/bch-vm-limits/blob/master/tests-and-benchmarks.md).

## Omission of VM Number Format or Operation Documentation

Beyond dropping the unnecessary limit on VM number length, this proposal does not modify any other properties of the VM. Notably, the precise format and behavior of VM numbers across all VM operations are part of network consensus and do not otherwise change as a result of this proposal.

Implementers should review all VM operation which consume or produce numeric values, particularly the subset of operations which directly manipulate numbers: `OP_NUM2BIN` (`0x80`), `OP_BIN2NUM` (`0x81`), `OP_1ADD` (`0x8b`), `OP_1SUB` (`0x8c`), `OP_NEGATE` (`0x8f`), `OP_ABS` (`0x90`), `OP_NOT` (`0x91`), `OP_0NOTEQUAL` (`0x92`), `OP_ADD` (`0x93`), `OP_SUB` (`0x94`), `OP_MUL` (`0x95`), `OP_DIV` (`0x96`), `OP_MOD` (`0x97`), `OP_BOOLAND` (`0x9a`), `OP_BOOLOR` (`0x9b`), `OP_NUMEQUAL` (`0x9c`), `OP_NUMEQUALVERIFY` (`0x9d`), `OP_NUMNOTEQUAL` (`0x9e`), `OP_LESSTHAN` (`0x9f`), `OP_GREATERTHAN` (`0xa0`), `OP_LESSTHANOREQUAL` (`0xa1`), `OP_GREATERTHANOREQUAL` (`0xa2`), `OP_MIN` (`0xa3`), `OP_MAX` (`0xa4`), and `OP_WITHIN` (`0xa5`).

Note that other VM operations also consume and produce numeric values, and may require specific review in some implementations. For the avoidance of doubt, all implementations should verify their functional correctness and performance properties using the [CHIP Limits: Tests & Benchmarks](https://github.com/bitjson/bch-vm-limits/blob/master/tests-and-benchmarks.md).
