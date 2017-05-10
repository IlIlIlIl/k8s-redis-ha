# Kubernetes Redis with High Availability

Kubernetes Redis Images and Samples implemented using the latest features:

* StatefulSet
* Init Container


# Requirements

* Kubernetes 1.6.* cluster
* Redis 3.2


# Quick Start

If you have already a Kubernetes cluster, you can deploy High Availability Redis using the command below:

```console
$ kubectl create -f example/
service "redis-sentinel" created
statefulset "redis-sentinel" created
service "redis-server" created
statefulset "redis-server" created
```


## Accessing redis

The way to access Redis in the cluster is to create a pod like below:

```console
$ kubectl create -f test/console.yaml
pod "console" created
```

And the pod "console" based on [`library/redis`](https://hub.docker.com/_/redis/),
so you can access Redis through the pod "console" using `kubectl exec -ti console /bin/bash`:

```console
$ export MASTER_IP="$(kubectl exec console -- redis-cli -h redis-sentinel -p 26379 sentinel get-master-addr-by-name mymaster | head -1)"
$ export SERVER_PASS="_redis-server._tcp.redis-server.default.svc.cluster.local"
$ # set foo=bar 
$ kubectl exec console -- redis-cli -h "${MASTER_IP}" -a "${SERVER_PASS}" set foo bar
OK
$ # get foo
$ kubectl exec -- redis-cli -h redis-server -a "${SERVER_PASS}" get foo 
bar
```

If you aren't on 'default' namespace, please replace default in `$SERVER_PASS` with your namespace:

```console
$ # if your namespace is 'abcde' ...
$ export SERVER_PASS="_redis-server._tcp.redis-server.abcde.svc.cluster.local"
```


## Scale in out

`tarosky/k8s-redis-ha` has abilities to scale in and scale out.
You can check this feature below:

```console
kubectl scale --replicas=5 statefulset/redis-sentinel
kubectl scale --replicas=5 statefulset/redis-server
```


# Sample code in Python to use High Availability Redis 

One of the simplest way to access Redis in cluster is port-foward which connects a local port with a remote container port:

```console
$ kubectl port-forward redis-server-0 6379:6379
Forwarding from 127.0.0.1:6379 -> 6379
Forwarding from [::1]:6379 -> 6379
```

'redis-server-0' is a Pod which are created from StatefulSet.
Here is the sample code on ipython which access Redis through redis-py.
To run code below, you must install [`redis-py`](https://pypi.python.org/pypi/redis):

```python
In [1]: from redis import StrictRedis

In [2]: a = StrictRedis(password='_redis-server._tcp.redis-server.default.svc.cluster.local')

In [3]: a.get('foo')
Out[3]: b'bar'
```


# Running test script

To check if Pods and others generated from `example/` and `images/` have High Availability, Test script is in `test/`.
Test script requires Python virtual environment to run it.
Here is an example how to build test environment.

```console
$ pyvenv .venv
$ source .venv/bin/activate
$ pip install -r test/requirements.txt
```

You can run test command using command below.

```console
$ KUBE_NAMESPACE='{{Your name space}}' nosetests test/test.py
```


