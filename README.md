# tf-news-service-deployment

GitOps deployment repo for `tim-feldhaus/tf-news-service`. Argo CD watches this repo and renders the Helm chart from GHCR with the per-environment values stored here.

## Layout

```
environments/
├── dev/values.yaml          # values for dev environment (kind cluster)
└── prod/values.yaml         # values for prod environment (skeleton)
argocd/
├── dev-application.yaml     # Argo CD Application — auto-sync
└── prod-application.yaml    # Argo CD Application — manual sync
```

## Source chart

Chart is pulled as an OCI artifact:

    oci://ghcr.io/tim-feldhaus/charts/news-service:<version>

Published from [`tf-news-service`](https://github.com/tim-feldhaus/tf-news-service) on tag push.

## How a release flows

1. `tf-news-service` tags `vX.Y.Z` → CI publishes 4 images + chart to GHCR.
2. `tf-pipeline/gitops-promote.yml` (called from `tf-news-service`'s promote workflow, Phase 7) opens a PR here bumping `image.tag` in `environments/dev/values.yaml` and `targetRevision` in `argocd/dev-application.yaml`.
3. PR auto-merges into `main`.
4. Argo CD detects the change → syncs the chart → new images roll out.
5. Promotion to prod is a manual PR bumping `environments/prod/values.yaml`.

## Image visibility

GHCR packages inherit visibility from their source repo. `tf-news-service` is private → the 4 images and the chart are private. For Argo CD (running in cluster) to pull, two options:

- **Make the packages public** (Settings → Packages on each package → Change visibility). Repo stays private; only the artifacts are public.
- **Configure imagePullSecrets** in the cluster. Argo CD's `argocd-cm` needs a Helm registry config; Kubernetes needs a `regcred` secret for the image pulls.

For initial migration, **making packages public is simpler** and avoids secret management. Revisit when prod cluster comes online.

## Editing values

Anyone with write access can hand-edit `environments/<env>/values.yaml`. The chart schema is defined in [`tf-news-service/chart/values.yaml`](https://github.com/tim-feldhaus/tf-news-service/blob/main/chart/values.yaml) — only OVERRIDE the keys that differ from chart defaults.
