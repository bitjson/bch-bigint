# Risk Assessment

The following security considerations, potential risks, and costs have been reviewed to verify the safety and advisability of this proposal.

Note that this risk assessment extends the parent proposal's risk assessment: [`Targeted Virtual Machine Limits: Risk Assessment`](https://github.com/bitjson/bch-vm-limits).

<details>

<summary><strong>Table of Contents</strong></summary>

- [Risks \& Security Considerations](#risks--security-considerations)
  - [User Impact Risks](#user-impact-risks)
    - [Reduced or Equivalent Node Validation Costs](#reduced-or-equivalent-node-validation-costs)
    - [Increased or Equivalent Contract Capabilities](#increased-or-equivalent-contract-capabilities)
  - [Consensus Risks](#consensus-risks)
    - [Full-Transaction Test Vectors](#full-transaction-test-vectors)
    - [New Performance Testing Methodology](#new-performance-testing-methodology)
    - [`Chipnet` Preview Activation](#chipnet-preview-activation)
  - [Denial-of-Service (DoS) Risks](#denial-of-service-dos-risks)
    - [Expanded Node Performance Safety Margin](#expanded-node-performance-safety-margin)
  - [Protocol Complexity Risks](#protocol-complexity-risks)
    - [Simplification of Protocol and Contract Review](#simplification-of-protocol-and-contract-review)
    - [Elimination of Potential Protocol Ambiguity](#elimination-of-potential-protocol-ambiguity)
    - [Obviation of Future Precision-Emulation Enhancements](#obviation-of-future-precision-emulation-enhancements)
- [Upgrade Costs](#upgrade-costs)
  - [Node Upgrade Costs](#node-upgrade-costs)
  - [Ecosystem Upgrade Costs](#ecosystem-upgrade-costs)
- [Maintenance Costs](#maintenance-costs)
  - [Node Maintenance Costs](#node-maintenance-costs)
  - [Ecosystem Maintenance Costs](#ecosystem-maintenance-costs)

</details>

## Risks & Security Considerations

This section reviews the foreseeable security implications of the proposed changes to the Bitcoin Cash network. Key technical considerations include user impact risks, consensus risks, denial-of-service (DoS) risks, and risks to protocol complexity or maintenance burden of newly introduced behavior.

### User Impact Risks

All upgrade proposals must carefully analyze proposed changes for potential impacts to existing Bitcoin Cash users and use cases. Virtual Machine (VM) upgrades can impact node operators and blockchain indexers (and therefore payment processors, exchanges, and other businesses), software development libraries, wallets, decentralized applications, and a wide range of pre-signed transactions, contract systems, and transaction-settled protocols.

This proposal preserves backwards-compatibility, minimizing user impact risks:

#### Reduced or Equivalent Node Validation Costs

See [Limits CHIP Risk Assessment: Reduced or Equivalent Node Validation Costs](https://github.com/bitjson/bch-vm-limits/blob/master/risk-assessment.md#reduced-or-equivalent-node-validation-costs).

#### Increased or Equivalent Contract Capabilities

See [Limits CHIP Risk Assessment: Increased or Equivalent Contract Capabilities](https://github.com/bitjson/bch-vm-limits/blob/master/risk-assessment.md#reduced-or-equivalent-node-validation-costs).

Additionally, this proposal highlights and evaluates the risk of impact to one additional, hypothetical set of advanced contract users. See [Removal of Overflow Behavior: Consideration of Potential Impacts](rationale.md#consideration-of-potential-impacts).

### Consensus Risks

All network consensus upgrade proposals must account for consensus risks arising from incorrect or inconsistent implementation of consensus-critical changes. For Virtual Machine (VM) upgrades, consensus risks primarily apply to node implementations and other software which performs VM evaluation as part of transaction validation.

This proposal inherits the parent proposal's consensus risk mitigations: 1) an extensive set of full-transaction test vectors, 2) a new cross-implementation performance testing methodology, and 3) a 6-month early activation on `chipnet`.

#### Full-Transaction Test Vectors

See [Limits CHIP Risk Assessment: Full-Transaction Test Vectors](https://github.com/bitjson/bch-vm-limits/blob/master/risk-assessment.md#full-transaction-test-vectors).

#### New Performance Testing Methodology

See [Limits CHIP Risk Assessment: New Performance Testing Methodology](https://github.com/bitjson/bch-vm-limits/blob/master/risk-assessment.md#new-performance-testing-methodology).

#### `Chipnet` Preview Activation

See [Limits CHIP Risk Assessment: `Chipnet` Preview Activation](https://github.com/bitjson/bch-vm-limits/blob/master/risk-assessment.md#chipnet-preview-activation).

### Denial-of-Service (DoS) Risks

All network consensus upgrade proposals which alter system limits carry risks related to Denial-of-Service (DoS) attacks. In particular, modifications to VM limits could 1) exacerbate the worst-case performance of transaction or block validation for both expensive-but-valid cases and excessively-invalid cases, and/or 2) decrease the cost or increase the practicality of attempting a particular VM-related DOS attack.

#### Expanded Node Performance Safety Margin

See [Limits CHIP Risk Assessment: Expanded Node Performance Safety Margin](https://github.com/bitjson/bch-vm-limits/blob/master/risk-assessment.md#expanded-node-performance-safety-margin).

### Protocol Complexity Risks

All upgrade proposals must carefully analyze proposed changes for both immediate and potential future impacts on overall protocol complexity.

In addition to the [risk mitigations of the parent proposal](https://github.com/bitjson/bch-vm-limits/blob/master/risk-assessment.md#protocol-complexity-risks), this proposal mitigates additional protocol complexity risks:

#### Simplification of Protocol and Contract Review

This proposal eliminates a variety of edge cases which add to the complexity of both VM implementations and contract development. By simplifying away these cases, protocol complexity risks are reduced and a class of potential contract vulnerabilities is eliminated. See [Rationale: Full Removal Simplifies Protocol and Contract Review](rationale.md#full-removal-simplifies-protocol-and-contract-review).

#### Elimination of Potential Protocol Ambiguity

This proposal fully eliminates a potential source of future protocol ambiguity from the Bitcoin Cash VM. See [Removal of Overflow Behavior: Consideration of Potential Impacts](#consideration-of-potential-impacts).

#### Obviation of Future Precision-Emulation Enhancements

By definitively removing the need for emulated-precision math, this proposal reduces future protocol complexity risks arising from the demands of precision-emulation use cases, as all existing use cases needing greater flexibility can be migrated to native-precision math. For example, the owners of [`CHIP-2023-07 Composite Arithmetic Opcodes`](https://github.com/mr-zwets/Composite-Arithmetic-Opcodes/blob/main/README.md) have paused work and plan to withdraw `CHIP-2023-07 Composite Arithmetic Opcodes` if this proposal is locked-in.

## Upgrade Costs

This section reviews the costs of implementing the proposed changes.

### Node Upgrade Costs

See [Limits CHIP Risk Assessment: Node Upgrade Costs](https://github.com/bitjson/bch-vm-limits/blob/master/risk-assessment.md#node-upgrade-costs).

Note that significant portions of expected node upgrade costs were already speculatively paid, in hopes of later activation, during the creation and review of this proposal. Additionally, rather than draining node resources, this proposal has attracted additional resources to fund further development and maintenance of multiple node implementations. See [Node Maintenance Costs](#node-maintenance-costs).

### Ecosystem Upgrade Costs

See [Limits CHIP Risk Assessment: Ecosystem Upgrade Costs](https://github.com/bitjson/bch-vm-limits/blob/master/risk-assessment.md#ecosystem-upgrade-costs).

## Maintenance Costs

All network upgrade proposals must evaluate their foreseeable impact on maintenance costs. Virtual Machine (VM) upgrades can increase the risks of [consensus divergence](#consensus-risks), [performance issues](#denial-of-service-dos-risks), and increased [implementation complexity](#protocol-complexity-risks). This section reviews foreseeable ongoing costs following activation of the proposed changes.

### Node Maintenance Costs

See [Limits CHIP Risk Assessment: Node Maintenance Costs](https://github.com/bitjson/bch-vm-limits/blob/master/risk-assessment.md#node-maintenance-costs).

Beyond the node maintenance costs of `CHIP: Targeted Virtual Machine Limits`, this proposal requires all future node implementations to implement big-integer arithmetic, either by:

1.  Importing a dependency, or
2.  Relying on utilities built-in to the programming language or execution environment in use.

Most modern programming languages and/or execution environments now include official support for arbitrary-precision arithmetic at far greater lengths than are [required by this proposal](./readme.md#technical-specification) – with the notable exceptions of C++ and Rust, which both have a variety of widely-used libraries, e.g.:

- **C++** – [GMP](https://gmplib.org/) (the library used by [BCHN's implementation of this proposal](readme.md#implementations))
- **Go** – [`math/big`](https://pkg.go.dev/math/big) standard library package
- **Java** – [`java.math.BigInteger`](https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/math/BigInteger.html)
- **JavaScript** – [`BigInt` primitive](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/BigInt)
- **Python** – [`int` Numeric Type](https://docs.python.org/3/library/stdtypes.html#numeric-types-int-float-complex)
- **Ruby** – [`Integer`](https://ruby-doc.org/core-2.5.1/Integer.html)
- **Rust** – [`rug` crate](https://crates.io/crates/rug) – (interface to [GMP](https://gmplib.org/))

Requiring high-precision arithmetic in the Bitcoin Cash VM **adds a non-zero review and maintenance cost to all future implementations** – particularly for node implementations written in languages without existing, official support (requiring a new direct dependency).

However, high-precision arithmetic is widely used across a variety of financial and safety-critical industries. With such wide usage, the relative implementation and maintenance cost of high-precision arithmetic is often even lower than the equivalent costs related to Bitcoin Cash's existing, relatively-niche cryptographic requirements – especially `RIPEMD160` and `Secp256k1` – which typically require implementations to rely on far less commonly used dependencies for cryptography than are available for high-precision arithmetic.

Finally, various implementations have produced additional BigInt-specific test sets and testing methodologies which may be useful for verification and maintenance:

- **vmb_tests** – This proposal's [functional tests and performance benchmarks](https://github.com/bitjson/bch-vm-limits/blob/master/tests-and-benchmarks.md) include combinatorial sets of tests exercising all arithmetic operations at varying operand lengths and byte-fill contents.
- **Property-based testing** – [Property Tests for Big Integer Arithmetic Script Operations](https://gitlab.com/bitcoin-cash-node/bitcoin-cash-node/-/blob/e6b2c2119a22e1a0fc75f8d9e9cd50e492c19921/doc/bigint-script-property-tests.md) (BCHN)
- **Python-Generated Test Vectors** – [`generate_bigint_test_vectors.py` from BCHN](https://gitlab.com/bitcoin-cash-node/bitcoin-cash-node/-/blob/e6b2c2119a22e1a0fc75f8d9e9cd50e492c19921/src/test/data/generate_bigint_test_vectors.py)
- **OpenSSL Test Vectors** – Used in [verifying the BCHN implementation](https://gitlab.com/cculianu/bitcoin-cash-node/-/commit/814a88c6f6dd14b461699fd540a357c4f4794ea8)

### Ecosystem Maintenance Costs

See [Limits CHIP Risk Assessment: Ecosystem Maintenance Costs](https://github.com/bitjson/bch-vm-limits/blob/master/risk-assessment.md#ecosystem-maintenance-costs).
