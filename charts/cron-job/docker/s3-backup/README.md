# S3 Backup Image

Custom image based on `amazon/aws-cli` with `tar` and `gzip` pre-installed for non-root execution.

## Build and Push

```bash
cd charts/cron-job/docker/s3-backup

# Build
docker build -t ghcr.io/expcmetana/s3-backup:2.33.17 .

# Test
docker run --rm --user 1000:1000 ghcr.io/expcmetana/s3-backup:2.33.17 sh -c "aws --version && tar --version"

# Push
docker push ghcr.io/expcmetana/s3-backup:2.33.17
```

## Security

- Runs as UID 1000 (non-root)
- Compatible with restrictive Pod Security Policies
- Minimal additions to base aws-cli image (only tar + gzip)
