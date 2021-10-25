## Build charms, if needed

## Deploy the charms
```
juju deploy ./charm-zookeeper-k8s/zookeeper-k8s_ubuntu-20.04-amd64.charm --resource zookeeper-image=zookeeper:latest -n3
juju deploy ./bookie/bookie_ubuntu-20.04-amd64.charm --resource bookie-image=apachepulsar/pulsar:latest -n3
juju deploy ./broker/broker_ubuntu-20.04-amd64.charm --resource broker-image=apachepulsar/pulsar:latest -n3
juju deploy ./pulsar-client-dummy/pulsar-client-dummy_ubuntu-20.04-amd64.charm --resource pulsar-image=apachepulsar/pulsar:latest -n2
```

## Setup zookeeper
```
# Setup znodes
kubectl exec --stdin --tty zookeeper-k8s-0 -n pulsar -c zookeeper -- bin/zkCli.sh -- create /ledgers
kubectl exec --stdin --tty zookeeper-k8s-0 -n pulsar -c zookeeper -- bin/zkCli.sh -- create /ledgers/available
juju add-relation bookie zookeeper-k8s
```

# Initialize metadata 
```
juju add-relation bookie broker
kubectl exec --stdin --tty broker-0 -n pulsar -c broker -- /bin/bash
. initialize-cluster-metadata.sh
juju add-relation broker pulsar-client-dummy
```

## Restart brokers
```
kubectl exec -i broker-0 -n pulsar -c broker  -- /pulsar/bin/pulsar broker
kubectl exec -i broker-1 -n pulsar -c broker  -- /pulsar/bin/pulsar broker
kubectl exec -i broker-2 -n pulsar -c broker  -- /pulsar/bin/pulsar broker
```

## Test pulsar
```
# Connect to first pulsar-client and listen for messages
kubectl exec --stdin --tty pulsar-client-dummy-0 -n pulsar -c pulsar-client -- /bin/bash
bin/pulsar-client consume persistent://public/default/test -n 100 -s 'consumer-test' -t 'Exclusive'

# Connect to second pulsar-client and publish a message
kubectl exec --stdin --tty pulsar-client-dummy-1 -n pulsar -c pulsar-client -- /bin/bash
bin/pulsar-client produce persistent://public/default/test -n 1 -m 'Hello Apache Pulsar in Juju!'
```
