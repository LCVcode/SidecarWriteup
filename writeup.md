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
[Bookie-k8s](https://code.launchpad.net/~lcvcode/+git/bookie-k8s)

[Pulsar-k8s](https://code.launchpad.net/~lcvcode/+git/pulsar-k8s)

[Pulsar-dummy-k8s](https://code.launchpad.net/~lcvcode/+git/pulsar-dummy-client)

## Design
* Pulsar-k8s went through several design iterations, and the final design is as follows:
    * Two charms:
        - Bookie-k8s (BookKeeper)
        - Pulsar-k8s (ZooKeeper & Broker)
    * Pulsar-k8s intended to be scaled any odd value >= 3.
    * Bookie-k8s can be scaled to 3+.
    * Bookie-k8s requires relation to pulsar-k8s to access ZooKeeper.
    * A dummy pulsar client charm is used to simulate an application connecting to the pulsar cluster.
* Initial design:
    * Two charms:
        - ZooKeeper-k8s (ZooKeeper)
        - Pulsar-k8s (Bookie & Broker)
    * This design used an existing ZooKeeper-k8s sidecar charm made by a member of the OpenStack team.
    * Bookie needs to be able to scale much larger than Broker, so this design was changed.
* First revision:
    * Three charms:
        - ZooKeeper-k8s
        - Bookie-k8s
        - Broker-k8s
    * This design used the same pre-made ZooKeeper-k8s charm.
    * Splitting Bookie and Broker worked well and was much simpler than the first design.
    * Failed when I found that ZooKeeper required pulsar-specific dependencies which were missing from the charm.
    * Resulted in final design, where ZooKeeper was rebuild and merged with Broker.

## The Positives
* I found it very intuitive and fast to create the first working prototype of a charm

## Obstacles / Feedback
### [Lack of One-Off Commands](https://github.com/canonical/pebble/issues/37)
* The most prevalent blocker was the inability to run one-off commands from pebble.
* This resulted in manual steps being required for deployment of pulsar.
* Slowed development time by requiring too much human interaction during deployments, which is error prone.
* Found a [possible workaround](https://github.com/canonical/pebble/issues/37#issuecomment-870942625) late into development.
* Would have preferred to be able to run commands with something like: `result = container.execute("<command>", stdout=True)`

### Accessing Containers
* Inability to `juju ssh` made it difficult to start development.
* Finding the `kubectl` command to access containers took time.
    ```
    $ kubectl exec --stdin --tty <charm-name>-<unit-num> -n <model-name> -c <container-name> -- /bin/bash
    ```
* I wish this tied into juju like: `juju ssh <unit> <optional-container-name>`.

### [Networking problems when using a VIP after Juju version 2.9.7](https://bugs.launchpad.net/juju/+bug/1943786)
* Due to [LP1922133](https://bugs.launchpad.net/juju/+bug/1922133), the Bookie-k8s could not connect to ZooKeeper-k8s after Juju version 2.9.8.
* ZooKeeper-k8s is built to avoid the bug linked above:
    - Prefers to provide the VIP over the client relation, but will fall back to a list of unit addresses if it cannot get the VIP.
* Bookie-k8s could not connect to ZooKeeper-k8s after it started getting a single VIP.
    - It tried to connect at `<vip>:<zookeeper-port>`, but there was nothing listening there.
* I avoided this bug by reverting my Juju controller to version 2.9.3, which I had used for prior Sidecar development.

### Charms Breaking After Redeploying a Previously Related Charm
* I found that all relations have a default maximum count of one.
* When relations are removed or related applications are removed, future relations will fail.
* As a result, I was not able to simply `juju refresh` my charms during stages of development where they needed relations.
* This slowed the development process by simply requiring more time between deployments.

### Microk8s Crashes
* This obstacle was much less significant than the others.
* Sometimes, microk8s would crash and juju would completely lose connection to the microk8s controller.
* When this happened, I would have to restart microk8s, rebuild the juju controller and model, and redeploy my environment.

## Future Improvements
* Final version of pulsar would have ZooKeeper and Broker in separate charms, with Broker & Bookie related to ZooKeeper.
* Expanded config options for all three applications.
* ZooKeeper:
    - Ability to create znodes using relation data.
    - Initialization of metadata after Broker connects to it.
* Broker:
    - Start of Broker process only after ZooKeeper is started and connected to Bookie.
* Implement remaining Pulsar functionality:
    - Pulsar Functions
    - Geo-Replication
