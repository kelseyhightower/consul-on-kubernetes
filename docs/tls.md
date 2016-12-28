# Generate TLS Certificates

```
cd ca
```

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

```
kubectl create secret generic consul \
  --from-file=ca.pem \
  --from-file=consul.pem \
  --from-file=consul-key.pem
```

```
kubectl describe secret consul
Name:		consul
Namespace:	default
Labels:		<none>
Annotations:	<none>

Type:	Opaque

Data
====
ca.pem:		1310 bytes
consul-key.pem:	1675 bytes
consul.pem:	1931 bytes
```


```
kubectl create configmap consul --from-file=configs/server.json
```
