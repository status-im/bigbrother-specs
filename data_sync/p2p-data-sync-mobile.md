# P2P Data Sync for mobile

## Introduction

How do we achieve user friendly data syncronization in a p2p network for resource restricted devices? In this paper we survey existing work, and propose a solution combining several of the most promising technologies.

In a p2p network you often want to reliably transmit and replicate data across participants. This can either be large files, or small messages that users want to exchange between each other in a private chat. The paticipants can either be human, or machines, or a combination thereof. P2P networks are one type of challenging network that has a few special characteristics: nodes often churn a lot (low uptime) and there might be malicious nodes if you haven't established trust beforehand. There are also considerations such as NAT traversal, which we treat as out of scope for this paper. Doing P2P on mobile and similar resource restricted devices is extra challenging, due to the churn being even more of an issue, as well as having to pay attention to limited bandwidth usage, battery, storage and computational requirements. A lot of these considerations overlap with other types of challenging networks, such as VANETs, DTNs and mesh networks.

In addition to synchronization data, you might also want other types of properties such as: enabling casual consistency for message ordering, privacy-preservation through self-exploding messages, censorship resistance, and so on.

This paper is aimed at introducing one part of a secure messaging stack.

## Related Work

Data synchronization in p2p networks have existed for a while. The most commonly used modern examples are probably Bittorrent and Git. With p2p cryptocurrencies like Bitcoin and Ethereum, we have seen an explosion in the amount of infrastructure being built to solve various use cases. In Ethereum you have projects like IPFS and Swarm, that deal with content storage and distribution, as well as associated protocols with it.

Outside of the cryptocurrency or Web3 community we also have seen many projects building decentralized networks and apps recently with open associated protocols. We have projects like Briar with its Bramble Synchronization protocol, Tribler with its Dispersy Bundle Synchroization, Matrix, Secure Scuttlebutt and a few others.

It's useful to compare these on a few dimensions to understand how they are similar and how they differ. There are many ways to slice a cake, and we'll focus in a few of the more relevant dimensions. Let's first give a brief overview of the main sources of inspiration.

### Server based model

In a server based model things are rather simple. The server is the source of truth, and you do updates synchronously with it. So I have a device, make a request to some server and send update. This means it is easy to guarantee otherwise impossible attributes, such as sequential consistency. This comes at the cost of centralization, with drawbacks such as availability issues (whether deliberate, by accident, by service provider or by third party), censorship, lack of ownership and control of your data, and surveillance. We mention it here only to provide contrast with the other models.

### Git

Git is a decentralized version control system. It was built out of the needs of Linux kernel development, which is inherently distributed and with many sources of truth that needs to be reconciled. Essentially a git repository is a DAG, and then you create branches and can pull down other remote references and merge them with yours. In case of different views, you either need to rebase (changing you own dependency tree), or create a merge commit. While you could use Git for other things than just code, it isn't what it is designed for. Often commits operate on the same single piece of data (codebase), which leads to things like merge conflicts etc. So while the commits themselves are immutable, the thing you are manipulating is not, and it often leads to conflicts and so on.

### Bittorrent

Bittorrent has quite a few parts to it, and it can acts as a private group-based network or as an open structured network. A common mode of operation is having a tracker, then getting other nodes that you share chunks of a single static file.

One interesting aspect of Bittorrent is that it has a form of incentive mechainsm that's tit for tat. Essentially to encourage pairwise contributions within the game that is sharing a single static torrent file. This game doesn't extend outside of sharing a single file, which has lead to private trackers using things like seed ratio as a reputation system in a centralized fashion, leading to exposure to censorship and coercion.

For its structured network, most Bittorrent implementations leverage a DHT of nodes, that allow you to figure out who is sharing a piece of data. It is worth noting that the object being shared is static, a torrent file once uploaded doesn't change, and the integrity/consistency of it is communicated in the form of a checksum that's usually communicated in the initial (centrally distributed) download.

### Matrix

Matrix is a tool for various forms of group chat, including on mobile. Matrix is in its current mode of operation operating a hybrid architecture. It is neither fully p2p nor client-server, instead it uses a federated mode or a super-peer architecture. As a user, you connect to a homeserver, which then syncs data with other known homeservers. While there are plans to move to a p2p architecture where each node is running its own homeserver, there appears to be some unsolved issues, similar to the ones noted in other protocols.

Matrix distinguishes between syncable messages and ephemeral messages, which allows it to have non-heavy typing notifications and so on. They also have two DAGs, where one is used for reconciling auth events to ensure people who are banned from a chat room can't easily just rejoin as unbanned. Since it uses a DAG, it also means a chat context can be offline, such as in a submarine, for an extended period of time, then come online and sync up with the rest of the world. Because you need to connect to a homeserver to sync, it requires the user to be online or have an active connection with a homeserver.

The super-peer archicture means there's only a simple TCP connection and no high availablity requirements for end users, which leads to a good user experience for mobile devices. However, it isn't pure peer to peer, and suffer many of the same centralizing aspects as other server based approaches, including security concerns, censorship resistance, and so on.

### Secure Scuttlebutt (SSB)

SSB is a form of group-based private p2p network based on trusted relationsihps append-only feeds. These feeds are single-master and offline/local by default. You can follow a feed to get updates from it, and  and by following a feed, and being followed, you become aware of more feeds. Essentially you see content 1-hop away, fetch in a cache content 2 hops away, and you are aware of 3 hop. The way bootstrapping works is that you connect to a known Pub server, which follows you back. By following you back, you become aware of the connection that pub has, and so on, in concentric circles, like a social graph owned by you. Feeds are unencrypted by default, and the way messaging works is that you post to your own feed with a message encrypted by the other person's public key. It's then a client detail how to make this into a good user interface.

### Bramble Synchronization Protocol (BSP)

BSP is used in Briar, which is a messaging app for mobile devices. It is based on a friend-to-friend network, so it leverages trusted connections. It uses a secure transport protocol to transfer data from one device to another securely. While BSP is made for mobile devices, it assumes devices are largely online. This has battery concerns, and also means some difficulty in the real-world for running on iPhone devices.

BSP works by keeping a log for each group context, and you replicate data asynchronously. Depending on the specific settings, you then offer to send or send messages to other peers that you are sharing with. If they receive it, they acknowledge this to you, and if they don't the sender resends. Each peer keeps track of the state of other nodes to know when to resend messages.

Because it makes no special network assumptions, it works just as well over sneakernet (USB stick).

Two drawbacks off this approach are:
- high churn is problematic, as can be verified in a simulation / back of the envelope calculation
- large group chats over a multicast network lead to a lot of overhead due to each message being pairwise

### Dispersy Bundle Synchronization (DBS)

Dispersy is a form of anti-entropy sync, which uses bloom filters to advertise locally available bundles to random nodes in the network. The receiving node then tests this bloom filter against their own locally available bundles, and requests missing messages. As each random interaction steps, we get closer and closer to a consistent world view among all nodes.

This approach is very bandwidth efficient, as Bloom Filters are very space efficient way of conveying probabilistic information, and the effects of false positives can be mitigated. One problem with using Bloom Filters is that if you have a lot of elements your false positive rate can shoot up quite quickly. To mitigate this, DBS uses a subset of elements approach. Essentially they cut up the locally available bundles in clever ways, which allows the false positive rate and the bloom filter size stay the same. The latter is important, as for challenging networks you want to keep the payload limited to allow for NAT puncturing with UDP, and too big of a payload leads to fragmentation.

DBS is used in Tribler, where it's used for their megacaches, which essentially contains various forms of social information (peers similar, videos liked, etc). It's important to note that this doesn't have to be 100% up to date to be useful information. There are two main downsides of this approach as far as the author can tell:

- it is probabilistic and doesn't aim to be used for guaranteed message ordering (i.e. casual consistency)
- you sync data with all peers, including potentially untrusted peers

### Swarm Feeds and chunks

Swarm is one of the legs of Web3 in Ethereum world. There are many related protocols to Swarm, such as access control, PSS for routing, incentivization, etc. It's essentially storage for the so called world computer. Swarm spreads data into chunks that are spread onto all the nodes in the network. It's leveraging a form of DHT, a structured network, where the data placement is related to the address of the content. This means individual nodes don't have a say in what they store, which is a form of censorship resistance, as no one is really in charge of data storage.

All data is immutable in Swarm by default, so to deal with mutability Swarm has Feeds. This is a mutable resource, similar to a pointer or DNS. It essentially acts as a replicated log. This is useful for smartphones, since you can replicate your log to the network, and thus get close to 100% available guarantees. There are a few downsides to this approach:

- Your log is by default accessible by anyone
- The receiver need to actively go fetch your log to sync
- In a group context, there might be a lot of feeds to sync

These can be mitigated to various degrees by using things like PSS, topic negotiation, and so on. There are trade-offs to all of these though.

## Problem motivation

Why do we want to do p2p sync for mobilephones in the first place? There's three components to that question. One is on the valuation of decentralization and peer-to-peer, the second is on why we'd want to reliably sync data at all, and finally why mobilephones and other resource restricted devices.

For decentralization and p2p, there are various reasons, both technical and social/philosophical in nature. Technically, having a user run network means it can scale with the number of users. Data locality is also improved if you query data that's close to you, similar to distributed CDNs. The throughput is also improved if there are more places to get data from.

Socially and philosophically, there are several ways to think about it. Open and decentralized networks also relate to the idea of open standards, i.e. compare AOL with IRC or Bittorrent and the longevity. One is run by a company and is shutdown as it stops being profitable, the others live on. Additionally increasingly control of data and infrastructure is becoming a liability [xx]. By having a network with no one in control, everyone is. It's ultimately a form of democratization, more similar to organic social structures pre Big Internet companies. This leads to properties such as censorship resistance and coercion resistance, where we limit the impact a 3rd party might have a voluntary interaction between individuals or a group of people. Examples of this are plentiful in the world of Facebook, Youtube and Twitter.

For reliably syncing data at all, it's often a requirement for many problem domains. You don't get this by default in a p2p world, as it is extra unreliable with nodes permissionslessly join and leave the network. In some cases you can get away with only ephemeral data, but usually you want some kind of guarantees. This is a must for reliable group chat experience, for example, where messages are expected to arrive in a timely fashion and in some reasonable order. The same is true for messages there represent financial transactions, and so on.

Why mobilephones? We live in an increasingly mobile world, and most devices people use daily are mobile phones. It's important to provide the same or at least similar guarantees to more traditional p2p nodes that might run on a desktop computer or computer. The alternative is to rely on gateways, which shares many of the drawbacks of centralized control and prone to censorship, control and surveillence.

More generally, resource restricted devices can differ in their capabilities. One example is smartphones, but others are: desktop, routers, Raspberry PIs, POS systems, and so on. The number and diversity of devices are exploding, and it's useful to be able to leverage this for various types of infrastructure. The alternative is to centralize on big cloud providers, which also lends itself to lack of democratization and censorship, etc.

### Minimal Requirements

In terms of minimal requirements or design goals for a solution we propose the following.

1. MUST sync data reliably between devices.
By reliably we mean having the ability to deal with messages being out of order, dropped, duplicated, or delayed.

2. MUST NOT rely on any centralized services for reliability.
By centralized services we mean any single point of failure that isn’t one of the endpoint devices.

3. MUST allow for mobile-friendly usage.
By mobile-friendly we mean devices that are resource restricted, mostly-offline and often changing network.

4. MAY use helper services in order to be more mobile-friendly.
Examples of helper services are decentralized file storage solutions such as IPFS and Swarm. These help with availability and latency of data for mostly-offline devices.

5. MUST have the ability to provide casual consistency.
By casual consistency we mean the commonly accepted definition in distributed systems literature. This means messages that are casually related can achieve a partial ordering.

6. MUST support ephemeral messages that don’t need replication.
That is, allow for messages that don’t need to be reliabily transmitted but still needs to be transmitted between devices.

7. MUST allow for privacy-preserving messages and extreme data loss.
By privacy-preserving we mean things such as exploding messages (self-destructing messages). By extreme data loss we mean the ability for two trusted devices to recover from a, deliberate or accidental, removal of data.

8. MUST be agnostic to whatever transport it is running on.
It should not rely on specific semantics of the transport it is running on, nor be tightly coupled with it. This means a transport can be swapped out without loss of reliability between devices.

We've already expanded on the rationale for 1-3. Let's briefly expand on the need for the other requirements. For 4, the reality is that mobile devices are largely offline, and you need somewhere to get data from. To get reasonable latency, this requires some additional form of helper service. It also ties into 3, which is to allow for a good UX on mobile, which means reliable and reasonable latency.

5 is a must to at least be able provide casual ordering between events, though it is up to clients of the protocol to use this or no. 6 is useful for non important messages, such as typing notifications, discovery beacons, and so on. The alternative here is to use a separate protocol, which would make this data sync layer less pluggable and general for various types of transports. It is similar to TCP and UDP running on IP. This also ties into 8, where the protocol should be able to use any underlying transport from device to device.

For 7, this is often a desirable feature for secure messaging applications. The idea is that you want to do exploding messages, or maybe wipe your device if you are afraid of local seizure [xx]. This is an explicit requirement as otherwise you get into hairy solutions when for example repairing a DAG.

## Solution / System Model

We propose a protocol that's heavily inspired by BSP, with some tweaks, some minor and some major. Let's first introduce BSP in more detail as specified before diving into enhancements.

We synchronize messages between devices. Each device has a set of peers, of which it chooses a subset to which it wants to synchronize messages with. Each synchronization happens within a data group, and in a data group there's a set of immutable messages. Each message has a message id that identifies it.

There are the following message types: OFFER, ACK, SEND, REQUEST. ACK acknowledges a set of message IDs, whereas REQUEST requsests it. OFFER offers a set of message ids without sending them, whereas SEND sends the actual messages. Depending on their sharing policy, a client can choose to OFFER messages or SEND messages first, depending on if latency (roundtrips) or bandwidth (payloads) is at a premium.

**Tweak 1:** For payloads we propose using Protobuf to allow for upgradability and implementation in multiple languages. For wire protocol we use a slightly different one, see next major section for this.

**Tweak 2:** One difference from BSP is that we allow multiple messages types to occur in one packet, so you can ACK messages at the same time as you OFFER messages. This is to allow for fewer roundtrips, as well as piggybacking ACKs. This is useful if a node might not want to disclose that they received a message straight away.

A device keeps track of its peers that it is synchronizing data with. When a device sends a message, it starts by appending it locally to its own log. It then tries to OFFER or SEND messages to each of the peers it is sharing with. This is operating on a positive retransmission with ACKs basis, simlar to e.g. TCP/IP. It uses exponential backoff and keeps track of send count to ensure it doesn't send too many messages to an offline node.

A receiving node might receive messages out of order. Inside the payload, a useful thing to have is a set of parent IDs. These are message dependencies that can be requested specifically before delivering the message. So a node knows what data it is missing, and can request it, by default from the sending node but additionally in other ways. Only once all message dependencies have been met does the message actually get delivered, as some order has been guaranteed. This is useful when casual consistency is needed, but isn't a strict requirement and it might be domain dependent.

### Enhancement 1: Leveraging multicast

Another difference with BSP is that we allow for leveraging multicast protocols. In these you might have a topic construct, which you know multiple receivers are listening to. If you are sending on a topic that you know a set of participants are listening to, this means a client counts this as a send to those participants. If you have 100 participant on a topic, this means a 100x factor improvement in bandwidth usage. A mapping is kept between known participants and a topic to facilitate this.

As a specific example, if you have a group chat with 100 participants, you can ACK a set of messages. This means you publish this ACK (message ids) to a topic. As a result, all participants who are able to read this message knows you recevied it. Additionally, they are now aware that this is a message id that exists, and may choose to request it. However, this still has the drawback that each node sends ACKs to the same topic, which can lead to undesirable bandwidth consumption. This points to using ACKs less aggressively and leveraging more randomized and optimized approaches. These additional enhancements in case of large group chats are possibly, see below for more radical changes.

This is an optional feature, as it isn't guaranteed the listeners of a topic are the intendendent recipients, which is a somewhat weaker assumption than friend to friend with mandatory trust establishment.

### Enhancement 2: Swarm for replicated log

One drawback mentioned earlier is that high churn is problematic. As a simple example, we can imagine a mobile node only being online 10% of the time. This means 9/10 sends will be failures, and this will cascade with failing to deliver ACKs, leading to more resends, and so on. Additionally, with exponential backoff and possibly offline devices at non-overlapping times, the latency might be unacceptable.

The fundamental issue here is one of low-availability. How do we solve this without reverting back to centralized solutions? One such approach is to replicate the log remotely. Similar to what is done in Git, but ideally in a 'no one owns this way', which points to IPFS and Swarm. This enables an interesting property, where you can essentially upload and forget, and, assuming incentives around storage policy and so on work out, you can be reasonably sure the data is still there for you even if you lose your local copy.

A replicated log can exist on Swarm and IPFS, assuming the message ids map to references used there. What Swarm Feeds provide is a way to show where the last message is, and doing so without requiring individual nodes to send or offer a message to that peer individually. This means as long you know which log to pull from, you can query it at some interval. Additionally, Swarm Feeds uses an adaptive algorithm that allows you to seamlessly communicate the expected update frequency.

One downside of this is that it requires Internet access and a specific overlay network, currently devp2p. This is an optional enhancement, and there are many ways to replicate remote logs, the important thing is that the other node is aware of it.

The way it'd work is as follows. A node commits a message to their local log, then they sync it to some decentralized file storage, such as Swarm or IPFS. This information is further confirmation that the data has been replicated and acts as a form of ACK where Swarm is seen as a node. Recipients then know, through previous communication with sender node, that they can look at this log. This means the two nodes don't need to be both online, and they can leverage Swarm, or other similar solutions, with less latency. Another way of looking at it is as chunks providing a linked list. Even if a similar solution lacks a feeds-like mechanism (e.g. like ENS or DNS), it can still be used for chunk storage, even though latency might be slightly higher.

Let's decompose the above into orthogonal concerns.

#### Content addressed storage (CAS)

Swarm with its chunks and references is a content addressed storage. This allows us to get and put data there, with various properties. There are also other content addressed storages, such as IPFS and local node's cache. What's important is that there's a way to refer to messages, and that this is based on a cryptographic hash function. This means we have two properties: collision resistance and preimage resistance. This allows us to uniquely refer to a piece of content, and it can be leveraged by clients in term of ordering of events. It also allows us to check the data integrity of a specific message.

A desirable property here is to be as agnostic as possible to the specific infrastructure used. How can we refer to the same piece of content through various stores? This is an open question, but tentatively multihashes and multicodec can be used for this.

#### Update pointers

In the above section, we suggest using Feeds. However, this is just one alternative. Other alternatives are ENS, DNS, etc. We can also use endpoint communicating the latest update through a push-message. It's desirable that this notion is kept abstract.

What it does is to (a) allow you to see last updated data and ideally (b) reference previous state, so you can fetch it via CAS, or at least be aware that it's missing.

### Enhancement 3: Dispersy Bloom Filters

Another drawback mentioned in earlier section is that large group data sync contexts over multicast network lead to a lot of overhead due to each message being pairwise. By leveraging the multicast capability when it exists we mitigate this somewhat. However, a simple back of the envelope calculation shows that this might still be too much. If you have 100 nodes in a group context, a node sending a message over a topic results in 100 naive ACKs over topic. These can clumped together and so on but it still might be an unacceptable bandwidth trade off. Without relying on centralized intermediaries, this leads to ACKs being relaxed somewhat, while still ensuring we keep reliablity. One way of doing this is by using some of the ideas in DSP.

By using bloom filters and repeat random interactions with other nodes in a data sync context to offer and acknowledge messages, we'll eventually reach a form of steady state, as this is a form of anti-entropy sync.

We can extend the message types as follows. In large group contexts, OFFER and ACK, all messages that have a set of message ids, can be replaced with BLOOM_AVAILABLE and some additional subset description. This conveys the same information as OFFER and ACK but probabilistically, in the following way. If we have a data sync context with 100 nodes, one node A has some messages locally available, and it connects to a random node B. A then sends a bloom filter along with some additional information. B then compares each locally available message (for some range) with the bloom filter sent. It gets a response 'maybe' or 'not' in set, and false positives here are fine. If it is not in set, it knows A is missing these messages, and in return it sends a specific REQUEST for these messages, along with its own Bloom filter. B then does vice versa. Messages that are in the 'maybe' category can be seen as probabilistic ACK, and no further action is needed from the receiving node. This information can be discarded, or it can be used as probabilistic information to inform things like future send time.

#### Subset description
How is the subset described? What's most important is that there's a way to divide up a set of messages into some equally sized buckets. This is important to ensure the bloom filter length and false positive rate stays the same. A good way of doing this is using Lamport timestamps. These are non-unique but monotonic, and they roughly evenly divide up the space of messages. I.e. we can select a range 0..1000 and communicate this in the BLOOM_AVAILABLE message, and the recipent knows which subset of messages to look for. What happens if there's a byzantine node that sends messages with the same timestamp? In that case, the sending node would select a smaller subset partition range. The universe size for the bloom filter is determined by sender, and it simply hints at the receiver where it should look in its own store.

We don't just want to arbitrarily divide the space, instead we want to make sure we send a request that's most likely to get the information we need. Similar to DSP, depending on if the node has almost caught up or is just starting, different heuristics can be used. For example, if a node just joined, it can use a modulo heuristic. This way a field in BLOOM_AVAILABLE would also be modulo and offset. As an example, if you expect there to be roughly 1000 messages, you might select modulo 10. This would then divide this up the space into 10 pieces, which can be requested in parallel. I.e. 1st, 11th, 21th message, and so on. This approximates a linear download, without requiring any state. For more recent messages, a pivot heuristic that biases recent messages is desirable, that probabilistically picks messages that are more recent. For further analysis of this, we refer to the DSP paper.

#### Sync scope and location of subset description
A brief experimental note on sync scope and Lamport clocks. In some cases we might want untrusted nodes so not use information inside payload. One thing worth noticing here is where we want the global time partial ordering Lamport clocks to be stored. By default, they will be inside a data sync context payload. There may be circumstances were it is desirable to expand this. As a concrete example: imagine you have several one on one chats, each their own data sync context, since the chats are encrypted. If these devices are mostly offline, and you don't leverage remote replicated logs, it may be desirable to let other nodes help you sync these messages for you. This is similar to having an encrypted torrent file, but still having a way to refer to roughly ordered chunks, so other nodes can seed it. It may be that it's desirable to have nested sync contexts, where participants can choose to let other nodes help them sync in order to reduce latency. By doing so, another level of Lamport clocks can be added in the header, and the sync scope ID can also be used to divie up the space. This design is experimental, and belongs in future work.

Notice that this general approach might still have to be complemented with REQUESTs. If we want casual consistency, it's desirable to have a way to refer to specific message ids to fetch these messages.

Also note that this would work without a specific structured overlay on top of Internet, for example over a mesh network.

## Specification and wire protocol

What follows is the actual specification and wire protocol. Note that this is the Minimal Version of Data Sync (MVDS). Both of these are in draft mode and currently being tested. They don't currently implement any of the enhancements listed above.

## Specification

*We expect implementations of MVDS to be generic, allowing for the underlying transport to be easily replaced*

## Types

### Custom Types

| Type        | Equivilant | Description                                               |
|-------------|------------|-----------------------------------------------------------|
| `MessageID` | `bytes32`  | 32 bytes of binary data obtained when hashing the message |

### Message Types

We define `messages` as packets sent and recieved by MVDS nodes. They have been taken from the BSP spec.

#### `ACK`

@TODO: DESCRIBE

```python
{
    'ids': ['MessageID']
}
```

#### `OFFER`

@TODO: DESCRIBE

```python
{
    'id': ['MessageID']
}
```

#### `REQUEST`

@TODO: DESCRIBE

```python
{
    'id': ['MessageID']
}
```

#### `MESSAGE`

@TODO: DESCRIBE

```python
{
    'group_id': 'bytes32',
    'timestamp': 'int64',
    'body': 'bytes'
}
```

### Wire protocol

### Example Clients

## Proof-Evaluation-Simulation

## Future work

## Conclusion

## References