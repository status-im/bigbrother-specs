# P2P Discovery Comparison

**WARNING: This is an early draft, and likely contains errors.**

This document comparse various protocols and methods used for peer discovery in p2p networks, with the intention of making it easier to chose one for any specific use case. It is part of a larger series on secure messaging, and therefore will likely contain certain biases towards that use-case.

> Request for comments: Please direct comments on this document to @decanus. These comments can be on the structure, content, things that are wrong, things that should be looked into more, things that are unclear, as well as anything else that comes to mind.

## [Kademlia](https://en.wikipedia.org/wiki/Kademlia)

### Reading
 - http://www.scs.stanford.edu/~dm/home/papers/kpos.pdf
 - https://arxiv.org/pdf/1408.3079.pdf

**Pros**
 - @TODO
 
**Cons**
 - Running a DHT on a mobile client
 
## Secure Scuttlebutt

## [Rendezvous](https://github.com/libp2p/specs/tree/e1083c1f9d8f7afc0d65a43a12b05492f3873385/rendezvous)

**Pros**
 - Leightweight
 
**Cons**
 - Susceptible to spam attacks (suggest PoW for anti-spam, this is no longer mobile friendly)
