# Running Consul on Kubernetes

## Usage

Clone this repo:

```
git clone https://github.com/kelseyhightower/consul-on-kubernetes.git
```

```
cd consul-on-kubernetes
```

### Generate TLS Certificates

```
cfssl gencert -initca ca/ca-csr.json | cfssljson -bare ca
```

```
cfssl gencert \
  -ca=ca.pem \
  -ca-key=ca-key.pem \
  -config=ca/ca-config.json \
  -profile=default \
  ca/consul-csr.json | cfssljson -bare consul
```

### Generate the Consul Gossip Encryption Key

```
CONSUL_GOSSIP_ENCRYPTION_KEY=$(consul keygen)
```

### Create the Consul Service

```
kubectl create -f services/consul.yaml
```

### Create the Consul StatefulSet

```
kubectl create -f statefulsets/consul.yaml
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

### Join the Consul Cluster Members

```
kubectl create -f jobs/consul-join.yaml
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

```
kubectl delete secrets consul
```

```
kubectl delete configmaps consul
```
