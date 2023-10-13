---
title: (t, n) case from Gennaro & Goldfeder (2019)
math: true
---

# (t, n) case from Gennaro & Goldfeder (2019)

**Important note:** this paper has some corrections addressing errors and wrong conjectures from the first version. Although we will point such errors, it is important to make sure that the implementation meet the updated version of the protocol.

**Note on notation:** different to the previous works presented in this document, this work uses multiplicative notation for groups.

In this work, Genaro & Goldfeder propose a new method for a $(t, n)$ signature scheme. However, we have to keep in mind that when they say $(t, n)$, they mean that any subset of $t + 1$ parties can jointly sign but any smaller set cannot, which is slightly different to the concept presented in other works reviewed in this document. Also, this work focuses on the DSA signature scheme but the results can be easily applyd to ECDSA as the latter is an elliptic curve variant of the former. Genaro & Goldfeder propose a solution based on Paillier encryption scheme to realize a functionality that transform multiplicative shares into aditive ones, along with non-malleable equivocable commitments to ensure a correct computation against malicious adversaries. Also he uses a Feldman's verifiable secret-sharing scheme (VSS) to ensure the threshold security of the signing process.

### Preliminaries

In this section, we present three definitions to understand the signing protocol by Gennaro & Goldfeder. First, we will explain what is a non-malleable equivocable commitment. Second, we will explain the specification of the DSA signature scheme. And third, we will explain the concept of Feldman's VSS.

#### DSA signature scheme

The Digital Signature Algorithm (DSA) is a signature scheme whose security relies on the discrete logarithm problem. The public paramenters of this scheme are a cyclic group $\mathcal{G}$ with order $q$, a generator $g$ for $\mathcal{G}$, a hash function $H: \{0, 1\}^* \rightarrow \mathbb{Z}_q$, and a hash function $H': \mathcal{G} \rightarrow \mathbb{Z}_q$. The specification of the algorithms are presented next:

- $\textsf{KeyGen}$: on input a security parameter, the algorithm outputs a secret key $x$ choosen at random from $\mathbb{Z}_q$, and also outputs a public key $y = g^x$ in $\mathcal{G}$.
- $\textsf{Sign}$: On imput a message $M$, do the following:
    - Compute $m = H(M)$.
    - Choose $k \in \mathbb{Z}_q$ at random.
    - Compute $R = g^{k^{-1}}$ in $\mathcal{G}$ and $r = H'(R)$ in $\mathbb{Z}_q$.
    - Compute $s = k(m + x \cdot r) \mod q$.
    - Output $\sigma = (r, s)$.
- $\textsf{Ver}$: on input $M$, $\sigma$ and $y$, do:
    - Compute $m = H(M)$.
    - Check that $r, s \in \mathbb{Z}_q$.
    - Compute $R' = g^{(ms^{-1} \mod q)}y^{(rs^{-1} \mod q)}$
    - Accept if $H'(R') = r$.

We stress that ECDSA is a particular case of DSA where the group $\mathcal{G}$ is an elliptic curve group. For this work, all the results presented also hold for ECDSA.

#### Share conversion protocol

The goal with the share conversion protocol is to transform multiplicative into additive shares of a secret. Specifically, suppose that $x = ab \mod q$ where $a, b \in \mathbb{Z}_q$ are the multiplicative shares held by Alice and Bob respectively. Using a share conversion protocol, Alice and Bob will compute $\alpha, \beta \in \mathbb{Z}_q$ such that $\alpha + \beta = x = ab \mod q$. At the end Alice will hold bot $\alpha$ and $a$, while Bob will hold $\beta$ and $b$.

For this version of the protocol, we assume that Alice has distributed a key $A$ for an additively homomorphic scheme $\mathcal{E}$ over an integer $N$. In the protocol, we assume that $B = g^b$ is public to force Bob to provide a correct $b$. This protocol (including the additional check) is denoted as $\textsf{MtAwc}$ ("multiplicative to aditive with chech"). The protocol without this check is denoted by $\textsf{MtA}$. We present the protocol next:

{{< figure src="https://hackmd-prod-images.s3-ap-northeast-1.amazonaws.com/uploads/upload_9f31e75533146f146f3153542bc64556.png?AWSAccessKeyId=AKIA3XSAAW6AWSKNINWO&Expires=1697229202&Signature=U30NpAyFCiDKblx%2FJHddOlVb56E%3D" >}}

#### Non-malleable equivocable commitments

A *trapdoor* commitment scheme is a commitment which is information-theoretic hidding and computationally binding, where for the later, the scheme admits a trapdoor such that when known, it reveals the commited message entirely. However, this trapdoor is hard to be computed efficiently.

A trapdoor commitment scheme consists in four algorithms $\textsf{KG}$, $\textsf{Com}$, $\textsf{Ver}$, $\textsf{Equiv}$ such that:

- $\textsf{KG}$ is the algorithm for key generation which outputs a pair $(\textsf{pk}, \textsf{tk})$, where $\textsf{pk}$ is the public-key associated with the commitment scheme and $\textsf{tk}$ is the trapdoor key.
- $\textsf{Com}$ is the commitment algorithm. On input $\textsf{pk}$ and a message $M$, it outputs $[C(M), D(M)] = \textsf{Com}(\textsf{pk}, M, R)$ where $R$ are the coin toses. $C(M)$ is the commitment string, while $D(M)$ is the decommitment string. The later is kept secret until the openning time.
- $\textsf{Ver}$ is the verification algorithm. On input $C$, $D$ and $\textsf{pk}$, it either outputs a message $M$ or $\bot$.
- $\textsf{Equiv}$ is the algorithm that opens a commitment in every possible way given the trapdoor string. On input $\textsf{pk}$, stringgs $M$, $R$ such that $[C(M), D(M)] = \textsf{Com}(\textsf{pk}, M, R)$, a message $M' \neq M$ and a string $T$. if $T = \textsf{tk}$, then $\textsf{Equiv}$ will output $D'$ such that $\textsf{Ver}(\textsf{pk}, C(M), D') = M'$.

In real-world applications, it is common to use a hash function $H$ to instantiate this commitments. For a message $m$, we define the commitment as $h = H(m, r)$ where $r$ is randomly chosen with lenght equal to the security paramenter $\lambda$.

#### Feldman's VSS

Feldman's VSS is an extension of the Shamir's secret-sharing scheme. To share a secret $\sigma \in \mathbb{Z}_q$, the dealer generates a random polynomial $p$ such that $p(0) = \sigma$. This means that the polynomial has the following structure:

$$
p(x) = \sigma + a_1 x + a_2 x^2 + \cdots + a_t x^t \mod q.
$$

As in Shamir's secret-sharing, each party $P_i$ receives $\sigma_i = p(i) \mod q$ as a share of the secret value $\sigma$. The extension comes in place when the dealer of the secret also publishes $v_i = g^{a_i}$ in the group $\mathcal{G}$ for all $i \in [t]$, and $v_0 = g^\sigma$. With this modification, each party $P_i$ can check the consistency of each share $\sigma_i$ by checking the equality

$$
g^{\sigma_i} \stackrel{?}{=} \prod_{j=0}^t v_j^{i^j}
$$

in $\mathcal{G}$. In the case that some party complains about the check, the protocol must abort.

### Key generation

The goal of the key generation is to obtain $(t, n)$ shares of the private key that will be used as input in the signing phase. For this specification, we assume that each party $P_i$ has an associated public key for the additively homomorphic encryption scheme $\mathcal{E}$. Below, present the protocol:

{{< figure src="https://hackmd-prod-images.s3-ap-northeast-1.amazonaws.com/uploads/upload_e5bae00beec7d73b596e96e9634f9315.png?AWSAccessKeyId=AKIA3XSAAW6AWSKNINWO&Expires=1697229221&Signature=QiWvDlo6IdzZ56X0Ri90hAFSURw%3D" >}}

### Signing

Now, we present the signing protocol. It receives $m = H(M)$ as input, where $M$ is the message that will be signed. Additionaly, the parties have $(t, n)$ shares of the private key $x$.

Let $S \subseteq [n]$ the set of parties involved in the signing protocol, such that $\vert S \vert = t + 1$. Knowing that the key generation protocol returned Shamir shares of the private key $x$, each party can construct the appropriate Lagrange coefficients $\lambda_{i, S}$ used to interpolate the polinomial used in the secret-sharing scheme. If $x_i$ are the Shamir secret shares of $x$, each party can compute $w_i = \lambda_{i, S} \cdot x_i$. Notice that $w_i$ is a (t, t + 1) share of $x$ held by the party $P_i$, which means that $x = \sum_{i \in S} w_i$. Also, the parties have public values $X_i = g^{x_i}$, therefore, each party can compute $W_i = g^{w_i} = X_i^{\lambda_{i, S}}$.

{{< figure src="https://hackmd-prod-images.s3-ap-northeast-1.amazonaws.com/uploads/upload_107d87a599c43fc6d5202632be25ad49.png?AWSAccessKeyId=AKIA3XSAAW6AWSKNINWO&Expires=1697229263&Signature=QFh6vEmlDEy3M6MsfYRHOik4o8I%3D" >}}

Notice that in Step (5B) of the previous protocol, the party $P$ broadcasts $V = R^s g^l$ and $A = g^\rho$ and he need to prove that he knows $s$, $l$ and $\rho$ for which the previous equations hold. To prove that he knows $\rho$, it can be used a classic Schnorr proof. To prove that he know $s$ and $l$, they can use the following protocol:

{{< figure src="https://hackmd-prod-images.s3-ap-northeast-1.amazonaws.com/uploads/upload_c66961c621421a4ecf71c5512cdb1129.png?AWSAccessKeyId=AKIA3XSAAW6AWSKNINWO&Expires=1697229279&Signature=nRgSABzmhdRFm%2BlRkWukCNlkiw8%3D" >}}

## Security concerns

- In the paper, Gennaro & Goldfeder say that the last version of the paper has some fixes in the protocol and its proofs of security for which the previous version is not correct. Therefore, it is important to double-check which version of the protocol has been implemented.
- At the end of the signing protocol, the parties need to verify that the produced signature is valid using the standalone version of the verify algorithm in the DSA/ECDSA signature scheme.
- The previous version of the protocol did not check the range of the values encrypted with the Paillier encryption scheme which produces a leak of information about honest parties' shares.
- Works like [Tymokhanov & Shlomovits (2021)](https://eprint.iacr.org/2021/1621) and [Makriyannis & Peled (2021)](https://info.fireblocks.com/hubfs/A_Note_on_the_Security_of_GG.pdf) have shown that removing the ZK-proofs lead into an insecure protocol. Therefore, it is important to check that all the proofs are correctly executed.
- The protocol assumes the existence of a broadcast channel as well as point-to-point channels connecting every pair of players. In this case, we need to check that these conditions are correctly realized.
- There is a small typo in the non-malleable equivocable commitment in the $\textsf{Com}$ algorithm where they say "where $r$ is the coin tosses", but the correct version is "where $R$ is the coin tosses".
- The authors say that non-malleable commitments that are not "concurrently" secure are not suitable for the protocol. Instead, an implementation should consider a concurrent secure non-malleable commitment scheme.
- It is necessary to check that whenever Feldman's VSS protocol complains about a bad check-in of the shares, the protocol will abort.
- The authors recommend that, in a typical choice of parameters, $N$ is approximately $q^8$. Also, they stress that the parties should check the size of N to ensure the ZK property in the proofs.
- It is important to remember that the message to be signed is $m = H(M)$ and not $M$ itself. 
- In the signing protocol there are a series of checks that need to be computed, specifically, in Steps 5B, 5D, and 5E.
- The paper presents Section 5 as a historical record, however, the simplified protocol presented in that section is not secure.
- If the implementation uses another additively homomorphic encryption scheme, it requires an assumption analogous to Paillier-EC or a ZK-proof for the statement in the $\textsf{MtAwc}$ protocol. Also, it is needed to guarantee security against "adversarially chosen" public keys.
- **[List all the ZK-proofs]**