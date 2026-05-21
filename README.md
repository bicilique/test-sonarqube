# test-sonarqube

Demo project for running SonarQube with PostgreSQL via Docker Compose and using Docker-based security scan commands for demos.

## What This Repo Provides

- SonarQube Community running on port `9000`
- PostgreSQL backing SonarQube
- Persistent Docker volumes for SonarQube data, logs, extensions, and Postgres data
- A bind mount from the current project directory to `/workspace` inside the `sonarqube` container

## Prerequisites

- Docker Desktop or Docker Engine with Docker Compose
- Enough memory for SonarQube to start cleanly
- A local project directory you want to scan

## Start The Stack

Run the SonarQube stack in the background:

```bash
docker compose up -d
```

Stop the stack:

```bash
docker compose down
```

Open SonarQube:

```text
http://localhost:9000
```

## Compose Layout

The current [`docker-compose.yml`](/Users/balaisertifikasielektronik/IdeaProjects/Github/project/test-sonarqube/docker-compose.yml) starts:

- `sonarqube` using `sonarqube:26.5.0.122743-community`
- `db` using `postgres:15`

It also mounts the current folder into the SonarQube container at `/workspace`, which is useful for demo scenarios where the project source lives beside the Compose file.

## Demo Flow

Use this repo for a simple demo flow:

1. Start SonarQube with Docker Compose.
2. Open `http://localhost:9000` and prepare a project/token in SonarQube.
3. Run Docker-based scan commands for:
   - Trivy filesystem scan
   - SonarQube analysis upload
   - OWASP ZAP baseline scan against a running demo app

All copy-paste commands live in [SCAN-COMMANDS.md](/Users/balaisertifikasielektronik/IdeaProjects/Github/project/test-sonarqube/SCAN-COMMANDS.md).

## Persistent Data

The Compose stack uses named volumes so data survives container restarts:

- `sonarqube_data`
- `sonarqube_logs`
- `sonarqube_extensions`
- `postgresql_data`

If you want a clean reset, remove the stack and the named volumes manually.
