# dag-documentation
Documentation for DAG protocol

* [Introduction](#Introduction)
  * [Yet Another Protocol?](#Yet-Another-Protocol)
  * [DLT Stack](#DLT-Stack)
  * [Minimal Viable Network](#Minimal-Viable-Network)
* [Separation of Concerns](#Separation-of-Concerns)
  * [Correctness](#Correctness)
  * [Fault Tolerance](#Fault-Tolerance)
* [Rules of Engagement](#Rules-of-Engagement)
  * [Resource Ownership Rules](#Resource-Ownership-Rules)
  * [Submitter Sequencing Rules](#Submitter-Sequencing-Rules)
    * [Simple Sequencing](#Simple-Sequencing)
    * [Sharded Sequencing](#Sharded-Sequencing)
  * [Transaction Rules](#Transaction-Rules)
  * [Double Spending Rules](#Double-Spending-Rules)
    * [Background](#Background)
    * [Problem Scenarios](#Problem-Scenarios)
    * [Traditional Solutions](#Traditional-Solutions)
    * [Trust-Net DAG Solution](#Trust-Net-DAG-Solution)
* [DAG Protocol](#DAG-Protocol)
  * [Transaction Schema](#Transaction-Schema)
  * [Transaction Submission](#Transaction-Submission)
  * [Network Transaction](#Network-Transaction)
  * [Headless Node](#Headless-Node)
  * [Shard Sync](#Shard-Sync)
    * [Shard Parent Sync](#Shard-Parent-Sync)
    * [Shard Uncles Sync](#Shard-Uncles-Sync)
  * [Submitter Sync](#Submitter-sync)
  * [Double Spend Resolution](#Double-Spend-Resolution)
    * [Flush or just shard sync?](#Flush-or-just-shard-sync)
    * [Which node should flush?](#Which-node-should-flush)

# Introduction
DAG protocol is a middleware network protocol geared towards building enterprise applications with DLT capabilities.

## Yet Another Protocol?
With interoperability and fragmentation of protocols being a major challenge in blockchain/DLT adoption, its only fair to question the rationale for designing a new protocol. Trust-Net's DAG protocol is designed to solve one main problem behind existing protocols -- the "Minimal Viable Network" challenge of enterprise applications.

This document intends to explain design philosophy and rationale behind Trust-Net's DAG protocol. However, to summarize, here are some key differences between Trust-Net and Ethereum-like protocols used for enterprise applications:

|Feature|TrustNet|Ethereum|
|----|----|----|
|*Objective*|Build DLT capability in native applications|Smart contracts based DApps|
|*Datastructures*|Weaved DAGs|Canonical Chain + MPT|
|*Transaction Orderer*|Submitter (self ordered)|Block producing node|
|*Consensus Model*|Scope/Access constraints based|PoW/PoS based|
|*Incentive Model*|Ammortization|Economic|
|*Privacy Model*|Strong privacy, application level encryption|Public/non-private transactions|
|*Application Model*|Native (full control) apps|EVM bytecode based DApp|
|*Stack Model*|Stack as a library, application agnostic|Stack as the controller for smart contract apps|

> We are only comparing against protocols that have symmetric nodes, i.e. they do not use a "privileged" node or co-ordinator based solution for bypassing throughput limitations. Intention here is to keep the network model independent of any "co-ordinator" or any other special purpose nodes that are typically used to finalize transactions on non traditional network models. Such a choice results in weaker network security by assigning critical role to a limited number of "privileged" nodes. With Trust-Net, all nodes in the network are equally capable and have equal role in the protocol security.

## DLT Stack
Key distinction and reasoning behind Trust-Net protocol is to invert the role of protocol in an application. Unlike the traditional blockchain protocols where stack is the controller for light weight applications, Trust-Net is designed to build DLT capability into traditional enterprise applications. This means, ability to instantiate and use DLT as a Stack into application, just like application instantiates any other protocol stack for its needs (e.g. HTTP, SIP, ...).

## Minimal Viable Network
Due to the protocol stack as the controller for application in traditional blockchains, its not possible for network to be agnostic to application logic. Therefore, all typical enterprise blockchain applications use a private network within a consortium. This model has inherent problem of "minimal viable network" -- i.e. weaked security due to smaller network size.

> Private/consortium networks with traditional blockchain/DLT protocols are analogous to each web application building it's own "internet" from scratch!!!

Just like a common internet (with its middle layer protocol suites) supports all different web applications with their independent security needs -- similarly "Trust-Net" is intended to be a common network for different DLT capable applications and use cases. Protocol is agnostic to applications, and hence a shared network by different applications results in "ammortization" of network security.

# Separation of Concerns
One key philosphy behind Trust-Net's DAG protocol is to separate out an application's correctness from network security. Since network is application agnostic, not all nodes in the network process an application's transaction. Only the nodes where an application's instance is registered will be the one where application's transaction will be processed (and hence correctness of those transaction established). However, independent of the transaction processing, all nodes in the network participate on network security ("fault tolerance") by validating transaction ordering, non-alterability and ownership.

## Correctness
**Q:**    Does this means consensus security (correctness) is weak in this network?   
**A:**    True that the number of nodes at which transaction is processed is less than the total number of nodes in the network. Hence, strength of consensus security is bounded by the number of application nodes (an application node is a node that is part of the application’s shard). This limitation (consensus security == application node count) does not changes. However, our protocol separates “consensus” from “non-alterability” — which means that the strength of “non-alterability” security is based on number of network nodes, regardless of the number of application nodes. Hence, overall our network provides better security, due to added “non-alterability” to any application node.

## Fault Tolerance
**Q:**    What is the fault-tolerance of the protocol?  
**A:**    Let N be the number of network nodes, and let A be the number of application nodes, for an application “A” on a network “N”. Hence,
* “non-alterability” = f(N)
* “consensus” = f(A)
* number of application nodes is significantly less than network nodes (i.e. A << N)

Lets assume attacker has exact knowledge of those “A” application nodes, and is able to implement a 51% attack, i.e. (A/2 + 1) number of application nodes were compromised at both application and the network layer. What this means is:
* “correctness” would be compromised (since majority of application nodes deviate from correct consensus)
* “non-alterability” will NOT be compromised, because network will not allow transaction sequence alterations to propagate (A << N)
* above means, network will ensure that any 51% attack will fail — because network will have “non-alterable” transaction history that can be used to challenge and refute the attack on application nodes
* additionally, if sharding layer maintains application specific transaction history — then that history can be used to recover from the 51% attack, by simply replaying the application's transaction history from any good node

> A detailed analysis of DAG protocol's properties and assumptions is described at [DAG Properties Analysis](./DAG-Properties-Analysis.md#Properties).

# Rules of Engagement

## Resource Ownership Rules
* A resource is a named asset with specific owner and value, within the scope of a logical application on Trust-Net
* There are 2 types of operation categories in any transaction for any application:
  * "outgoing" operation where some value gets transferred out from resources owned by a network identity, and
  * "incoming" operation where some value gets transferred into resources belonging to a network identity
* In any transaction, sum of "outgoing" values must be equal to sum of "incoming" values (preservation of total value)
* A transaction that has an "outgoing" operation against a resource can only be submitted by the resource owner
* A transaction that has an "incoming" operation against a resource can be submit by anyone
* A resource's scope would be limited to an individual shard (i.e. all instances of an application), so that a faulty implementation of an application can not access and violate resources belonging to other shards
* Application implementation will be responsible for "access control", i.e., making sure that an outbound value transfer (or any update in general) is only performed by eligible resource owner
* DLT stack will provide interface for accessing resources in the application's scope, however stack does not have any visibility into actual content/value of those resources
* DLT implies time consensus -- i.e., value of resources can change over time -- however, probability of them changing decreases significantly over time


## Submitter Sequencing Rules
* Each Transaction will have a Submitter’s ID, Submitter’s Current Sequence and Submitter’s Last Transaction ID (a.k.a. Submitter Sequence)
* Submitters are responsible for tracking and providing correct values for submitter's last transaction ID and current transaction sequence
* Submitter sequencing is a strictly monotonically increasing sequence and submitter's last transaction ID links to a transaction history specific to the submitter
* A shard cannot have two transactions from same Submitter with same value for Submitter's Sequence
* Submitters can maintain a common Submitter Sequencing across all shards (simple sequencing) or can use separate one for each shard (sharded sequencing)

### Simple Sequencing
![Simple Sequencing][simple-seq]

### Sharded Sequencing
![Sharded Sequencing][sharded-seq]

## Transaction Rules
* A transaction will consist of an “Anchor”, “Payload” and “Signature”
* A submitter will request an Anchor from a node that hosts a shard for which transaction is intended
* Submitter will provide its next sequence and last transaction id as reference when requesting the Anchor
* Shard hosting node will provide a signed Anchor as proof of valid submitter request
* Submitter will submit a transaction using node’s anchor and signed payload to the same node that issued the anchor
* Node will validate transaction’s anchor and payload signature and process the transaction

## Double Spending Rules

### Background
* Let there be a submitter with latest transaction T(N-1) known at peers P1, P2, P3 and P4
* submitter submits double spending transactions with Seq N at peers P1 and P2, lets call these double spending transactions as T(N)-1 and T(N)-2, 
* P1 accepts T(N)-1 as valid and broadcasts to network
* P2 accepts T(N)-2 as valid and broadcasts to network
* submitter submits T(N+1) on peer P1
* P1 accepts T(N+1) as valid and broadcasts to network
* P4 receives T(N)-2 from P2 (via network) and accepts as valid and broadcasts to network

### Problem Scenarios
* P1 receives T(N)-2 from P2 (via network)
    * P1 already has a later transaction T(N+1)
    * How does DAG protocol handle this stale n/w transaction?
* P2 receives T(N)-1 from peer 1 (via network)
    * P2 already has another transaction for same sequence T(N)-2
    * How does DAG protocol handle this duplicate n/w transaction?
* P3 receives T(N+1) from P1 (via network)
    * P3 does not have the preceding transaction T(N)-1 from submitter
    * How does DAG protocol handle this out of seq n/w transaction?
* P4 receives T(N+1) from P1 (via network)
    * P4 has T(N)-2 as preceding transaction, that is not parent of T(N+1)
    * How does DAG protocol hanlde this orphan n/w transaction?

> In all of the above scenarios, rolling back a shard DAG recorded transaction is not an option (there may be other valid submitted transactions from other submitters that are descendants to contentious transaction).

> Also, a system generated compensation transaction (i.e. complementing/cancelling transaction) is not an option (cancelling just this one duplicate transaction without handling the network effect of cascading transactions is incorrect).

### Traditional Solutions
**Q:** _How does Ethereum, or any other traditional blockchain handles these scenarios?_   
**A:** Any traditional blockchain can simply “switch” the world state view back and forth between blocks (canonical chain tips) and resolve above situations appropriately. However these solutions do not apply in our case (trust-net DLT) because:
* transaction processing is outside the scope of DLT stack (it’s delegated to application). So, by extension, world state is also not within the scope of DLT stack!
* we are using DAG structure to allow multiple transactions from submitters and shards to record in parallel, hence there is no canonical chain. No canonical chain means no canonical chain tip that every node can agree on and switch to!

### Trust-Net DAG Solution
**Q:** _What are some of the assumptions and facts that can help with solution?_   
**A:** Following facts and assumptions exist in trust-net DAG model:
* each submitter is required to keep track of its latest transaction sequence and transaction hash
* there cannot be two transaction's from same submitter with same transaction sequence within a shard
* when a new Anchor is issued for a transaction, it coalesces all the tips of shard's DAG, effectively making all transactions processed so far by the shard as ancestor
* any transaction that uses an "incoming" value from a previous transaction is also a descendent of that transaction in the shard's DAG (by virtue of shard's tips coalescing)
* only the resource owner can submit a transaction with "outgoing" operation for that resource

With above constraints, DAG protocol gaurantees that:
* all transactions that depend on an "incoming" operation from a previous transaction will only be processed after its ancestors have been processed (see [Issue #47](https://github.com/trust-net/dag-lib-go/issues/47) and [Issue #51](https://github.com/trust-net/dag-lib-go/issues/51))
* all transactions with an "outgoing" operation that deducts value from a resource are processed in same sequential order across all application nodes in the network (by virtue of resource ownership and submitter sequencing rules)
* if there are 2 different transactions with same submitter sequence — then this is evidence of intentional double spending by the submitter and will be rejected by all honest application nodes

# DAG protocol
tbd

## Transaction Schema
Refer to [transaction schema documentation](https://github.com/trust-net/dag-lib-go/blob/master/docs/Transaction.md#Contents) for correct specs as per latest version of the protocol.

## Transaction Submission
![Submitted Transaction Processing][submitted-tx]

## Network Transaction
![Network Transaction Processing][network-tx]

## Headless Node
A headless node is one that does not have any shard registered/hosted. Also, transactions that do not belong to the registered/hosted shard are also handled like a headless node.

![Headless Transaction Processing][headless-tx]

## Shard Sync
tbd

### Shard Parent Sync
tbd

### Shard Uncles Sync
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
The algorithm to determine which node must flush its local shard must guarantee that same shard partition will get flushed between any two peers with same pair of double spending transactions. Therefore, using Anchor weights is not correct because different nodes (with same double spending transactions pair) can have different anchors at different times. Therefore, we'll need to instead use the actual transactions from double spending pair themselves to decide which shard partition must get flushed. Since whichever transaction came into network first should win, we'll use the transaction with "least" weight as the winning transaction -- i.e., node that had that transaction gets to keep its shard, whereas other node needs to flush and sync its shard.


[headless-tx]: https://raw.githubusercontent.com/trust-net/dag-documentation/master/images/Headless%20Node.png "Headless Transaction Processing"
[network-tx]: https://raw.githubusercontent.com/trust-net/dag-documentation/master/images/Network%20Transaction.png "Network Transaction Processing"
[submitted-tx]: https://raw.githubusercontent.com/trust-net/dag-documentation/master/images/Transaction%20Submission.png "Submitted Transaction Processing"
[simple-seq]: https://raw.githubusercontent.com/trust-net/dag-documentation/master/images/SimpleSequencing.png "Simple Sequencing Example"
[sharded-seq]: https://raw.githubusercontent.com/trust-net/dag-documentation/master/images/ShardedSequencing.png "Sharded Sequencing Example"
