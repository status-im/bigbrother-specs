# Interfaces

Document that outlines what each layer requires and provides in more detail.

Goal of this document: provide a clear interface for other teams working on other parts of the stack. Specifically: Validity Labs and Nym, working on mixnet designs at transport privacy layer.

## Data Sync clients

TODO: Later

Data sync clients are...

They require... (reliable transmission of data between pairs of peers or groups of peers)

They provide ... (end user functionality like group chat, 1:1 chat, other forms of socio-economic coordination mechanisms)

## Data sync

The data sync layer is responsible for synchronizing data across multiple devices.

It requires a transport layer security protocol that is able to deliver data securely from one device to another. It may also deliver from one device to many other devices. Data can be delayed, lost, reordered or duplicated. The transport layer security protocol is responsible for properties such as confidentiality, integrity, authenticity and forward secrecy. This layer does not care about what underlying transport is used.

It also requires that messages are immutable/idempotent, i.e. they can be referred to by some cryptographic hash.

It provides reliable data synchronization with casual consistency guarantees in a specific group text without relying on centralized services.

(Inspired by https://code.briarproject.org/briar/briar-spec/blob/master/protocols/BSP.md)

## Secure transport

TODO: Connection-oriented vs connectionless, make this more explicit

The secure transport layer is responsible for providing a secure channel between two - or more -
 peers, ensuring confidentiality, integrity, authenticity, forward secrecy and post-compromise security, over a variety of underlying transports.

It requires a transport that can deliver a sequence of bytes from one device to another. These...

Additionally, it requires a long term identity key. 

Looking at MLS specifically, it requires a messaging service that provides:
- a long term identity key
- a broadcast channel for each group which relays messages to it
- a place where participants publish initialization keys and download them

TODO: Is there any difference between initialization keys in MLS and prekeys in DR?

TODO: Are requirements different for Double Ratchet? E.g. see https://github.com/status-im/status.im/blob/52b9e1c5f5d4b35dd4111952c9114f6ebfe83b96/source/research/pfs.md - it doesn't require a broadcast channel. So this can be seen as an enhancement.

It provides confidentiality, authenticity etc.

(Inspired by https://code.briarproject.org/briar/briar-spec/blob/master/protocols/BTP.md)

(Inspired by https://tools.ietf.org/html/draft-barnes-mls-protocol-01)

## Transport privacy

TODO: Later

Transport privacy is ...

It requires... (p2p overlay)

It provides... (metadata protection, etc)

## P2P Overlay

TODO: Later

P2P overlay is...

It requires...(transport TCP/RTC/etc)

It provides...
