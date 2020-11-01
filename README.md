# Pre-requistes

helmfile https://github.com/roboll/helmfile and Helm V3

# Deploying radix-core-emulator in a local kubernetes cluster
kind is a tool for running local Kubernetes clusters using Docker container “nodes”. To install it check https://kind.sigs.k8s.io/

```
kind create cluster --config kind/config
```

`NOTE`
Make sure you have enough CPU and Memory. At least 2GB of available RAM is needed

## Deploy radixdlt-emulator
```
helmfile  -f helm/helmfile.yaml apply
```

## Enable ingress
To avoid to have to do Kubernetes port-forwards to access the emulator:

```
helmfile  -f helm/helmfile.yaml apply --set ingress.enabled=true
```

And install an Ingress Controller configured for Kind (https://kind.sigs.k8s.io/docs/user/ingress/#ingress-nginx)
```
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/master/deploy/static/provider/kind/deploy.yaml
```

## Enable persistence
If you want to keep data in case a pod is restarted:
```
helmfile  -f helm/helmfile.yaml apply --set persistence.enabled=true
```


# Deploying radix-core-emulator in an existing cluster
```
helmfile  -f helm/helmfile.yaml apply
```

# Enable persistence
```
helmfile  -f helm/helmfile.yaml apply --set persistence.enabled=true
```

*NOTE* be sure to set the right storageclass with --set persistence.storageClass=NAME_OF_YOUR_STORAGE_CLASS


# Readiness and liveness
I've used the /api/ping endpoint for the readiness. Ideally specifc readiness and livensss endpoints should be created.


# Issues

It looks like there's a value hardcoded somewhere or that an environment variable is not documented and uses a default value.

```
11:25:18,542 ERROR: [messaging] a:231 - 6 -> system:OUTBOUND:a669b78580b3ba1baa0c4d5887c5050f @ 1604229918559 to UDP RADIX://core:30000 ID:00000000000000000000000000000000 failed
java.net.UnknownHostException: core: Name does not resolve
	at java.net.Inet4AddressImpl.lookupAllHostAddr(Native Method)
	at java.net.InetAddress$2.lookupAllHostAddr(InetAddress.java:929)
	at java.net.InetAddress.getAddressesFromNameService(InetAddress.java:1324)
	at java.net.InetAddress.getAllByName0(InetAddress.java:1277)
	at java.net.InetAddress.getAllByName(InetAddress.java:1193)
	at java.net.InetAddress.getAllByName(InetAddress.java:1127)
	at java.net.InetAddress.getByName(InetAddress.java:1077)
	at org.radix.p.z.h.f(SourceFile:67)
	at org.radix.p.u.a.f(SourceFile:223)
	at org.radix.h.c.v.run(SourceFile:47)
	at java.lang.Thread.run(Thread.java:748)
```

To work around that I exposed port 3000 in the service and used `core` as the name of the service so the host could be resolved

