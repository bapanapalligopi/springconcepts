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

kafka broker---> cosumers

============
how data is stored in kafka broker
===========
kafka topic
===================
topic is category where the messages is stored, data can be any type of format
categorize the messages in kafka brokers
topic is identified by a name
you can't query data form the topic like sql
===================
kafka partions
===================
topics are divided into a number of partitions, which contain records in an unchangeble sequence.

topi1-: partition-1,2,3,,,,....

===========
offset 
======
it is id given to messages as the arrive at parttions
==================
topic->partition-1-> 0,1,2,3,4,5,6,7... are the offsets for the messages
=========
consumer groups
===========
consumer group contains one or more  consumers to consumer the messages .
