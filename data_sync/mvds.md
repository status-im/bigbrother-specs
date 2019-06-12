# Minimum Viable Data Synchronization

**DRAFT VERSION 1.0**

*Written by Oskar Thorén oskar@status.im & Dean Eigenmann dean@status.im*

## Table of Contents

1. [Abstract](#abstract)
2. [Definitions](#definitions)
3. [Wire Protocol](#wire-protocol)
    1. [Payloads](#payloads)
4. [Synchronization](#synchronization)
    1. [State](#state)
    2. [Flow](#flow)
    3. [Retransmission](#retransmission)
5. [Footnotes](#footnotes)

## Abstract

In this specification we describe a minimum viable protocol for data synchronization inspired by the Bramble Synchronization Protocol<sup>1</sup>. This protocol is built on top of the Bramble Synchronization Protocol and was designed in order to ensure reliable messaging between peers communicating across an unreliable peer-to-peer (P2P) network where they may be unreachable or unresponsive. 

We present a functional specification for future implementation<sup>2</sup> as well as reference simulation data which demonstrates its performance.

## Definitions

| Term | Description |
|------|-------------|
| **Peer** | The other nodes that a peer is connected to. |

## Wire Protocol

### Payloads

```protobuf
message Payload {
  Ack ack = 1;
  Offer offer = 2;
  Request request = 3;
  repeated Message messages = 4;
}

message Ack {
  repeated bytes id = 1;
}

message Message {
  bytes group_id = 1;
  int64 timestamp = 2;
  bytes body = 3;
}

message Offer {
  repeated bytes id = 1;
}

message Request {
  repeated bytes id = 1;
}
```

## Synchronization

### State

State is kept for any message of the types `OFFER`, `REQUEST` and `MESSAGE` we do not keep states for `ACK` messages as we do not retransmit those periodically. State is stored per peer. The following information is stored for messages:

 - **Type** - Either `OFFER`, `REQUEST` or `MESSAGE`
 - **Send Count** - The amount of times a message has been sent to a peer.
 - **Send Epoch** - The next epoch at which a message can be sent to a peer.

### Flow

A maximum of one payload is sent to peers per epoch, this payload contains all `ACK`, `OFFER`, `REQUEST` and `MESSAGE` messages for the specific peer. Payloads are created in reaction to messages recieved by peers or new messages being sent by a node. 

Nodes have two modes with which they can send messages, `BATCH` and `INTERACTIVE` mode. The following rules dictate how nodes construct payloads every epoch for any given peer for both modes.

#### Interactive Mode

 - A node initially offers a message, this means an `OFFER` is added to the next payload and the state for the given peer.
 - When a node recieves an `OFFER`, a `REQUEST` is added to the next payload and the state for the given peer. 
 - When a node recieves a `REQUEST` for a previously sent `OFFER`, the `OFFER` is removed from the state and the corresponding `MESSAGE` is added to the next payload and the state for the given peer.
 - When a node receives a `MESSAGE`, the `REQUEST` is removed from the state and an `ACK` is added to the next payload for the given peer.
 - When a node receives an `ACK`, the `MESSAGE` is removed from the state for the given peer.
 - All messages that require retransmission are added to the payload, given `Send Epoch` has been reached.

<p align="center">
    <img src="https://notes.status.im/uploads/upload_4256a743dc961a67446940dd1bd36107.png" />
    <br />
    Figure 1: Message delivery without retransmissions in interactive mode.
</p>

#### Batch Mode

 - When a node sends a `MESSAGE`, it is added to the next payload and the state for the given peer.
 - When a node receives a `MESSAGE`, an `ACK` is added to the next payload for the corresponding peer.
 - When a node receives an `ACK`, the `MESSAGE` is removed from the state for the given peer.
 - All messages that require retransmission are added to the payload, given `Send Epoch` has been reached.

<!-- diagram -->

<!-- Interactions with state, flow chart with retransmissions? -->

### Retransmission

The message of the type `Type` is retransmitted every time `Send Epoch` is smaller than or equal to the current epoch.

`Send Epoch` and `Send Count` are increased every time a message is retransmitted. Although no function is defined on how to increase `Send Epoch`, it should be exponentially increased until reaching an upper bound where it then goes back to a lower epoch in order to prevent message `Send Epoch`'s from becoming too large.

## Footnotes

1. https://code.briarproject.org/briar/briar-spec/blob/master/protocols/BSP.md
2. https://github.com/status-im/mvds
