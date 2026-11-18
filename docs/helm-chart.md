# Helm chart

The chart at [`chart/k8coins`](../chart/k8coins) deploys all five services to a
Kubernetes cluster. In this repo it is installed from the folder, not from a
published chart registry:

```bash
helm install k8coins ./chart/k8coins
```

The four app images come from the upstream k8coins release, pinned in
[`values.yaml`](../chart/k8coins/values.yaml) (`image.tag: v0.5.17`), so every
recovery run deploys exactly the same versions. See `values.yaml` for what's
configurable: per-service resource requests/limits, autoscaling (off by
default), and an optional Ingress for `webui` (also off; for local access use
`kubectl port-forward svc/webui 8000:80`).

## Redis persistence (optional)

Off by default, so existing installs don't change. Turn it on with:

```bash
helm install k8coins ./chart/k8coins --set redis.persistence.enabled=true
```

| Value | Default | What |
|---|---|---|
| `redis.persistence.enabled` | `false` | Create a PVC (`redis-data`) and mount it at `/data` |
| `redis.persistence.size` | `1Gi` | The data is a counter plus a few thousand hashes, and cloud disks round up to ~1Gi anyway |
| `redis.persistence.storageClass` | `""` | Empty = the line is left out, so the cluster's default class is used. Set a name to pick a disk type |

With persistence on, Redis runs with AOF (`--appendonly yes`) and the Deployment
uses `strategy: Recreate`: the disk is `ReadWriteOnce` (one node at a time), so
the old pod must stop before the new one can attach it. The cost is a few
seconds without Redis during an update. Why not `ReadWriteMany`: see
[`CHALLENGES.md`](../CHALLENGES.md).

Tested on a one-node k3s (18 Nov): data kept across a `rollout restart`. On one
node the RWO conflict can't show up, so the real check is a multi-node cluster.

The Redis password is generated on install. It's a setting, not data (it isn't
stored in the AOF), so restored data works with a new password.

`rng`, `hasher`, `worker` and `webui` reach each other and `redis` by fixed
Service names hardcoded in each service's source code, so the chart's Service
names are fixed too, not prefixed with the release name. Only one release can
run per namespace.

Originally from [platformfix/k8coins](https://github.com/platformfix/k8coins),
where the chart is also published as an OCI artifact and a Helm repo.
