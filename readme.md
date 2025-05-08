# CHIP-2024-07 BigInt: High-Precision Arithmetic for Bitcoin Cash

        Title: High-Precision Arithmetic for Bitcoin Cash
        Type: Standards
        Layer: Consensus
        Maintainer: Jason Dreyzehner
        Status: Final
        Initial Publication Date: 2024-07-24
        Latest Revision Date: 2024-10-01
        Version: 1.1.1

## Summary

This proposal augments [`CHIP: Targeted Virtual Machine Limits`](https://github.com/bitjson/bch-vm-limits) by removing one additional Bitcoin Cash Virtual Machine limit: the number length limit (A.K.A. `nMaxNumSize`).

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

## Deployment

Deployment of this specification is proposed for the May 2025 upgrade.

- Activation is proposed for `1731672000` MTP, (`2024-11-15T12:00:00.000Z`) on `chipnet`.
- Activation is proposed for `1747310400` MTP, (`2025-05-15T12:00:00.000Z`) on the BCH network (`mainnet`), `testnet3`, `testnet4`, and `scalenet`.

This proposal requires [`CHIP: Targeted Virtual Machine Limits`](https://github.com/bitjson/bch-vm-limits).

## Technical Specification

The limit on the maximum length of Bitcoin Cash VM numbers (A.K.A. `nMaxNumSize`) is removed.

Note that the [10,000-byte stack element length limit](https://github.com/bitjson/bch-vm-limits#increased-stack-element-length-limit) (A.K.A. `MAX_SCRIPT_ELEMENT_SIZE`) continues to limit both numeric operands and numeric results, and [Arithmetic Operation Cost](https://github.com/bitjson/bch-vm-limits#arithmetic-operation-cost) prevents excessive usage of arithmetic operations.

## Tests & Benchmarks

[`CHIP: Targeted Virtual Machine Limits`](https://github.com/bitjson/bch-vm-limits) includes a suite of functional tests and benchmarks to verify the behavior and performance of all operations within virtual machine implementations, including high-precision arithmetic operations. See [CHIP Limits: Tests & Benchmarks](https://github.com/bitjson/bch-vm-limits/blob/master/tests-and-benchmarks.md) for details.

## Implementations

Please see the following reference implementations for additional examples and test vectors:

- **C++**:
  - [Bitcoin Cash Node (BCHN)](https://bitcoincashnode.org/) – A professional, miner-friendly node that solves practical problems for Bitcoin Cash. [Merge Request !1876](https://gitlab.com/bitcoin-cash-node/bitcoin-cash-node/-/merge_requests/1876).
- **Go**:
  - [BCHD](https://bchd.cash/) – An alternative full node bitcoin cash implementation written in Go (golang). [OPReturnCode/bchd PR #1](https://github.com/OPReturnCode/bchd/pull/1)
- **Java**:
  - [Bitcoin Verde](https://bitcoinverde.org/) – Bitcoin Verde is a Java full-node implementation of the Bitcoin Cash protocol. Bitcoin Verde provides a block explorer, development library, and network implementation diversification. [Issue #24](https://github.com/SoftwareVerde/bitcoin-verde/issues/24)
- **JavaScript/TypeScript**:
  - [Libauth](https://github.com/bitauth/libauth) – An ultra-lightweight, zero-dependency JavaScript library for Bitcoin Cash. [Pull Request #139](https://github.com/bitauth/libauth/pull/139).
  - [Bitauth IDE](https://github.com/bitauth/bitauth-ide) – An online IDE for bitcoin (cash) contracts. [Pull Request #101](https://github.com/bitauth/bitauth-ide/pull/101).

## Risk Assessment

- [Appendix: Risk Assessment &rarr;](risk-assessment.md)
  - [Risks \& Security Considerations](risk-assessment.md#risks--security-considerations)
    - [User Impact Risks](risk-assessment.md#user-impact-risks)
      - [Reduced or Equivalent Node Validation Costs](risk-assessment.md#reduced-or-equivalent-node-validation-costs)
      - [Increased or Equivalent Contract Capabilities](risk-assessment.md#increased-or-equivalent-contract-capabilities)
    - [Consensus Risks](risk-assessment.md#consensus-risks)
      - [Full-Transaction Test Vectors](risk-assessment.md#full-transaction-test-vectors)
      - [New Performance Testing Methodology](risk-assessment.md#new-performance-testing-methodology)
      - [`Chipnet` Preview Activation](risk-assessment.md#chipnet-preview-activation)
    - [Denial-of-Service (DoS) Risks](risk-assessment.md#denial-of-service-dos-risks)
      - [Expanded Node Performance Safety Margin](risk-assessment.md#expanded-node-performance-safety-margin)
    - [Protocol Complexity Risks](risk-assessment.md#protocol-complexity-risks)
      - [Simplification of Protocol and Contract Review](risk-assessment.md#simplification-of-protocol-and-contract-review)
      - [Elimination of Potential Protocol Ambiguity](risk-assessment.md#elimination-of-potential-protocol-ambiguity)
      - [Obviation of Future Precision-Emulation Enhancements](risk-assessment.md#obviation-of-future-precision-emulation-enhancements)
  - [Upgrade Costs](risk-assessment.md#upgrade-costs)
    - [Node Upgrade Costs](risk-assessment.md#node-upgrade-costs)
    - [Ecosystem Upgrade Costs](risk-assessment.md#ecosystem-upgrade-costs)
  - [Maintenance Costs](risk-assessment.md#maintenance-costs)
    - [Node Maintenance Costs](risk-assessment.md#node-maintenance-costs)
    - [Ecosystem Maintenance Costs](risk-assessment.md#ecosystem-maintenance-costs)

## Rationale

- [Appendix: Rationale &rarr;](rationale.md)
  - [Extraction from `CHIP: Targeted Virtual Machine Limits`](rationale.md#extraction-from-chip-targeted-virtual-machine-limits)
  - [Removal of Number Length Limit](rationale.md#removal-of-number-length-limit)
    - [Alternative: Preserve 8-Byte Limit](rationale.md#alternative-preserve-8-byte-limit)
      - [Scenario: Never Increase](rationale.md#scenario-never-increase)
      - [Scenario: Delay Activation](rationale.md#scenario-delay-activation)
    - [Alternative: Select a Higher Constant Limit](rationale.md#alternative-select-a-higher-constant-limit)
    - [Full Removal Simplifies Protocol and Contract Review](rationale.md#full-removal-simplifies-protocol-and-contract-review)
    - [Removal of Overflow Behavior](rationale.md#removal-of-overflow-behavior)
      - [Consideration of Potential Impacts](rationale.md#consideration-of-potential-impacts)
  - [Omission of Implementation-Specific Technical Documentation](rationale.md#omission-of-implementation-specific-technical-documentation)
  - [Omission of VM Number Format or Operation Documentation](rationale.md#omission-of-vm-number-format-or-operation-documentation)

## Evaluation of Alternatives

Notable alternatives evaluated in the rationale are summarized here for ease of review:

- **Indefinitely retained 8-byte limit** – Wasteful in every applicable measure when compared to this proposal, requiring: greater overall network bandwidth usage, larger individual transaction sizes, greater overall block storage requirements, and greater net computation costs of equivalent arithmetic operations across all validating nodes on the network. See [Scenario: Never Increase](rationale.md#scenario-never-increase).

- **Temporarily retained 8-byte limit** – Wasteful of node implementation resources, requiring two entirely independent review cycles (separated by one or more years), with significantly greater implementation risks, significant discarded first-cycle effort in the second cycle, a potential decrease in new resources available to all node implementations, and non-trivial disruption risks. See [Scenario: Delay Activation](rationale.md#scenario-delay-activation).

- **Selection of higher constant limit** – Selection of any particular constant would be arbitrary. See [Alternative: Select a Higher Constant Limit](rationale.md#alternative-select-a-higher-constant-limit)

- **Full removal of limit** – This proposals chosen solution – reduces overall protocol complexity, simplifies contract review, and reduces future protocol complexity risks. See [Full Removal Simplifies Protocol and Contract Review](rationale.md#full-removal-simplifies-protocol-and-contract-review).

## Stakeholder Responses & Statements

[Stakeholder Responses & Statements &rarr;](stakeholders.md)

## Feedback & Reviews

- [BigInt CHIP Issues](https://github.com/bitjson/bch-bigint/issues)
- [`CHIP 2024-07 BigInt` - Bitcoin Cash Research](https://bitcoincashresearch.org/t/chip-2024-07-bigint-high-precision-arithmetic-for-bitcoin-cash/1356)

## Acknowledgements

Thank you to the following contributors for reviewing and contributing improvements to this proposal, providing feedback, and promoting consensus among stakeholders:
[Calin Culianu](https://github.com/cculianu), [bitcoincashautist](https://github.com/A60AB5450353F40E), [Andrew#128](https://gitlab.com/andrew-128), [Fernando Pelliccioni](https://gitlab.com/fpelliccioni), [Mathieu Geukens](https://github.com/mr-zwets), [Joshua Green](https://github.com/joshmg), [OPReturnCode](https://github.com/OPReturnCode), [Jeremy](https://bitcoincashpodcast.com/), [Kallisti.cash](https://kallisti.io), [Corbin Fraser](https://corbinfraser.com/), [imaginary_username](https://gitlab.com/im_uname), [John Nieri](https://gitlab.com/emergent-reasons), [Jonathan Silverblood](https://gitlab.com/monsterbitar), [Josh Ellithorpe](https://github.com/zquestz), [John Moriarty](https://x.com/BitcoinOutLoud), [minisatoshi](https://minisatoshi.cash/), [Andrew Groot](https://github.com/thesquaregroot), [Tom Zander](https://github.com/zander), [Rosco Kalis](https://github.com/rkalis), [Richard Brady](https://github.com/rnbrady).

## Changelog

This section summarizes the evolution of this document.

- **v1.1.1 – 2024-10-01** ([`d1406b69`](https://github.com/bitjson/bch-bigint/commit/d1406b6984c5528983a029c79111646e95286b8c))
  - Add [Risk Assessment](risk-assessment.md)
  - Expand [Rationale](rationale.md)
  - Add [Evaluation of Alternatives](#evaluation-of-alternatives)
- **v1.1.0 – 2024-08-28** ([`616a7a94`](https://github.com/bitjson/bch-bigint/commit/616a7a948dca97aef1126715aa6fe8b3edbe35f8))
  - Remove the VM number length limit ([#1](https://github.com/bitjson/bch-bigint/issues/1))
- **v1.0.0 – 2024-07-24** ([`b114c957`](https://github.com/bitjson/bch-bigint/commit/b114c95729e670f4b0780d4fd14590c35d281d77))
  - Initial publication

## Copyright

This document is placed in the public domain.
