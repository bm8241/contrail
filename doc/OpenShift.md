
# 1 Overview

* Contrail 5.0.0 with OpenShift 3.7
* Contrail 5.0.1 with OpenShift 3.9
* Contrail 5.0.2 with OpenShift 3.9


# 2 Project/Namespace

With OpenShift, each OpenShift project/namespace maps to a project in Contrail. All unisolated projects share the same virtual network 'k8s-default-pod-network' in project 'default'. Here is an example of unisolated project.
```
apiVersion: v1
kind: Namespace
metadata:
  name: demo
```

For each isolated project, virtual network 'k8s-<project name>-pod-network' is created in the Contrail project. Here is an example of isolated project.
```
apiVersion: v1
kind: Namespace
metadata:
  name: private
  annotations: {
    opencontrail.org/isolation: "true"
  }
```


# 3 Distributed SNAT

This feature is released with Contrail 5.0.0. SNAT is done by each vrouter, instead of HAProxy in namespace as previous releases. Source address is vrouter vhost0 address. Source port is allocated from the pool configured by user.

Once distributed SNAT is enabled for a virtual network, containers launched on this virtual network will have the access to underlay.

To enable distributed SNAT, add this annotation to project YAML.
```
opencontrail.org/ip_fabric_snat: "true"
```

For all unisolated projects, update project 'default' with the annotation. It will enable SNAT on virtual network 'k8s-default-pod-network'.
```
oc edit namespace default
# Update YAML with the annotation
```

For isolated project, SNAT will be enabled on the specific virtual network in Contrail.
```
apiVersion: v1
kind: Namespace
metadata:
  name: private
  annotations: {
    opencontrail.org/isolation: "true"
    opencontrail.org/ip_fabric_snat: "true"
  }
```

Note, `SNAT Port Translation Pools` has to be configured on Contrail web UI.
```
Global Config -> Virtual Routers -> Forwarding Options
```


# 4 Pod

```
apiVersion: v1
kind: Pod
metadata:
  name: ubuntu
  labels:
    app: ubuntu
spec:
  containers:
    - name: ubuntu
      image: ubuntu-upstart
```

```
apiVersion: v1
kind: Pod
metadata:
  name: web
  labels:
    app: web
spec:
  containers:
  - name: nginx
    image: nginx
```

# 5 Service

```
apiVersion: v1
kind: ReplicationController
metadata:
  name: web
spec:
  replicas: 2
  selector:
    app: web
  template:
    metadata:
      name: web
      labels:
        app: web
    spec:
      containers:
      - name: nginx
        image: nginx
```

## 5.1 ClusterIP

The default service type is `ClusterIP`. A service IP address (cluster IP) is always allocated for any type of service.


## 5.2 LoadBalancer

Update ConfigMap 'kube-manager' to add this line.
```
KUBERNETES_PUBLIC_FIP_POOL: "{'domain': 'default-domain', 'project': 'default', 'network': 'public-pool', 'name': 'default'}"
```
```
oc edit cm kube-manager-config -n kube-system
```

Delete kube-manager pod to restart it, to apply the update.
```
oc delete pod kube-manager-*
```

```
kind: Service
apiVersion: v1
metadata:
  name: web
spec:
  selector:
    app: web
  ports:
    - protocol: TCP
      port: 80
      targetPort: 80
  type: LoadBalancer
```
Given type 'LoadBalancer', a loadbalancer based on vrouter ECMP is created.

```
# oc get service web
NAME      CLUSTER-IP       EXTERNAL-IP                              PORT(S)        AGE
web       10.111.153.110   172.29.109.75,10.100.0.3,172.29.109.75   80:32147/TCP   2m
```
CLUSTER-IP is the loadbalancer VIP.

Check Contrail, FIP 10.100.0.3 is allocated from configured KUBERNETES_PUBLIC_FIP_POOL.

Login pod 'ubuntu' to check name resovling and connectivity to service.
```
root@ubuntu:/# ping web.default.svc.cluster.local
PING web.default.svc.cluster.local (10.111.153.110) 56(84) bytes of data.
^C
--- web.default.svc.cluster.local ping statistics ---
2 packets transmitted, 0 received, 100% packet loss, time 999ms
```


# 6 CI/DI

By default, when run CI on web UI to build app, the service created by builder is type ClusterIP. The service can be updated with type LoadBalancer to get external IP. When rebuild pod, service remains untouched.

For example, create project 'portal-dev'. Select catalog Python and create application 'portal'. After pod is built, change service to LoadBalancer.
```
oc edit service portal -n portal-dev
```
An EXTERNAL-IP will be allocated.



