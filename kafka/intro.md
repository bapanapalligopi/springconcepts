kafka cluster
brokers in cluster
producers,consumers
producers can produce any type of data and send tp kafka cluster
conssumer can consume the produced item
topic -> create a topic in cluster

zookeeper manages state of all the kafka brokers,maintain the configuartion for kafka topics , producer and consumers
distibuted streaming platform
fault tolerance-> if one broker down then other will support
data will store in any duatin in broker

use cases:

messaging-> replacement for activemq ,rabitmq
website activity tracking

kafka cluster:
============
consists of set of brokers,a cluster has a minimum of 3 brokers in production

kafka broker:
===========
kafka broker is a kafka server, act as a message broker between producer and consumer
producer---->kafka_broker---->consumer

producer:
========
application sends a messages to the kafka broker
producer-->messages->send to->kafkabroker

consumer:
=========
consumer is an application , reads messages from kafka broker.
