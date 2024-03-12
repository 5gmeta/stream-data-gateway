
# Stream Data Gateway 
@author damendola@vicomtech.org - Vicomtech

Stream Data Gateway is the module used in 5GMETA Cloud platform to stream the messages from the MECs to the Thirth Parties.
It is implemented using Kafka ecosystem technologies.

## Table of Contents
In this repository we provide two version of the code (for the moment only dev):
- a development version: using docker-compose. This could be used to run the Stream Data GW on your machine easily.
- a production version: with Helm Chart that will be deployed into Kubernetes in the Cloud Platform of 5GMETA. (will be avilable soon...)

We will use a docker-compose (previously LensesBox) only for developement purpose, the production verion will use only opensource free software with a preference for Apache-2.0 License.

### Dev-version (LensesBox, now changed with a docker-compose)
You can find a docker compose file into the folder 'src/dev-version' that contains all the Kafka dockers that need to be execute (plus some dev tools).

How to use it: 

`cd src/dev-version`

`sudo docker-compose up -d`

Now the service are running, and you can find at this address: 

- http://localhost:8080/ --> UI for Apache Kafka  

#### LensesBox (deprecated - please see the docker-compose in dev)
Previoysly we were using Lenses Box, now deprecated. LensesBox is a Docker image which contains Lenses and a full installation of Kafka with all its relevant components. The image packs Lenses’ Stream Reactor Connector collection as well. It is all pre-setup and the only requirement is having Docker installed. It contains Lenses, Kafka Broker, Schema Registry, Kafka Connect, 25+ Kafka Connectors with SQL support and various CLI tools.

link: https://docs.lenses.io/

To get a free license: https://lenses.io/downloads/lenses/?path=docs-box


## Kakfa Ecosystem

Some paragraph to illustreate the main Kafka modules used in 5GMETA platform.

### Kafka Connect: How to use and how to create a Connector from CLI
Apache Kafka Connect is a part of Kafka and provides scalable and reliable way to copy data between Karka and other datastore. It provide APIs and a runtime to develope and run connector plug-ins (library that Kafka Connect executes and that are responsible for moving the data. )
Official: https://kafka.apache.org/documentation/#connect

A command line interface to control the Kafka Connect is available here: https://github.com/lensesio/kafka-connect-tools

We will use JMS connector from Lenses (open source, under Apache 2.0 license) https://docs.lenses.io/3.2/connectors/source/jms.html:
- A Source Connector JSM-Source: from ActiveMQ on the MEC to Kakfa on the Cloud platform,
- A Sink Connector JMS-Sink: from Kafka on the Cloud to ActiveMQ on the MEC platform.  

A Converter could be necessari in the furure developements.
At the moment we are using a `io.confluent.connect.avro.AvroConverter`

This is the default implementation. The payload is taken as is: an array of bytes and sent over Kafka as an AVRO record with Schema.BYTES. You don’t have to provide a mapping for the source to get this converter!!

### How to use Connect-CLI and create a Connector
using the Kafka-connect-tools (download the latest release built binary file) and use the following command:

- Download the latest connect-cli release: wget https://github.com/lensesio/kafka-connect-tools/releases/download/v1.0.9/connect-cli
- It requires Java: `apt-get install openjdk-11-jre`
- Configure connect-cli: `export KAFKA_CONNECT_REST="http://localhost:8083"` (MUST be localhost, some problems with external ip addresses)
- Create the connector:
 `./connect-cli create jmssource < jms-source.properties` (for cits messages)
 `./connect-cli create jmsimagesource < jms-source-image.properties` (for images)
 `./connect-cli create jmssink < jms-sink.properties (for downlink messages)`
Here a resource with more details: https://docs.lenses.io/3.2/connectors/source/jms.html#starting-the-connector-distributed

An usefull article about Connect: https://www.confluent.io/blog/kafka-connect-deep-dive-converters-serialization-explained/

Some othe examples of connect-cli
* ./connect-cli ps
* ./connect-cli status



### Kafka registry
Kafka registry is used transparently by the Connector, Consumer and Producer to validate the messages schema in Avro. Avro is the Serialization and Deseriazation methods for the messages in 5GMETA.

In case it would be necessary to delete or manage a schema from the Kafka Registry, an APIs is exposed. Useful APIs on the Registry:

- Delete a schema in the Registry: `curl -X DELETE http://localhost:8081/subjects/5gmeta-cits-value`


## KSQLdb: general information
KSQLdb (KSQL is to be intended as a synomym of KSQLdb) is a streaming SQL engine that enables real-time data processing against Apache Kafka.

5GMETA platform will use KSQLDB internali (not expose the APIs of KSQL externaly). The minimal tools to filter and manage topics in Kakfa is reppresetned by the KStreams, KSQL under the hood create KStream and KTable (more info here: https://developer.confluent.io/tutorials/filter-a-stream-of-events/kstreams.html).

### KSQLDB: some command tested

In the following I will try to provide some useful command that could be used in 5GMETA, however to have a overall perspective on KSQLdb have a look to the official documentation: https://docs.ksqldb.io/en/latest/

Run the KSQL CLI deployed with docker-compose:

`sudo docker exec -it ksqldb-cli ksql http://ksqldb-server:8088`

KSQLdb Documentation: https://docs.ksqldb.io/en/latest/developer-guide/ksqldb-reference/

First of all here is how to istantiate a Connector from the KSQL API interface: 

```
CREATE SOURCE CONNECTOR JMS_SOURCE WITH (
  'connector.class'= 'com.datamountaineer.streamreactor.connect.jms.source.JMSSourceConnector',
  'connect.jms.initial.context.factory'= 'org.apache.activemq.jndi.ActiveMQInitialContextFactory',
  'tasks.max'= '1',
  'connect.jms.queues'= 'jms-queue',
  'connect.jms.password'= '<password>',
  'connect.progress.enabled'= 'true',
  'connect.jms.username'= '<user>',
  'connect.jms.url'= 'tcp://<ip>:61616 ',
  'connect.jms.kcql'= 'INSERT INTO cits SELECT * FROM cits WITHTYPE TOPIC ',
  'value.converter.enhanced.avro.schema.support'= 'true',
  'value.converter.schema.registry.url'= 'http://schema-registry:8081',
  'connect.jms.connection.factory'= 'ConnectionFactory',
  'name'= 'JMS_SOURCE',
  'value.converter'= 'io.confluent.connect.avro.AvroConverter',
  'key.converter'= 'org.apache.kafka.connect.storage.StringConverter'
  );
```

Create a stream from a topic:

`CREATE STREAM 5GMETA_CITS2_STREAM (destination varchar, bytes_payload varchar ) WITH (kafka_topic='5gmeta-cits', value_format='AVRO');`

Show all streams: 

 `show streams;`

See the column names of the stream:

`DESCRIBE 5GMETA_CITS_STM;`


See the data from a stream:

`SELECT * FROM cits_avro_stream  EMIT CHANGES;`

#### How to access the properties:
How to acccess to the properties fields of our topics.

The field properties is declared by the Connector are using a type of field MAP<String, VARCHAR(String)>, 
this means that is a special field and cannot be filtered in the WHERE clause.
Heva a look here for more information about the MAP and ARRAY fields in KSQLDB https://docs.ksqldb.io/en/0.7.1-ksqldb/developer-guide/query-with-arrays-and-maps/

Usie the DESCRIBE command to see the field, type of the stream:
`DESCRIBE CITS_AVRO_STREAM;`

```
Name                 : CITS_AVRO_STREAM
 Field             | Type                         
--------------------------------------------------
 MESSAGE_TIMESTAMP | BIGINT                       
 CORRELATION_ID    | VARCHAR(STRING)              
 REDELIVERED       | BOOLEAN                      
 REPLY_TO          | VARCHAR(STRING)              
 DESTINATION       | VARCHAR(STRING)              
 MESSAGE_ID        | VARCHAR(STRING)              
 MODE              | INTEGER                      
 TYPE              | VARCHAR(STRING)              
 PRIORITY          | INTEGER                      
 BYTES_PAYLOAD     | BYTES                        
 PROPERTIES        | MAP<STRING, VARCHAR(STRING)> 
--------------------------------------------------
```

For runtime statistics and query details run: `DESCRIBE <Stream,Table> EXTENDED;`

Creating a TABLE from the topic (pay attention to the KEY field https://www.confluent.io/blog/ksqldb-0-10-updates-key-columns/)


With specific fields:

`CREATE TABLE CITS_AVRO_TABLE (rowkey BIGINT PRIMARY KEY, MESSAGE_TIMESTAMP BIGINT, MESSAGE_ID VARCHAR, MODE INTEGER, TYPE  VARCHAR, CORRELATION_ID VARCHAR(STRING), BYTES_PAYLOAD BYTES, PROP MAP<STRING, VARCHAR(STRING)>) WITH (KAFKA_TOPIC = 'cits-avro', VALUE_FORMAT='AVRO');`

Or with all fields:

`CREATE TABLE CITS_AVRO_TABLE (ID INT PRIMARY KEY) WITH (KAFKA_TOPIC = 'cits-avro', VALUE_FORMAT='AVRO');`

Execute the command to see the table 

`SHOW TABLES;` 


##### How to destructurate the field PROPERTIES:
How to manage the MAP fields:
https://docs.ksqldb.io/en/0.11.0-ksqldb/developer-guide/syntax-reference/#map


```
SELECT `MESSAGE_TIMESTAMP` AS TIMESTAMP, `PROPERTIES`['vehicle_id'] FROM CITS_AVRO_STREAM EMIT CHANGES LIMIT 10;
```

And, finaly here is how to filter a strem and create a new topic with the message that have the properties->vehicle_id = 'v1'

```
CREATE STREAM CITS_AVRO_FILTERED_V1_STREAM AS SELECT * FROM CITS_AVRO_STREAM WHERE `PROPERTIES`['vehicle_id']='v1';
```
