# Discovery

## Bootstrap

In order to facilitate a more decentralized (democratic) implementation of bootstrapping for a p2p network, we introduce an `on-chain` smart contract containing a list of bootstrapping nodes. This smart contract is permissionless and operates with a stake / slashing model. Ideally we use an optmistic smart contract model<sup>1</sup> for the submission process.
A node is added to the set of nodes unless a challenge was voted valid by `2/3` of the current bootstrap nodes.

*We consider this approach naive and requiring further investigation.*

## Further Reading
 - [P2P Discovery Comparison](discovery-comparison.md)
 - [Bootstrap Nodes Smart Contract](https://discuss.status.im/t/bootstrap-nodes-smart-contract/1135)

## Footnotes
 [1] https://medium.com/@decanus/optimistic-contracts-fb75efa7ca84
