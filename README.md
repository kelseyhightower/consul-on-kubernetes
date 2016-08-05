# Running Consul on Kubernetes

These are my rough notes on how to run a 3 node consul cluster on Kubernetes. You MUST understand how consul works, this is just my notes.

## Overview

- This guide is only showing the high level steps to provision a basic cluster
- The cluster will be insecure. Use the consul docs to help you secure the cluster
- This is NOT production ready. Do not run consul like this in production. These are just notes on the basics.

## What makes this work

- 1 deployment per consul member
- 1 service per consul member
- 1 service to expose the consul UI
- Use `spec.securityContext.fsGroup` to ensure the volume is writable by the consul process which is running as non-root.

```
spec:
  securityContext:
    fsGroup: 1000
```

## Usage

Clone this repo:

```
git clone https://github.com/kelseyhightower/consul-on-kubernetes.git
```

```
cd consul-on-kubernetes
```

### Create Volumes

```
gcloud compute disks create consul-1 consul-2 consul-3
```

### Consul Services

Create one service per consul member with a fixed cluster IP. 

```
kubectl create -f services/
```

```
service "consul-1" created
service "consul-2" created
service "consul-3" created
service "consul-http" created
```

```
kubectl get svc
NAME          CLUSTER-IP       EXTERNAL-IP     PORT(S)                                                                   AGE
consul-1      10.215.243.61    <none>          8500/TCP,8400/TCP,8301/TCP,8301/UDP,8302/TCP,8302/UDP,8300/TCP,8600/TCP   5m
consul-2      10.215.243.62    <none>          8500/TCP,8400/TCP,8301/TCP,8301/UDP,8302/TCP,8302/UDP,8300/TCP,8600/TCP   5m
consul-3      10.215.243.63    <none>          8500/TCP,8400/TCP,8301/TCP,8301/UDP,8302/TCP,8302/UDP,8300/TCP,8600/TCP   5m
consul-http   10.215.243.191   104.198.9.159   8500/TCP                                                                  29m
kubernetes    10.215.240.1     <none>          443/TCP                                                                   13d
```

### Consul Deployment

Create one deployment per consul member.

```
kubectl create -f deployments/
```
```
deployment "consul-1" created
deployment "consul-2" created
deployment "consul-3" created
```

```
kubectl get pods
```
```
NAME                        READY     STATUS    RESTARTS   AGE
consul-1-3104874582-6o4n4   1/1       Running   0          1m
consul-2-3080431481-7mf6r   1/1       Running   0          1m
consul-3-3678840700-3bp3k   1/1       Running   0          1m
```

### Verification

```
kubectl logs consul-1-3104874582-6o4n4
```
```
==> WARNING: Expect Mode enabled, expecting 3 servers
==> Starting Consul agent...
==> Starting Consul agent RPC...
==> Consul agent running!
         Node name: 'consul-1-3104874582-6o4n4'
        Datacenter: 'dc1'
            Server: true (bootstrap: false)
       Client Addr: 0.0.0.0 (HTTP: 8500, HTTPS: -1, DNS: 8600, RPC: 8400)
      Cluster Addr: 10.215.243.61 (LAN: 8301, WAN: 8302)
    Gossip encrypt: false, RPC-TLS: false, TLS-Incoming: false
             Atlas: <disabled>

==> Log data will now stream in as it occurs:

    2016/08/05 19:48:36 [INFO] raft: Node at 10.215.243.61:8300 [Follower] entering Follower state
    2016/08/05 19:48:36 [INFO] serf: EventMemberJoin: consul-1-3104874582-6o4n4 10.215.243.61
    2016/08/05 19:48:36 [INFO] serf: EventMemberJoin: consul-1-3104874582-6o4n4.dc1 10.215.243.61
    2016/08/05 19:48:36 [INFO] consul: adding LAN server consul-1-3104874582-6o4n4 (Addr: 10.215.243.61:8300) (DC: dc1)
    2016/08/05 19:48:36 [INFO] consul: adding WAN server consul-1-3104874582-6o4n4.dc1 (Addr: 10.215.243.61:8300) (DC: dc1)
    2016/08/05 19:48:36 [ERR] agent: failed to sync remote state: No cluster leader
    2016/08/05 19:48:38 [WARN] raft: EnableSingleNode disabled, and no known peers. Aborting election.
consul $ kubectl logs consul-1-3104874582-6o4n4 -f
==> WARNING: Expect Mode enabled, expecting 3 servers
==> Starting Consul agent...
==> Starting Consul agent RPC...
==> Consul agent running!
         Node name: 'consul-1-3104874582-6o4n4'
        Datacenter: 'dc1'
            Server: true (bootstrap: false)
       Client Addr: 0.0.0.0 (HTTP: 8500, HTTPS: -1, DNS: 8600, RPC: 8400)
      Cluster Addr: 10.215.243.61 (LAN: 8301, WAN: 8302)
    Gossip encrypt: false, RPC-TLS: false, TLS-Incoming: false
             Atlas: <disabled>

==> Log data will now stream in as it occurs:

    2016/08/05 19:48:36 [INFO] raft: Node at 10.215.243.61:8300 [Follower] entering Follower state
    2016/08/05 19:48:36 [INFO] serf: EventMemberJoin: consul-1-3104874582-6o4n4 10.215.243.61
    2016/08/05 19:48:36 [INFO] serf: EventMemberJoin: consul-1-3104874582-6o4n4.dc1 10.215.243.61
    2016/08/05 19:48:36 [INFO] consul: adding LAN server consul-1-3104874582-6o4n4 (Addr: 10.215.243.61:8300) (DC: dc1)
    2016/08/05 19:48:36 [INFO] consul: adding WAN server consul-1-3104874582-6o4n4.dc1 (Addr: 10.215.243.61:8300) (DC: dc1)
    2016/08/05 19:48:36 [ERR] agent: failed to sync remote state: No cluster leader
    2016/08/05 19:48:38 [WARN] raft: EnableSingleNode disabled, and no known peers. Aborting election.
    2016/08/05 19:48:53 [INFO] serf: EventMemberJoin: consul-3-3678840700-3bp3k 10.215.243.63
    2016/08/05 19:48:53 [INFO] consul: adding LAN server consul-3-3678840700-3bp3k (Addr: 10.215.243.63:8300) (DC: dc1)
    2016/08/05 19:48:56 [ERR] agent: coordinate update error: No cluster leader
    2016/08/05 19:48:57 [ERR] agent: failed to sync remote state: No cluster leader
    2016/08/05 19:49:16 [INFO] serf: EventMemberJoin: consul-2-3080431481-7mf6r 10.215.243.62
    2016/08/05 19:49:16 [INFO] consul: adding LAN server consul-2-3080431481-7mf6r (Addr: 10.215.243.62:8300) (DC: dc1)
    2016/08/05 19:49:16 [INFO] consul: Attempting bootstrap with nodes: [10.215.243.61:8300 10.215.243.63:8300 10.215.243.62:8300]
    2016/08/05 19:49:17 [INFO] consul: New leader elected: consul-3-3678840700-3bp3k
    2016/08/05 19:49:19 [INFO] agent: Synced service 'consul'
```
