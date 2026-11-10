# Chronolox Security Architecture & Source-Code Escrow Specification

**Document Version:** 1.0  
**Target Audience:** Security Auditors, Technical Evaluators, Beta Testers, and Privacy Engineers  
**Classification:** Public Specification  

---

## 1. Executive Security Thesis: The Golden Rule

The foundational axiom of the Chronolox Protocol is:

> **The customer must always be able to retrieve their secrets after the time-lock expires, even if Chronolox as a business completely ceases to exist.**

To fulfill this guarantee, Chronolox abandons traditional centralized SaaS architecture—such as centralized user databases, proprietary auth tokens, and managed secret vaults—in favor of hardware-enforced cryptographic boundaries and public state machines.

---

## 2. Core Cryptographic Model

### 2.1 Hardware-Enforced Custodianship (Oasis Sapphire TEE)
- **Zero Operator Visibility:** Encryption keys are generated, stored, and managed inside hardware-isolated Trusted Execution Environments (TEEs) on the Oasis Sapphire network.
- **Zero Admin Overrides:** No private developer key, governance multisig, or legal demand can force decryption before `block.timestamp >= unlockTime`. Once committed, the contract logic is immutable.
- **Double-Lock Archival (≤ 1 Year & Frozen):** Short-term (under 365 days) and frozen secrets leverage dual custodianship combining Sapphire TEE enclaves with decentralized threshold network beacons (drand `tlock`). Even during a total network consensus halt, the payload can be decrypted mathematically offline.

### 2.2 Client-Side Isolation & Calldata Architecture
- **Local Key Derivation:** All payload encryption occurs client-side using authenticated AEAD ciphers (Deoxys-II-256-128 / AES-256-GCM) and Argon2id key derivation before any network packet is dispatched.
- **On-Chain Calldata:** Encrypted payloads are permanently committed into public EVM calldata (Polygon), guaranteeing data availability without depending on private storage servers.
- **Integrity Anchoring (Proof of Existence):** Document and container fingerprints use quantum-resistant SHA3-512 hashes anchored directly to the public ledger via `ChronoloxNotary.sol`, proving exact data existence at a specific block time without exposing contents.

---

## 3. Commercial Closed-Source Beta Policy

During the initial deployment and beta validation phases, Chronolox maintains a **source-available, commercial closed-beta distribution model** for its primary application binaries (`secret-maker`, `chronolox-heir`, `chronolox-cli`) and core routing relayers.

### 3.1 Why Is the Code Not Fully Open-Source Yet?
1. **Economic Sustainability:** We are an independent, engineering-driven team bootstrapping protocol development without predatory VC lockups. Protecting the core implementation during the pre-revenue beta window ensures financial viability and protects early stakeholder equity.
2. **Security by Design, Not by Obscurity:** Chronolox relies strictly on publicly auditable cryptographic primitives (AES-GCM, Argon2id, SHA3-512, Oasis Sapphire TEE, drand). The privacy guarantees depend exclusively on mathematics and hardware-isolated enclaves—not on hidden client code.
3. **Audited Milestones:** Core smart contracts and binary components will transition to permissive open-source licenses as operational milestones and economic sustainability are satisfied.

---

## 4. The Cryptographic Dead-Man's Switch for Source Code

To eliminate counterparty risk while source code remains proprietary, Chronolox implements an automated, on-chain **Cryptographic Source Escrow Failsafe** directly inside distributed binaries.

```mermaid
flowchart TD
    subgraph Binary["Chronolox Distributed Binary (chronolox-cli / chronolox-heir)"]
        Engine["Standard Engine & Runtime<br/>(Compiled Go / WASM)"]
        Archive["Embedded Source Escrow<br/>(source_archive.7z · Encrypted)"]
    end

    subgraph Blockchain["Oasis Sapphire Network (Hardware TEE)"]
        Contract["SecretVault Contract<br/>Time-Locked Master Secret<br/>(Holds Escrow Password)"]
    end

    Archive -. Requires Password .-> Contract

    Contract -->|Company Active: Extends Unlock Date| ActiveState["Normal Operations<br/>- Secret postponed forward<br/>- Source remains sealed"]
    Contract -->|Company Inactive: Timer Expires| AbandonedState["Autonomous Trigger<br/>- Timer expires naturally<br/>- Password released on-chain"]

    AbandonedState --> Extract["Community Recovery Command:<br/>chronolox-cli extract-source --secret-id 0x..."]
    Extract --> Audit["Full Open Source Codebase Extracted<br/>- Independent compilation<br/>- Community-run relayers<br/>- Complete protocol survival"]

    style Binary fill:#f8f9fa,stroke:#495057,stroke-width:1px
    style Blockchain fill:#e8f4fd,stroke:#0c63e4,stroke-width:1px
    style ActiveState fill:#e6fcf5,stroke:#0ca678,stroke-width:1px
    style AbandonedState fill:#fff4e6,stroke:#f76707,stroke-width:1px
    style Audit fill:#edf2ff,stroke:#4263eb,stroke-width:1px
```

### 4.1 How It Works
1. **Encrypted Source Bundling:** Every official compiled binary (`chronolox-heir`, `chronolox-cli`) contains an embedded, encrypted archive (`source_archive.7z`) containing the complete Go source code, build scripts, and contract definitions.
2. **On-Chain Key Custody:** The decryption password for `source_archive.7z` is stored in an immutable, time-locked secret on the Oasis Sapphire blockchain.
3. **Active Postponement:** While the company is operational, funded, and maintaining the platform, the Chronolox team periodically postpones the unlock date forward via governance transactions.
4. **Autonomous Release on Disappearance:** If Chronolox ever dissolves, ceases business operations, or fails to maintain the protocol, the timer expires naturally. The decryption password unlocks automatically on-chain.
5. **Community Extraction:** Any user with the compiled binary can execute:
   ```bash
   chronolox-cli extract-source --secret-id <PUBLIC_ESCROW_SECRET_ID>
   ```

   The binary queries the blockchain, retrieves the unlocked password, unpacks the entire codebase, and allows the community to audit, self-host, and compile independent relayers and clients indefinitely.

---

## 5. Recipient Verification & Air-Gapped Operation

Heirs do not need network connectivity to inspect or prepare their recovery environments:

* **Portable Standalone Executables:** The Heir Unlocker requires zero external package installations, zero browser dependencies, and zero administrative daemon privileges.
* **Checksum Verification:** Every release published to `github.com/Chronolox/releases` is paired with cryptographic checksums (`SHA256SUMS`) signed by the official Chronolox Release Key.
* **Independent On-Chain Verification:** Even if `chronolox.com` is offline, heirs can verify capsule timestamps directly against the Polygon network using any public EVM RPC node.

---

## 6. Responsible Disclosure

If you identify a security vulnerability in our protocol contracts, client cryptography, or binary packages:

1. **Do NOT open a public GitHub issue or pull request.**
2. Follow the disclosure procedure in [`SECURITY.md`](./SECURITY.md).
3. Send an encrypted or direct report to **`security@chronolox.com`**.
4. Allow up to 48 hours for initial triage and assessment before coordinated disclosure.

---

## 7. Contributing

We welcome feedback, specification review, and security analysis from the community. Please read [`CONTRIBUTING.md`](./CONTRIBUTING.md) for details on submitting issues and proposals.

---

## 8. License

This specification is licensed under the [Creative Commons Attribution 4.0 International License (CC BY 4.0)](./LICENSE). You are free to share and adapt this material with appropriate credit.
