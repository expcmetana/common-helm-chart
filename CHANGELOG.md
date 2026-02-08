# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.0.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [1.0.0] - 2026-02-08

### Added

- Initial release of cron-job Helm chart
- Security-hardened pod configuration with non-root user
- Support for CronJob scheduling with timezone configuration (default: Europe/Moscow GMT+3)
- Configurable concurrency policies: Forbid, Allow, Replace
- Resource limits and requests configuration
- ServiceAccount with configurable annotations
- Support for environment variables via `env` and `envFrom`
- Optional ConfigMap and Secret generation
- Support for custom volumes and volume mounts
- Node selector, tolerations, and affinity configuration
- TTL for completed jobs
- Comprehensive documentation and examples

### Security

- Non-root execution (UID/GID 1000)
- Read-only root filesystem
- Dropped all capabilities
- SecurityContext with RuntimeDefault seccomp profile
- ServiceAccount token auto-mount disabled by default
