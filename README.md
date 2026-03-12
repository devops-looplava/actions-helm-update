# actions-helm-update

Composite GitHub Action that updates a Helm chart's `values.yaml` with a new image tag, updates cron job tags, and optionally packages and uploads the chart to ChartMuseum.

Used by `ci-main` and `ci-test` pipelines for Helm-based deployments.

## Usage

Production (full pipeline with chart upload):

```yaml
- name: Update Helm Chart
  uses: devops-looplava/actions-helm-update@main
  with:
    HELM_REPO: onoctave/helm-onoctave
    HELM_TOKEN: ${{ secrets.GH_PAT_DEVOPS_HELM }}
    REPO: ${{ github.repository }}
    UPDATED_TAG: prod-${{ needs.build-artifact-and-scan.outputs.version }}
    PUSH_CHANGES: "true"
    UPLOAD_CHART: "true"
```

Test (update values only, no push, no chart upload):

```yaml
- name: Update Helm Chart
  uses: devops-looplava/actions-helm-update@main
  with:
    HELM_REPO: onoctave/helm-onoctave
    HELM_TOKEN: ${{ secrets.GH_PAT_DEVOPS_HELM }}
    REPO: ${{ github.repository }}
    UPDATED_TAG: test-${{ github.sha }}
    PUSH_CHANGES: "false"
    UPLOAD_CHART: "false"
```

## Inputs

| Input | Required | Default | Description |
|-------|----------|---------|-------------|
| `HELM_REPO` | Yes | - | Helm chart GitHub repository |
| `HELM_TOKEN` | Yes | - | GitHub PAT for the Helm repo |
| `CHART_DIR` | No | `onoctave` | Chart directory inside the repo |
| `REPO` | Yes | - | Service GitHub repository (`owner/name`) |
| `REPO_NAME` | No | *(derived from REPO)* | Repository short name |
| `UPDATED_TAG` | Yes | - | New image tag |
| `VALUES_FILE` | No | `values.yaml` | Path to values.yaml relative to CHART_DIR |
| `ECR_ACCOUNT_ID` | No | `888577026587` | ECR account ID for cron updates |
| `ECR_REGION` | No | `us-west-2` | ECR region |
| `PUSH_CHANGES` | No | `true` | Push commits to origin |
| `UPLOAD_CHART` | No | `true` | Package and upload chart to ChartMuseum |
| `CHARTMUSEUM_URL` | No | `http://chartmuseum.chartmuseum:8080/api/charts` | ChartMuseum API URL |
| `HELM_CHART_BUCKET` | No | `onoctave-helm-charts` | S3 bucket for chart version lookup |

## What it does

1. Checks out the Helm chart repository
2. Installs `yq`
3. Derives service name from repo (`onoctave-some-service` -> `someService`)
4. Updates the service's `.image.tag` in `values.yaml` using `yq`
5. Updates cron job image tags using `awk` pattern matching
6. Commits and pushes changes (if `PUSH_CHANGES=true`)
7. Calculates next chart version from S3 bucket (if `UPLOAD_CHART=true`)
8. Packages Helm chart and uploads to ChartMuseum (if `UPLOAD_CHART=true`)
