Zookeeper is used by Contrail services for locking various resources.

* Zookeeper locks mastership for schema transformer and service monitor who work in active-standby mode.

* Zookeeper locks FQ name (store fq-name and ID mapping) for configuration. In case the requests with the same FQ name coming to multiple configuration API servers, Zookeeper will ensure only one of thoes requests to be accepted.

* Zookeeper locks IP for IP allocation to avoid IP duplication in case of multiple requests coming concurrently.

* Zookeeper locks partition for alarm-gen for work load balance.

