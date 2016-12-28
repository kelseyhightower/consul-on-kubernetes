# Consul Configuration

## Generate TLS Certificates

```
cfssl gencert -initca ca-csr.json | cfssljson -bare ca
```

```
cfssl gencert \
  -ca=ca.pem \
  -ca-key=ca-key.pem \
  -config=ca-config.json \
  -profile=default \
  consul-csr.json | cfssljson -bare consul
```

## Generate the Gossip Encryption Key

```
CONSUL_GOSSIP_ENCRYPTION_KEY=$(consul keygen)
```

## Create the consul secret

```
kubectl create secret generic consul \
  --from-literal="gossip-encryption-key=${CONSUL_GOSSIP_ENCRYPTION_KEY}" \
  --from-file=ca.pem \
  --from-file=consul.pem \
  --from-file=consul-key.pem
```

```
kubectl describe secret consul
```
```
Name:           consul
Namespace:      default
Labels:         <none>
Annotations:    <none>

Type:    Opaque

Data
====
ca.pem:                 1310 bytes
consul-key.pem:         1679 bytes
consul.pem:             1464 bytes
gossip-encryption-key:  24 bytes
```

## Create the consul configmap

```
kubectl create configmap consul --from-file=configs/server.json
```
