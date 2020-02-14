## Overview

Cassandra is used by Contrail services as the database to store data. The number of Cassandra nodes in the cluster has to be odd (2n + 1 to cover n failures). This is required by configuration read/write consistency.

## Services

### Configuration API Server

Configuration API server reads/writes configuration from/to Cassandra. Replication factor is the number of Cassandra nodes in the cluster, so data is replicated on all nodes. Read and write consistency is QUORUM.


### Collector

Collector reads/writes analytics data from/to Cassandra. Replication factor is always 2. Even there is only one Cassandra node, it can be handled by Cassandra. Only 1 failure is covered when there are more than 1 Cassandra nodes. Read and write consistency is ONE.


### Discovery

Discovery service reads/writes server and client data from/to Cassandra. Replication factor is the number of Cassandra nodes in the cluster. Read and write consistency is ONE.


### Schema Transformer and Service Monitor

Schema transformer and service monitor read configuration from Cassandra after
getting notification from RabbitMQ. Read consistency is QUORUM.


## Troubleshoot

### State and Port

TCP ports opened by Cassandra are configured in /etc/cassandra/cassandra.yaml. By default, TCP port 9160 is for client to connect to Cassandra.


### Nodetool Utility

"nodetool status" shows the state of each node in the cluster.

