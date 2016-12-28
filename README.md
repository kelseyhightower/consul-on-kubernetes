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

### Create the Consule Secret and Configmap

```
kubectl create secret generic consul \
  --from-literal="gossip-encryption-key=${CONSUL_GOSSIP_ENCRYPTION_KEY}" \
  --from-file=ca.pem \
  --from-file=consul.pem \
  --from-file=consul-key.pem
```

```
kubectl create configmap consul --from-file=configs/server.json
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

### Verification

```
kubectl logs consul-0
```

```
kubectl port-forward consul-0 8400:8400
```
```
Forwarding from 127.0.0.1:8400 -> 8400
Forwarding from [::1]:8400 -> 8400
```

```
consul members
```
```
Node      Address           Status  Type    Build  Protocol  DC
consul-0  10.176.4.30:8301  alive   server  0.7.2  2         dc1
consul-1  10.176.4.31:8301  alive   server  0.7.2  2         dc1
consul-2  10.176.1.16:8301  alive   server  0.7.2  2         dc1
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
