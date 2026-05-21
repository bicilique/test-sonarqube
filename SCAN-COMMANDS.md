# SCAN-COMMANDS

Copy-paste Docker commands for demo scans.

## Before You Run These Commands

- Start SonarQube first with `docker compose up -d`
- Replace placeholder values such as `<your_sonar_token>`, `<your_project_key>`, and `http://host.docker.internal:<port>`
- These examples assume Docker Desktop. On Linux, add host mapping support if `host.docker.internal` does not resolve

## Trivy Filesystem Scan

Scan the current project directory from a Docker container:

```bash
docker run --rm \
  -v "$(pwd):/workspace" \
  -w /workspace \
  aquasec/trivy:latest \
  fs --severity HIGH,CRITICAL .
```

## SonarQube Scan

Send analysis from the current project directory to the SonarQube server started by Compose:

```bash
docker run --rm \
  -v "$(pwd):/usr/src" \
  sonarsource/sonar-scanner-cli:latest \
  -Dsonar.projectKey=<your_project_key> \
  -Dsonar.sources=. \
  -Dsonar.host.url=http://host.docker.internal:9000 \
  -Dsonar.token=<your_sonar_token>
```

## OWASP ZAP Baseline Scan

Run a baseline web scan against a running demo app. Replace the placeholder target URL with your real application URL:

```bash
docker run --rm \
  -v "$(pwd):/zap/wrk/:rw" \
  -t ghcr.io/zaproxy/zaproxy:stable \
  zap-baseline.py \
  -t http://host.docker.internal:<port> \
  -r zap-report.html
```

## Linux Note

If you are not using Docker Desktop and `host.docker.internal` is unavailable, add this option to the Docker commands that must reach services on your host:

```bash
--add-host=host.docker.internal:host-gateway
```
