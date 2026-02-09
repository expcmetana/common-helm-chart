# CronJob Helm Chart

Production-grade Kubernetes CronJob Helm chart with security hardening and best practices.

## Features

- **Security Hardened**: Non-root execution, read-only filesystem, dropped capabilities, seccomp profile
- **Flexible Scheduling**: Cron expressions with timezone support (default: GMT+3)
- **Concurrency Control**: Forbid, Allow, or Replace policies for overlapping jobs
- **Resource Management**: TTL for completed jobs, history limits, backoff policies
- **Environment Configuration**: Support for env, envFrom, ConfigMaps, and Secrets
- **Cloud Native**: ServiceAccount, RBAC-ready, External Secrets integration

## Installation

### From Git Repository

```bash
helm install my-cronjob oci://ghcr.io/your-org/common-helm-chart/cron-job --version 1.2.6
```

### From Local Path

```bash
helm install my-cronjob ./charts/cron-job -f values.yaml
```

### With ArgoCD

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

## Configuration

### Basic Example

```yaml
cronjob:
  schedule: "0 2 * * *" # Daily at 2 AM

image:
  repository: busybox
  tag: "1.36"

command:
  - /bin/sh
  - -c

args:
  - echo "Hello World"
```

### Database Backup Example

```yaml
nameOverride: "db-backup"

cronjob:
  schedule: "0 2 * * *"
  timeZone: "Europe/Moscow"
  concurrencyPolicy: Forbid
  restartPolicy: OnFailure

job:
  backoffLimit: 3
  activeDeadlineSeconds: 3600
  ttlSecondsAfterFinished: 86400

image:
  repository: postgres
  tag: "16-alpine"

command:
  - /bin/sh
  - -c

args:
  - |
    pg_dump -h $DB_HOST -U $DB_USER $DB_NAME > /backup/backup-$(date +%Y%m%d-%H%M%S).sql

envFrom:
  - secretRef:
      name: db-credentials

volumes:
  - name: backup
    persistentVolumeClaim:
      claimName: backup-pvc

volumeMounts:
  - name: backup
    mountPath: /backup
```

### External Secrets Integration

```yaml
envFrom:
  - secretRef:
      name: api-env # Managed by External Secrets Operator

env:
  - name: ENVIRONMENT
    value: "production"
  - name: API_KEY
    valueFrom:
      secretKeyRef:
        name: app-secrets
        key: api-key
```

## Parameters

### CronJob Configuration

| Parameter                            | Description                                                       | Default           |
| ------------------------------------ | ----------------------------------------------------------------- | ----------------- |
| `cronjob.schedule`                   | Cron schedule expression                                          | `"0 0 * * *"`     |
| `cronjob.timeZone`                   | Timezone for schedule                                             | `"Europe/Moscow"` |
| `cronjob.concurrencyPolicy`          | How to handle concurrent executions: `Forbid`, `Allow`, `Replace` | `Forbid`          |
| `cronjob.successfulJobsHistoryLimit` | Number of successful jobs to keep                                 | `3`               |
| `cronjob.failedJobsHistoryLimit`     | Number of failed jobs to keep                                     | `1`               |
| `cronjob.startingDeadlineSeconds`    | Deadline to start the job if missed                               | `300`             |
| `cronjob.suspend`                    | Suspend all executions                                            | `false`           |
| `cronjob.restartPolicy`              | Pod restart policy: `OnFailure` or `Never`                        | `OnFailure`       |

### Job Configuration

| Parameter                     | Description                                    | Default |
| ----------------------------- | ---------------------------------------------- | ------- |
| `job.backoffLimit`            | Number of retries before marking job as failed | `3`     |
| `job.activeDeadlineSeconds`   | Maximum execution time in seconds              | `600`   |
| `job.ttlSecondsAfterFinished` | Time to keep completed jobs                    | `86400` |

### Image Configuration

| Parameter          | Description                | Default        |
| ------------------ | -------------------------- | -------------- |
| `image.repository` | Container image repository | `""`           |
| `image.tag`        | Container image tag        | `""`           |
| `image.pullPolicy` | Image pull policy          | `IfNotPresent` |
| `imagePullSecrets` | Image pull secrets         | `[]`           |

### Security Context

| Parameter                                  | Description                        | Default |
| ------------------------------------------ | ---------------------------------- | ------- |
| `podSecurityContext.runAsUser`             | User ID to run container           | `1000`  |
| `podSecurityContext.runAsGroup`            | Group ID to run container          | `1000`  |
| `podSecurityContext.fsGroup`               | Filesystem group ID                | `1000`  |
| `securityContext.readOnlyRootFilesystem`   | Mount root filesystem as read-only | `true`  |
| `securityContext.allowPrivilegeEscalation` | Allow privilege escalation         | `false` |

### Resources

| Parameter                   | Description    | Default |
| --------------------------- | -------------- | ------- |
| `resources.limits.cpu`      | CPU limit      | `500m`  |
| `resources.limits.memory`   | Memory limit   | `512Mi` |
| `resources.requests.cpu`    | CPU request    | `100m`  |
| `resources.requests.memory` | Memory request | `128Mi` |

## Concurrency Policy

The `concurrencyPolicy` determines how Kubernetes handles overlapping job executions:

- **Forbid** (Recommended): Prevents concurrent runs. If the previous job is still running when a new schedule time arrives, the new execution is skipped. Best for jobs that shouldn't run simultaneously (e.g., database backups, data processing).

- **Allow**: Permits multiple jobs to run at the same time. Use with caution as it can lead to resource contention and race conditions.

- **Replace**: Terminates the currently running job and starts a new one. Useful for jobs where only the latest execution matters.

## Environment Variables

### Direct Values

```yaml
env:
  - name: LOG_LEVEL
    value: "info"
  - name: ENVIRONMENT
    value: "production"
```

### From ConfigMap or Secret

```yaml
env:
  - name: DATABASE_URL
    valueFrom:
      secretKeyRef:
        name: db-credentials
        key: url
  - name: API_ENDPOINT
    valueFrom:
      configMapKeyRef:
        name: app-config
        key: api_endpoint
```

### From Entire ConfigMap/Secret

```yaml
envFrom:
  - secretRef:
      name: api-env
  - configMapRef:
      name: app-settings
```

## Timezone Configuration

The chart uses `Europe/Moscow` (GMT+3) by default. To change:

```yaml
cronjob:
  timeZone: "America/New_York"  # EST/EDT
  # or
  timeZone: "Asia/Tokyo"        # JST
  # or
  timeZone: "UTC"               # Coordinated Universal Time
```

Valid timezone values: [IANA Time Zone Database](https://en.wikipedia.org/wiki/List_of_tz_database_time_zones)

## Testing

### Lint Chart

```bash
helm lint charts/cron-job
```

### Dry Run

```bash
helm install test-cronjob charts/cron-job --dry-run --debug
```

### Template Output

```bash
helm template test-cronjob charts/cron-job
```

### Manual Trigger

```bash
kubectl create job --from=cronjob/my-cronjob my-cronjob-manual
```

## Monitoring

### Check CronJob Status

```bash
kubectl get cronjob my-cronjob
```

### View Job History

```bash
kubectl get jobs -l app.kubernetes.io/instance=my-cronjob
```

### View Logs

```bash
kubectl logs -l app.kubernetes.io/instance=my-cronjob --tail=100
```

### Describe CronJob

```bash
kubectl describe cronjob my-cronjob
```

## Troubleshooting

### Job Not Starting

Check starting deadline:

```yaml
cronjob:
  startingDeadlineSeconds: 600 # Increase if nodes are slow
```

### Jobs Running Concurrently

Ensure proper concurrency policy:

```yaml
cronjob:
  concurrencyPolicy: Forbid
```

### Pod Crashing

Check security context if using read-only filesystem:

```yaml
securityContext:
  readOnlyRootFilesystem: false # Disable if app needs write access

volumeMounts:
  - name: tmp
    mountPath: /tmp

volumes:
  - name: tmp
    emptyDir: {}
```

## Security Best Practices

1. **Never use `root` user**: Keep `runAsNonRoot: true`
2. **Use read-only filesystem** when possible
3. **Drop all capabilities**: Keep `capabilities.drop: [ALL]`
4. **Use External Secrets**: Don't embed secrets in values
5. **Limit resources**: Always set limits and requests
6. **Use specific image tags**: Avoid `latest` tag
7. **Enable seccomp profile**: Keep `seccompProfile.type: RuntimeDefault`

## License

MIT
