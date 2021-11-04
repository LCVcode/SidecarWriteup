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
juju ssh --container zookeeper zookeeper-k8s/0 "bin/zkCli.sh -- create /ledgers"
juju ssh --container zookeeper zookeeper-k8s/0 "bin/zkCli.sh -- create /ledgers/available"
juju add-relation bookie zookeeper-k8s
```

# Initialize metadata 
```
juju add-relation bookie broker
juju ssh --container broker broker/0 ". ./initialize-cluster-metadata.sh"
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
juju ssh --container pulsar-client pulsar-client-dummy/0 "bin/pulsar-client consume persistent://public/default/test -n 100 -s 'consumer-test' -t 'Exclusive'"

# Connect to second pulsar-client and publish a message
juju ssh --container pulsar-client pulsar-client-dummy/1 "bin/pulsar-client produce persistent://public/default/test -n 1 -m 'Hello Apache Pulsar in Juju'"
```
