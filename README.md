# dag-documentation
Documentation for DAG protocol

* [Introduction](https://github.com/trust-net/dag-documentation#Introduction)
  * [DLT Stack](https://github.com/trust-net/dag-documentation#DLT-Stack)
  * [Minimal Viable Network](https://github.com/trust-net/dag-documentation#Minimal-Viable-Network)
* [Rules of Engagement](https://github.com/trust-net/dag-documentation#Rules-of-Engagement)
  * [Resource Ownership Rules](https://github.com/trust-net/dag-documentation#Resource-Ownership-Rules)
  * [Submitter Sequencing Rules](https://github.com/trust-net/dag-documentation#Submitter-Sequencing-Rules)
    * [Simple Sequencing](https://github.com/trust-net/dag-documentation#Simple-Sequencing)
    * [Sharded Sequencing](https://github.com/trust-net/dag-documentation#Sharded-Sequencing)
  * [Transaction Rules](https://github.com/trust-net/dag-documentation#Transaction-Rules)
  * [Double Spending Rules](https://github.com/trust-net/dag-documentation#Double-Spending-Rules)
    * [Background](https://github.com/trust-net/dag-documentation#Background)
    * [Problem Scenarios](https://github.com/trust-net/dag-documentation#Problem-Scenarios)
    * [Traditional Solutions](https://github.com/trust-net/dag-documentation#Traditional-Solutions)
    * [Trust-Net DAG Solution](https://github.com/trust-net/dag-documentation#Trust-Net-DAG-Solution)
* [DAG Protocol](https://github.com/trust-net/dag-documentation#DAG-Protocol)
  * [Transaction Schema](https://github.com/trust-net/dag-documentation#Transaction-Schema)
  * [Transaction Submission](https://github.com/trust-net/dag-documentation#Transaction-Submission)
  * [Network Transaction](https://github.com/trust-net/dag-documentation#Network-Transaction)
  * [Shard Sync](https://github.com/trust-net/dag-documentation#Shard-Sync)
  * [Submitter Sync](https://github.com/trust-net/dag-documentation#Submitter-sync)
  * [Double Spend Resolution](https://github.com/trust-net/dag-documentation#Double-Spend-Resolution)
    * [Flush or just shard sync?](https://github.com/trust-net/dag-documentation#Flush-or-just-shard-sync)
    * [Which node should flush?](https://github.com/trust-net/dag-documentation#Which-node-should-flush)

# Introduction
tbd

## DLT Stack
tbd

## Minimal Viable Nework
tbd

# Rules of Engagement

## Resource Ownership Rules
* There are 2 types of transactions defined for any application:
  * "outgoing" transactions where some value gets transferred out from assets belonging to a network identity, and
  * "incoming" transactions where some value gets transferred into assets belonging to a network identity.
* Only asset owners can submit "outgoing" transactions against an asset, whereas anyone can submit an "incoming" transaction towards an asset
* Application implementation will be responsible for "access control", i.e., making sure that an outbound value transfer (or any update in general) is only performed by eligible submitter identity
* DLT stack will provide interface to access resources along with their ownership, however stack does not have any visibility into actual content/value for those resources
* Resource scope would be constraints within a shard, so that a faulty application can not access and violate resources beloging to other shards
* DLT implies time consensus -- i.e., value of resources can change over time, however probability of them changing decreases significantly over time
* world state is only updated on a node during transaction processing when there is an app registered. So, world state does not apply to a headless nodes, and no updates to world state after an app has deregistered (since no transaction processing callback to app)


## Submitter Sequencing Rules
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

## Double Spending Rules

### Background
* Let there be a submitter with latest transaction T(N-1) known at peers P1, P2, P3 and P4
* submitter submits double spending transactions with Seq N, lets call them T(N)-1 and T(N)-2, at peers P1 and P2
* P1 accepts T(N)-1 as valid and broadcasts to network
* P2 accepts T(N)-2 as valid and broadcasts to network
* submitter submits T(N+1) on peer P1
* P1 accepts T(N+1) as valid and broadcasts to network
* P4 receives T(N)-2 from P2 (via network) and accepts as valid and broadcasts to network

### Problem Scenarios
* P1 receives T(N)-2 from P2 (via network)
    * P1 already has a later transaction T(N+1)
    * what does consensus protocol decides? (Stale Transaction)
* P2 receives T(N)-1 from peer 1 (via network)
    * P2 already has another transaction for same sequence T(N)-2
    * what does consensus protocol decides? (Duplicate Transaction)
* P3 receives T(N+1) from P1 (via network)
    * P3 does not have the preceding transaction T(N)-1 from submitter
    * what does consensus protocol decides? (Out of Seq Transaction)
* P4 receives T(N+1) from P1 (via network)
    * P4 has T(N)-2 as preceding transaction, that is not parent of T(N+1)
    * what does consensus protocol decides? (Orphan Transaction)

> In all of the above scenarios, rolling back a shard DAG recorded transaction is not an option (there may be other valid submitted transactions from other submitters that are descendants to contentious transaction).

> Also, a system generated compensation transaction (i.e. complementing/cancelling transaction) is not an option (cancelling just this one duplicate transaction without handling the network effect of cascading transactions is incorrect).

### Traditional Solutions
**Q:** _How does Ethereum, or any other traditional blockchain handles these scenarios?_   
**A:** Any traditional blockchain (e.g. trust-net blockchain) can simply “switch” the world state view back and forth between blocks (canonical chain tips) and resolve above situations appropriately. However these solutions do not apply in our case (trust-net DLT) because:
* transaction processing is outside the scope of DLT stack (it’s delegated to application). So, by extension, world state is also not within the scope of DLT stack!
* we are using DAG structure to allow multiple transactions from submitters and shards to record in parallel, hence there is no canonical chain. No canonical chain means no canonical chain tip that every node can agree on and switch to!

### Trust-Net DAG Solution
**Q:** _What are some of the assumptions and facts that can help with solution?_   
**A:** Following facts and assumptions exist in trust-net DAG model:
* each submitter is required to keep track of its latest transaction sequence and transaction hash
* if there are 2 different transactions with same submitter sequence — then this is evidence of intentional double spending by the submitter!
* submitter identities are global and shared at the platform level across all apps/shards
* asset values owned by transaction submitters are application/shard specific


# DAG protocol
tbd

## Transaction Schema
Refer to [transaction schema documentation](https://github.com/trust-net/dag-lib-go/blob/master/docs/Transaction.md) for correct specs as per latest version of the protocol.

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

### Flush or just shard sync?
A double spending means local copy of the shard does not have a transaction that remote peer's shard copy has (double spending == same submitter/seq/shard different transaction). So a double spending event can be seen as a "partitioning" of the shard at the point of double spending. This can lead to a
naive solution using a shard sync to resolve the situation. However, **a shard sync is designed to _"merge"_ local and remote copies of a shard across two nodes, so that both nodes have same DAG. Its not designed to merge conflicting partitions.**

Therefore, during double spending, its important that one of the nodes flush (i.e. completely discard) its shard and then sync with other node. That is the only way to guarantee that no conflicting transactions appear on the shard _and_ the world state stays consistent on both nodes. This also means some of the valid transactions from the flushing node may be lost (temporarily) if they were not on the other node. But, eventually node should sync and receive those transactions.

### Which node should flush?
The algorithm to determine which node must flush its local shard must guarantee that same shard partition will get flushed between any two peers with same pair of double spending transactions. Therefore, using Anchor weights is not correct because different nodes (with same transaction from double spending pair) can have different anchors at different times. Therefore, we'll need to instead use the actual transactions from double spending pair themselves to decide which shard partition must get flushed. Since whichever transaction came into network first should win, we'll use the transaction with "least" weight as the winning transaction -- i.e., node that had that transaction gets to keep its shard, whereas other node needs to flush and sync.


[network-tx]: https://raw.githubusercontent.com/trust-net/dag-documentation/master/images/Network%20Transaction.png "Network Transaction Processing"
[submitted-tx]: https://raw.githubusercontent.com/trust-net/dag-documentation/master/images/Transaction%20Submission.png "Submitted Transaction Processing"
[simple-seq]: https://raw.githubusercontent.com/trust-net/dag-documentation/master/images/SimpleSequencing.png "Simple Sequencing Example"
[sharded-seq]: https://raw.githubusercontent.com/trust-net/dag-documentation/master/images/ShardedSequencing.png "Sharded Sequencing Example"
