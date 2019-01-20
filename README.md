# dag-documentation
Documentation for DAG protocol

* [Introduction](https://github.com/trust-net/dag-documentation#Introduction)
  * [DLT Stack](https://github.com/trust-net/dag-documentation#DLT-Stack)
  * [Minimal Viable Network](https://github.com/trust-net/dag-documentation#Minimal-Viable-Network)
* [Rules of Engagement](https://github.com/trust-net/dag-documentation#Rules-of-Engagement)
  * [Submitter Sequencing Rule](https://github.com/trust-net/dag-documentation#Submitter-Sequencing-Rule)
    * [Simple Sequencing](https://github.com/trust-net/dag-documentation#Simple-Sequencing)
    * [Sharded Sequencing](https://github.com/trust-net/dag-documentation#Sharded-Sequencing)
  * [Resource Ownership Rule](https://github.com/trust-net/dag-documentation#Resource-Ownership-Rule)
* [DAG Protocol](https://github.com/trust-net/dag-documentation#DAG-Protocol)
  * [Transaction Submission](https://github.com/trust-net/dag-documentation#Transaction-Submission)
  * [Network Transaction](https://github.com/trust-net/dag-documentation#Network-Transaction)
  * [Shard Sync](https://github.com/trust-net/dag-documentation#Shard-Sync)
  * [Submitter Sync](https://github.com/trust-net/dag-documentation#Submitter-sync)
  * [Double Spend Resolution](https://github.com/trust-net/dag-documentation#Double-Spend-Resolution)

# Introduction
tbd

## DLT Stack
tbd

## Minimal Viable Nework
tbd

# Rules of Engagement

## Submitter Sequencing Rule
* Each Transaction will have a Submitter’s ID, Submitter’s Sequence and Submitter’s Last Transaction ID
* A shard cannot have two transactions that have same Submitter ID and Submitter Sequence
* Submitters are responsible to provide correct Sequence and Last Submitter Transaction ID (a.k.a. Submitter Sequencing)
* Submitter sequencing is a strictly monotonically increasing sequence and transaction history link specific to a submitter
* Submitters can maintain a common Submitter Sequencing across all shards (simple sequencing) or can use one for each shard (sharded sequencing)

### Simple Sequencing
![Simple Sequencing][simple-seq]

### Sharded Sequencing
![Sharded Sequencing][sharded-seq]

## Transaction Rules
* A transaction will consist of an “Anchor”, “Payload” and “Signature”
* A submitter will request an Anchor from a node that hosts a shard
* Submitter will provide its next sequence and last transaction id as reference when requesting the Anchor
* Node will provide a signed Anchor as proof of valid submitter request
* Submitter will submit a transaction using node’s anchor and signed payload to the same node that issued the anchor
* Node will validate transaction’s anchor and payload signature and process the transaction

## Resource Ownership Rule
tbd

# DAG protocol
tbd

## Transaction Submission
![Submitted Transaction Processing][submitted-tx]

## Network Transaction
![Network Transaction Processing][network-tx]

## Shard Sync
tbd

## Submitter Sync
tbd

## Double Spend Resolution
tbd


[network-tx]: https://raw.githubusercontent.com/trust-net/dag-documentation/master/images/Network%20Transaction.png "Network Transaction Processing"
[submitted-tx]: https://raw.githubusercontent.com/trust-net/dag-documentation/master/images/Transaction%20Submission.png "Submitted Transaction Processing"
[simple-seq]: https://raw.githubusercontent.com/trust-net/dag-documentation/master/images/SimpleSequencing.png "Simple Sequencing Example"
[sharded-seq]: https://raw.githubusercontent.com/trust-net/dag-documentation/master/images/ShardedSequencing.png "Sharded Sequencing Example"
