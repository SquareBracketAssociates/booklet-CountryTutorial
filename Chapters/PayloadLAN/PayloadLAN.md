## A simple network simulator with payload
@ch:lan

In this chapter, we will develop a simulator for a computer network, from scratch, step by step. The program starts with a simplistic model of a computer network, made of objects that represent different parts of a local network such as packets, nodes, links, workstations, and hubs.

At first, we will just simulate the different steps of packet delivery and have fun with the system. As a second step, we will extend the basic functionalities by adding extensions such as a hub and different packet routing strategies. In doing so we will revisit many object-oriented concepts such as polymorphism, encapsulation, and hooks and templates. Finally, this system could be further refined to become an experimentation platform to explore and understand networking and distributed algorithms.

!!note that this version of the exercise is more abstract than the one in the new version of the book: _Learning OOD with TDD in Pharo_ and _Advanced Micro Projects_ (the book available with the Advanced Design Mooc).


![Two little networks composed of nodes and sending packets over links. %width=80](figures/lan-overview.pdf )

#### Basic definitions and a starting point


We need to establish a basic mental model, so what does the description above tell us about a network? A _network_ is a number of interconnected _nodes_, which exchange _data packets_. We will therefore probably need to model the nodes, the connection links, and the packets. Let's fill out a little more detail:

- Nodes have _addresses_, and can send and receive packets.
- _Links_ connect two nodes together, and transmit the packets between them.
- A _packet_ transports a payload and includes the address of the node to which it should be delivered; if we want nodes to be able to answer \(after they receive the packet\), packets should also have the address of the node that originally sent it.



### Packets are simple value objects


Packets seem to be the simplest objects in our model: we only need to create them, ask them about the data they contain, and... that's about it. Once created, a packet object is a passive data structure; it will not change its data, knows nothing of the surrounding network, and has no real behavior to speak of. This sort of object is often referred to as a _value object_.

Let's start by defining a test class and a first test, sketching out what creating and looking at packets should look like:

```language=smalltalk
TestCase << #KANetworkEntitiesTest
    package: 'NetworkSimulator-Tests'
```


```language=smalltalk
KANetworkEntitiesTest >> testPacketCreation
    | src dest payload packet |
    src := Object new.
    dest := Object new.
    payload := Object new.

    packet := KANetworkPacket from: src to: dest payload: payload.

    self assert: packet sourceAddress equals: src.
    self assert: packet destinationAddress equals: dest.
    self assert: packet payload equals: payload
```


By writing this unit test, we have described how we think packets should be created, using a `from:to:payload:` constructor message, and how they should be accessed, using three messages: `sourceAddress`, `destinationAddress`, and `payload`. Since we have not yet decided what addresses and payloads should look like, we will just pass arbitrary objects as parameters. All that matters is that when we tell the packet to give us one of those three pieces of information, it returns the correct object back.

Of course, if we compile and run this test method, it will fail because the class `KANetworkPacket` has not been created yet, nor have any of the four above messages. You can either execute the test and let the system prompt you to define the missing objects and messages when needed or we can define the class from the start:

```language=smalltalk
Object << #KANetworkPacket
    slots: { #sourceAddress . #destinationAddress . #payload};
    package: 'NetworkSimulator-Core'
```


The class-side constructor method creates an instance which it returns after sending it an initialization message, so nothing too original as far as constructors go:

```language=smalltalk
KANetworkPacket class >> from: sourceAddress to: destinationAddress payload: anObject
    ... Your code ...
```


That constructor will need to pass the initialization parameters to the new instance. It's preferable to define a single initialization method that takes all needed parameters at once, since it is only supposed to be called when creating packets and should not be confused with a setter:

```language=smalltalk
KANetworkPacket >> initializeSource: *sourceAddress destination: destinationAddress payload: aPayload
    ... Your code ...
```


Once a packet is created, all we need to do with it is to obtain its payload, or the addresses of its source or destination nodes. Define the following getters:

```language=smalltalk
KANetworkPacket >> sourceAddress
	... Your code ...
KANetworkPacket >> destinationAddress
	... Your code ...
KANetworkPacket >> payload
	... Your code ...
```


Now our test should be running and passing. That's enough for our admittedly simplistic model of packets. We've completely ignored the layers of the [OSI model](https://en.wikipedia.org/wiki/OSI_model), but it could be an interesting exercise to model them more precisely.

### Nodes are known by their address


The first obvious thing we can say about a network node is that if we want to be able to send packets to it, then it should have an address. Let's translate that into a test:

```language=smalltalk
KANetworkEntitiesTest >> testNodeCreation
    | address node |
    address := Object new.
    node := KANetworkNode withAddress: address.
    self assert: node address equals: address
```


Like before, to run this test to completion, we will have to define the `KANetworkNode` class:

```language=smalltalk
Object << #KANetworkNode
    slots: { #address};
    package: 'NetworkSimulator-Core'
```


Then a class-side constructor method takes the address of the new node as its parameter:

```language=smalltalk
KANetworkNode class >> withAddress: aNetworkAddress
    ^ self new
        initializeAddress: aNetworkAddress
```


The constructor relies on an instance-side initialization method, and the test asserts that the `address` accessor works as expected. Let's define them:

```language=smalltalk
KANetworkNode >> initializeAddress: aNetworkAddress
	... Your code ...
```


```
KANetworkNode >> address
	... Your code ...
```


Again, our tests should now pass.

### Links are one-way connections between nodes


After nodes and packets, what about looking at links? In the real world, network cables are bidirectional, but that's because they have wires going both ways. Here, we're going to keep it simple and define links as simple one-way connections; to make a two-way connection, we will just use two links, one in each direction.

However, creating links that know their source and destination node is not sufficient: nodes also need to know about their outgoing links, otherwise they cannot send packets. Let's write a test to cover this.

```language=smalltalk
KANetworkEntitiesTest >> testNodeLinking
    | node1 node2 link |
    node1 := KANetworkNode withAddress: #address1.
    node2 := KANetworkNode withAddress: #address2.
    link := KANetworkLink from: node1 to: node2.

    link attach.

    self assert: (node1 hasLinkTo: node2)
```


This test creates two nodes and a link. After telling the link to _attach_ itself, we check that it did so: the source node should confirm that it has an outgoing link to the destination node. Note that the constructor could have registered the link with `node1`, but we opted for a separate message `attach` instead, because it's bad practice to have a constructor change other objects. This way we can build links between arbitrary nodes and still have control of when the connection really becomes part of the network model. For symmetry, we could have specified that `node2` has an incoming link from `node1`, but that ends up not being necessary, so we leave that out for now.

Again, we need to define the class of links:

```language=smalltalk
Object << #KANetworkLink
    slots: { #source . #destination};
    package: 'NetworkSimulator-Core'
```


A constructor that passes the two required parameters to an instance-side initialization message:

```language=smalltalk
KANetworkLink class >> from: sourceNode to: destinationNode
    ^ self new
        initializeFrom: sourceNode to: destinationNode
```


As well as the initialization method and accessors:

```language=smalltalk
KANetworkLink >> initializeFrom: sourceNode to: destinationNode
	... Your code ...
KANetworkLink >> source
	... Your code ...
KANetworkLink >> destination
	... Your code ...
```


The `attach` method of a link should not (and cannot) directly modify the source node, so it must delegate to it instead.

```language=smalltalk
KANetworkLink >> attach
    source attach: self
```


This is an example of separation of concerns: the link knows which node has to do what, but only the node itself knows precisely how to do that. If a node knows about all its outgoing links then it means it has a collection of them, and attaching a link adds it to that collection:

```language=smalltalk
KANetworkNode >> attach: anOutgoingLink
    outgoingLinks add: anOutgoingLink
```


For this method to compile correctly, we will need to extend `KANetworkNode` with the new instance variable `outgoingLinks`, and with the corresponding initialization code:

```language=smalltalk
KANetworkNode >> initialize
    outgoingLinks := Set new.
```


And finally the unit test relied on a predicate method which we define in `KANetworkNode`:

```language=smalltalk
KANetworkNode >> hasLinkTo: anotherNode
    ... Your code ...
```


The method `hasLinkTo:` should verify that there is at least one outgoing link whose destination is the node passed in as an argument. We suggest to have a look at the iterator `anySatisfy:` to express this logic.

Again, all the tests should now pass.

![Current API of our three main classes. %width=80](figures/API1.pdf)

### Making our objects more understandable


When programming we often make mistakes and it is important to help developers to address them. Let us put a breakpoint in to try and understand the objects we have been working with:

```language=smalltalk
KANetworkEntitiesTest >> testNodeLinking
	| node1 node2 link |
	node1 := KANetworkNode withAddress: #address1.
	node2 := KANetworkNode withAddress: #address2.
	link := KANetworkLink from: node1 to: node2.
	link attach.
	self halt.
	self assert: (node1 hasLinkTo: node2)
```


Running the test will open a debugger as the one shown in Figure *@debugger@*. We can see objects, but their textual representation is too generic to be of any help.

![Navigating specific objects having a generic presentation.](figures/debugger.png width=100&label=debugger)

The method `printOn:` is responsible for printing the object's representation. So we should redefine this method for our objects:

```language=smalltalk
KANetworkNode >> printOn: aStream
	aStream nextPutAll: 'Node ('.
	aStream nextPutAll: address , ')'
```


```language=smalltalk
KANetworkLink >> printOn: aStream
	aStream nextPutAll: 'Link'.
	source
		ifNotNil: [ aStream
				nextPutAll: ' ';
				nextPutAll: source address ].
	destination
		ifNotNil: [ aStream
				nextPutAll: ' -> ';
				nextPutAll: destination address ]
```


Now if we rerun the test we obtain a better user experience as shown in Figure *@debuggerXP@*: we can see the address of a node and the source and destination of a link.

![Navigating objects offering a customized presentation. %width=100&anchor=debuggerXP](figures/betterDebugerXP.png )

### Simulating the steps of packet delivery


The next big feature is that nodes should be able to send and receive packets, and links should be able to transmit them.

```language=smalltalk
KANetworkEntitiesTest >> testSendAndTransmit
    | srcNode destNode link packet |
    srcNode := KANetworkNode withAddress: #src.
    destNode := KANetworkNode withAddress: #dest.
    link := (KANetworkLink from: srcNode to: destNode) attach; yourself.
    packet := KANetworkPacket from: #address to: #dest payload: #payload.

    srcNode send: packet via: link.
    self assert: (link isTransmitting: packet).
    self deny: (destNode hasReceived: packet).

    link transmit: packet.
    self deny: (link isTransmitting: packet).
    self assert: (destNode hasReceived: packet)
```


We create and setup two nodes, a link between them, and a packet. Now, to control which packets get delivered in which order, we specify that it happens in separate, controlled steps. This will allow us to model packet delivery precisely, to simulate latency, out-of-order reception, etc.

- First, we tell the node to send the packet using the message `send:via:`. At that point, the packet should be passed to the link for transmission, but not be delivered yet.
- Then, we tell the link to actually transmit the packet along using the message `transmit:`, and thus the packet should be received by the destination node.


### Sending a packet


To send a packet, the node emits it on the link:

```language=smalltalk
KANetworkNode >> send: aPacket via: aLink
    aLink emit: aPacket
```


For the simulation to be realistic, we do not want the packet to be delivered right away. Instead, emitting a packet just stores it in the link until the user selects this packet be transmitted using the `transmit:` message. Storing packets requires adding an instance variable to `KANetworkLink`, as well as specifying how this instance variable should be initialized.

```language=smalltalk
Object << #KANetworkLink
    slots: {#source . #destination . #packetsToTransmit}; 
    package: 'NetworkSimulator-Core'
```


```language=smalltalk
KANetworkLink >> initialize
    packetsToTransmit := OrderedCollection new
```


```language=smalltalk
KANetworkLink >> emit: aPacket
    "Packets are not transmitted right away, but stored.
    Transmission is explicitly triggered later by sending #transmit:."

    packetsToTransmit add: aPacket
```


We also add a testing method to check whether a given packet is currently being transmitted by a link:

```language=smalltalk
KANetworkLink >> isTransmitting: aPacket
    ... Your code ...
```


### Transmitting across a link


Transmitting a packet means telling the link's destination node to receive it. Nodes only consume packets addressed to them; fortunately this is what will happen in our test, so we can worry about the alternative case later \(`notYetImplemented` is a special message that we can use in place of code that we will have to write eventually, but prefer to ignore for now\).

```language=smalltalk
KANetworkNode >> receive: aPacket from: aLink
    aPacket destinationAddress = address
        ifTrue: [
            self consume: aPacket.
            arrivedPackets add: aPacket ]
        ifFalse: [ self notYetImplemented ]
```


Consuming a packet represents what the node will do with it at the application level; for now let's just define an empty `consume:` method, as a placeholder:

```language=smalltalk
KANetworkNode >> consume: aPacket
    "Default handling is to do nothing."
```


After consuming the packet, we note that it did arrive; this is mostly for testing and debugging, but someday we might want to simulate packet losses and re-emissions.
Don't forget to declare and initialize the `arrivedPackets` instance variable, along with its accessor:

```language=smalltalk
KANetworkNode >> hasReceived: aPacket
	... Your code ...
```


Now we can implement the `transmit:` message. A link can not transmit packets that have not been sent via it, and once transmitted, the packet should not be in the link anymore. We should remove it from the link's list of packets to be transmitted and tell the destination to receive it using the message `receive:from:`.

```language=smalltalk
KANetworkLink >> transmit: aPacket
    "Transmit aPacket to the destination node of the receiver link."
    ... Your code ...
```


At this point all our tests should pass.

Note that the message `notYetImplemented` is not called, since our tests do not yet require routing. Figure *@Api2@* shows that the API of our classes is getting richer than before.

![Richer API. % width=90&anchor=Api2](figures/API2.pdf)


### The loopback link


On a real network, when a node wants to send a packet to itself, it does not need any connection to do so. In real-world networking stacks, loopback routing shortcuts the lower networking layers, but this is more detail than we are modeling here.

Still, we do want to model the fact that the loopback link is a little special, so each node will store its own loopback link, separately from the outgoing links. We start to define a test.

```language=smalltalk
KANetworkEntitiesTest >> testLoopback
    | node packet |
    node := KANetworkNode withAddress: #address.
    packet := KANetworkPacket from: #address to: #address payload: #payload.

    node send: packet.
    node loopback transmit: packet.

    self assert: (node hasReceived: packet).
    self deny: (node loopback isTransmitting: packet)
```


The loopback link is implicitely created as part of the node itself. We also introduce a new `send:` message, which takes the responsibility of selecting the link to emit the packet. For triggering packet transmission, we have to use a specific accessor get to the loopback link of the node.

First, we have to add yet another instance variable in nodes:

```language=smalltalk
Object << #KANetworkNode
    slots: {#address . #outgoingLinks .  #loopback . #arrivedPackets};
    package: 'NetworkSimulator-Core'
```


As with all instance variables, we have to remember to make sure it is correctly initialized; we thus modify `initialize`:

```language=smalltalk
KANetworkNode >> initialize
    ... Your code ...
```


The accessor has nothing special:

```language=smalltalk
KANetworkNode >> loopback
    ^ loopback
```


And finally we can focus on the `send:` method and automatic link selection. The method `send:` should be more generic than the method `send:via:` and will be one exposed as a public entry point.

This method has to rely on some routing algorithm to identify which links will transmit the packet closer to its destination. Since some routing algorithms select more than one link, we will implement routing as an _iteration_ method, which evaluates the given block for each selected link.

```language=smalltalk
KANetworkNode >> send: aPacket
    "Send aPacket, leaving the responsibility for routing to the node."
    self
        linksTowards: aPacket destinationAddress
        do: [ :link | self send: aPacket via: link ]
```


One of the simplest routing algorithms is _flooding_, which just sends the packet to every single outgoing link. Obviously this is a waste of bandwidth, but it works without any knowledge of the network topology beyond the list of outgoing links.

There is, however, one case where we know how to route the packet: if the destination address matches the one of the current node, we can select the loopback link. The logic of `linksTowards:do:` should look something like:

- compare the packet's destination address with the one of the node
- if it is the same, we execute the block using the loopback link
- else we simply iterate on the outgoing links of the receiver.


```language=smalltalk
KANetworkNode >> linksTowards: anAddress do: aBlock
    "Simple flood algorithm: route via all outgoing links.
    However, just loopback if the receiver node is the routing destination."
    ... Your code ...
```


Now we have the basic model working we can try more realistic examples.


### Modeling the network itself


More realistic tests will require non-trivial networks. We therefore need an object that represents the network as a whole, to avoid keeping many nodes and links in individual variables. We will introduce a new class `KANetwork`, whose responsibility is to help us build, assemble then find the different nodes and links involved in a network.

Let's start by creating another test class, to keep things in order:

```language=smalltalk
TestCase << #KANetworkTest
    slots: { #net . #hub . #alone};
    package: 'NetworkSimulator-Tests'
```


Since every test will need to rebuild the example network from scratch, we'll move those steps out to a method called `buildNetwork`. We'll then send that message in the `setUp` method of our test, which is called before each test method is run.

```language=smalltalk
KANetworkTest >> setUp
    super setUp.
    self buildNetwork
```


We'll worry about the implementation of `buildNetwork` shortly.

Before we go any further with the test setup, let's write a test that will pass once we've made progress; we want to be able to access network nodes given their addresses. Here we check that we get a the node `hub` based on its address:

```language=smalltalk
KANetworkTest >> testNetworkFindsNodesByAddress
    self
        assert: (net nodeAt: hub address ifNone: [ self fail ])
        equals: hub
```


We will have to implement `nodeAt:ifNone:` on our `KANetwork` class, but first we need to decide how we're going to build our example network. This is what it's going to look like:
- The nodes `pc1`, `pc2`, `mac`, and `impr` connected in a star-shape around a central `hub` node
- A pair of connected nodes, `ping` and `pong`, which are part of the network but not linked to any other nodes
- And the node `alone` which sits all by itself, not even a part of our network


We can picture this as in Figure *@net@*.

![Our network `net`. % width=50&anchor=net](figures/lan-star.pdf)

Expanding a network implies adding new connections and possibly new nodes to it. If the `net` object understands a `connect: aNode to: anotherNode` message, we should be able to build nodes and connect them into a network that matches the figure. This can then be the content of our `buildNetwork` method:

```language=smalltalk
KANetworkTest >> buildNetwork
	alone := KANetworkNode withAddress: #alone.
	net := KANetwork new.
	hub := KANetworkNode withAddress: #hub.
	#( mac pc1 pc2 impr ) do: [ :addr | 
		| node |
		node := KANetworkNode withAddress: addr.
		net connect: node to: hub ].
	net
		connect: (KANetworkNode withAddress: #ping)
		to: (KANetworkNode withAddress: #pong)
```


The name of the `connect:to:` message suggests that establishing the bidirectional links is the responsibility of the `net` object. It also has to remember enough information to allow us to inspect the network topology; we can simply store nodes and links in a couple of sets, even though that representation is a little redundant.
Let's define the class with two instance variables:

```language=smalltalk
Object << #KANetwork
    slots: {#nodes . #links};
    package: 'NetworkSimulator-Core'
```


Whenever we define an instance variable, initialization \(if any\) comes next:

```language=smalltalk
KANetwork >> initialize
    ... Your code ...
```



Now we should give the network the ability to create links.  We'll use this method to add links to the network.

```
KANetwork >> makeLinkFrom: aNode to: anotherNode
	^ KANetworkLink from: aNode to: anotherNode
```


We will also add a low level method `add:` to add a node in a network:

```language=smalltalk
KANetwork >> add: aNode
	nodes add: aNode
```


And to test the network construction we add a little test method:

```language=smalltalk
KANetwork >> doesRecordNode: aNode
	^ nodes includes: aNode
```


Now we can add isolated nodes to the network.

#### Connecting nodes.

Connecting nodes without ensuring that they are part of the network doesn't make sense.
Therefore, when connecting nodes, we will also:

- ensure that nodes are added to the network \(by adding them in the node Set of the network\)
- create and attach links to the nodes in _both_ directions
- and finally store both links in the network


Here is a test covering connecting nodes in a network:

```
KANetworkTest >> testConnect
	| netw hubb mac pc1 |
	netw := KANetwork new.
	hubb := KANetworkNode withAddress: #hub.
	mac := KANetworkNode withAddress: #mac.
	pc1 := KANetworkNode withAddress: #pc1.

	netw connect: hubb to: mac.
	self assert: (hubb hasLinkTo: mac).
	self assert: (mac hasLinkTo: hubb).
	self assert: (netw doesRecordNode: hubb).
	self assert: (netw doesRecordNode: mac).

	netw connect: hubb to: pc1.
	self assert: (hubb hasLinkTo: pc1).
	self assert: (mac hasLinkTo: hubb)
```


Now implement the `connect:to:` method. For concision, note that the `attach` method we defined previously effectively returns the link.

```language=smalltalk
KANetwork >> connect: aNode to: anotherNode
    ... Your code ...
```


The test `testConnect` should now be green.

### Looking up nodes


At this point, the test `testNetworkFindsNodesByAddress` should run through `setUp` method, but fail in the unit test itself. This is because we still need to implement node lookup. The base lookup should find the first node that has the requested address, or evaluate a fall-back block \(a perfect case for the `detect:ifNone:` message\):

```language=smalltalk
KANetwork >> nodeAt: anAddress ifNone: noneBlock
    ... Your code ...
```


We can also make a convenience method, `nodeAt:`, for node lookup. This will raise the predefined `NotFound` exception if it does not find the node. Let's first write a test which validates this behavior:

```language=smalltalk
KANetworkTest >> testNetworkOnlyFindsAddedNodes
    self
        should: [ net nodeAt: alone address ]
        raise: NotFound
```


Then we can simply express `nodeAt:` by delegating to `nodeAt:ifNone:`. Note that to raise an exception, you can simply send the message `signal` to the exception class. Here we use the specific class method `signalFor:in:` defined on the `NotFound` class. `NotFound` exceptions are used when a collection does not contain an object; the `signalFor:in:` method adds both the collection and the missing object to the exception, generating a more detailed description of the problem for the user:

```language=smalltalk
KANetwork >> nodeAt: anAddress
    ^ self
        nodeAt: anAddress
        ifNone: [ NotFound signalFor: anAddress in: self ]
```


### Looking up links


Next, we want to be able to lookup links between two nodes. Again we define a new test:

```language=smalltalk
KANetworkTest >> testNetworkFindsLinks
    | link |
    self
        shouldnt: [ link := net linkFrom: #pong to: #ping ]
        raise: NotFound.
    self
        assert: link source
        equals: (net nodeAt: #pong).
    self
        assert: link destination
        equals: (net nodeAt: #ping)
```


And we define the method `linkFrom:to:`, returning the link between source and destination nodes with matching addresses, and signalling `NotFound` if no such link is found:

```language=smalltalk
KANetwork >> linkFrom: sourceAddress to: destinationAddress
    ... Your code ...
```


#### Final check.

As a final check, let's try some of the previous tests, first on the isolated `alone` node, showing that loopback works even without a network connection:

```language=smalltalk
KANetworkTest >> testSelfSend
    | packet |
    packet := KANetworkPacket
        from: alone address
        to: alone address
        payload: #something.
    self assert: (packet isAddressedTo: alone).
    self assert: (packet isOriginatingFrom: alone).

    alone send: packet.
    self deny: (alone hasReceived: packet).
    self assert: (alone loopback isTransmitting: packet).

    alone loopback transmit: packet.
    self deny: (alone loopback isTransmitting: packet).
    self assert: (alone hasReceived: packet)
```


You can see that we used new convenience testing methods `isAddressedTo:` and `isOriginatingFrom:` which help inspect the state of a simulated network without explicitly comparing addresses.
However, those methods should not take part in network simulation code, since in the real world nodes can never know their peers other than through their addresses.

```language=smalltalk
KANetworkPacket >> isAddressedTo: aNode
    ^ destinationAddress = aNode address
```


```language=smalltalk
KANetworkPacket >> isOriginatingFrom: aNode
    ^ sourceAddress = aNode address
```


The second test attempts transmitting a packet in the network, between the directly connected nodes `ping` and `pong`:

```language=smalltalk
KANetworkTest >> testDirectSend
    | packet ping pong link |
    packet := KANetworkPacket from: #ping to: #pong payload: #ball.
    ping := net nodeAt: #ping.
    pong := net nodeAt: #pong.
    link := net linkFrom: #ping to: #pong.

    ping send: packet.
    self assert: (link isTransmitting: packet).
    self deny: (pong hasReceived: packet).

    link transmit: packet.
    self deny: (link isTransmitting: packet).
    self assert: (pong hasReceived: packet)
```


Both tests should pass with no additional work, since they just reproduce what we already tested in `KANetworkEntitiesTest` and adding `KANetwork` did not impact the established behavior of nodes, links, and packets.


### Packet delivery with forwarding


Up until now, we have only tested packet delivery between directly connected nodes; now let's try sending to a node so that the packet has to be forwarded through the `hub`.

```language=smalltalk
KANetworkTest >> testSendViaHub
    | hello mac pc1 firstLink secondLink |
    hello := KANetworkPacket from: #mac to: #pc1 payload: 'Hello!'.
    mac := net nodeAt: #mac.
    pc1 := net nodeAt: #pc1.
    firstLink := net linkFrom: #mac to: #hub.
    secondLink := net linkFrom: #hub to: #pc1.

    self assert: (hello isAddressedTo: pc1).
    self assert: (hello isOriginatingFrom: mac).

    mac send: hello.
    self deny: (pc1 hasReceived: hello).
    self assert: (firstLink isTransmitting: hello).

    firstLink transmit: hello.
    self deny: (pc1 hasReceived: hello).
    self assert: (secondLink isTransmitting: hello).

    secondLink transmit: hello.
    self assert: (pc1 hasReceived: hello).
```


If you run this test, you will see that it fails because of the `notYetImplemented` message we left earlier in `receive:from:`; it's time to fix that! When a node receives a packet, but it is not the recipient, it should forward the packet:

```language=smalltalk
KANetworkNode >> receive: aPacket from: aLink
    aPacket destinationAddress = address
        ifTrue: [
            self consume: aPacket.
            arrivedPackets add: aPacket ]
        ifFalse: [ self forward: aPacket from: aLink ]
```


Now we need to implement packet forwarding, but there is a trap...

An easy solution would be to simply `send:` the packet again: the hub would send the packet to all its connected nodes, one of which happens to be `pc1`, the recipient, so all is good!

_Wrong!_

The packet would be also sent to other nodes than the recipient. What would those nodes do when they receive a packet not addressed to them? Forward it. Where? To all their neighbours, which would forward it again... and again... and again... so when would the forwarding stop?

We should improve our test to demonstrate that hubs don't send the same packet back to the originating node:

```language=smalltalk
KANetworkTest >> testSendViaHub
    | hello mac pc1 firstLink secondLink |
    hello := KANetworkPacket from: #mac to: #pc1 payload: 'Hello!'.
    mac := net nodeAt: #mac.
    pc1 := net nodeAt: #pc1.
    firstLink := net linkFrom: #mac to: #hub.
    secondLink := net linkFrom: #hub to: #pc1.
    backLink := net linkFrom: #hub to: #mac.

    self assert: (hello isAddressedTo: pc1).
    self assert: (hello isOriginatingFrom: mac).

    mac send: hello.
    self deny: (pc1 hasReceived: hello).
    self assert: (firstLink isTransmitting: hello).

    firstLink transmit: hello.
    self deny: (pc1 hasReceived: hello).
    self deny: (backLink isTransmitting: hello).
    self assert: (secondLink isTransmitting: hello).

    secondLink transmit: hello.
    self assert: (pc1 hasReceived: hello).
```


The link `backLink` monitors whether `hub` sends the `hello` packet back to `mac` when it forwards the message upon its receipt. 

To get this test to pass we need hubs to behave differently from nodes. In reality hubs work at the lower layers of the OSI model, but our simplified model does not have that level of detail. We can approximate this by saying that upon reception of a packet addressed to another node, a hub should forward the packet, but a normal node should just ignore it.

Let's first define an empty `forward:from:` method for nodes, then add a new class for hubs, which will be modeled as nodes with an actual implementation of forwarding:

```language=smalltalk
KANetworkNode >> forward: aPacket from: arrivalLink
    "Do nothing. Normal nodes do not route packets."
```


### Introducing a new kind of node


Now we define the class `KANetworkHub` that will be the recipient of hub specific behavior:

```language=smalltalk
KANetworkNode << #KANetworkHub
    package: 'NetworkSimulator-Core'
```


We can also create this class through the _Refactorings_ menu option on the `KANetworkNode` class. To do this we open the menu, select _New subclass_ and then enter the new subclass name.

Now it's time to implement `forward:from:` for our `KANetworkHub`. As the hub has no routing information we're forced to rely on a flood routing algorithm. But... remember the constraint as outlined in our test above: we do not want our hub forwarding the packet back to the originating node. We suggest taking advantage of the message `linksTowards:do:` that performs an action for all given links to one address:

```language=smalltalk
KANetworkHub >> forward: aPacket from: arrivalLink
   ... Your code ...
```


Now we can use a proper hub in our test, replacing the relevant line in `buildNetwork`, and then check that the `testSendViaHub` unit test passes.

```language=smalltalk
    hub := KANetworkHub withAddress: #hub.
```



You have now a nice basis for simulating networks. But we can do more! In the following section we will present some possible extensions.

### Other examples of specialized nodes

In this section we will present some extensions to our network simulation package that will allow us to support new scenarios. We will propose some tasks to make sure that the extensions are fully working. In this section we do not define tests - but we _strongly_ encourage you to start to write your own tests. At this point in the book you should be comfortable writing your own tests, and should see their value for improving your development process. You should also consider writing your tests _first_, before you start implementing the examples below, following the practice of Test Driven Development. So please take this opportunity to practice.

#### Workstations counting received packets


We would like to know how many packets specific nodes are receiving. In particular when a _workstation_ node consumes a packet, it increments a packet counter.

%  consider asking reader to write the test now? Something like...

!!important Can you write a test for the requirement above? Do you think you could use it to drive the development of a `KANetworkWorkstation` class, whose instances implement the above behaviour? What other behaviours should the instances exhibit? If you can, compare your solution to the one below. If you don't feel comfortable writing your test first yet, don't worry; just continue reading and implement the solution below.

Let's start by subclassing `KANetworkNode`:

```language=smalltalk
KANetworkNode << #KANetworkWorkstation
    slots: {#receivedCount};
    category: 'NetworkSimulator-Nodes'
```


%  comment about the new package

We need to initialize the `receivedCount` instance variable for workstations. Redefining `initialize` should do the trick but, because we inherit instance variables from the superclass \(`address`, `outgoingLinks`, `arrivedPackets` and `loopback`\), it's really important to send the `super initialize` message. If we didn't, some of the those instance variables would not receive their default values \(`address` is initialized in the  `KANetworkNode >> withAddress:` constructor, which our workstations will also use\).

```language=smalltalk
KANetworkWorkstation >> initialize
    super initialize.
    receivedCount := 0
```


Now we can override `consume:` in `KANetworkWorkstation`:

```language=smalltalk
KANetworkWorkstation >> consume: aPacket
    receivedCount := receivedCount + 1
```


We should also:

- define accessors and the `printOn:` method for debugging.
- define a test \(or multiple tests\) for the behavior of workstation nodes.


#### Printers accumulating printouts


When a printer consumes a packet, it prints it. We can model the output tray of a printer as a list where packet payloads get stored, and the supply tray as the number of blank sheets remaining in the printer.

!!important Again, if you feel comfortable writing a test first based on these requirements, feel free to do so before reading any further and then compare your solution to the one below.

The implementation is very similar to what we did for workstations: we subclass `KANetworkNode` and redefine the `consume:` method:

```language=smalltalk
KANetworkNode << #KANetworkPrinter
    slots: { #supply . #output};
    package: 'NetworkSimulator-Nodes'
```


```language=smalltalk
KANetworkPrinter >> consume: aPacket
    supply > 0 ifFalse: [ ^ self "no paper, do nothing" ].

    supply := supply - 1.
    output add: aPacket payload
```


Initialization is a bit different, though; since the standard `initialize` method has no argument, the only sensible initial value for the `supply` instance variable is zero:

```language=smalltalk
KANetworkPrinter >> initialize
    super initialize.
    supply := 0.
    tray := OrderedCollection new
```


We therefore need a way to pass the initial supply of paper available to a fresh instance:

```language=smalltalk
KANetworkPrinter >> resupply: paperSheets
    supply := supply + paperSheets
```


For convenience, we can provide an extended constructor to create printers with a non-empty supply in one message:

```language=smalltalk
KANetworkPrinter class >> withAddress: anAddress initialSupply: paperSheets
    ^ (self withAddress: anAddress)
        resupply: paperSheets;
        yourself
```


To finish up we can:

- define some useful accessors and our good friend the `printOn:` method for debugging purpose
- define some test methods for the behavior of printer nodes \(if you haven't done so already\)
- extend the behaviour further. Are there more messages that a printer could usefully respond to?


#### Servers that answer a request


Let's make a server which responds to a packet by taking its payload, converting it to uppercase, and then sends it back to the server which sent it.

!!important Again, you can stop here and try to drive the implementation our new server with tests if you're comfortable doing that. Otherwise, read on. 

This is another subclass of `KANetworkNode` which redefines the `consume:` method, but this time the node is stateless, so we have no initialization or accessor methods to write:

```language=smalltalk
KANetworkNode << #KANetworkEchoServer
    package: 'NetworkSimulator-Nodes'
```


```language=smalltalk
KANetworkEchoServer >> consume: aPacket
    | response |
    response := aPacket payload asUppercase.
    self send: (KANetworkPacket
        from: self address
        to: aPacket sourceAddress
        payload: response)
```


%  I think we have a problem here. Please correct me if I'm wrong, but at this point there is no way for a ==KANetworkLink== to transmit a packet along to the destination of the link when we don't have a reference to the packet: ==transmit:== needs a packet argument. So I don't think this will work!

%  I've not got any good ideas for this. Some of my bad ideas are:

%  - expose the Set of packets in a link through an accessor so we can get a reference
%  - a new method to transmit all the packets in a link
%  - a specialisation on ==KANetworkLink== for a link which ==transmit:== whenever we call ==emit:==

%  Probably the first is best.

%  The other question is how to handle this diactically. Paragraph to explain the problem, with a solution? The problem, with hints towards the solution? Or just hints towards the problem?

Define a test for the behavior of server nodes.

### Conclusion


In this chapter we built a little network simulation system, step by step. We showed the benefit of good protocol decompositions.

As a further extension, we suggest modeling a more realistic network with cycles, as shown in Figure *@routing@*.
Making this work properly will require replacing our primitive hubs with real routers, and flood routing with more realistic routing algorithms.

![A possible extension: a more realistic network with a cycle between three router nodes. %width=50&anchor=routing](figures/lan-routes.pdf )

To start you off, here is a possible setup for a new family of tests:

```language=smalltalk
KARoutingNetworkTest >> buildNetwork
    | routers |
    net := KANetwork new.

    routers := #(A B C) collect:
        [ :each | KANetworkHub withAddress: each ].
    net connect: routers first to: routers second.
    net connect: routers second to: routers third.
    net connect: routers third to: routers first.

    #(a1 a2) do: [ :addr |
        net connect: routers first
            to: (KANetworkNode withAddress: addr) ].
    #(b1 b2 b3) do: [ :addr |
        net connect: routers second
            to: (KANetworkNode withAddress: addr) ].
    net connect: routers third
        to: (KANetworkNode withAddress: #c1)
```
