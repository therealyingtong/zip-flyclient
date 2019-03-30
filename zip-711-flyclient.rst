::

  ZIP: 711 (?)
  Title: FlyClient ZCash SPV
  Champions: Zooko <zooko@z.cash> (?)
  Status: Draft
  Category: Standards Track (?)
  Created: 2019-03-30
  License: MIT (?)


Terminology
===========
The key words "MUST", "SHOULD", and "MAY" in this document are to be interpreted as
described in RFC 2119. [#RFC2119]_

**Light client**
  A client that is not a full participant in the network of Zcash peers. It can send and
  receive payments, but does not store or validate a copy of the blockchain.

**High probability**
  An event occurs with high probability if it occurs with probability 1-O(1/2^λ), where λ is the security parameter.

**Negligible probability**
  An event occurs with negligible probability if it occurs with probability O(1/2^λ), where λ is the security parameter.

**Velvet fork**
  A backwards compatible protocol upgrade in which blocks by outdated nodes are not rejected. These rely on clients reinterpreting blockchain data.
 

Abstract
========
A proposed implementation of the probabilistic verification FlyClient protocol [#FlyClient]_ on top of the existing ZCash reference light client protocol [#ZIPXXX]_.

Motivation
==========
FlyClient reduces the number of block headers needed for light client verification of a valid chain, from linear (as in the current reference protocol) to logarithmic in blockchain length. This verification is correct with high probability. It also allows creation of subchain proofs, so light clients need only check blocks later than the most recently verified block index. Following that, verification of a transaction inclusion within that block follows the usual reference protocol [#ZIPXXX]_. 


Specification
=============

New block header
`````````````````
Each FlyClient prover has to maintain a commitment to all its block headers in a Merkle mountain range (MMR). Each block header in the MMR must include the MMR root of all the blocks before it. This can be implemented in a Velvet fork of the ZCash protocol. All evaluations assume a block header of size 508 bytes and a hash output of 32 bytes.

Merkle mountain range
``````````````````````
Each MMR node contains an additional field of 8 bytes to store the cumulative difficulty of the leaves below it.

Verifier
`````````
Verifying a valid chain: the hash of the block B_n at the head of the chain specifies a random subset of blocks to sample. For each block, the prover provides the corresponding block header and an MMR proof that the block is located at the correct height of the chain committed by B_n. The verifier checks that the MMR root stored in each sampled block correctly commits to the correct subchain of the chain committed to in B_n. If the PoW solution or the MMR proofs of any of the sampled blocks is invalid, then the verifier rejects the proof.  Otherwise, it accepts B_n as the last block of the honest chain.

Verifying a block in the valid chain: MMR 

Verifying transaction inclusion in the block: To ensure that tx is included in some block in the honest chain, the client proceeds similar to a regular SPV client, i.e., verifies the Merkle proof provided by the prover against the root of the transaction Merkle tree included the block header along with another MMR proof that the block is in the MMR rooted at M_n.


Rationale
=========
Unlike previous skip list SPVs (PoPoW, NIPoPoW), the FlyClient protocol allows for variable block difficulty, which is appropriate for ZCash's variable difficulty [#difficulty]_. FlyClient also avoids bribing attacks since the selection of blocks on its skip list is random and not rule-based.

Security and privacy considerations
===================================
This protocol is probabilistic, meaning there is a negligible probability that an invalid chain can be verified. Assumes adversary mining power is bounded by a known fraction of combined mining power of honest nodes, and cannot drop or tamper with messages between client and full nodes. Assume client is connected to at least one full node and knows the genesis block.

applicable, security and privacy considerations should be explicitly described, particularly if the ZIP makes explicit trade-offs or assumptions. For guidance on this section consider RFC 3552. as a starting point.

Reference implementation
========================
The authors of FlyClient

References
==========
.. [#RFC2119] `Key words for use in RFCs to Indicate Requirement Levels <https://tools.ietf.org/html/rfc2119>`_

.. [#FlyClient] `FlyClient protocol (2019) <https://eprint.iacr.org/2019/226.pdf>`_

.. [#ZIPXXX] `ZCash reference light client protocol <https://github.com/gtank/zips/blob/light_payment_detection/zip-XXX-light-payment-detection.rst>`_

.. [#difficulty] `ZCash historical block difficulty <https://www.coinwarz.com/difficulty-charts/zcash-difficulty-chart>`_