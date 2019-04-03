# Different approaches to p2p data sync
*Written March 29, 2019 by Oskar. Updated April 1, April 2.*

**WARNING: This is an early draft, and likely contains errors.**

This document compares various forms of data sync protocols and applications, along various dimensions, with the goal of making it easier for you to choose which one is for you. It is part of a larger series on secure messaging.

>Request for comments: Please direct comments on this document to Oskar. These comments can be on the structure, content, things that are wrong, things that should be looked into more, things that are unclear, as well as anything else that comes to mind.

## Table of contents

1. [Introduction](#1-introduction)
2. [Background and definitions](#2-background-and-definitions)
3. [Methodology](#3-methodology)
4. [Comparison](#4-comparison)
5. [Summary](#5-summary)
6. [Acknowledgements](#6-acknowledgements)
7. [References](#7-references)

## 1. Introduction
In a p2p network you often want to reliably transmit and replicate data across participants. This can be either large files, or messages that users want to exchange between each other in a private chat. This is especially challenging on mobile devices. Additionally, you might want security properties beyond robustness, such as privacy-preservation, censorship resistance and coercion resistance.

In this paper we go through various approaches to data sync in a p2p context. We then compare them across relevant dimensions to make it easier for you to make an informed decision about what solution you want to use.

## 2. Background and definitions

Let's start with a raw list of definitions. These are phrases with some specific meaning that we'll use further down the line. Some of it the terms are general, and some are more specific to data sync in particular.

*Node*: Some process that is able to store data, do processing and communicate with other nodes.

*Peer*: The other nodes that a peer is connected to.

*Peer-to-peer (P2P)*: Protocols where resources are divided among multiple peers, without the need of central coordination.

*Device*: A node capable of storing some data locally.

*Identity*: A user's cryptographic identity, usually a single or several public keypair.

*User*: A (human) end-user that may have multiple *devices*, and some form of identity.

*Data replication*: Storing the same piece of data in multiple locations, in order to improve availability and performance.

*Data sync*: Achieving *consistency* among a set of nodes storing data

*Mobile-friendly*: Multiple factors that together make a solution suitable for mobile. These are things such as dealing with *mostly-offline* scenarios, *network churn*, *limited resources* (such as bandwidth, battery, storage or compute).

*Replication object*: Also known as the minimal unit of replication, the thing that we are replicating.

*Friend-to-friend network (F2F)*: A private P2P network where there are only connections between mutually trusted peers.

*Content addressable storage (CAS)*: Storing information such that it can be retrieved by its content, not its location. Commonly performed by the use of cryptographic hash functions.

*Public P2P*: Open network where peers can connect to each other.

*Structured P2P network*: A public p2p network where data placement is related to the network topology. E.g. a DHT.

*Unstructed P2P network*: A public p2p network where peers connect in an ad hoc manner, and a peer only knows its neighbors, not what they have. 

*Super-peer P2P network*: A non-pure p2p network, hybrid client/server architecture, where certain peers do more work. 

*Private P2P*: Private network where peers must mutually trust each other before connecting, can either be F2F or group-based.

*Group-based P2P network*: A private p2p network where you need some form of access to join, but within group you can connect to new peers, e.g. friend of a friend or a private BT tracker.

**These have yet to be filled out**
*Consistency model*: ...

*Mostly-offline:* ...

*Network churn:* ...

*Light node:* ...

*Cryptographic hash function*: ...

*Multi-device:*...

*Robustness*: ...

*Privacy-preservation*: ...

*Censorship-resistance*: ...

*Coercion-resistance*: ...

## 3. Methodology
  We look at generally established dimensions in the literature [xx1], and evaluate protocols and applications based on these. Additionally we add some dimensions that aren't necessarily captured in the literature, such as mobile-friendliness and practical implementation. Specifically the focus is on p2p applications that perform some form of data synchronization, with a bias towards secure messaging applications.

All notes are tentative and are based on the provided documentation and specification. Code has generally not been looked into, nor has any empirical simulations been performed. These results are tentative as they have yet to be reviewed.

### Compared dimensions

**Request for comments: What's a  better way to decompose these properties? Ideally something like 2-5 dimensions that matter most. Right now it reads a bit like a laundry list, or like an ad hoc classification. Update April 1: Factored other considerations out into guarantees and practical application/protocol considerations. Also tweaked How section to also deal with transport requirements and peer discovery.**

These dimensions are largely taken from the survey paper by Martins as well, with some small tweaks. To make it easier to survey, we divide up the dimensions into rough sections.

#### 1. Why and what are we syncing?
- *Problem domain*. Why are we syncing stuff in the first place?
- *Minimal unit of replication* . The minimal entity that we can replicate. The actual unit of replication is the data structure that we are interested in, usually a collection of entities. Related: version history abstraction (linear vs DAG).
- *Read-only or read and write*. Is the (actual unit of replication) data static or not?

#### 2. Who is participating?
- *Active vs passive replication*. Are participants replicating data that they are not interested in themselves?

####  3. When and where are we syncing?
Replication control mechanisms.

- *Single-master vs multi-master*. Both entity and collection.
- *Synchronous (eager) vs asynchronous (lazy)*.
- *If asynchronous, optimistic or not*.
- *Replica placement*. Full or partial replication.

#### 4. How are they syncing?
- *P2P Topology*. Public or private P2P? Unstructured/structured/super-peer or friend-to-friend/group-based?
- *Peer Discovery*. How do peers discover and connect with each other?
- *Transport requirements*. Any specific requirements on the underlying transport layer?

#### 5. What guarantees does it provide?
- *Consistency-model*. What are the guarantees provided?
- *Privacy-preservation*. Assuming underlying layers provide privacy, are these guarantees upheld?

#### 6. Engineering and application, how does it work in practice?
- *Actively used*. Is an instance of the protocol used in the real-world?
- *Mobile friendliness*. Any specific considerations for mobile?
- *Well-defined spec*. Is there a well-defined and documented spec to refer to?
- *Protocol upgradability*. Are there any mechanisms for upgrading the protocol?
- *Framing considerations*. Does it support messages that don't need replication?

## Notes on single-master vs multi-master
For single-master, there's only a single node that writes to a specific piece of data. The other peers purely replicate it and don't change it.

For many of the studied systems, *content addressable storage* is used. This means the replication object is usually immutable, and it is only written to once. As a side effect, many systems are naturally single-master from this point of view.

This is in comparison with the Martins survey paper, where they are more interested in update-in-place programming through replicating SQL DBs and other rich data structures.

However, if we look at what is semantically interesting for the user, this is usually not an individual message or chunk of a file. Instead it is usually a conversation or a file. Seen from that point of view, we often employ some form of linear or DAG-based version history. In this case, many participants might update the relevant sync scope. Thus, the system is better seen as a multi-master one. To capture this notion we have divided the minimal unit of replication into entity and collection.

## Compared technologies

**RFC: Which way is the most useful to look at it? Worried about confusion between the two, especially since many applications have tightly coupled set of protocols.**

This includes both applications and specification. For example p2p messengers such as Briar (Bramble) [xx2], Matrix [xx3] and Secure Scuttlebutt (SSB) [xx4]. As well as general p2p storage solutions such as Swarm [xx5].

In order to compare apples to apples, we compare Briar and Matrix as in how they de facto use sync. An additional consideration would be to compare Bramble and Matrix Server-to-Server API/spec directly. This also means Swarm doesn't necessarily make sense to compare directly to these applications.

This application focus is justified by the following observations:

a) most protocols in this space are young, underspecified and somewhat coupled with the application using them, especially compared to protocols such as IP/TCP/UDP/TLS/Bittorrent, etc

b) we are ultimately interested in considerations that only manifest themselves as the protocol is used on e.g. mobile devices

## 4. Comparison

### What and why are we syncing?

#### Briar

*Problem domain*. Bramble is the protocol Briar uses for synchronizing application layer data. Briar is a messaging app, and application data can be events such as sending messages, joining a room, etc. 
 
*Minimal unit of replication*. Immutable messages. The actual unit of replication is a DAG, which is encoded inside the encrypted message payload. This produces a message graph.

*Read-only or read and write*. Each message is immutable, but since we are dealing with instant messaging a group context, such as a conversation, is changing. Thus it is read and write.

#### Matrix

*Problem domain*. Matrix is a set of open HTTP APIs that allow for real-time synchronization and persistance of arbitrary JSON over a federation of servers. Often this is done in a room context, with participants talking to each other. More generally, for messaging between humans.

*Minimal unit of replication*. Messages that also form a DAG.

*Read-only or read and write*. Message domain so read and write. Individual messages are generally immutable.

#### Secure Scuttlebutt

*Problem domain*.

*Minimal unit of replication*

*Actual unit of replication*. 

*Read-only or read and write*.

#### Swarm

*Problem domain*.

*Minimal unit of replication*

*Actual unit of replication*. 

*Read-only or read and write*.

#### Comparison

| Dimension | Briar | Matrix | SSB | Swarm |
|------------|------|------| ----| ---|
| *Problem domain*. | Messaging  | Messaging | Social network | Storage |
| *Minimal unit of replication* | Messages | Messages | Messages | Chunks |
| *Actual unit of replication*.  | DAG | DAG  | Log | | 
| *Read-only or read and write*. | Read and write | Read and write | Read and write | Read and write | 

### Who is participating?

#### Bramble
*Active vs passive replication*. In Bramble, only the participants in a group chat are syncing messages. It is thus a form of passive replication. There have some proposals to have additional nodes to help with offline inboxing.

#### Matrix
*Active vs passive replication*. In Matrix, homeservers are used for replication. Homeservers themselves don't care about the messages, so it is a form of active replication.

#### Secure Scuttlebutt
*Active vs passive replication*.

#### Swarm
*Active vs passive replication*. In Swarm, nodes replicate data that they are "assigned" to by virtue of being close to it.

#### Comparison

| Dimension | Briar | Matrix | SSB | Swarm |
|------------|------|------| ----| ---|
| Active replication? | Passive | Active | | Passive |

###  When and where are we syncing?

#### Multi-master
*Single-master vs multi-master*. Anyone who has access to the DAG can write to it. Each individual message is immutable and thus write-once. But it's multi-master since multiple people can update the principal data structure.

*Synchronous (eager) vs asynchronous (lazy)*. Asynchronous, you write to your own node first.

*If asynchronous, optimistic or not*. Optimistic, since each message update is a hash conflicts are likely to be rare.

*Replica placement*. Full or partial replication.

#### Matrix
*Single-master vs multi-master*. Essentially same as Briar. Multi-master.

*Synchronous (eager) vs asynchronous (lazy)*. Partial, because there's a HTTP API to Matrix server that needs to acknowledge before a message is transmitted.

*If asynchronous, optimistic or not*. Optimistic. Though there are sone considerations for the conflict resolutions when it comes to auth events, which are synced separately from normal message events. This is part of room state, which is a shared dictionary and they use room state resolution to agree on current state. This is in order to deal with soft failure / ban evasion prevention.

More on ban evasion: In order to prevent user from evading bans by attaching to an older part of DAG. These events may be valid, but a federation homeserver checks if such an event passes the current state auth checks. If it does, homeserver doesn’t propagate it. A similar construct does not appear in Briar, since there’s no such federation contruct and there’s no global notion of the current state.

*Replica placement*. Partial. DAG and homeserver can choose to get previous state when the join the network.

#### Secure Scuttlebutt
*Single-master vs multi-master*. The principal data structure in SSB is a log, and only you can write to your own log. Single-master.

*Synchronous (eager) vs asynchronous (lazy)*. Asynchronous, you add to your own log first.

*If asynchronous, optimistic or not*. Optimistic.

*Replica placement*. Full, because you start syncing your log from scratch.

#### Swarm
*Single-master vs multi-master*.

*Synchronous (eager) vs asynchronous (lazy)*.

*If asynchronous, optimistic or not*.

*Replica placement*. Full or partial replication.

#### Comparison

| Dimension | Briar | Matrix | SSB | Swarm |
|------------|------|------| ----| ---|
| *Multi-master?*  | Mulit-master | Multi-master | Single-master | |
| *Asynchronous?*  | Asynchronous | Partial | Asynchronous| |
|  *Optimistic?*  | Optimistic | Optimistic | Optimistic | | 
| *Replica placement*  | Partial | Partial | Full | 

### How are they syncing?

#### Bramble
*P2P Topology*. Multiple options, but basically friend to friend. Can work either directly over Bluetooth or Tor (Internet). For BT it works by directly connecting to known contact, i.e. friend to friend. For Tor it uses onion addresses , it also leverages a DHT (structured).

*Peer Discovery*. Usually add contact locally. beacon signal locally, then keep track of and update transport properties for each contact. Leverage Tor onion addresses.

*Transport requirements*. Requires a transport layer security protocol which provides a secure channel between two peers. For Briar, that means Bramble Transport Protocol.

#### Matrix
*P2P Topology*. Super-peer based on homeservers. Pure P2P discused discussed but not live yet.

*Peer Discovery*.

*Transport requirements*. Written as a set of HTTP JSON API:s, so runs on HTTP. Though there are some alternative transports as well.

#### Secure Scuttlebutt
*P2P Topology*. Private p2p, group-based.

*Peer Discovery*. Gossip-based, friends and then different levels of awareness 1-hop, 2-hop and 3-hop.

*Transport requirements*.

#### Swarm
*P2P Topology*. Public structured DHT.

*Peer Discovery*. Kademlia, and caching peers in various proximity bins.

*Transport requirements*.

#### Comparison

| Dimension | Briar | Matrix | SSB | Swarm |
|------------|------|------| ----| ---|
|*P2P Topology* | F2F | Super-peer | Group-based | Structured |
| *Peer Discovery* | | | | |
| *Transport requirements*  | | | | | 

### Guarantees provided

#### Bramble
*Consistency model*. Casual consistency, since each message encodes its (optional) message dependencies. This DAG is hidden for non-participants.

*Privacy-preservation*. It is a friend to friend network, and preserves whatever privacy underlying layer provides, so it has Tor's properties, and for short-distance the threat model is justifably weaker due to the increased cost of surveillence. However, a peer knows when another message is received by the receiving ACK (though it is possible to delay sending this, if on wishes).

#### Matrix
*Consistency model*. DAG so casual consistency, generally speaking.

*Privacy-preservation*.  Relies on homeservers, so sender and receiver anonymity not provided. Generally, if adversary controls homeserver they have a lot of information.

#### Secure Scuttlebutt
*Consistency model*. Sequential consistency due to append-only log for each participant.

*Privacy-preservation*.

#### Swarm
*Consistency model*.

*Privacy-preservation*.

#### Comparison
*Consistency model*.

*Privacy-preservation*.

| Dimension | Briar | Matrix | SSB | Swarm |
|------------|------|------| ----| ---|
|*Consistency model* | Casual | Casual | Sequential | |
| *Privacy-preservation* | Yes | | | |
| *Transport requirements*  | | | | | 

### Engineering and application

#### Bramble
*Actively used*. Released in the Play Store and on F-Droid. Seems like it, but not huge. ~300 reviews on Google Play Store, if that means anything.

*Mobile friendliness*. Partial. Yes in that it is built for Android and works well, even without Internet and with high network churn. Note that battery consumption is still an issue, and that it runs in the background. The latter makes an iOS app difficulty.

*Well-defined spec*. Yes/Partial, see Bramble specifications. Not machine readable though.

*Protocol upgradability*. Haven't seen any specific work in this area, though they do have multiple versioned specs, which generally seem based on accretion.

*Framing considerations*. As far as I can tell this is not a consideration.

#### Matrix
*Actively used*. Matrix has users in the millions and there are multiple clients. So definitely.

*Mobile friendliness*. Due to its super-peer architecture, mobile is straightforward and it can leverage basic HTTP PUT/long-lived GETs like normal messenger apps. DNS for homeservers can also be leveraged.

*Well-defined spec*. Partial/Yes. Well-described enough to have multiple clients, but not machine-readable as far as I can tell.

*Protocol upgradability*. Spec is versioned and rooms can have different versions that can be upgraded. Exact process around that is currently a bit unclear, though it does appear to have been a major consideration.

*Framing considerations*. Matrix supports multiple forms of message types. Aside from synced Persisted Data Units (PDU) they also have Ephemeral (EDU) and queries. PDUs record history of message and state of room. EDUs don’t need to be replied to, necessarily. Queries are simple req/resp to get snapshot of state.

#### Secure Scuttlebutt
*Actively used*. In production and many clients. Unclear exactly how active.

*Mobile friendliness*.

*Well-defined spec*.

*Protocol upgradability*.

*Framing considerations*.

#### Swarm
*Actively used*. Currently in POC testing phase with limited real-world usage.

*Mobile friendliness*.

*Well-defined spec*.

*Protocol upgradability*.

*Framing considerations*.

#### Comparison


| Dimension | Briar | Matrix | SSB | Swarm |
|------------|------|------| ----| ---|
|*Actively used* | Yes | Very | Yes | POC |
| *Mobile friendliness* | Partial | Yes | | |
| *Well-defined spec*  | Partial | Partial | | | 
| *Protocol upgradability* | | | | 
| *Framing considerations* | No | Yes | | 

### Table Summary

**RFC: Does a full table summary makes sense? Should it list all dimensions? What would be the most useful?**

| Dimension | Bramble | Matrix | SSB | Swarm |
| ---------- | -------- | -------- | --- |--- | 
| Replication object | Message (DAG) | Message* (DAG/*) | Messages (?) (Log) | Chunks (File/Manifest) |
| Single-master? | Single-master | Depends* | Single-master |  |
| Asynchronous? |Yes  | Partial | Yes | Yes | Yes |
| Asynchronous optimistic? | Yes | Yes | Yes | Yes |
| Full vs partial replication? | Partial | Partial | Partial? | Partial |
| Consistency model | Casual consistency | Casual | Eventual / Casual |
| Active replication? | No | Yes | Yes | Yes |
| P2P Topology | F2F Network | Super-peer | Group-based | Structured |

#### Brief discussion

What these results mean. Things that come together or not, independent considerations.

## 5. Summary

TODO.

Should answer what this means for you, given your constraint. For us specifically: mobile-friendly considerations, etc. Possibly a section on future work.

## 6. Acknowledgements

TODO.

Should reflect people who have helped shape this document.

## 7. References

xx1
[Martins, Pacitti, and Valduriez, “Survey of Data Replication in P2P Systems.”](https://hal.inria.fr/inria-00122282/PDF/Survey_of_data_replication_in_P2P_systems.PDF)

xx2
[Bramble Synchronisation Protocol, version 1](https://code.briarproject.org/briar/briar-spec/blob/master/protocols/BSP.md)

xx3
[Matrix Specification](https://matrix.org/docs/spec/)

xx4
[Scuttlebutt Protocol Guide](https://ssbc.github.io/scuttlebutt-protocol-guide/)

xx5
[Swarm documentation, release 0.3](https://buildmedia.readthedocs.org/media/pdf/swarm-guide/latest/swarm-guide.pdf)



## TODO

- Add graphs of linear vs DAG version history
- Clarify user, identity, multi-device definitions
- Mention alternative to data sync, i.e. for messaging RTC and framing, but motivate why useful especially for p2p
- Better document structure, e.g. see SoK Secure Messaging for example of survey paper
- Apples to apples: break apart e.g. Matrix and Swarm since they aren't directly comparable
- Capture in how are they syncing: API and push/pull.
- Consider collapasing actual replication unit.
- Matrix should capture auth state semantics.
- Mentions: How deal with other things that are kind of replicated but differently? E.g. metadata, etc. Auth state, ephemeral, manifests, feeds, etc.
- Remove redundancy. In the case dimensions are generally the same, consider treating it under same header.
- Fill in rest of definitions.
 
### Further work and research questions

**1. Multiple-device management and data sync.** (minor)
How do various solutions deal with multiple-devices and identities? I.e. Matrix, Briar etc. What does this look like for SSB if you have a single log but write from mobile and desktop? Related: ghost user problem and mitigations.

**2. CRDTs and version history.** (minor)
Elaborate on what problem CRDTs solve, their consistency model and problem domain, as well as how this related to logs and DAGs for version history.

**3. Breaking apart data sync into better components.**
The dimensions right now reads like a laundry list, and I'm not confident these are orthogonal considerations. Especially the "other considerations" section. Ideally there'd be just a few different dimensions that matter the most, with possible sub problems. A la principal component analysis, likely only a few really fundamental differences. Related to breaking things apart into similarties and differences.

**4. Extend comparison with more applications and protocols.**
E.g. Bittorrent, Git, Tribler, IPFS, Whisper, Status. Revisit Tox/TokTok.

**5. Clarify requirements of sync protocol in question.**
E.g. if it has transport requirements or certain protocol dependencies. This also relates to how tightly coupled a sync protocol is to other protocols. It is also relevant for engineering reality.

**6. Ensure knowns in data sync research log are captured**
I.e. for the comparative work that has been done in https://discuss.status.im/t/data-sync-research-log/1100, this should be summarized under the various dimensions.

**7. Ensure previously unknowns brought up considerations are captured.**
I.e. all of these https://discuss.status.im/t/data-sync-next-steps-and-considerations/1056 should be captured in some way or another. As well as all the loose ends/questions posted in https://discuss.status.im/t/data-sync-research-log/1100. This might fork into new future work items.

**8. Peer discovery.**
Capture how who wants to sync finds each other. Consider how encryption plays a role here in terms of visibility.

**9. User-centric, capture client implementers and requirements.**
The user of a data sync layer is the sync client writer, not end user. What do they want to think about and not think about? Capture requirements better.

**10. Separate out Swarm vs IPFS storage and distribution comparison.**
Looking at similarities and differences between these probably more informative, even though there's overlap. Also keeps the comparison scope more tight.
