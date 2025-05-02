+++
title = "Notes on Coinbase CB-MPC Cryptography audit"
draft = "true"
+++

# Notes on Coinbase CB-MPC cryptography audit by Cure53

Around March 27th, 2025, [Coinbase](https://www.coinbase.com) released its [Coinbase Open Source MPC Library](https://github.com/coinbase/cb-mpc) (CB-MPC). This library provides implementations of primitive and protocols that are commonly used in the Secure Multi-party Computation (MPC) community. At the time of the release, Coinbase also disclosed an [audit](https://github.com/coinbase/cb-mpc/blob/master/docs/cure53-audit.pdf) conducted on CB-MPC by [Cure53](https://cure53.de/). This blogpost is a set of notes, explaination and extension of the findings in the mentioned audit based on the video ["005 Guarding the Gates: Lessons from the Coinbase CB-MPC Cryptography Audit with Nadim Kobeissi"](https://youtu.be/2wR25jFgPSo).

## CB-MPC

The Coinbase Open Source MPC Library is a C++ implementation of a set of MPC protocols and cryptographic primitives. The aim of these implementations is to secure key assets in blockchain applications in a decentralized manner. This library focuses on the following principles: safety, network architecture adaptability, general-purpose usage, and a very comprehensive documentation including both practical and theoretical aspects. The CBC-MPC library includes the following implementations:

- Basic cryptographic primitives:
    - Hashing and random oracles.
    - Key derivation.
    - Pseudorandom generators (PRG) and sampling.
    - Signcryption.
    - Commitment schemes.
    - Broadcast protocols.
    - Coin tossing protocols.
    - ElGamal commitments.
    - Paillier encryption. 
    - Secret-sharing schemes.
- Zero knowledge proofs.
    - Elliptic curve $\Sigma$-protocols.
    - Paillier proofs.
    - Encryption proofs.
- Elliptic curve distributed key generation (EC-DKG):
    - With additive sharing.
    - With threshold sharing.
- Threshold ECDSA signature protocols:
    - For two-party computation.
    - For multi-party computation.
- Oblivious transfer (OT) protocols.
- Publicly verifiable encryption.
- Theshold Schnorr signature protocols.
- Threshold encryption protocols.



