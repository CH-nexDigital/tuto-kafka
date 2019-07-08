# Apache Kafka & Confluent Kafka
A simple tutorial to get started with Apache Confluent Kafka.





# General presentation of Kafka

Initially developped at LinkedIn, **Apache Kafka** is an event streaming platform capable of handling trillions of events a day. It is designed to reduce the pipelines complexity, handling streaming data at very large scale.
<br>
Data are no more sent from A to a B. Using Kafka, data are stored in Kafka and reachable for any systems that are interested in, whenever they want. There may be systems willing to consume in real-time, ie immediately when data are available in Kafka as systems that want to batch daily data at the end of the day. Kafka offers both approaches: Real-time streaming & Batching.

### :triangular_flag_on_post: **Kafka Architecture**
An instance of Kafka is called **Broker** and multiple brokers form a **Kafka Cluster**.
The cluster is monitored by a cluster of **ZooKeepers** aimed to coordinate different brokers and keep important infomations such as configuration, security and etc.

Systems send data to Kafka Cluster and data on Kafka can be consumed by any applications which are interested in, whenever they want thanks to the **Publish and Subscribe** architecture. 
<br>
Systems which write data to Kafka are called **Producers** and those who fetch data from Kafka are called **Consumers**.

### :triangular_flag_on_post: **Data in Kafka**

The basic unit of data in Kafka is called **message** and they each message belongs to one **Topic**. A topic is a logical categorization of data.
Whitin a topic, messages are organized into **partitions**. A partition is an ordered and immutable log of messages which is represented as ByteArray files on the brokers disks.

### :triangular_flag_on_post: **Replication**

In order to provide **High-Avalaibility** and make data more **persistent**, data should be replicated across brokers on the Cluster.


# Pre-requisites
The following tutorial show command lines for **Linux** systems, for other OS please adapt the command lines to your computer system.

For this tutorial, you should have installed **Confluent Platform**.

It is also recommended to install the JSON Processor **jq**:
```
$ sudo apt-get update
$ sudo apt-get install jq
```


# Install Confluent Platform

Please refer to this [**page**](https://docs.confluent.io/current/installation/installing_cp/deb-ubuntu.html#systemd-ubuntu-debian-install) for local installation using Systemd (**recommended for this tutorial**).
<br>
For other installation methodes, please refer to this [page](https://docs.confluent.io/current/installation/installing_cp/index.html).

The configuration for this tutorial is:

| Components  | Port |
| ------------- | ------------- |
| Zookeeper  | 2181  |
| Kafka Brokers  | 9092  |
| Confluent Control Center | 9021|
| Kafka Connect REST API | 8083 |
| KSQL Server REST API | 8088 |
| REST Proxy | 8082 |
| Schema Registry REST API | 8081 |


If you have different settings, please adapt the command lines to your own configuration.

# Confluent Platform

There are some useful commands for Confluent Platform.

```bash
# Start Confluent Platform
$ confluent start

# Show components' status
$ confluent status

# Stop Confluent Platform
$ confluent stop

# Destroy Confluent Platform
$ confluent destroy

# Log a component       ex: confluent log kafka
$ confluent log <component>
```

# Topics

## Create a topic
By default, if a producer write messages to a non-existing topic, it will automatically create it. If you want to create topics by yourself, you can execute the following command line:
```bash
# Create a topic
$ kafka-topics --zookeeper localhost:2181 --create \
  --replication-factor 1 --partitions 1 --topic my_topic
```

## List Topics
```bash
# List topics
$ kafka-topics --zookeeper localhost:2181 --list
```

## Delete a Topic Content
```bash
# First, set the retention policy to 0ms to delete the topic content
$ kafka-configs --zookeeper localhost:2181  --entity-type topics \
    --entity-name <topic_name> --alter --add-config retention.ms=0
# Wait for 5 minutes and then delete the previous retention policy
$ kafka-configs --zookeeper localhost:2181  --entity-type topics \
    --entity-name <topic_name> --alter --delete-config retention.ms
```

## Delete a Topic
To delete a topic, you must navigate to the broker configuration file **/etc/kafka/server.properties** and add :
```apache
# Enable topic deletion
delete.topic.enable=true
```
After that, you have to restart your broker and run the following command to delete a topic:

```bash
# Delete the topic
$ kafka-topics --zookeeper localhost:2181 --delete --topic <topic_name>
```

# Produce messages

## Console Producer
```bash
# Create a message
$ kafka-console-producer --broker-list localhost:9092 --topic <topic_name>

Example:
$ kafka-console-producer --broker-list localhost:9092 --topic test
> This is a message
> This is another message
```

## Avro Data Format
Avro is an open source data serialization system that helps with data exchange between systems, programming languages, and processing frameworks. Avro helps define a binary format for your data, as well as map it to the programming language of your choice. Using Avro makes possible the data validation that is not supported using other data formats such as JSON.
For more details, please refer to this [page](https://www.confluent.io/blog/avro-kafka-data/).

## Console Avro Producer
There is a console CLI for producing Avro Messages with Confluent. To do thaht, please create another topic to not mix String format data you formerly created and Avro Format data.
<br>
You need to define the schema of your Avro data when you produce Avro messages. 
An Avro schema defines the data structure in a JSON format. The following is an example Avro schema that specifies a user record with two fields: name and index of type string and int.
```json
{"namespace":"kafka-tutorial"
 "type": "record",
 "name": "pokemon",
 "fields": [
     {"name": "name", "type": "string"},
     {"name": "index",  "type": "int"}
 ]
}
```
Now, let's produce some messages with this schema.
```bash
$ kafka-avro-console-producer \
    --broker-list localhost:9092 --topic <topic_name> \
    --property value.schema=<value_schema>


Example:
$ kafka-avro-console-producer \
    --broker-list localhost:9092 --topic pokemon \
    --property value.schema='{"type":"record","name":"pokemon","fields":[{"name": "name", "type": "string"},{"name": "index",  "type": "int"}]}'

> {"name": "Pikachu", "index":25}
> {"name": "Mr. Mime", "index":122}
> {"name": "Eevee", "index":133}
```

# Consume messages

## Console Consumer
```bash
# Read messages
$ kafka-console-consumer.sh --bootstrap-server localhost:9092 --topic <topic_name> \
    --from-beginning


Example: 
$ kafka-console-consumer.sh --bootstrap-server localhost:9092 --topic test \
    --from-beginning

> This is a message
> this is another message
```
## Console Avro Consumer
```bash
# Read Avro messages
$ kafka-avro-console-consumer --bootstrap-server localhost:9092  --topic <topic_name> \
    --from-beginning


Example:
$ kafka-avro-console-consumer --bootstrap-server localhost:9092  --topic pokemon \
    --from-beginning

> {"name":"Pikachu","index":25}
> {"name":"Mr. Mime","index":122}
> {"name":"Eevee","index":133}
```

# Schema Registry
Schema Registry provides a serving layer for your metadata. It provides a RESTful interface for storing and retrieving Avro schemas. It stores a versioned history of all schemas, provides multiple compatibility settings and allows evolution of schemas according to the configured compatibility setting. It provides serializers that plug into Kafka clients that handle schema storage and retrieval for Kafka messages that are sent in the Avro format.


## List all subjects (topics)
```bash
$ curl -s "http://localhost:8081/subjects/" | jq '.'
```
You may notice that **pokemon-value** is listed. Actually, Schema Registry automatically register schemas when using Avro Serialization. Since, we used Avro Console Producer before which implies Avro Serialization, the **pokemon-value** is registered. You may realize that there are no **pokemon-key**, since previously our avro messages did not have any keys.

## List all schema versions registered under a subject
```bash
$ curl -s "http://localhost:8081/subjects/<subject>/versions/" | jq '.'


Example:
$ curl -s "http://localhost:8081/subjects/pokemon-value/versions/" | jq '.'
```

## Fetch a schema by globally unique id
```bash
$ curl -s "http://localhost:8081/schemas/ids/<id>" | jq '.'


Example:
$ curl -s "http://localhost:8081/schemas/ids/1" | jq '.'
```

## Fetch a given version of the schema registered under a subject
```bash
$ curl -s "http://localhost:8081/subjects/<subject>/versions/<version>" | jq '.'


Example:
$ curl -s "http://localhost:8081/subjects/pokemon-value/versions/1" | jq '.'
```

##  Fetch the most recently registered schema under a subject
```bash
$ curl -s "http://localhost:8081/subjects/<subject>/versions/latest" | jq '.'
```

## Register a new version of a schema under a subject

```bash
$ curl -X POST -i -H "Content-Type: application/vnd.schemaregistry.v1+json" \
    --data '{"schema": "{\"type\": \"string\"}"}' \
    http://localhost:8081/subjects/<subject>/versions
```


## Register a new version of a schema under a subject
It can be used to create a new schema under a subject too.

```bash
$ curl -X POST -i -H "Content-Type: application/vnd.schemaregistry.v1+json" \
    --data <schema> \
     http://localhost:8081/subjects/<subject>/versions


Example:
$ curl -X POST -i -H "Content-Type: application/vnd.schemaregistry.v1+json" \
    --data '{"schema": "{\"type\":\"record\",\"name\":\"pokemon.v2\", \"namespace\":\"kafka-tutorial\",\"fields\":[{\"name\":\"name\",\"type\":\"string\"},{\"name\":\"index\",\"type\":\"int\"},{\"name\":\"colour\",\"type\":\"string\",\"default\":\"\"}]}"}'
    http://localhost:8081/subjects/pokemon-value/versions
```
> Since the Schema Registry automatically create/update schemas. That means, you can update schemas without calling the Schema Registry APIs. <br> If you directly pass a new schema in the Avro Console Producer, it will automatically update your schema version.


## Backwards Compatibility
Your schemas should be backwards compatible. If you want to add a new field, you must to pass a default value to make compatible the former messages that you already produced.

For example, if you produce two different messages:
```bash
# Produce a new message that you previously registered
$ kafka-avro-console-producer --broker-list localhost:9092 --topic pokemon \
    --property value.schema='{"type":"record","name":"pokemon","fields":[{"name": "name", "type": "string"},{"name": "index",  "type": "int"}, {"name": "colour",  "type": "string"}]}'
> {"name":"Jigglypuff","index":39,"colour":"pink"}

# Generate a new version of schema with Avro Console Producer
$ kafka-avro-console-producer --broker-list localhost:9092 --topic pokemon \
    --property value.schema='{"type":"record","name":"pokemon","fields":[{"name": "name", "type": "string"},{"name": "index",  "type": "int"}, {"name": "colour",  "type": "string"}, {"name": "generation",  "type": "int", "default":1}]}'
> {"name":"Chansey","index":113,"colour":"pink","generation":1}
```
> The **default** field is very important for the backwards compatibility.

# Rest-Proxy
Confluent offers a REST Proxy that makes possible operations through APIs. Using those APIs, you can build your own applications based on the Confluent Kafka.

## Get a list of topics
 ```bash
 $ curl "http://localhost:8082/topics"
 ```

## Get info about one topic
```bash
$ curl "http://localhost:8082/topics/<topic_name>"
```
## Produce a message with JSON data
```bash
$ curl -X POST -H "Content-Type: application/vnd.kafka.json.v2+json" \
          --data <json_data> \
          "http://localhost:8082/topics/<topic_name>"


Example:
$ curl -X POST -H "Content-Type: application/vnd.kafka.json.v2+json" \
    --data '{"records":[{"key":"gen1","value":{"name": "Bulbasaur"}}, {"key":"gen2","value":{"name": "Chikorita"}}, {"key":"gen3","value":{"name": "Mudkip"}}]}' \
    "http://localhost:8082/topics/json-pokemon"
```

## Create a consumer for JSON data, starting at the beginning of the topic's
The consumer group is called "my_json_consumer" and the instance is "my_consumer_instance".
```bash
$ curl -X POST -H "Content-Type: application/vnd.kafka.v2+json" -H "Accept: application/vnd.kafka.v2+json" \
    --data '{"name": <instance_name>, "format": "json", "auto.offset.reset": "earliest"}' \
    http://localhost:8082/consumers/<consumer_group>


Example:
$ curl -X POST -H "Content-Type: application/vnd.kafka.v2+json" -H "Accept: application/vnd.kafka.v2+json" \
    --data '{"name": "my_consumer_instance", "format": "json", "auto.offset.reset": "earliest"}' \
    http://localhost:8082/consumers/my_json_consumer
> {"instance_id":"my_consumer_instance","base_uri":"http://localhost:8082/consumers/my_json_consumer/instances/my_consumer_instance"}

```
## Subscribe the consumer to a topic
```bash
$ curl -X POST -H "Content-Type: application/vnd.kafka.v2+json" --data '{"topics":<list_of_topics>}' \
    http://localhost:8082/consumers/<consumer_group>/instances/<instance_name>/subscription


Example:
$ curl -X POST -H "Content-Type: application/vnd.kafka.v2+json" --data '{"topics":["json-pokemon"]}' \
    http://localhost:8082/consumers/my_json_consumer/instances/my_consumer_instance/subscription
```

## Consume data
```bash
$ curl -X GET -H "Accept: application/vnd.kafka.json.v2+json" \
    http://localhost:8082/consumers/<consumer_group>/instances/<instance_name>/records


Example:
$ curl -X GET -H "Accept: application/vnd.kafka.json.v2+json" \
    http://localhost:8082/consumers/my_json_consumer/instances/my_consumer_instance/records

> [{"topic":"json-pokemon","key":"gen1","value":{"name":"Bulbasaur"},"partition":0,"offset":0},{"topic":"json-pokemon","key":"gen2","value":{"name":"Chikorita"},"partition":0,"offset":1},{"topic":"json-pokemon","key":"gen3","value":{"name":"Mudkip"},"partition":0,"offset":2}]
```
> This API will return messages that did not have been read by this consumer. If you retry this API, there will be an empty array.

## Stop the consumer
```bash
$ curl -X DELETE -H "Accept: application/vnd.kafka.v2+json" \
          http://localhost:8082/consumers/<consumer_group>/instances/<instance_name>

Example:
$ curl -X DELETE -H "Accept: application/vnd.kafka.v2+json" \
          http://localhost:8082/consumers/my_json_consumer/instances/my_consumer_instance
```

# KSQL

KSQL is the streaming SQL engine for Apache Kafka®. It provides an easy-to-use yet powerful interactive SQL interface for stream processing on Kafka, without the need to write code in a programming language such as Java or Python. KSQL is scalable, elastic, fault-tolerant, and real-time. It supports a wide range of streaming operations, including data filtering, transformations, aggregations, joins, windowing, and sessionization.

KSQL provides multiple functionalities:
- Streaming ETL
- Real-Time Monitoring and Analytics
- Data exploration and discovery
- Anomaly detection
- Personalization
- Sensor data and IoT
- Customer 360-view

<!--

## STREAMS

## TABLE

## SELECT, JOINTURE

# Java APIs

```bash

```

-->