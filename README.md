# k8s-inference-platform

Running an LLM inference gateway on Kubernetes — written as raw manifests first,
Helm second, so the generated output can be compared against the hand-written version.

Built on a local two-node cluster (minikube, Kubernetes v1.35, containerd) as a
deliberate constraint: 4 GB per node means resource requests, limits and eviction
behaviour have to be reasoned about rather than ignored.

## Status

Early. The platform is being built one layer at a time.

| | Layer | State |
|---|---|---|
| ✅ | Pods — bare Pod, restart semantics, node placement | done |
| ✅ | Deployments — ReplicaSets, rolling update, rollback | done |
| ✅ | Services — ClusterIP, NodePort, EndpointSlices, in-cluster DNS | done |
| ⬜ | LiteLLM gateway — Deployment + Service | next |
| ⬜ | Config and Secrets — provider keys, model config | |
| ⬜ | Probes — startup vs readiness vs liveness | |
| ⬜ | Storage — Postgres backend on a PVC | |
| ⬜ | Autoscaling — HPA sized to the database, not to the node | |
| ⬜ | Helm — official chart, diffed against these manifests | |

## Layout

```
01-pods/          bare Pod, and why nothing recreates it
02-deployments/   the same workload, managed
```

## Notes

**Image tags are pinned by digest, not by tag.** `:latest` and rolling `-main`
tags are avoided deliberately — a March 2026 incident briefly compromised two
published LiteLLM releases, and a mutable tag offers no protection against that.

**HPA limits are sized to the database, not to available CPU.** Upstream chart
defaults are tuned to install cleanly on any cluster, not to run in production;
scaling a gateway past what its Postgres can serve converts a traffic spike into
a database outage.

## Running it

```sh
minikube start --nodes 2 --kubernetes-version=v1.35.8 \
  --driver=docker --container-runtime=containerd --memory=4096 --cpus=2

kubectl apply -f 02-deployments/
```
