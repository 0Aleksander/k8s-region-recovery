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

Redis currently has **no persistence** (no volume, no RDB/AOF), and its
password is generated fresh on every install. Both are being changed for the
recovery experiment, see [`CHALLENGES.md`](../CHALLENGES.md).

`rng`, `hasher`, `worker` and `webui` reach each other and `redis` by fixed
Service names hardcoded in each service's source code, so the chart's Service
names are fixed too, not prefixed with the release name. Only one release can
run per namespace.

Originally from [platformfix/k8coins](https://github.com/platformfix/k8coins),
where the chart is also published as an OCI artifact and a Helm repo.
