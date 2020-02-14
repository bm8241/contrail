# 1 Overview

As a logical component in Contrail, configuration consists of a group of services to provide configuration management.

Configuration includes the following services.
* API server
* Schema Transformer
* Device Manager
* Service Monitor
* Node Manager

Configuration depends on the following services.
* Cassandra
* RabbitMQ
* Zookeeper
* Keystone (for authentication optionally)


# 2 Workflow

Configuration provides REST API for client to CRUD (Create, Read, Update and Delete) configurations. Configuration is created by client, stored in database and published to control and data planes. Figure "Workflow" shows the configuration work flow.

![Figure Workflow](Configuration/Figure-Workflow.png)


# 3 Configuration services

## 3.1 API Server

API server provides REST API for client to CRUD configuration. After received the request from client, API server connects to authentication server to validate the token.

For reading request, API server reads configuration from Cassandra and sends it back to the client. For other requests, API server processes the request, updates Cassandra, publishes notification to RabbitMQ and sends response back to the client.

Opening ports.
* 8082: REST API
* 8084: Introspec


## 3.2 Schema Transformer

Schema transformer gets notification of configuration update from RabbitMQ, reads configuration from Cassandra and creates some system required configurations, like routing instance, route target, ACL, etc.

In case of HA, schema transformer works in active/backup mode. Only one schema transformer is active. It's elected by Zookeeper.

Opening ports.
* 8087: Instrospec


## 3.3 Service Monitor

Service monitor manages service instance and service chaining. It gets notification from RabbitMQ, reads configuration from Cassandra and creates required configuration for building service instance and service chain.

In case of HA, service monitor works in active/backup mode. Only one service monitor is active. It's elected by Zookeeper.

Opening ports.
* 8088: Instrospec


## 3.4 Device Manager

Device manager manages physical devices, like switch (eg. QFX), gateway (eg. MX) or other physical appliances. It gets notification from RabbitMQ, reads configuration from Cassandra and creates required configuration for managing devices.

In case of HA, device manager works in active/backup mode. Only one service monitor is active. It's elected by Zookeeper.

Opening ports.
* 8096: Instrospec


# 4 Client

Examples of client are Contrail Web UI, Neutron Contrail plug-in in case of OpenStack integration, schema transformer and service monitor (internal client) and utilities developed by users. In case of OpenStack integration, client has to connect to Keystone service to get authentication. If the client is authenticated, a token will be returned from Keystone. This token has to be sent to configuration API server along with each request.


# 5 XMPP Server

XMPP server on control node receives notification from RabbitMQ, read data from Cassandra and store locally.


# 6 Vrouter Agent

Vrouter agent connects to XMPP server on control node to get configuration.


