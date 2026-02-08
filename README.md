# Common Helm Charts

Production-grade, reusable Helm charts for Kubernetes workloads.

[![Lint and Test](https://github.com/your-org/common-helm-chart/actions/workflows/lint.yaml/badge.svg)](https://github.com/your-org/common-helm-chart/actions/workflows/lint.yaml)
[![Release Charts](https://github.com/your-org/common-helm-chart/actions/workflows/release.yaml/badge.svg)](https://github.com/your-org/common-helm-chart/actions/workflows/release.yaml)

## Available Charts

### [cron-job](./charts/cron-job)

Production-grade Kubernetes CronJob chart with security hardening, flexible scheduling, and environment configuration support.

**Features:**

- Security hardened (non-root, read-only filesystem, dropped capabilities)
- Timezone support (default: GMT+3)
- Concurrency control policies
- External Secrets integration
- Resource management with TTL

## Usage

### Use with ArgoCD (Recommended)

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: my-cronjob
  namespace: argocd
spec:
  project: default
  source:
    repoURL: https://github.com/your-org/common-helm-chart.git
    targetRevision: HEAD
    path: charts/cron-job
    helm:
      values: |
        # Your values here
  destination:
    server: https://kubernetes.default.svc
    namespace: default
```

### Install from GitHub Release

```bash
helm install my-cronjob https://github.com/your-org/common-helm-chart/releases/download/v1.0.0/cron-job-1.0.0.tgz
```

## Development

### Prerequisites

- Helm 3.14+
- Kubernetes 1.25+
- yamllint
- kind (for local testing)

### Linting

```bash
# Lint YAML files
yamllint -c .yamllint charts/

# Lint Helm charts
helm lint charts/cron-job --strict
```

### Testing

```bash
# Template generation
helm template test charts/cron-job --debug

# Local installation (requires Kubernetes cluster)
helm install test-release charts/cron-job \
  --set image.repository=busybox \
  --set image.tag=latest \
  --set 'command[0]=/bin/sh' \
  --set 'args[0]=echo "test"'
```

## CI/CD

### GitHub Actions Workflows

- **Lint and Test** (`.github/workflows/lint.yaml`): Validates YAML, lints Helm charts, runs security scans, tests installation
- **Release** (`.github/workflows/release.yaml`): Detects changed charts, creates GitHub releases with packaged charts (parallel multi-chart support)

### Versioning

This project follows [Semantic Versioning](https://semver.org/). The release workflow automatically:

1. Detects which charts have `Chart.yaml` changes
2. Extracts chart name and version from each changed chart
3. Creates separate releases for each chart (parallel execution)
4. Skips release if version tag already exists

**Release Process:**

- Update version in `charts/{chart-name}/Chart.yaml` (e.g., `1.0.0` → `1.1.0`)
- Commit and push to `main` branch
- Release workflow triggers automatically
- Each chart gets its own release: `cron-job-v1.0.0`, `common-v1.0.0`, etc.

**Multi-chart updates:**

- Update multiple charts in one commit → separate releases created in parallel
- Each release is independent with chart-specific tags

## Contributing

1. Create a feature branch
2. Make changes and update chart version in `charts/{chart-name}/Chart.yaml`
3. Update `CHANGELOG.md` with changes
4. Create PR and ensure CI passes
5. After merge to main, release workflow detects changed charts and creates releases

**Adding a new chart:**

1. Create `charts/new-chart/` directory
2. Add `Chart.yaml`, `values.yaml`, and `templates/`
3. CI will automatically lint, test, and release on merge

## License

MIT
