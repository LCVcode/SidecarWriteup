# Sidecar Feedback

## Pulsar Summary
* Pulsar is a cloud-native, distributed messaging and streaming platform and is a possible alternative to RabbitMQ.
* Pulsar is a top-level Apache Software Foundation project which was originally created at Yahoo!
* Pulsar uses a publish-subscribe messaging model built in Java.
* Key features:
    * [Pulsar Functions](https://pulsar.apache.org/docs/en/functions-overview/)
        * Consume messages from topic(s)
        * Apply user-specificed logic to each message
        * Publish results to another topic
        * Similar to:
            - AWS Lambda
            - Google Cloud Functions
            - Etc
    * [Horizontal Scalability](https://pulsar.apache.org/docs/en/concepts-architecture-overview/)
    * [Multi-tenancy](https://pulsar.apache.org/docs/en/concepts-multi-tenancy/)
    * [Persistent Storage](https://pulsar.apache.org/docs/en/concepts-architecture-overview/#persistent-storage)
        * Guaranteed delivery of any message that Broker receives .
        * Non-acknowledged messages are stored in BookKeeper until their delivery has been acknowledged.
    * [Python Client Library](https://pulsar.apache.org/docs/en/client-libraries-python/)
        * Also libraries for: Java, Go, C++, Node.js, WebSocket, C#
    * [Geo-Replication](https://pulsar.apache.org/docs/en/administration-geo/)
        * Replication of persistently stored message data across multiple clusters of a Pulsar instance.
* Three other Apache applications work in concert to create Apache Pulsar:
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

[Broker](https://code.launchpad.net/~lcvcode/+git/broker)

[Pulsar-client-dummy](https://code.launchpad.net/~lcvcode/+git/pulsar-client-dummy)

## Design
* Design of this Pulsar cluster is as follows:
    * Three charms:
        - Bookie (BookKeeper)
        - Broker 
        - [ZooKeeper-k8s](https://github.com/openstack-charmers/charm-zookeeper-k8s)
    * Broker and ZooKeeper are scaled to any odd value >= 3.
    * Bookie is scaled to 3+.
    * Bookie relates to ZooKeeper and Broker.
    * Broker exposes a "pulsar" relation which is the endpoint for all clients.
    * Dummy pulsar clients are used to for demonstration purposes.

## Demo

## Positives of Sidecar 
* Documentation was valuable but needs expansion.
    * Often, it was more useful to scan Charmhub channels in Mattermost before searching through the docs.
    * I mostly used the Index for navigation.
    * Would have been helpful if the Module Index extended all the way to the method level.
* I found it very intuitive and fast to create the first working prototype of a charm.

## Obstacles / Feedback
### [Lack of One-Off Commands](https://github.com/canonical/pebble/issues/37)
* Executing one-off commands via pebble would remove all manual steps required for deployment of Pulsar.
* Use a work around to achieve this effect.

### Networking problems when using cluster IP after Juju version 2.9.7 - [LP#1943786](https://bugs.launchpad.net/juju/+bug/1943786)
* After Juju version 2.9.7, applications have cluster IPs.
* `ops.model.network.ingress_address` returns the cluster IP, if possible.
* Sidecar charms trying to communicate by `<ingress-address>:<port>` fail. 
* The Kubernetes service gets a default target port of 65535 and isn't configurable from within a charm.
* I avoided this issue by using `ops.model.network.bind_address` instead of `ops.model.network.ingress_address`.

## Future Pulsar Improvements
* Expanded config options for all three applications.
    - Proxies
    - Topic compaction
    - Replication
* Bookie:
    - External message storage.
* ZooKeeper:
    - [Znode creation based on relation data.](https://github.com/openstack-charmers/charm-zookeeper-k8s/issues/1)
* Implement authentication & authorization.
* Implement additional Pulsar functionality.
* Implement tiered storage with Apache Hadoop.
* Use unique images for each application rather than an all-in-one Pulsar image.

## Other
* I submitted a documentation bug to Apache regarding initialization of ZooKeeper metadata:
    * https://github.com/apache/pulsar/issues/12488
