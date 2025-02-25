# practice-debezium

## What, How & Why
  - What: An open source CDC tool, used for streaming database chnges. Built on top of kafka, which enables it to let applications consume changed data, correctly and ocmpletly.
  - How: Reads transaction log and uses Kafka to publish it to consumer(s).
  - Why: To solve business problems like data sycn, data refresh etc.

## Tutorial
  - https://debezium.io/documentation/reference/3.0/tutorial.html

## Steps

### Docker
  - Docker is used for containrize services required, like Kafka, connector etc.
  - Refer here: https://docs.docker.com/engine/install/
  - Docker Desk top is used for this tutorial.

### Zookeeper
  - Image pull: docker pull quay.io/debezium/zookeeper:latest
  - Start: $ docker run -it --rm --name zookeeper -p 2181:2181 -p 2888:2888 -p 3888:3888 quay.io/debezium/zookeeper:latest


