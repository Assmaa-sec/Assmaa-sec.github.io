---
title: "Threshold Signature Blockchain Wallet"
description: "3-of-5 threshold ECDSA wallet where the private key is never assembled in one place. Uses Distributed Key Generation, Shamir Secret Sharing, and Verifiable Secret Sharing across five admin nodes, with mTLS transport and a SHA-256 hash-chained audit log."
category: "Applied Cryptography"
status: "Complete"
featured: true
date: 2024-09-01
tech:
  - Python
  - Flask
  - Threshold ECDSA
  - Shamir Secret Sharing
  - DKG / VSS
  - mTLS
  - SQLite
github: "https://github.com/Assmaa-sec/Blockchain-wallet"
---

## Overview

Standard blockchain wallets rely on a single private key. If that key is lost or stolen, the funds are gone with no recovery path. This wallet requires at least 3 of 5 admins to cooperate on every transaction. The full private key is never assembled in one place at any point in the signing flow.

## Architecture

```
Client  ->  API  ->  Wallet Core  ->  Signing Layer  ->  Admin Nodes (x5)
                                                               |
                                                        Crypto Primitives
                                                     (DKG, Shamir SS, VSS)
```

## Cryptographic Design

**Distributed Key Generation (DKG):** Each admin generates a random polynomial and sends shares to the others. The final key share per admin is the sum of all received shares. Nobody ever sees the full private key.

**Shamir Secret Sharing:** The secret is split using a degree-2 polynomial over the secp256k1 field. Any 3 shares reconstruct via Lagrange interpolation. Fewer than 3 shares reveal nothing.

**Verifiable Secret Sharing (VSS):** Admins publish polynomial coefficient commitments so everyone can verify their received share is valid. A malicious admin handing out bad shares is caught immediately.

**Threshold ECDSA (3-round protocol):**
1. Each signer commits to a random nonce
2. Nonces are revealed and verified; joint nonce is computed
3. Each signer produces a partial signature; coordinator combines them

## Security Properties

| Threat | Mitigation |
|---|---|
| Admin node compromise | One share cannot sign anything, needs 2 more |
| Less than 3 admins collude | Mathematically cannot produce a valid signature |
| Man-in-the-middle | mTLS between all components |
| Replay attack | Fresh random nonces + UUID in transaction hash |
| Audit log tampering | SHA-256 hash chain, any change breaks the chain |
| Key brute force | 2^256 search space; shares stored in SGX sealed storage |
