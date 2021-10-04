# Sidecar Feedback

## Pulsar Summary
* Puslar is a cloud-native, distributed messaging and streaming platform and is a possible alternative to RabbitMQ.
* Pulsar is a top-level Apache Software Foundation project which was originally created at Yahoo!
* Pulsar uses a publish-subscribe messaging model built in Java.
* Key features:
    * [Pulsar Functions](https://pulsar.apache.org/docs/en/functions-overview/)
        * Consume messages from topic(s)
        * Apply user-specificed logic to each message
        * Publish results to another topic
    * [Horizontal Scalability](https://pulsar.apache.org/docs/en/concepts-architecture-overview/)
    * [Multi-tenancy](https://pulsar.apache.org/docs/en/concepts-multi-tenancy/)
    * [Persistent Storage](https://pulsar.apache.org/docs/en/concepts-architecture-overview/#persistent-storage)
        * Guaranteed delivery of any message that gets to Broker
        * Non-acknowledged messages are stored until they are delievered and acknowledged
    * [Python Client Library](https://pulsar.apache.org/docs/en/client-libraries-python/)
        * Also libraries for: Java, Go, C++, Node.js, WebSocket, C#
    * [Geo-Replication](https://pulsar.apache.org/docs/en/administration-geo/)
* Three other Apache applications work in concert to create Pulsar:
    * ZooKeeper
        - Manages the other applications.
    * BookKeeper
        - Provides storage. 
    * Pulsar Broker
        - Handles all Pulsar message transfers.
* A "pulsar instance" is one or more pulsar clusters managed by a "global ZooKeeper" cluster.
* A "pulsar cluster" consists of at least nine containers:
    * 3 clustered ZooKeepers
    * 3 clustered Brokers
    * 3+ clustered BookKeepers
* A client can connect to one of the Brokers to begin consuming and producing messages.
    * There is a pulsar python package that can be used in charms.

## Code
[Bookie](https://code.launchpad.net/~lcvcode/+git/bookie)

[Pulsar](https://code.launchpad.net/~lcvcode/+git/pulsar)

[Pulsar-client-dummy](https://code.launchpad.net/~lcvcode/+git/pulsar-client-dummy)

## Design
* Design of this Pulsar cluster is as follows:
    * Two charms:
        - Bookie (BookKeeper)
        - Pulsar (ZooKeeper & Broker)
    * Pulsar intended to be scaled any odd value >= 3.
    * Bookie can be scaled to 3+.
    * Bookie requires relation to pulsar to access ZooKeeper.
    * A dummy pulsar client charm is used to simulate an application connecting to the pulsar cluster.

## Demo

## Positives of Sidecar 
* Documentation was good and it was easy to navigate.
* I found it very intuitive and fast to create the first working prototype of a charm.

## Obstacles / Feedback
### [Lack of One-Off Commands](https://github.com/canonical/pebble/issues/37)
* Executing one-off commands via pebble would have been useful.

### [Networking problems when using application IP after Juju version 2.9.7](https://bugs.launchpad.net/juju/+bug/1943786)
* After Juju version 2.9.7, applications have IPs, in addition to individual units.
* Charms relating via this application IP cannot find daemons listening at `<application-ip>:<port>`.
* I avoided this bug by reverting my Juju controller to version 2.9.3.

### Unclear Required Changes Between Juju Versions
* Between periods of development, a new Juju version was released which required the addition of a charmcraft.yaml file.
* Without it, `charmcraft build` would fail.
* I could not easily find documentation about this change.

## Future Pulsar Improvements
* Final version of pulsar would have ZooKeeper and Broker in separate charms, with all three charms related to each other.
* Expanded config options for all three applications.
* ZooKeeper:
    - Create of znodes using a one-off command after connection from Bookie.
    - Initialization of metadata after Broker connects to it.
* Broker:
    - Start of Broker process only after ZooKeeper is started and connected to Bookie.
* Implement remaining Pulsar functionality:
    - Pulsar Functions
    - Geo-Replication
