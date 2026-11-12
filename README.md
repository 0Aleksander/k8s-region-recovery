# k8s-region-recovery

Measuring and automating the recovery of a Kubernetes environment after a total region loss.

*Diploma thesis: "Merjenje in avtomatizacija obnove Kubernetes okolja po popolni izgubi regije". VSŠ Informatika, ŠC Nova Gorica, 2026/27.*

## The experiment

A small Kubernetes cluster runs [k8coins](https://github.com/platformfix/k8coins) with its data in Redis. Then the whole region is lost on purpose. Only the Git repo and the last data backup survive. The cluster is rebuilt in another region, and the time until the app is serving again with its data back is measured.

| Version | How the environment comes back |
|---|---|
| V0 | By hand |
| V1 | Infrastructure with Terraform |
| V2 | V1 + apps deployed by Flux (GitOps) |
| V3 | V2 + scripted data restore |

Each version is run 3 times (12 recoveries), measuring recovery time (RTO) and data loss (RPO).

## Status

Work in progress. What broke along the way and why: [`CHALLENGES.md`](CHALLENGES.md).

## Run the app locally

```bash
docker compose up -d --build
curl -s localhost:8000/json   # "hashes" should go up
```

Dashboard: http://localhost:8000. Architecture: [`docs/architecture.md`](docs/architecture.md). Kubernetes: [`docs/helm-chart.md`](docs/helm-chart.md).

## Credits

Based on [k8coins](https://github.com/platformfix/k8coins) by Steve Wade / Platform Fix, itself adapted from DockerCoins by Jérôme Petazzoni. See [`NOTICE`](NOTICE). Licensed under Apache 2.0.
