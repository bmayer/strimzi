
# Deploy & Config Strimzi Kafka

## Download the release artifacts from GH

```shell=
mkdir strimzi; cd strimzi

export STRIMZI_VER=0.27.0

wget https://github.com/strimzi/strimzi-kafka-operator/releases/download/${STRIMZI_VER}/strimzi-${STRIMZI_VER}.tar.gz

tar xvf strimzi-${STRIMZI_VER}.tar.gz
```

## Deploy the Cluster Operator
- This will watch Strimzi resources in a single namespace in your Kubernetes cluster

- This will create a number of CRDs, \*RoleBindings\* & a Deployment/single instance of strimzi-cluster-operator

```shell=
export ns=<namespace>

kubectl create ns ${ns}

cd strimzi-${STRIMZI_VER}

sed -i "s/namespace: .*/namespace: ${ns}/" install/cluster-operator/*RoleBinding*.yaml

kubectl create -f install/cluster-operator -n ${ns}
```

## Deploying the Kafka cluster
- When installing Kafka, Strimzi also installs a ZooKeeper cluster and adds the necessary configuration to connect Kafka with ZooKeeper.

- The location of `emptyDir` should be in `/var/lib/kubelet/pods/{podid}/volumes/kubernetes.io~empty-dir/` on the given Node where your Pod is running.

## Kafka Config
https://strimzi.io/docs/operators/latest/using.html#assembly-deployment-configuration-str

## Ephemeral Deployment
- edit `strimzi/strimzi-${STRIMZI_VER}/examples/kafka/kafka-ephemeral.yaml`

- Define the name of the Kafka cluster via `.metadata.name`

- Define the namespace where Kafka will be installed `.metadata.namespace`
  
- Define the [Service Type](https://strimzi.io/docs/operators/latest/using.html#assembly-accessing-kafka-outside-cluster-str) using `.spec.kafka.listeners.type: loadbalancer` (loadbalancer type services and loadbalancers are created for each Kafka broker, as well as an external bootstrap service. The bootstrap service routes external traffic to all Kafka brokers. DNS names and IP addresses used for connection are propagated to the status of each service.)

- run `kubectl apply -f strimzi/strimzi-${STRIMZI_VER}/examples/kafka/kafka-ephemeral.yaml`


# Topics
k get kafka,kafkatopic -n ${ns}

## Create a topic
```shell=
apiVersion: kafka.strimzi.io/v1beta2
kind: KafkaTopic
metadata:
  name: test-topic
  namespace: kafka
  labels:
    strimzi.io/cluster: yogi-kafka
spec:
  partitions: 1
  replicas: 1
  config:
    retention.ms: 7200000
    segment.bytes: 1073741824
```

## Deploy a Kafka producer/consumer

```shell=
kubectl run kafka-producer -ti --image=quay.io/strimzi/kafka:0.27.0-kafka-3.0.0 -n ${ns} --rm=true --restart=Never -- bin/kafka-console-producer.sh --broker-list yogi-kafka-kafka-bootstrap:9092 --topic test-topic



kubectl run kafka-consumer -ti --image=quay.io/strimzi/kafka:0.27.0-kafka-3.0.0 -n ${ns} --rm=true --restart=Never -- bin/kafka-console-consumer.sh --bootstrap-server yogi-kafka-kafka-bootstrap:9092 --topic test-topic --from-beginning




kubectl run kafka-admin -ti --image=quay.io/strimzi/kafka:0.27.0-kafka-3.0.0 --rm=true --restart=Never -- ./bin/kafka-topics.sh --bootstrap-server localhost:9092 --topic __strimzi-topic-operator-kstreams-topic-store-changelog --delete && ./bin/kafka-topics.sh --bootstrap-server localhost:9092 --topic __strimzi_store_topic --delete



kubectl run kafka-admin -ti --image=quay.io/strimzi/kafka:0.27.0-kafka-3.0.0 -n ${ns} -- sh


# Resources
k get kafkatopic -nkafka
