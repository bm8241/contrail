
## Links

[https://docs.openstack.org/mitaka/networking-guide/config-lbaas.html](https://docs.openstack.org/mitaka/networking-guide/config-lbaas.html)
[https://wiki.openstack.org/wiki/Neutron/LBaaS](https://wiki.openstack.org/wiki/Neutron/LBaaS)
[https://wiki.openstack.org/wiki/Network/LBaaS/docs/how-to-create-tls-loadbalancer](https://wiki.openstack.org/wiki/Network/LBaaS/docs/how-to-create-tls-loadbalancer)


# 1 Architecture

LBaaS in OpenStack is implemented by Neutron service. Neutron API provides user interface to take requests.
```
      User Request
           |
           v
       Neutron API
           |
           v
   Neutron LBaaS plugin
   service_plugins in neutron.conf
           |
           v
   LBaaS provider driver
   service_provider in neutron.conf
```
Neutron LBaaS plugin loads LBaaS driver based on 'service_provider' setting and only one provider is supported.


# 2 Neutron CLI

LBaaS can be configured via Neutron API. This section is about how to configure the following components by Neutron CLI.
* Loadbalancer
* Listener
* Pool
* Healthmonitor
* Member

[Appendix A.1](#appendix-a1-script) has an example script.


## 2.1 Loadbalancer

Create a loadbalancer.
```
neutron lbaas-loadbalancer-create --name lb \
    --vip-address $vip $ext_subnet_id
```
With Neutron native LBaaS plugin, since it doesn't support multi-provider, no need to specify provider when creating loadbalancer.

With Neutron Contrail LBaaS plugin, who supports multi-provider, user may specify provider to choose desired driver to implement LB. By default (no provider specified by user), the provider is 'opencontrail' and the driver implements LB with HAProxy.

The argument --tenant-id is for creating object in a specific tenant other than the tenant used for authentication. If that argument is not specified, the object will be created in the same tenant used for authentication.


## 2.2 Listener

LBaaS v2 supports multiple listeners. The following protocols are supported.
* TCP: generic TCP
* HTTP: HTTP
* HTTPS: HTTPS pass-through
* TERMINATED_HTTPS: HTTPS terminate

Create a listener on port 80 for HTTP.
```
neutron lbaas-listener-create --name http \
    --loadbalancer lb --protocol HTTP --protocol-port 80
```

Create a listener on port 22 for TCP.
```
neutron lbaas-listener-create --name ssh \
    --loadbalancer lb --protocol TCP --protocol-port 22
```

Create a listener on port 443 for terminated HTTPS.
```
neutron lbaas-listener-create --name https-terminate \
    --loadbalancer lb --protocol TERMINATED_HTTPS --protocol-port 443 \
    --default-tls-container $container
```
LB with terminated HTTPS requires Barbican service from OpenStack. See [Appendix B](#appendix-b-secret) for how to generate secrets and configure container.


## 2.3 Pool

A pool needs to be created for each listener.

Create the pool for listener 'http'.
```
neutron lbaas-pool-create --name http \
    --lb-algorithm ROUND_ROBIN --listener http --protocol HTTP
```

Create the pool for listener 'ssh'.
```
neutron lbaas-pool-create --name ssh \
    --lb-algorithm ROUND_ROBIN --listener ssh --protocol TCP

```

Create the pool for listener 'https-terminate'.
```
neutron lbaas-pool-create --name https-terminate \
    --lb-algorithm ROUND_ROBIN --listener https-terminate --protocol HTTP
```


## 2.4 Healthmonitor

Create a TCP health monitor for each pool.
```
neutron lbaas-healthmonitor-create --name http \
     --type TCP --pool http \
     --delay 12 --max-retries 5 --timeout 18

neutron lbaas-healthmonitor-create --name ssh \
     --type TCP --pool ssh \
     --delay 12 --max-retries 5 --timeout 18

neutron lbaas-healthmonitor-create --name https-terminate \
     --type TCP --pool https-terminate \
     --delay 12 --max-retries 5 --timeout 18
```


## 2.5 Member

Create members for each pool.
```
neutron lbaas-member-create --name server1 \
    --subnet $int_subnet_id --address 192.168.20.3 \
    --protocol-port 80 http
neutron lbaas-member-create --name server2 \
    --subnet $int_subnet_id --address 192.168.20.4 \
    --protocol-port 80 http

neutron lbaas-member-create --name server1 \
    --subnet $int_subnet_id --address 192.168.20.3 \
    --protocol-port 80 ssh
neutron lbaas-member-create --name server2 \
    --subnet $int_subnet_id --address 192.168.20.4 \
    --protocol-port 80 ssh

neutron lbaas-member-create --name server1 \
    --subnet $int_subnet_id --address 192.168.20.3 \
    --protocol-port 80 https-terminate
neutron lbaas-member-create --name server2 \
    --subnet $int_subnet_id --address 192.168.20.4 \
    --protocol-port 80 https-terminate
```


# 3 Test

Cirros cloud image is used for the client and servers.

On each server, run nc to simulate web service.
```
sudo nc -lk -p 80 -e echo -e "HTTP/1.1 200 OK\nContent-Length: 24\n\n\nServer 1\n192.168.20.3\n" &
```

On the client, run curl to test LB on web services.
```
curl http://<VIP>
curl -k https://<VIP>
```


# Appendix A.1 Script
```
#!/bin/sh

vip=192.168.100.100
ext_subnet=192.168.100.0
int_subnet=192.168.20.0

set_env()
{
    export OS_USERNAME=admin
    export OS_PASSWORD=contrail123
    export OS_TENANT_NAME=demo
    export OS_AUTH_URL=http://10.87.68.166:5000/v2.0/
    export OS_IDENTITY_API_VERSION=2
}

create_lb()
{
    ext_subnet_id=$(neutron subnet-list | awk "/$ext_subnet/"'{print $2}')
    neutron lbaas-loadbalancer-create --name ltm --provider f5 \
        --vip-address $vip $ext_subnet_id
}

delete_lb()
{
    neutron lbaas-loadbalancer-delete ltm
}

create_http()
{
    neutron lbaas-listener-create --name http \
        --loadbalancer ltm --protocol HTTP --protocol-port 80

    neutron lbaas-pool-create --name http \
        --lb-algorithm ROUND_ROBIN --listener http --protocol HTTP

    neutron lbaas-healthmonitor-create --name http \
         --type TCP --pool http \
         --delay 12 --max-retries 5 --timeout 18

    int_subnet_id=$(neutron subnet-list | awk "/$int_subnet/"'{print $2}')
    neutron lbaas-member-create --name server1 \
        --subnet $int_subnet_id --address 192.168.20.3 \
        --protocol-port 80 http

    neutron lbaas-member-create --name server2 \
        --subnet $int_subnet_id --address 192.168.20.4 \
        --protocol-port 80 http
}

delete_http()
{
    delete_list member http
    hm=$(neutron lbaas-pool-show http | awk "/health/"'{print $4}')
    neutron lbaas-healthmonitor-delete $hm
    neutron lbaas-pool-delete http
    neutron lbaas-listener-delete http
}

create_ssh()
{
    neutron lbaas-listener-create --name ssh \
        --loadbalancer ltm --protocol TCP --protocol-port 22

    neutron lbaas-pool-create --name ssh \
        --lb-algorithm ROUND_ROBIN --listener ssh --protocol TCP

    neutron lbaas-healthmonitor-create --name ssh \
         --type TCP --pool ssh \
         --delay 12 --max-retries 5 --timeout 18

    int_subnet_id=$(neutron subnet-list | awk "/$int_subnet/"'{print $2}')
    neutron lbaas-member-create --name server1 \
        --subnet $int_subnet_id --address 192.168.20.3 \
        --protocol-port 22 ssh

    neutron lbaas-member-create --name server2 \
        --subnet $int_subnet_id --address 192.168.20.4 \
        --protocol-port 22 ssh
}

delete_ssh()
{
    delete_list member ssh
    hm=$(neutron lbaas-pool-show ssh | awk "/health/"'{print $4}')
    neutron lbaas-healthmonitor-delete $hm
    neutron lbaas-pool-delete ssh
    neutron lbaas-listener-delete ssh
}

create_https_terminate()
{
    export OS_TENANT_NAME=admin
    container=$(barbican -q secret container list | awk "/tls/"'{print $2}')

    export OS_TENANT_NAME=demo
    neutron lbaas-listener-create --name https-terminate \
        --loadbalancer ltm --protocol TERMINATED_HTTPS --protocol-port 443 \
        --default-tls-container $container

    neutron lbaas-pool-create --name https-terminate \
        --lb-algorithm ROUND_ROBIN --listener https-terminate --protocol HTTP

    neutron lbaas-healthmonitor-create --name https-terminate \
         --type TCP --pool https-terminate \
         --delay 12 --max-retries 5 --timeout 18

    int_subnet_id=$(neutron subnet-list | awk "/$int_subnet/"'{print $2}')
    neutron lbaas-member-create --name server1 \
        --subnet $int_subnet_id --address 192.168.20.3 \
        --protocol-port 80 https-terminate

    neutron lbaas-member-create --name server2 \
        --subnet $int_subnet_id --address 192.168.20.4 \
        --protocol-port 80 https-terminate
}

delete_https_terminate()
{
    delete_list member https-terminate
    hm=$(neutron lbaas-pool-show https-terminate | awk "/health/"'{print $4}')
    neutron lbaas-healthmonitor-delete $hm
    neutron lbaas-pool-delete https-terminate
    neutron lbaas-listener-delete https-terminate
}

create()
{
    create_lb
    create_http
    create_ssh
}

delete_list()
{
    id_list=$(neutron lbaas-$1-list $2 | awk "/True/"'{print $2}')
    for id in $id_list;
    do
        neutron lbaas-$1-delete $id $2
    done
}

create_secret()
{
    mkdir -p secret
    openssl req -newkey rsa:1024 -nodes -keyout secret/ca-root.key \
        -x509 -days 3650 -out secret/ca-root.crt
    openssl req -newkey rsa:1024 -nodes -keyout secret/ca-sign.key \
        -out secret/ca-sign.csr -subj "/CN=ca-sign@ca.com"
    openssl x509 -req -days 3650 -in secret/ca-sign.csr \
        -CA secret/ca-root.crt -CAkey secret/ca-root.key \
        -set_serial 01 -out secret/ca-sign.crt 
    openssl req -newkey rsa:1024 -nodes -keyout secret/server.key \
        -out secret/server.csr -subj "/CN=admin@acme.com"
    openssl x509 -req -days 3650 -in secret/server.csr \
        -CA secret/ca-sign.crt -CAkey secret/ca-sign.key \
        -set_serial 01 -out secret/server.crt 
}

create_container()
{
    export OS_TENANT_NAME=admin

    barbican secret store --name certificate \
        --payload-content-type "text/plain" \
        --payload "$(cat secret/server.crt)"

    barbican secret store --name "private-key" \
        --payload-content-type "text/plain" \
        --payload "$(cat secret/server.key)"

    cert=$(barbican -q secret list | awk "/certificate/"'{print $2}')
    priv_key=$(barbican -q secret list | awk "/private-key/"'{print $2}')

    barbican secret container create --name tls \
        --type certificate \
        --secret certificate=$cert \
        --secret private_key=$priv_key
}

add_user()
{
    export OS_TENANT_NAME=admin

    user=$(openstack user list | awk "/neutron/"'{print $2}')
    list=$(barbican -q secret list | awk "/secrets/"'{print $2}')
    for i in $list;
    do
        barbican -q acl user add --user $user $i
    done
    list=$(barbican -q secret container list | awk "/containers/"'{print $2}')
    for i in $list;
    do
        barbican -q acl user add --user $user $i
    done
}

clear_user()
{
    export OS_TENANT_NAME=admin

    list=$(barbican -q secret list | awk "/secrets/"'{print $2}')
    for i in $list;
    do
        barbican -q acl submit $i
    done
    list=$(barbican -q secret container list | awk "/containers/"'{print $2}')
    for i in $list;
    do
        barbican -q acl submit $i
    done
}

test()
{
    echo "test"
}

help()
{
    echo help
}

main()
{
    set_env
    case "$1" in
        create)
            shift
            create "$@"
            ;;
        create-lb)
            shift
            create_lb "$@"
            ;;
        delete-lb)
            shift
            delete_lb "$@"
            ;;
        create-http)
            shift
            create_http "$@"
            ;;
        delete-http)
            shift
            delete_http "$@"
            ;;
        create-ssh)
            shift
            create_http "$@"
            ;;
        delete-ssh)
            shift
            delete_http "$@"
            ;;
        create-https-terminate)
            shift
            create_https_terminate "$@"
            ;;
        delete-https-terminate)
            shift
            delete_https_terminate "$@"
            ;;
        delete)
            shift
            delete "$@"
            ;;
        create-secret)
            shift
            create_secret
            ;;
        create-container)
            shift
            create_container
            ;;
        add-user)
            shift
            add_user
            ;;
        clear-user)
            shift
            clear_user
            ;;
        test)
            shift
            test "$@"
            ;;
        *)
            help
            ;;
    esac
}

main "$@"
exit 0
```


# Appendix B Secret

To support terminating HTTPS, LB requires server private key and server certificate.

## B.1 Generate secrets

Create CA root key (key pair of private and public keys) and self-signed CA root certificate.
```
openssl req -newkey rsa:1024 -nodes -keyout secret/ca-root.key \
    -x509 -days 3650 -out secret/ca-root.crt
```

Create CA signing key and generate CA signing certificate request.
```
openssl req -newkey rsa:1024 -nodes -keyout secret/ca-sign.key \
    -out secret/ca-sign.csr -subj "/CN=ca-sign@ca.com"
```

Create CA signing certificate signed by CA root key and certificate.
```
openssl x509 -req -days 3650 -in secret/ca-sign.csr \
    -CA secret/ca-root.crt -CAkey secret/ca-root.key \
    -set_serial 01 -out secret/ca-sign.crt
```

Create server key and generate server certificat request.
```
openssl req -newkey rsa:1024 -nodes -keyout secret/server.key \
    -out secret/server.csr -subj "/CN=admin@acme.com"
```

Create server certificated signed by CA signing key and certificate.
```
openssl x509 -req -days 3650 -in secret/server.csr \
    -CA secret/ca-sign.crt -CAkey secret/ca-sign.key \
    -set_serial 01 -out secret/server.crt
```

## B.2 Store secrets

Store server key and certificate in Barbican service, and create a container for them.
```
barbican secret store --name certificate \
    --payload-content-type "text/plain" \
    --payload "$(cat secret/server.crt)"

barbican secret store --name "private-key" \
    --payload-content-type "text/plain" \
    --payload "$(cat secret/server.key)"


barbican secret container create --name tls \
    --type certificate \
    --secret certificate=$cert \
    --secret private_key=$priv_key
```

Secret and container are created by the user used for authentication. If any other user needs to retrieve them, the user has to be added into ACL. For example, objects are created by user 'admin' in tenant 'admin', user 'neutron' in tenant 'service' needs to retrieve objects, then user 'neutron' has to be added into ACL for all objects.
```
user=$(openstack user list | awk "/neutron/"'{print $2}')
list=$(barbican -q secret list | awk "/secrets/"'{print $2}')
for i in $list;
do
    barbican -q acl user add --user $user $i
done
list=$(barbican -q secret container list | awk "/containers/"'{print $2}')
for i in $list;
do
    barbican -q acl user add --user $user $i
done
```

The secret container ID (href) will be specified when creating listener for terminated HTTPS.
```
barbican -q secret container list | awk "/containers/"'{print $2}'
```


