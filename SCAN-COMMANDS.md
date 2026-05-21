# SCAN-COMMANDS

Copy-paste Docker commands for demo scans.

## Before You Run These Commands

- Start SonarQube first with `docker compose up -d`
- Replace placeholder values such as `<your_sonar_token>`, `<your_project_key>`, and `http://host.docker.internal:<port>`
- These examples assume Docker Desktop. On Linux, add host mapping support if `host.docker.internal` does not resolve
- If your branch was seeded using `.env ADMIN_PASSWORD`, replace `lesson-01-admin123` with that value

---

## Trivy Filesystem Scan

Scan the current project directory from a Docker container:

```bash
docker run --rm \
  -v "$(pwd):/workspace" \
  -w /workspace \
  aquasec/trivy:latest \
  fs --severity HIGH,CRITICAL .
```

---

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

---

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

---

## First Scan with SonarQube CLI

### Linux/macOS

```bash
docker run --rm \
  --network host \
  -e SONAR_HOST_URL="http://localhost:9000" \
  -e SONAR_TOKEN="YOUR_TOKEN_HERE" \
  -v "$PWD:/usr/src" \
  sonarsource/sonar-scanner-cli \
  -D"sonar.projectKey=local-project" \
  -D"sonar.sources=src" \
  -D"sonar.tests=tests" \
  -D"sonar.exclusions=node_modules/**,src/public/uploads/**"
```

### Windows PowerShell

```powershell
docker run --rm `
  -e SONAR_HOST_URL="http://host.docker.internal:9000" `
  -e SONAR_TOKEN="$TOKEN" `
  -v "${PWD}:/usr/src" `
  sonarsource/sonar-scanner-cli `
  -D"sonar.projectKey=$PROJECT_KEY" `
  -D"sonar.sources=src" `
  -D"sonar.tests=tests" `
  -D"sonar.exclusions=node_modules/**,src/public/uploads/**"
```

---

## First Scan with Trivy

### Linux/macOS

```bash
docker run --rm \
  -v "$PWD:/work" \
  -w /work \
  aquasec/trivy fs .
```

### Windows PowerShell

```powershell
docker run --rm `
  -v "${PWD}:/work" `
  -w /work `
  aquasec/trivy fs .
```

### Linux/macOS with JSON Report

```bash
docker run --rm \
  -v "$PWD:/work" \
  -w /work \
  aquasec/trivy fs \
  --format json \
  --output trivy-report.json \
  .
```

### Windows PowerShell with JSON Report

```powershell
docker run --rm `
  -v "${PWD}:/work" `
  -w /work `
  aquasec/trivy fs `
  --format json `
  --output trivy-report.json `
  .
```

---

## Linux Note

If you are not using Docker Desktop and `host.docker.internal` is unavailable, add this option to the Docker commands that must reach services on your host:

```bash
--add-host=host.docker.internal:host-gateway
```

---

# ZAP Authenticated Baseline Scan

This section shows how to run an authenticated OWASP ZAP baseline scan using Docker.

The flow is:

1. Log in with `curl` and save the session cookie.
2. Build a `Cookie` header from the saved cookie jar.
3. Run `zap-baseline.py` with the cookie injected into requests.

Target page:

```text
http://host.docker.internal:3000/admin/products
```

---

## Linux/macOS Bash Version

### Step 1: Save cookie jar

```bash
curl -i \
  -c /tmp/minicart.cookies \
  -d 'username=admin&password=lesson-01-admin123' \
  -H 'Content-Type: application/x-www-form-urlencoded' \
  http://127.0.0.1:3000/login
```

If your branch was seeded using `.env ADMIN_PASSWORD`, replace:

```text
lesson-01-admin123
```

with that value.

---

### Step 2: Build cookie header

```bash
SESSION_COOKIE="$(awk 'BEGIN{FS="\t"} !/^#/ {print $6 "=" $7}' /tmp/minicart.cookies | paste -sd '; ' -)"
echo "$SESSION_COOKIE"
```

---

### Step 3: Run authenticated ZAP baseline

```bash
mkdir -p reports

docker run --rm \
  -v "$PWD/reports:/zap/wrk/:rw" \
  ghcr.io/zaproxy/zaproxy:stable \
  zap-baseline.py \
  -t http://host.docker.internal:3000/admin/products \
  -m 1 \
  -I \
  -r zap-auth.html \
  -w zap-auth.md \
  -z "-config replacer.full_list(0).description=authcookie -config replacer.full_list(0).enabled=true -config replacer.full_list(0).matchtype=REQ_HEADER -config replacer.full_list(0).matchstr=Cookie -config replacer.full_list(0).regex=false -config replacer.full_list(0).replacement=$SESSION_COOKIE"
```

---

### Linux Host Mapping Version

If `host.docker.internal` does not resolve on Linux, use this version:

```bash
mkdir -p reports

docker run --rm \
  --add-host=host.docker.internal:host-gateway \
  -v "$PWD/reports:/zap/wrk/:rw" \
  ghcr.io/zaproxy/zaproxy:stable \
  zap-baseline.py \
  -t http://host.docker.internal:3000/admin/products \
  -m 1 \
  -I \
  -r zap-auth.html \
  -w zap-auth.md \
  -z "-config replacer.full_list(0).description=authcookie -config replacer.full_list(0).enabled=true -config replacer.full_list(0).matchtype=REQ_HEADER -config replacer.full_list(0).matchstr=Cookie -config replacer.full_list(0).regex=false -config replacer.full_list(0).replacement=$SESSION_COOKIE"
```

---

## Windows PowerShell Version

### Step 1: Save cookie jar

```powershell
curl.exe -i `
  -c "$env:TEMP\minicart.cookies" `
  -d "username=admin&password=lesson-01-admin123" `
  -H "Content-Type: application/x-www-form-urlencoded" `
  http://127.0.0.1:3000/login
```

If your branch was seeded using `.env ADMIN_PASSWORD`, replace:

```text
lesson-01-admin123
```

with that value.

---

### Step 2: Build cookie header

```powershell
$SESSION_COOKIE = Get-Content "$env:TEMP\minicart.cookies" |
  Where-Object { $_ -and -not $_.StartsWith("#") } |
  ForEach-Object {
    $cols = $_ -split "`t"
    "$($cols[5])=$($cols[6])"
  }

$SESSION_COOKIE = $SESSION_COOKIE -join "; "
$SESSION_COOKIE
```

---

### Step 3: Run authenticated ZAP baseline

```powershell
New-Item -ItemType Directory -Force -Path reports | Out-Null

docker run --rm `
  -v "${PWD}\reports:/zap/wrk/:rw" `
  ghcr.io/zaproxy/zaproxy:stable `
  zap-baseline.py `
  -t http://host.docker.internal:3000/admin/products `
  -m 1 `
  -I `
  -r zap-auth.html `
  -w zap-auth.md `
  -z "-config replacer.full_list(0).description=authcookie -config replacer.full_list(0).enabled=true -config replacer.full_list(0).matchtype=REQ_HEADER -config replacer.full_list(0).matchstr=Cookie -config replacer.full_list(0).regex=false -config replacer.full_list(0).replacement=$SESSION_COOKIE"
```

---

## ZAP Authenticated Report Output

The authenticated ZAP reports will be written to:

```text
reports/zap-auth.html
reports/zap-auth.md
```

---

## Expected ZAP Authenticated Results

### Expected on `lesson/01-vulnerable`

You may see findings such as:

```text
Missing security headers
Cookie No HttpOnly Flag
Cookie Without Secure Flag
Anti-CSRF warnings on admin forms
```

### Expected on `lesson/04-dast-fixes` and `main`

You should see improvements such as:

```text
Many header findings reduced
Cookie No HttpOnly Flag should disappear
Cookie Without Secure Flag may still remain locally on HTTP
Anti-CSRF warnings may still remain
```
