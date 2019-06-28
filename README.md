# Apache Kafka & Confluent Kafka
A simple tutorial to get started with Apache Kafka and Confluent Kafka.





# :small_orange_diamond: General presentation of Kafka

Initially developped at LinkedIn, **Apache Kafka** is an event streaming platform capable of handling trillions of events a day. It is designed to reduce the pipelines complexity, handling streaming data at very large scale. 

An instance of Kafka is called **Broker** and multiple brokers form a **Kafka Cluster**.
The cluster is monitored by a cluster of **ZooKeepers** aimed to coordinate different brokers and keep important infomations such as configuration, security and etc.

Systems send data to Kafka Cluster and data on Kafka can be consumed by any applications which are interested in, whenever they want thanks to the **Publish and Subscribe** architecture. 
<br>
Systems which write data to Kafka are called **Producers** and those who fetch data from Kafka are called **Consumers**.

Data in Kafka

The basic unit of data in Kafka is called **message** and they each message belongs to one **Topic**. A topic is a logical categorization of data.
Whitin a topic, messages are organized into **partitions**. A partition is an ordered and immutable log file.







# :small_orange_diamond: Pre-requisites
The following tutorial show command lines for **Linux** systems, for other OS please adapt the command lines to your computer system.

For this tutorial, you should have installed **Confluent Platform**.



# :small_orange_diamond: Install Confluent Platform

# :small_orange_diamond: Topics

# :small_orange_diamond: Produce messages

# Consume messages

# Schema-Registry

# Rest-Proxy

# KSQL

# Java APIs

