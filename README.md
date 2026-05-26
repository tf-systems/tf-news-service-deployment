# tf-news-service-deployment

GitOps deployment repo for [`tf-news-service`](https://github.com/tf-systems/tf-news-service). Argo CD watches `main` and renders the Helm chart from GHCR with the per-environment values stored here.

[![License](https://img.shields.io/badge/license-Apache--2.0-blue.svg)](LICENSE)

> Part of [`tf-systems`](https://github.com/tf-systems/tf-workspace). The `tf-workspace` repo is the **central entry point** — open it in a Codespace to bring up the full development environment in one container.

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

```
oci://ghcr.io/tf-systems/charts/news-service:<version>
```

Published from [`tf-news-service`](https://github.com/tf-systems/tf-news-service) on each tag (cut automatically by release-please).

## How a release flows

1. `tf-news-service` merges a release-please PR → tag `vX.Y.Z` → CI publishes 4 images + chart to GHCR.
2. [`release.yml`](https://github.com/tf-systems/tf-news-service/blob/main/.github/workflows/release.yml) in `tf-news-service` chains into [`tf-pipeline/gitops-deploy.yml`](https://github.com/tf-systems/tf-pipeline/blob/main/.github/workflows/gitops-deploy.yml), which opens a PR here bumping `image.tag` in [`environments/dev/values.yaml`](environments/dev/values.yaml) and `targetRevision` in [`argocd/dev-application.yaml`](argocd/dev-application.yaml).
3. PR auto-merges into `main`.
4. Argo CD detects the change → syncs the chart → new images roll out in the cluster.
5. Promotion to prod is a manual PR bumping [`environments/prod/values.yaml`](environments/prod/values.yaml) and the prod Argo Application.

## Image visibility

GHCR packages are **public** so kubelet + Argo CD can pull without registry credentials. Source repos stay private; only the build artifacts are public — standard open-source-binary-distribution pattern.

If a future config secret has to live in this repo, switch to private + an Argo CD `repo-creds` Secret backed by a dedicated GitHub App (`tf-argocd-reader`), persisted via Sealed Secrets or similar.

## Editing values

Anyone with write access can hand-edit `environments/<env>/values.yaml`. The chart schema is defined in [`tf-news-service/chart/values.yaml`](https://github.com/tf-systems/tf-news-service/blob/main/chart/values.yaml) — only OVERRIDE the keys that differ from chart defaults.

## Support

For questions and help, see [SUPPORT](https://github.com/tf-systems/.github/blob/main/SUPPORT.md).

## License

[Apache-2.0](LICENSE)
