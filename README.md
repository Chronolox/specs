# Chronolox Security Architecture & Source-Code Escrow Specification

**Document Version:** 1.0  
**Target Audience:** Security Auditors, Technical Evaluators, Beta Testers, and Privacy Engineers  
**Classification:** Public Specification  

---

## 1. Executive Security Thesis: The Golden Rule

The foundational axiom of the Chronolox Protocol is:

> **The customer must always be able to retrieve their secrets after the time-lock expires, even if Chronolox as a business completely ceases to exist.**

To fulfill this guarantee, Chronolox abandons traditional centralized SaaS architecture (user databases, proprietary auth tokens, managed secret management services) in favor of hardware-enforced cryptographic boundaries and public state machines[cite: 2].

---

## 2. Core Cryptographic Model

### 2.1 Hardware-Enforced Custodianship (Oasis Sapphire TEE)
- **No Operator Visibility:** Encryption keys are generated, stored, and managed inside hardware-isolated Trusted Execution Environments (TEEs) on the Oasis Sapphire network[cite: 2].
- **Zero Admin Overrides:** No private developer key, governance multisig, or subpoena can force decryption before `block.timestamp >= unlockTime`[cite: 2, 11]. Once committed, the contract logic is immutable[cite: 2, 13].
- **Double-Lock Archival (≤ 1 Year):** Short-term and frozen secrets leverage dual-custodianship combining Sapphire TEE enclaves with decentralized threshold network beacons (drand `tlock`)[cite: 2, 26]. Even during a total network consensus halt, the payload can be decrypted mathematically offline[cite: 2, 34].

### 2.2 Client-Side Isolation & Calldata Architecture
- **Local Key Derivation:** All payload encryption occurs client-side using authenticated AEAD ciphers (Deoxys-II-256-128 / AES-256-GCM) and Argon2id key derivation before any network packet is dispatched[cite: 2, 8].
- **On-Chain Calldata:** Encrypted payloads are permanently committed into public EVM calldata (Polygon), guaranteeing 100% data availability without depending on private storage servers[cite: 2, 18].
- **Integrity Anchoring:** Document and container fingerprints use quantum-resistant SHA3-512 hashes anchored directly to the public ledger via `ChronoloxNotary.sol`[cite: 2, 6].

---

## 3. Commercial Closed-Source Beta Policy

During the initial deployment and beta validation phases, Chronolox maintains a **source-available, commercial closed-beta distribution model** for its primary application binaries (`secret-maker`, `chronolox-heir`, `chronolox-cli`) and core routing relayers[cite: 2].

### 3.1 Why Is the Code Not Fully Open-Source Yet?
1. **Economic Sustainability:** We are an independent, engineering-driven team bootstrapping protocol development without predatory VC lockups. Protecting the core implementation during the pre-revenue beta window ensures financial viability and protects early stakeholder equity[cite: 2, 9].
2. **Security by Design, Not by Obscurity:** Chronolox relies strictly on publicly auditable cryptographic primitives (AES-GCM, Argon2id, SHA3-512, Oasis Sapphire TEE, drand)[cite: 2, 8]. The privacy guarantees depend exclusively on mathematics and hardware-isolated enclaves—not on hidden client code[cite: 2].
3. **Audited Milestones:** Core smart contracts and binary components will transition to permissive open-source licenses as operational milestones and economic sustainability are satisfied[cite: 2].

---

## 4. The Cryptographic Dead-Man's Switch for Source Code

To eliminate counterparty risk while source code remains proprietary, Chronolox implements an automated, on-chain **Cryptographic Source Escrow Failsafe** directly inside distributed binaries.
