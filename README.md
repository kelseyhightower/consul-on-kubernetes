# Running Consul on Kubernetes

These are my rough notes on how to run a 3 node consul cluster on Kubernetes. You MUST understand how consul works, this is just my notes.

## Overview

- This guide is only showing the high level steps to provision a basic cluster
- The cluster will be insecure. Use the consul docs to help you secure the cluster
- This is NOT production ready. Do not run consul like this in production. These are just notes on the basics.

## What makes this work

- 1 StatefulSet per consul cluster
- 1 Service per consul cluster
- 1 Service to expose the consul UI
- 1 Job to bootstrap the cluster members
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

### Consul Services

Create one service per consul member with a fixed cluster IP. 

```
kubectl create -f services/
```

```
service "consul" created
service "consul-http" created
```

```
kubectl get svc
```
```
NAME          CLUSTER-IP       EXTERNAL-IP   PORT(S)                                                                   AGE
consul        None             <none>        8500/TCP,8400/TCP,8301/TCP,8301/UDP,8302/TCP,8302/UDP,8300/TCP,8600/TCP   3h
consul-http   XX.XXX.XXX.XXX   XX.XXX.XX.X   8500:30624/TCP                                                            12s
kubernetes    XX.XXX.XXX.X     <none>        443/TCP                                                                   4h
```

### Consul StatefulSet

Create the consul StatefulSet:

```
kubectl create -f statefulsets/consul.yaml
```
```
statefulset "consul" created
```

```
kubectl get pods
```
```
NAME       READY     STATUS    RESTARTS   AGE
consul-0   1/1       Running   0          50s
consul-1   1/1       Running   0          29s
consul-2   1/1       Running   0          15s
```

### Join the cluster members

```
kubectl create -f jobs/consul-join.yaml
```
```
job "consul-join" created
```

```
kubectl get jobs
```
```
NAME          DESIRED   SUCCESSFUL   AGE
consul-join   1         1            33s
```

```
kubectl logs consul-0
```
```
==> WARNING: Expect Mode enabled, expecting 3 servers
==> Starting Consul agent...
==> Starting Consul agent RPC...
==> Consul agent running!
           Version: 'v0.7.2'
         Node name: 'consul-0'
        Datacenter: 'dc1'
            Server: true (bootstrap: false)
       Client Addr: 0.0.0.0 (HTTP: 8500, HTTPS: -1, DNS: 8600, RPC: 8400)
      Cluster Addr: 10.176.4.17 (LAN: 8301, WAN: 8302)
    Gossip encrypt: false, RPC-TLS: false, TLS-Incoming: false
             Atlas: <disabled>

==> Log data will now stream in as it occurs:

    2016/12/27 20:25:05 [INFO] raft: Initial configuration (index=0): []
    2016/12/27 20:25:05 [INFO] raft: Node at 10.176.4.17:8300 [Follower] entering Follower state (Leader: "")
    2016/12/27 20:25:05 [INFO] serf: EventMemberJoin: consul-0 10.176.4.17
    2016/12/27 20:25:05 [INFO] serf: EventMemberJoin: consul-0.dc1 10.176.4.17
    2016/12/27 20:25:05 [INFO] consul: Adding LAN server consul-0 (Addr: tcp/10.176.4.17:8300) (DC: dc1)
    2016/12/27 20:25:05 [INFO] consul: Adding WAN server consul-0.dc1 (Addr: tcp/10.176.4.17:8300) (DC: dc1)
    2016/12/27 20:25:11 [WARN] raft: no known peers, aborting election
    2016/12/27 20:25:12 [ERR] agent: failed to sync remote state: No cluster leader
    2016/12/27 20:25:41 [ERR] agent: failed to sync remote state: No cluster leader
    2016/12/27 20:25:41 [ERR] agent: coordinate update error: No cluster leader
    2016/12/27 20:26:07 [ERR] agent: coordinate update error: No cluster leader
    2016/12/27 20:26:10 [ERR] agent: failed to sync remote state: No cluster leader
    2016/12/27 20:26:39 [ERR] agent: coordinate update error: No cluster leader
    2016/12/27 20:26:41 [ERR] agent: failed to sync remote state: No cluster leader
    2016/12/27 20:27:06 [ERR] agent: coordinate update error: No cluster leader
    2016/12/27 20:27:07 [ERR] agent: failed to sync remote state: No cluster leader
    2016/12/27 20:27:34 [ERR] agent: failed to sync remote state: No cluster leader
    2016/12/27 20:27:36 [INFO] agent.rpc: Accepted client: 10.176.4.19:50894
    2016/12/27 20:27:36 [INFO] agent: (LAN) joining: [consul-1.consul.default.svc.cluster.local consul-2.consul.default.svc.cluster.local]
    2016/12/27 20:27:36 [INFO] serf: EventMemberJoin: consul-1 10.176.1.12
    2016/12/27 20:27:36 [INFO] consul: Adding LAN server consul-1 (Addr: tcp/10.176.1.12:8300) (DC: dc1)
    2016/12/27 20:27:36 [INFO] serf: EventMemberJoin: consul-2 10.176.4.18
    2016/12/27 20:27:36 [INFO] agent: (LAN) joined: 2 Err: <nil>
    2016/12/27 20:27:36 [INFO] consul: Adding LAN server consul-2 (Addr: tcp/10.176.4.18:8300) (DC: dc1)
    2016/12/27 20:27:36 [INFO] consul: Found expected number of peers, attempting bootstrap: 10.176.4.17:8300,10.176.1.12:8300,10.176.4.18:8300
    2016/12/27 20:27:38 [WARN] raft: Heartbeat timeout from "" reached, starting election
    2016/12/27 20:27:38 [INFO] raft: Node at 10.176.4.17:8300 [Candidate] entering Candidate state in term 2
    2016/12/27 20:27:38 [INFO] raft: Election won. Tally: 2
    2016/12/27 20:27:38 [INFO] raft: Node at 10.176.4.17:8300 [Leader] entering Leader state
    2016/12/27 20:27:38 [INFO] raft: Added peer 10.176.1.12:8300, starting replication
    2016/12/27 20:27:38 [INFO] raft: Added peer 10.176.4.18:8300, starting replication
    2016/12/27 20:27:38 [INFO] consul: cluster leadership acquired
    2016/12/27 20:27:38 [INFO] consul: New leader elected: consul-0
    2016/12/27 20:27:38 [WARN] raft: AppendEntries to {Voter 10.176.1.12:8300 10.176.1.12:8300} rejected, sending older logs (next: 1)
    2016/12/27 20:27:38 [INFO] raft: pipelining replication to peer {Voter 10.176.4.18:8300 10.176.4.18:8300}
    2016/12/27 20:27:38 [INFO] raft: pipelining replication to peer {Voter 10.176.1.12:8300 10.176.1.12:8300}
    2016/12/27 20:27:38 [INFO] consul: member 'consul-0' joined, marking health alive
    2016/12/27 20:27:38 [INFO] consul: member 'consul-1' joined, marking health alive
    2016/12/27 20:27:38 [INFO] consul: member 'consul-2' joined, marking health alive
    2016/12/27 20:27:39 [INFO] agent: Synced service 'consul'
```


## Cleanup

```
kubectl delete statefulset consul
```

```
kubectl delete pvc data-consul-0 data-consul-1 data-consul-2
```

```
kubectl delete svc consul consul-http
```

```
kubectl delete jobs consul-join
```
