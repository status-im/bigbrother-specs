# Glossary

**Node**: Some process that is able to store data, do processing and communicate with other nodes.

**Peer**: The other nodes that a peer is connected to.

**Peer-to-peer (P2P)**: Protocols where resources are divided among multiple peers, without the need of central coordination.

**Device**: A node capable of storing some data locally.

**Identity**: A user's cryptographic identity, usually a single or several public keypairs.

**User**: A (human) end-user that may have multiple *devices*, and some form of identity.

**Data replication**: Storing the same piece of data in multiple locations, in order to improve availability and performance.

**Data sync**: Achieving *consistency* among a set of nodes storing data

**Mobile-friendly**: Multiple factors that together make a solution suitable for mobile. These are things such as dealing with *mostly-offline* scenarios, *network churn*, *limited resources* (such as bandwidth, battery, storage or compute).

**Replication object**: Also known as the minimal unit of replication, the thing that we are replicating.

**Friend-to-friend network (F2F)**: A private P2P network where there are only connections between mutually trusted peers.

**Content addressable storage (CAS)**: Storing information such that it can be retrieved by its content, not its location. Commonly performed by the use of cryptographic hash functions.

**Public P2P**: Open network where peers can connect to each other.

**Structured P2P network**: A public p2p network where data placement is related to the network topology. E.g. a DHT.

**Unstructed P2P network**: A public p2p network where peers connect in an ad hoc manner, and a peer only knows its neighbors, not what they have.

**Super-peer P2P network**: A non-pure p2p network, hybrid client/server architecture, where certain peers do more work.

**Private P2P**: Private network where peers must mutually trust each other before connecting, can either be F2F or group-based.

**Group-based P2P network**: A private p2p network where you need some form of access to join, but within group you can connect to new peers, e.g. friend of a friend or a private BT tracker.
