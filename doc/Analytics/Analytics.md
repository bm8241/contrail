# 1 Overview

Analytics collects system running statistics from all services and provide REST API for client to query. Figure 1 shows all services for analytics. Services are grouped into analytics node and analytics-db node.

![Figure Overview](Analytics/Figure-Overview.png)


# 2 Service

## 2.1 API server

Analytics API server provides REST API for clients to request UVE, query flow stats and log, and upstream UVE or alarm.

Each API server subscribes to all REDIS for getting update of aggregated UVE and alarm, and partition allocation.

For UVE request with '?flat', API server will determine which partition this UVE belongs to and find out which alarm generator is responsible for this partition, then connects to the REDIS server on the same analytics node as that alarm generator, to get aggregated UVE. In case of no '?flat' in UVE request, API server will collect and aggregate UVE from REDIS on all analytics nodes. This is not recommended for sake of performance.

For query request, API server passes query to query engine who will process the query, get data from Cassandra and send result back to API server. Database contains logs, flow stats and individual UVEs in logging format. Other than logs and flow stats, historical individual UVE info can also be queried.

For UVE and alarm streaming, API server subscribes to REDIS server for update. For UVE and alarm update notifications, API server reads UVE from aggregated UVE DB and alarm on REDIS and streams back to client.


## 2.2 Collector

Collector receives reports of UVE and log from all services (generators) and flow stats from vrouter agent.

After receiving reports from services/generators, collector
* publishes individual UVE info onto local REDIS,
* writes all reports to Cassandra, other than log and flow stats, individual UVE is also written to Cassandra as ObjectLog,
* extracts tagged stats from reports and save it in stats tables on Cassandra,
* publishes UVE notification and partition info on Kafka.

Collector needs Zookeeper for election, because with latest cassandra only 1 client can create schema/table at one time and so Zookeeper is used to elect one collector to do it.


## 2.3 REDIS

REDIS serves as both message bus and in-memory storage/cache. It supports publish-subscribe mode. REDIS stores original UVE published by collector and aggregated UVE published by alarm generator. For now, REDIS is not used as a cluster service.

UVE info on REDIS server is not persistent. It will get lost in case of REDIS restart. To handle this case, each generator keeps a cache of reports. In case of REDIS restart, collector will ask all connected generators to re-synchronize (re-send cached reports). Also, in case of collector down, generator will connect to a new collector and re-send cached reports. Alarm generator will also re-create all aggregated UVEs.

API server subcribes to REDIS server for UVE and alarm update. This is for streaming purpose.


## 2.4 Kafka

Kafka is the message bus serving collector and alarm generator for now. It only carries notification, not the complete data. In HA, multiple Kafka instances form the cluster.

REDIS is not used for this purpose because it was not a cluster service when it was used. Kafka may be replaced by REDIS when REDIS supports clustering.


## 2.5 Alarm Generator

Alarm generator builds aggregated UVE on REDIS and raises alarm when the alarm criteria is met. Alarm generator subscribes to RabbiMQ to get alarm configuration updates and store it locally.

To balance the workload in HA case, all UVEs are partitioned. For now, there are 30 partitions. When each alarm generator starts, it connects to Zookeeper to get partitions allocated based on certain algorithm. When the collector receives an UVE report, it maps UVE key to a partition based on an algorithm and publishes notification onto Kafka for that specific partition. Kafka is the messge bus cluster across all analytics nodes. The alarm generator for that specific partition will pick up the notification from Kafka and process it.

In case of adding or deleting UVEs, the whole workload will be re-partitioned and all aggregated UVEs will be re-created based on new partition schema.

After alarm generator receives notification of UVE update on Kafka, it reads and aggregates UVE info from REDIS on all analytics nodes, and publishes such aggregated UVE onto local REDIS. Alarm generator also checks alarm definition, raises the alarm (add alarm content into UVE) if criteria is met, sends updated UVE to collector (whichever got from discovery) just like other generators. Then the collector will update REDIS and Cassandra with updated UVE (with alarm) and publish notification onto Kafka so alarm generator will update aggregated UVE DB with the alarm.


## 2.6 Query Engine

Query engine is for handling query request from API server. Once API server receives query from client, it publishes it onto REDIS. Query engine will get the query from REDIS and process it. Based on the query, query engine reads data from Cassandra, aggregates and filters them, then put the result onto REDIS. Then API server will get the result and send it back to client.


## 2.7 SNMP Collector

SNMP collector subscribes to RabbitMQ for configuration update, based on which it connects to physical device and walk through SNMP tables. Then it will create UVE with SNMP info and send UVE to collector.

Workload (physical devices) for SNMP collector is partitioned and allocated by Zookeeper.


## 2.8 Topology

This service is for building the coorelation between overlay and underlay. It subscribes to RabbitMQ for configuration update, reads UVEs, creates topology UVEs, and send it to collector.

Workload for topology is partitioned and allocated by Zookeeper.


# 3 Data Flow

## 3.1 Log and Flow Stats

Two types of log are reported by each service, system log and trace log. Log report is driven by event. Whenever log is generated by service, it's reported to analytics.

Flow stats is reported by vrouter agent periodically. It's determined by flow export rate, which is 0 (disabled) by default.

Log and flow stats are reported in Sandesh message to collector in analytics. Then collector saves it to Cassandra DB.


## 3.2 Query Log and Flow Stats

![Figure Query](Analytics/Figure-Query.png)

Query is sent to API server as a HTTP POST request with JSON body containing query info. API server receives the query and publishes it on REDIS. Query engine will pick up the query, read data from database, aggregate data and post query result back to REDIS. API server picks up the query result and sends it back to user.


## 3.3 UVE

![Figure UVE](Analytics/Figure-UVE.png)

Collector publishes original UVE on local REDIS, posts notification and partition info onto Kafka, and also writes it to database as object log. Alarm generator who owns that partition reads UVE from REDIS and create or update aggregated UVE.


## 3.4 Request UVE

UVE request is sent to API server as a HTTP GET request. API server will get aggregated UVE from the REDIS who has the partition of that UVE, then send UVE back to user.


## 3.5 Streaming

When open streaming for aggregated UVE or alarm, API server subscribes to REDIS on all analyitcs nodes. Once there is update published by alarm generator, API server will stream the update back to user.


### Notes
Trace in-memory circuler buffer, hardcoded parameters, triggered by analytics API.
/analytics/send-tracebuffer/<source>/<module>/<instance_id>/<name>
source: hostname
modules: name of service
instance_id: 0
name: name of trace buffer, get from introspect (sandish tracebuffer).

