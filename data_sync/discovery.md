# Discovery

## Bootstrap

In order to facilitate a more decentralized implementation of bootstrapping for a p2p network, we introduce an `on-chain` smart contract containing a list of bootstrapping nodes. This smart contract is permissionless and operates with a stake / slashing model. Ideally we use an optmistic smart contract model<sup>1</sup> for the submission process.
A node is added to the set of nodes except if a challenge was voted valid by `2/3` of the current bootstrap nodes.

## Footnotes
 [1] https://medium.com/@decanus/optimistic-contracts-fb75efa7ca84
