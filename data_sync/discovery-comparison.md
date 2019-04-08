# Approaches to discovery in P2P networks

*Written April 8th, 2019 by Dean Eigenmann*

**WARNING: This is an early draft, and likely contains errors.**

This document compares various protocols and methods used for peer discovery in p2p networks, with the intention of making it easier to choose one for any specific use case. It is part of a larger series on secure messaging, and therefore will likely contain certain biases towards that use case.

> Request for comments: Please direct comments on this document to @decanus. These comments can be on the structure, content, things that are wrong, things that should be looked into more, things that are unclear, as well as anything else that comes to mind.

## Table of Contents
1. [Introduction](#introduction)
2. [Methodology](#methodology)
3. [Compared technologies](#compared-technologies)
4. [Comparison](#comparison)
5. [Summary](#summary)
6. [Acknowledgements](#acknowledgements)
7. [References](#references)

## Introduction

An important aspect of p2p networks is the actual discovery of peers, this dictates how a node finds other nodes in the network. It can also determine how on which nodes information is stored and whom a node most connect to in order to retrive said information. In this paper we analyze various methods for peer discovery, what network types they work for and what their benefits and drawbacks are when applying various limitations such as the challenge of running these methods on mobile devices.

## Methodology

Literature comparing various discovery methods for p2p networks seems rather limited, therefore most of the methodologies used within this paper are not based on previous literature.

The notes presented below are based on provided documentation and specification, code has not been looked at generally or fully reviewed. All results are tentative.

### Compared Dimensions

## Compared technologies

@TODO Write some stuff on each technology, quick abstract

### Kademlia

### Secure Scuttlebutt

### Rendezvous

## Comparison

## Summary

## Acknowledgements

## References

---
**OLD**

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
