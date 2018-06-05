# manna
Synchronise multiple containers/nodes competing for limited cloud quotas with a single point of failure.

## Proposed usage 
Create a Deployment with a single replica and a service that exposes it via the Kubernetes DNS addon:
```
kubectl create --save-config -f k8s-dep.yaml
kubectl create --save-config -f k8s-svc.yaml
```
The per-second growth-rate for manna is defined as environment variables in the deployment, along with the maximum attainable value for manna.

Usage in apps running across multiple pods:
```
manna.Timeout(60) // In seconds - give up if no available manna for a minute in this case. 0 = infinite
manna.Protect(func() {
    // This block is synchronous - i.e., returns only when protected func returns
})
```

## Functional Specifications/Behaviour
### In case of available manna
Require a point of manna and execute block. Open finishing the block, release manna back.

### In case of no manna available
Block and poll with a one second sleep until manna is available. Respect the timeout.

### In case of connection failure to the service
Treat this as no manna available.

### In case of pod crashing
On every restart, manna is to grow at its per-second growth rate. Thus, a crash is treated as a reset of manna to zero.
In the future, we can consider exploiting the `SIGTERM` issued by Kubernetes to our pod to save our count-state and elegantly restore it upon pod restart.

### Misc
All clients talk to the master manna note (the central point-of-failure) via gRPC.
