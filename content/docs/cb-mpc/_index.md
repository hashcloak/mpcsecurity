+++
title = "Notes on Coinbase CB-MPC Cryptography audit"
draft = "true"
+++

# Notes on Coinbase CB-MPC cryptography audit by Cure53

Around March 27th, 2025, [Coinbase](https://www.coinbase.com) released its [Coinbase Open Source MPC Library](https://github.com/coinbase/cb-mpc) (CB-MPC). This library provides implementations of primitives and protocols that are commonly used in the Secure Multi-party Computation (MPC) community. At the time of the release, Coinbase also disclosed an [audit](https://github.com/coinbase/cb-mpc/blob/master/docs/cure53-audit.pdf) conducted on CB-MPC by [Cure53](https://cure53.de/). This blog post is a set of notes about the findings in the mentioned audit based on the video ["005 Guarding the Gates: Lessons from the Coinbase CB-MPC Cryptography Audit with Nadim Kobeissi"](https://youtu.be/2wR25jFgPSo).

## CB-MPC

The Coinbase Open Source MPC Library is a C++ implementation of MPC protocols and cryptographic primitives. These implementations aim to secure key assets in blockchain applications in a decentralized manner. This library focuses on the following principles: safety, network architecture adaptability, general-purpose usage, and very comprehensive documentation, including both practical and theoretical aspects. The CB-MPC library includes the following implementations:

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
- Zero-knowledge proofs.
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
- Threshold Schnorr signature protocols.
- Threshold encryption protocols.

The library can be used to implement mechanisms that allow blockchain users to execute functionalities without reconstructing the secret key in a single host. For example, for a customer to send an asset, it must sign the transaction with its private key. The problem is that if an attacker compromises the customer's computer, it can steal the private key and move assets without authorization. Using MPC techniques and distributed key management, the customer can split the key into fragments so that compromising a subset of those fragments does not reveal information about the whole secret key. Once the customer wants to sign a transaction, it can run an MPC protocol between the holders of the secret key fragments to obtain the signature. The signature generation protocol is executed so that the secret key is not held by any protocol participant at any time during its execution. These techniques reduce the attack surface, given that the attacker must compromise not only one computer but many of them.

## Cure53 audit

### Key findings

#### CBS-02-003 WP1: Missing check stipulated by `ECC-Refresh-MP` security proof

The `ECC-Refresh-MP` function refreshes secret key shares for multi-party elliptic curve cryptographic schemes (ECC). During the execution of an MPC protocol, the parties must agree on some public parameters, such as the session identifiers (SID), the public key matching the shared secret key, and the shares of the public key. During the audit, the Cure53 team found that the function `ECC-Refresh-MP` did not check that these public values match for all the participants in the protocol. The lack of these checks may affect the security and the correctness of the protocol implementation. Also, checking absence may allow attackers to execute replay attacks or state injection attacks.

During audits conducted at HashCloak, we have found this issue multiple times. This raises an opportunity to remind software engineers and cryptographers implementing MPC protocols to be aware of checking the consistency of public parameters. We think that sometimes this may happen given that the mentioned consensus is not the key part on the protocol and it may remain inadvertent during the implementation process as engineer tend to focus on more complex or protocol-specific features. Sometimes, these are routine checks in MPC protocols that may be assumed but not considered in the implementation process. However, the lack of this consensus may produce equally damaging consequences to security as the more complex and protocol-specific features. Also, we encourage researchers to write these checks explicitly in the theoretical protocols so that engineers consider this. The Cure53 team pointed out this issue in their audit process because the Coinbase team highlighted this check in their theoretical specification clearly and explicitly, which is a good practice that engineers and auditors value a lot.

#### CBS-02-00 WP1: `ECC-Refresh-MP` vulnerable to small subgroup attacks

In ECC implementations, we often deal with sending, receiving, and manipulating EC points. These points must lie in the correct group so that the computation is correct and secure. If the implementation does not consider this fact, an adversary can take advantage by sending elements belonging to the wrong group so that it can execute so-called small subgroup attacks.

Considering the context of ECC, in a small subgroup attack, an adversary can send an invalid public key belonging to a small group to the honest parties. Then, the honest parties compute the shared secret for this key, but given that it belongs to a small group, the adversary can search for the shared key or at least learn some information about the shared key. In the reference video, Nadim Kobeissi recommends the paper ["Prime, Order Please! Revisiting Small Subgroup and Invalid Curve Attacks on Protocols using Diffie-Hellman"](https://eprint.iacr.org/2019/526) by Cas Cremers and Dennis Jackson. In the mentioned paper, the authors develop an extension of the symbolic model of Diffie-Hellman groups. This model captures additional behaviors that allow the detection of attacks using small subgroups or invalid point usage. In this paper, the authors also propose practical applications finding attacks on Scuttlebutt gossip protocol and Tendermint's secure handshake.

In the audit, the Cure53 team found that the source code does not validate whether the set of EC points $\{R_{j, \ell}\}_{\ell, j}$ are valid elements belonging to the prime-order subgroup as it is specified in the reference specification provided by Coinbase's team.

### Miscellaneous findings

There are other miscellaneous findings pointed out by Cure53's team involving the following topics:

- The implementation of the Ed25519 signing deviates from the standard specification that was worth documenting.
- Variable time branching and recalculation in modular inversion are important for achieving constant time execution.
- Missing parameter checks in internal functions.
- A proposal to execute batch exponentiation faster.
- Typographical errors in the documentation.

One issue that has significant relevance is **"CBS-02-006 WP1: Absent checks in Paillier Key generation"**. In the Paillier cryptosystem, we set a value $N = p \cdot q$ where $p, q$ are large primes. It must hold that:

- $p \neq q$, and
- $\gcd(N, (p-1)(q-1)) = 1$.

Although those properties should hold a high probability for randomly chosen prime numbers, these checks were missing in the implementation. However, these checks are considered mandatory for Paillier's security proofs.

## Lessons learned and conclusions

The Cure53 team considers that "CB-MPC library is the *gold standard* for how cryptographic libraries - or various cryptographic implementations - can be documented." At HashCloak, we have found that this is a common issue in the interaction between auditors and clients. As auditors, we do not commonly know all the quirks, modifications, assumptions, and misconceptions that the client may have. This hidden information may damage the audit process in two ways: (1) the auditor may have personal assumptions that do not align with the client's vision of the implementation, or (2) the audit process can become highly inefficient given that the auditor must constantly confirm and extract this hidden information to achieve a reasonable coverage and understanding of the implementation. Strong and rigorous documentation can increase the coverage of the code and achieve "the maximum value out of the audit" (as mentioned by Nadim Kobeissi). As Kobeissi also mentions, good documentation does not help only in an audit process but also in case you need to transfer the implementation rationale to other people in an efficient, structured, and organized way (for example, when a new employee arrives at the client's company and need to start working on the implementation).
