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

## Solution



## Proof-Evaluation-Simulation

## System Model

## Future work

## Conclusion

## References