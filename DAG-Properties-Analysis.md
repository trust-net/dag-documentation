## Properties
* [`An application “instantiates” the DLT stack`](#app-instantiates-stack)
* [`A transaction is only part of its submitter’s strand`](#tx-part-of-submitter-strand)
* [`A transaction is only processed at an application node`](#tx-only-processed-at-app-node)

## <a name="app-instantiates-stack"></a>Property
    An application “instantiates” the DLT stack
## Analysis
**Q:** Since DLT stack needs to be instantiated by an application, how will “headless” node work?  
**A:** A headless node is simply a stand alone node that has a shell application that simply instantiates the DLT stack to run as a stand alone executable, with no application registered with sharding layer. Essentially, this results in node only acting as an endorsement peer for network’s security.

**Q:** Can headless node be used to submit a transaction?  
**A:** A transaction is an application specific construct, and hence should only be submittable from an application node. Now an application may define API’s for clients to submit transaction — but that would be application specific business logic. Applications are native, and hence have complete freedom on how they implement the user interface and interaction.

## <a name="tx-part-of-submitter-strand"></a>Property
    A transaction is only part of its submitter’s strand
## Analysis
**Q:** Does this means transaction would be localized to the nodes that submitter connects to?   
**A:** No, due to sync and handshake protocol, that transaction should get propagated to the whole network. Also, even though a transaction may only be “consumed” by nodes that are part of the application shard that the transaction belonged to, it would still get propagated to all the nodes (regardless of whether they are part of the shard or not).

**Q:**    Will transaction only be accessible on the node that submitter connects to?   
**A:**    It should be noted — even though a transaction is part of submitter’s strand, it is actually tied to a specific application. In other words, a transaction has two attributes:
* submitter
* application

So, the first attribute “submitter” determines how the transaction is “ordered”, w.r.t. to other transactions from the same submitter. The second attribute “application” determines who the transaction is “accessible” to. In other words, transaction (and its impact to the application’s world state) will be accessible on any node that is part of the application's shard.

**Q:**    Is the “non-alterable” security of a transaction proportional to how many transactions from that submitter exist after that transaction?  
**A:**    Undeniably there is some benefit to the “non-alterable” security of a transaction if there are later transactions (due to self-pow hash computation, needed for DoS prevention), however that factor is not significant. Actual non-alterability comes from the “signing” of the transaction using private keys. Submitting an “altered” version of the transaction is not possible, without possession of the private key of the submitter.

**Q:**    But what if the submitter itself goes back and “alters” the transaction sequence it submitted earlier, using its own private keys?  
**A:**    Our protocol separates transaction submission (when it is signed), from transaction acceptance (where its validated for sequencing). Hence, any honest node will not allow a transaction sequence “alteration” from the submitter. 

## <a name="tx-only-processed-at-app-node"></a>Property
    A transaction is only processed at an application node
## Analysis
**Q:**    Does this means consensus security (correctness) is weak in this network?   
**A:**    True that the number of nodes at which transaction is processed is less than the total number of nodes in the network. Hence, strength of consensus security is bounded by the number of application nodes (an application node is a node that is part of the application’s shard). This limitation (consensus security == application node count) does not changes. However, our protocol separates “consensus” from “non-alterability” — which means that the strength of “non-alterability” security is based on number of network nodes, regardless of the number of application nodes. Hence, overall our network provides better security, due to added “non-alterability” to any application node.

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
