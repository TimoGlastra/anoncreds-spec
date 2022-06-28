## Cryptographic protocols

### Terminology

_TODO: general terms here_

### Protocols

Most protocols described here are zero-knowledge proofs for different statements. They follow the patterns of three-move rounds protocols with a commit, a challenge and a response phase which we shall describe as three separate algorithms. The challenge phase is unique, whereas the commit and response phases have a specific instance for each of the statements being proved.

#### Proving knowledge of a signature with selective disclosure of messages (`ProveCL`)

`ProveCLCommit` and `ProveCLResponse` are used by a Holder who possesses one or more signatures from one or more Issuers and uses them to derive a proof of knowledge for them. The algorithms are invoked once per signature.

_TODO: considerations about the algorithm and its security_

_TODO: clarify exact format and encoding of inputs and outputs_

`ProveCLCommit` finds has its reference implementation [here](https://github.com/hyperledger/ursa/blob/ece6ce32a59df4e1f99fa38243c1236423066bc2/libursa/src/cl/prover.rs#L1313-L1394).

```
( A', v', e', Z~, e~, v~ ) = ProveCLCommit( PK, signature, (m_1,..., m_L), RevealedIndexes, R )

Inputs:

 - PK (REQUIRED), the public key of the issuer
 - signature (REQUIRED), the output of the issuance protocol
 - (m_1,..., m_L) (OPTIONAL), the set of signed messages
 - RevealedIndexes (OPTIONAL), indices of revealed messages
 - R (OPTIONAL), the set of random factors to blind unrevealed messages; each random has length l_m + l_0 + l_H

Parameters:

 - TBD

Definitions:

 - l_n, the bitlength of the RSA modulus n of the issuer public key
 - l_m, the bitlength of messages
 - l_e, the bitlength of the prime e
 - l_v, the bitlength of the randomization factor v
 - l'_e, the size of the interval in which e is chosen
 - l_0, security parameter that governs the statistical zero-knowledge property
 - l_H, the bitsize of values hashed by the hash function H used for the Fiat-Shamir heuristic
 - L, the number of messages signed by the issuer

Outputs:

 - A', term of the rerandomised signature
 - v', term of the rerandomised signature
 - e', term of the rerandomised signature
 - Z~. t-value for the signature
 - e~, randomness used to generate t-value
 - v~, randomness used to generate t-value

Procedure:

 1. (i1, i2,..., iR) = RevealedIndexes       # the indices of messages that are revealed in this proof

 2. (j1, j2,..., jU) = [L] \ RevealedIndexes # the indices of messages that are kept hidden

 3. (m~_1, m~_2,..., m~_U) = R               # the random factors blinding each hidden message

 4. (n, S, Z, R_1, ..., R_L) = PK            # unpack the issuer public key

 5. (A, e, v) = signature                    # unpack the signature

 6. r = PRNG(l_n + l_0)                      # choose random to blind the signature

 7. A' = A * S^r mod n                       # compute the randomised signature

 8. v' = v - e * r                           # recompute v given the randomisation

 9. e' = e - 2^{l_e - 1}                     # prepare value to prove that e is positive

10. e~ = PRNG(l'_e + l_0 + l_H)              # random for t-value of the signature

11. v~ = PRNG(l_v + l_0 + l_H)               # random for t-value of the signature

12. Z~ = A'^{e~} * S^{v~} mod n              # compute t-value for the signature

13. foreach j in (j1, j2,..., jU):

14.     Z~ = Z~ * R_j^{m~_j} mod n           # add component for each undisclosed message

15. return ( A', v', e', Z~, e~, v~ )
```

`ProveCLResponse` finds has its reference implementation [here](https://github.com/hyperledger/ursa/blob/ece6ce32a59df4e1f99fa38243c1236423066bc2/libursa/src/cl/prover.rs#L1533-L1633).


```
pi = ProveCLResponse( (m_1,..., m_L), RevealedIndexes, R, c, ( A', v', e', Z~, e~, v~ ) )

Inputs:

 - (m_1,..., m_L) (OPTIONAL), the set of signed messages
 - R (OPTIONAL), the set of random factors to blind unrevealed messages; each random has length l_m + l_0 + l_H
 - c, an octet string representing the Fiat-Shamir challenge value
 - A', term of the rerandomised signature
 - v', term of the rerandomised signature
 - e', term of the rerandomised signature
 - Z~. t-value for the signature
 - e~, randomness used to generate t-value
 - v~, randomness used to generate t-value

Parameters:

 - TBD

Definitions:

 - TBD

Outputs:

 - A', term of the rerandomised signature
 - Z~. t-value for the signature
 - e^, s-value for the signature
 - v^, s-value for the signature
 - (m^_1,..., m^_L), s-value for the signature

Procedure:

 1. (j1, j2,..., jU) = [L] \ RevealedIndexes # the indices of messages that are kept hidden

 2. (m~_1, m~_2,..., m~_U) = R               # the random factors blinding each hidden message

 3. v^ = v~ + c * v'                         # the response for the term v'

 4. e^ = e~ + c * e'                         # the response for the term e'

 5. foreach j in (j1, j2,..., jU):

 6.     m^_j = m~_j + c * m_j                # the response for the undisclosed messages

 7. pi = (A', Z~, e^, v^, (m^_1,..., m^_L) ) # the terms that constitute the proof that will be verified

 8. return pi
```
