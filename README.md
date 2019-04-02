# BigBrother Specs <img align="right" src="https://thegraphicsfairy.com/wp-content/uploads/2013/10/Free-Public-Domain-Watching-Eye-Image-GraphicsFairy.jpg" height="80px" />

*Big Brother can't watch you!*

## Table of Contents
- [Abstract](#abstract)
- [Motivation](#motivation)
- [Terms](#terms)
- [Design Goals](#design-goals)
- [System Design / Architecture](#system-design--architecture)
  - [SOLID](#solid)
- [Phases](#phases)
  - [Phase 0](#phase-0---xkeyscore-data-sync)
  - [Phase 1](#phase-1---bullrun-transport-privacy-layer)
  - [Phase 2](#phase-2---stoneghost-p2p-overlay)
  - [Phase 3](#phase-3---credible-secure-transport)
  - [Phase 4](#phase-4---cointelpro-data-sync-clients)
  - [Phase 5](#phase-5---nativeflora-trust-establishment)
- [Footnotes](#footnotes)

## Abstract

BigBrother defines a family of protocols that when combined create a decentralized peer-to-peer messaging stack. BigBrother should enable various [messaging types](message-types.md), in order to be called a messaging stack.

The family of protocols SHOULD be flexible enough to allow various use case implementations not limited to that being defined by the entire family. The BigBrother protocol itself defines mainly the way the family of protocols interact with one another allowing for customizability on each layer.

## Motivation

**TODO: TALK ABOUT SHORTCOMINGS OF WHISPER**

## Terms

| Term | Definition|
|:---:|--|
| **Stack** | Defines the entire family of protocols, how they interact and what goals are achieved. |
| **Protocol** | Defines a single layer in the *stack* along with its endpoints used for communication with various other protocols in the stack.

## Design Goals

**1. Anonymity** - `sender` / `receiver` MUST remain anonymous.

**2. Unlinkability** - It should not be identifiable which parties are talking to eachother. 

**3. Scalable** - @TODO

**4. Incentivized** - @TODO

**5. Decentralized** - @TODO

**6. Resistant** - @TODO

**8. Inclusive** - Protocols within the stack should be designed to work on resource restricted devices, this allows for a higher participation making the entire BigBrother protocol more usable and reliable. 

## System Design / Architecture

The protocols summarized by BigBrother all follow a standard of architectural design principles with the goal of keeping each component noncomplicated. We architect a stack where each protocol interacts holistically, without the desire to create unnecessary complexity. Each protocol included in the stack should be simplistic enough to allow for multiple implementations creating client diversity, allowing us to ensure that the entire stack is unambiguous<sup>1</sup>.

### SOLID

Throughout the design of the stack we follow the SOLID design principles, redefining them slightly so they make sense in the context of protocols.

 - **S**ingle Responsibility principle: Each protocol in the family should only have a single responsibility.
 - **O**pen/closed principle: @TODO
 - **L**iskov substitution principle: @TODO
 - **I**nterface segregation principle: @TODO
 - **D**ependency inversion principle: @TODO
 
## Stack

The below table shows the intended layers of the network from the *highest* to the *lowest*.

| Layer / Protocol  | Purpose                         | Example                       |
|-------------------|---------------------------------|-------------------------------| 
| Sync Clients      | End user functionality          | 1:1, group chat, tribute, ... |
| Data Sync         | Syncing data/state              | Bramble Sync Protocol (ish)   |
| Secure Transport  | Confidentiality, PFS, etc       | Double Ratchet                |
| Transport Privacy | Metadata protection             | Mixnet?                       |
| P2P Overlay       | Overlay routing, NAT traversal  | libp2p?                       |
 
## Phases

Inspired by the ETH 2.0 implementation process, we have decided to roll out BigBrother in multiple phases. The phases can mostly be linked to various layers in the [stack](#stack). Some of these phases are sometimes more loosely coupled than ETH 2.0 components meaning they can be done in parallel as only the communication API is relevant.

### Phase 0 - [XKEYSCORE (Data Sync)](/data_sync/README.md)

### Phase 1 - BULLRUN (Transport Privacy Layer)

 - Mixnet
 - [PSS](https://gist.github.com/zelig/d52dab6a4509125f842bbd0dce1e9440)
 - Bluetooth local network
 - Sneakernet

### Phase 2 - STONEGHOST (P2P Overlay)

### Phase 3 - CREDIBLE (Secure Transport)

- Message Layer Security (MLS)

### Phase 4 - COINTELPRO / JTRIG OPS (Data Sync Clients)

### Phase 5 - NATIVEFLORA / TAO OPS (Trust Establishment)

## Footnotes

- [1] http://ethdocs.org/en/latest/ethereum-clients/choosing-a-client.html
