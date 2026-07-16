# Copilot CLI App Blueprint

A single, self-contained **master prompt** that drives GitHub Copilot CLI to
reconstruct a complete full-stack application from scratch in an empty folder —
no repo clone required.

The target app, **Workforce Analytics AI**, is a fully-local, containerized
chat + interactive analytics platform:

- **Frontend:** Angular 18 standalone SPA, D3 v7 charts, crossfilter dashboards,
  and a d3-geo world map — all rendered inline from a strict, parse-as-data
  payload contract.
- **Backend:** `aiohttp` async server that streams a local OpenAI-compatible LLM
  (LM Studio) over SSE, with **grounded** answers via a safe allow-listed query
  tool, session auth + roles, application-layer row-level security, PII masking,
  a governance/retention worker, and a CSV/XLSX/Parquet ingestion pipeline.
- **Data:** synthetic Workday-style HCM dataset over SQLite (all fictional).
- **Infra:** Docker multi-stage builds, k3d/Kubernetes, Traefik, docker-compose.

## How to use

1. Open a terminal in an **empty folder** on the target machine.
2. Start GitHub Copilot CLI.
3. Copy **everything from the `MASTER RECONSTRUCTION PROMPT` heading below** to the
   end of this file, and paste it as your first message.
4. Let it scaffold, then follow the verification steps at the bottom of the prompt.

> **Note:** the target app is ~11k lines. This prompt specifies exact schemas,
> configs, API surface, and logic so Copilot can generate a *functionally
> equivalent* rebuild — expect to iterate on the visual-heavy D3 components and
> long HTML/SCSS, which are described by behavior rather than reproduced verbatim.

License: [MIT](LICENSE).

---

# MASTER RECONSTRUCTION PROMPT — "Workforce Analytics AI"

You are GitHub Copilot CLI operating in an **empty folder**. Build a complete,
runnable full-stack application called **Workforce Analytics AI** exactly as
specified below. Create every folder and file, with the given contents and
behavior. Where a file's full contents are given verbatim (fenced), reproduce
them exactly. Where a file is described as a blueprint, generate functional code
that matches the described schema, signatures, endpoints, and logic precisely.

Work top-to-bottom. After creating the backend and frontend, verify the backend
imports (`python -m compileall backend/app`) and the frontend type-checks
(`cd frontend && npm i && npm run build`).

---

## 0. WHAT THIS APP IS

A fully-local, containerized **chat + interactive analytics** platform:

- Angular 18 SPA (standalone components, "Tron" neon theme), behind a login gate.
- `aiohttp` async Python backend that proxies a **local OpenAI-compatible LLM**
  (LM Studio on the host, `:1234`) as an **SSE token stream**, and renders inline
  D3 charts/dashboards/maps from a strict `<analytics_payload>` contract.
- Data answers are **grounded**: the model calls a safe `query_workforce` tool
  (picks a `dimension`+`measure` from fixed enums); the backend builds the SQL and
  the chart from real result rows.
- **Security**: session auth + roles (admin/analyst/viewer), application-layer
  row-level-security (RLS) on every query chokepoint, and role-based PII masking.
- **Governance**: in-process asyncio worker for data retention + detectors +
  alerts; an admin "Overwatch" page.
- **Data ingestion**: admins upload CSV/XLSX/Parquet worker files; an LLM proposes
  a column mapping; admins review/apply, then **activate** the snapshot as the live
  dataset (one-click rollback), with upstream-health auto-fallback.
- Three SQLite databases: `analytics.db` (regenerated synthetic HCM data),
  `app.db` (users/scopes/history/logs/settings/alerts/retention/loads), and
  `ingest.db` (uploaded snapshots).
- Runs via k3d (Kubernetes) or docker-compose. Everything synthetic; no branding.

Tech: Angular 18 standalone + D3 v7 + crossfilter2 + topojson/world-atlas;
Python 3.12 + aiohttp + aiosqlite + Faker + cryptography + openpyxl + pyarrow;
Docker multi-stage; k3d/k3s + Traefik; local-path PVC.

---

## 1. EXACT FOLDER HIERARCHY

Create precisely this tree (omit committed venvs / build artifacts):

```
.
├── README.md
├── CHANGELOG.md
├── Makefile
├── docker-compose.yml
├── k3d-config.yaml
├── .gitignore
├── .github/workflows/regression.yml
├── docs/
│   ├── ARCHITECTURE.md
│   ├── INGESTION_ROADMAP.md
│   └── SECURITY_GOVERNANCE_ROADMAP.md
├── backend/
│   ├── Dockerfile
│   ├── .dockerignore
│   ├── docker-entrypoint.sh
│   ├── pytest.ini
│   ├── requirements.txt
│   ├── requirements-dev.txt
│   ├── app/
│   │   ├── __init__.py
│   │   ├── config.py
│   │   ├── runtime_config.py
│   │   ├── db.py
│   │   ├── store.py
│   │   ├── crypto.py
│   │   ├── auth.py
│   │   ├── scopes.py
│   │   ├── pii.py
│   │   ├── metrics.py
│   │   ├── prompts.py
│   │   ├── llm_client.py
│   │   ├── analytics_query.py
│   │   ├── server.py
│   │   ├── governance/
│   │   │   ├── __init__.py
│   │   │   ├── retention.py
│   │   │   └── detectors.py
│   │   ├── ingest/
│   │   │   ├── __init__.py
│   │   │   ├── schema.py
│   │   │   ├── parsers.py
│   │   │   ├── mapping.py
│   │   │   ├── llm_prompt.py
│   │   │   ├── loader.py
│   │   │   ├── store.py
│   │   │   ├── serving.py
│   │   │   ├── source.py
│   │   │   └── fallback.py
│   │   └── routes/
│   │       ├── __init__.py
│   │       ├── auth.py
│   │       ├── chat.py
│   │       ├── analytics.py
│   │       ├── conversations.py
│   │       ├── insights.py
│   │       ├── settings.py
│   │       └── admin.py
│   ├── scripts/
│   │   ├── generate_mock_data.py
│   │   └── seed_ci.py
│   └── tests/
│       ├── __init__.py
│       ├── test_mock_data.py
│       ├── test_chat_helpers.py
│       ├── test_auth.py
│       ├── test_scopes.py
│       ├── test_pii.py
│       ├── test_alerts.py
│       ├── test_retention.py
│       ├── test_admin_users.py
│       ├── test_ingest.py
│       ├── test_ingest_mapping.py
│       ├── test_ingest_upload.py
│       ├── test_ingest_serving.py
│       ├── test_ingest_delta.py
│       └── test_ingest_fallback.py
├── frontend/
│   ├── Dockerfile
│   ├── .dockerignore
│   ├── nginx.conf
│   ├── package.json
│   ├── angular.json
│   ├── tsconfig.json
│   ├── tsconfig.app.json
│   ├── tsconfig.spec.json
│   ├── proxy.conf.json
│   └── src/
│       ├── index.html
│       ├── main.ts
│       ├── styles.scss
│       ├── types/
│       │   ├── crossfilter2.d.ts
│       │   └── world-atlas.d.ts
│       └── app/
│           ├── app.config.ts
│           ├── app.component.ts
│           ├── data/
│           │   ├── city-coords.ts
│           │   └── world-geo.ts
│           ├── models/
│           │   ├── chat.models.ts
│           │   ├── worker.models.ts
│           │   ├── metrics.models.ts
│           │   ├── insights.models.ts
│           │   ├── settings.models.ts
│           │   └── admin.models.ts
│           ├── services/
│           │   ├── auth.service.ts
│           │   ├── chat.service.ts
│           │   ├── payload-parser.service.ts
│           │   ├── payload-parser.service.spec.ts
│           │   ├── conversation.service.ts
│           │   ├── feedback.service.ts
│           │   ├── dataset.service.ts
│           │   ├── metrics.service.ts
│           │   ├── insights.service.ts
│           │   ├── settings.service.ts
│           │   ├── admin.service.ts
│           │   └── samples.ts
│           └── components/
│               ├── login/login.component.ts
│               ├── sidebar/sidebar.component.ts
│               ├── message-bubble/message-bubble.component.ts
│               ├── prompt-input/prompt-input.component.ts
│               ├── analytics-chart/analytics-chart.component.ts
│               ├── analytics-dashboard/analytics-dashboard.component.ts
│               ├── analytics-map/analytics-map.component.ts
│               ├── fullscreen/fullscreen-overlay.component.ts
│               ├── metrics/metrics.component.ts
│               ├── metrics/sparkline.component.ts
│               ├── ai-feedback/ai-feedback.component.ts
│               ├── insights-dashboard/insights-dashboard.component.ts
│               ├── settings/settings.component.ts
│               └── admin-dashboard/
│                   ├── admin-dashboard.component.ts
│                   └── ingestion-panel.component.ts
├── host-agent/
│   ├── requirements.txt
│   └── metrics_agent.py
└── k8s/
    ├── 00-namespace.yaml
    ├── 09-backend-secret.example.yaml
    ├── 10-backend-configmap.yaml
    ├── 11-backend-deployment.yaml
    ├── 12-backend-service.yaml
    ├── 13-backend-pvc.yaml
    ├── 20-frontend-deployment.yaml
    ├── 21-frontend-service.yaml
    ├── 30-ingress.yaml
    ├── 31-frontend-nodeport.yaml
    └── kustomization.yaml
```

(Angular components that have separate `.html`/`.scss` are fine to keep inline in
the `.ts` via `template:`/`styles:`; some in the source use external files —
either is acceptable as long as the component behaves as described.)

---

## 2. DEPENDENCIES & CONFIG (VERBATIM)

### backend/requirements.txt
```
aiohttp==3.9.5
aiohttp-cors==0.7.0
aiosqlite==0.20.0
python-dotenv==1.0.1
Faker==25.2.0
cryptography==42.0.8
openpyxl==3.1.5
pyarrow==16.1.0
```

### backend/requirements-dev.txt
```
-r requirements.txt
pytest==8.2.1
pytest-asyncio==0.23.7
```

### backend/pytest.ini
```ini
[pytest]
asyncio_mode = auto
testpaths = tests
```

### host-agent/requirements.txt
```
psutil==5.9.8
```

### frontend/package.json
```json
{
  "name": "analytics-chat-frontend",
  "version": "1.0.0",
  "description": "Angular SPA: streaming chat with inline D3 analytics rendering.",
  "scripts": {
    "ng": "ng",
    "start": "ng serve --proxy-config proxy.conf.json",
    "build": "ng build",
    "build:prod": "ng build --configuration production",
    "watch": "ng build --watch --configuration development",
    "test": "ng test",
    "test:ci": "ng test --watch=false --browsers=ChromeHeadless"
  },
  "private": true,
  "dependencies": {
    "@angular/animations": "^18.2.0",
    "@angular/common": "^18.2.0",
    "@angular/compiler": "^18.2.0",
    "@angular/core": "^18.2.0",
    "@angular/forms": "^18.2.0",
    "@angular/platform-browser": "^18.2.0",
    "@angular/platform-browser-dynamic": "^18.2.0",
    "@angular/router": "^18.2.0",
    "crossfilter2": "^1.5.4",
    "d3": "^7.9.0",
    "topojson-client": "^3.1.0",
    "world-atlas": "^2.0.2",
    "rxjs": "~7.8.0",
    "tslib": "^2.6.0",
    "zone.js": "~0.14.10"
  },
  "devDependencies": {
    "@angular-devkit/build-angular": "^18.2.0",
    "@angular/cli": "^18.2.0",
    "@angular/compiler-cli": "^18.2.0",
    "@types/d3": "^7.4.3",
    "@types/jasmine": "~5.1.0",
    "@types/topojson-client": "^3.1.5",
    "jasmine-core": "~5.2.0",
    "karma": "~6.4.0",
    "karma-chrome-launcher": "~3.2.0",
    "karma-coverage": "~2.2.0",
    "karma-jasmine": "~5.1.0",
    "karma-jasmine-html-reporter": "~2.1.0",
    "typescript": "~5.5.4"
  }
}
```

### frontend/angular.json
```json
{
  "$schema": "./node_modules/@angular/cli/lib/config/schema.json",
  "version": 1,
  "newProjectRoot": "projects",
  "projects": {
    "analytics-chat-frontend": {
      "projectType": "application",
      "root": "",
      "sourceRoot": "src",
      "prefix": "app",
      "architect": {
        "build": {
          "builder": "@angular-devkit/build-angular:application",
          "options": {
            "outputPath": "dist/analytics-chat-frontend",
            "index": "src/index.html",
            "browser": "src/main.ts",
            "polyfills": ["zone.js"],
            "tsConfig": "tsconfig.app.json",
            "assets": [],
            "styles": ["src/styles.scss"],
            "scripts": []
          },
          "configurations": {
            "production": {
              "budgets": [
                { "type": "initial", "maximumWarning": "1mb", "maximumError": "2mb" },
                { "type": "anyComponentStyle", "maximumWarning": "4kb", "maximumError": "8kb" }
              ],
              "outputHashing": "all"
            },
            "development": { "optimization": false, "extractLicenses": false, "sourceMap": true }
          },
          "defaultConfiguration": "production"
        },
        "serve": {
          "builder": "@angular-devkit/build-angular:dev-server",
          "configurations": {
            "production": { "buildTarget": "analytics-chat-frontend:build:production" },
            "development": { "buildTarget": "analytics-chat-frontend:build:development" }
          },
          "defaultConfiguration": "development"
        },
        "test": {
          "builder": "@angular-devkit/build-angular:karma",
          "options": {
            "polyfills": ["zone.js", "zone.js/testing"],
            "tsConfig": "tsconfig.spec.json",
            "styles": ["src/styles.scss"],
            "scripts": []
          }
        }
      }
    }
  }
}
```

### frontend/tsconfig.json (strict; ES2022; add the other two tsconfigs extending it)
Use Angular 18 defaults: `"strict": true`, `"target": "ES2022"`,
`"moduleResolution": "bundler"`, `"experimentalDecorators": true`,
`"typeRoots": ["node_modules/@types", "src/types"]`. `tsconfig.app.json` extends it
with `files: ["src/main.ts"]`; `tsconfig.spec.json` adds jasmine types + `.spec.ts`.

### frontend/proxy.conf.json
```json
{
  "//": "Dev-server proxy: forwards /api to the local backend.",
  "/api": { "target": "http://localhost:8080", "secure": false, "changeOrigin": true, "logLevel": "debug" }
}
```

### frontend/src/main.ts
```ts
import { bootstrapApplication } from '@angular/platform-browser';
import { appConfig } from './app/app.config';
import { AppComponent } from './app/app.component';

bootstrapApplication(AppComponent, appConfig).catch((err) => console.error(err));
```

### frontend/src/app/app.config.ts
```ts
import { ApplicationConfig, provideZoneChangeDetection } from '@angular/core';
import { provideHttpClient, withFetch } from '@angular/common/http';

export const appConfig: ApplicationConfig = {
  providers: [
    provideZoneChangeDetection({ eventCoalescing: true }),
    // withFetch() enables the fetch-based HttpClient — needed for SSE streaming.
    provideHttpClient(withFetch()),
  ],
};
```

### frontend/src/index.html
Minimal: `<app-root></app-root>`, title "Workforce Analytics AI", a dark
`<meta name="color-scheme" content="dark">`, and Google Fonts links for a display
font (e.g. Orbitron) + a mono font (e.g. Share Tech Mono / JetBrains Mono).

### frontend/src/styles.scss
Global "Tron" dark theme. Define CSS variables on `:root`:
`--bg:#04060d; --bg-elevated:#0a1020; --bg-glow / --bg-glow-2` (faint cyan/violet);
`--accent:#00e5ff; --accent-2:#ffc24b; --accent-soft` (translucent cyan);
`--text:#d8f6ff; --text-muted; --border; --danger:#ff5470; --grid-line` (faint cyan);
`--glow-cyan: 0 0 10px rgba(0,229,255,.4); --display; --mono` font stacks. Set body
background `var(--bg)`, `color var(--text)`, the mono/display fonts, custom neon
scrollbars. Everything is dark neon-on-black.

### docker-compose.yml
```yaml
services:
  backend:
    build: ./backend
    environment:
      LLM_BASE_URL: "http://host.docker.internal:1234/v1"
      LLM_MODEL: "qwen/qwen3.5-9b"
      DB_PATH: "/app/data/analytics.db"
      DB_APP_PATH: "/app/data/app.db"
      CORS_ORIGINS: "*"
    extra_hosts:
      - "host.docker.internal:host-gateway"
    ports: ["8080:8080"]
    healthcheck:
      test: ["CMD", "python", "-c", "import urllib.request,sys; sys.exit(0 if urllib.request.urlopen('http://localhost:8080/api/health').status==200 else 1)"]
      interval: 5s
      timeout: 3s
      retries: 30
    volumes: ["backend-data:/app/data"]
  frontend:
    build: ./frontend
    depends_on:
      backend:
        condition: service_healthy
    ports: ["8081:80"]
volumes:
  backend-data:
```

### backend/Dockerfile (multi-stage, verbatim)
```dockerfile
# syntax=docker/dockerfile:1
FROM python:3.12-slim AS builder
ENV PIP_NO_CACHE_DIR=1 PIP_DISABLE_PIP_VERSION_CHECK=1
WORKDIR /install
COPY requirements.txt .
RUN pip install --prefix=/install/prefix -r requirements.txt

FROM python:3.12-slim AS runtime
ENV PYTHONUNBUFFERED=1 PYTHONDONTWRITEBYTECODE=1 \
    PATH="/install/prefix/bin:$PATH" \
    PYTHONPATH="/install/prefix/lib/python3.12/site-packages"
RUN useradd --create-home --uid 10001 appuser
WORKDIR /app
COPY --from=builder /install/prefix /install/prefix
COPY app ./app
COPY scripts ./scripts
COPY docker-entrypoint.sh ./docker-entrypoint.sh
RUN mkdir -p /app/data && chown -R appuser:appuser /app && chmod +x docker-entrypoint.sh
USER appuser
EXPOSE 8080
ENTRYPOINT ["./docker-entrypoint.sh"]
CMD ["python", "-m", "app.server"]
```

### backend/docker-entrypoint.sh (verbatim)
```sh
#!/bin/sh
set -e
DB_FILE="${DB_PATH:-./data/analytics.db}"
needs_gen() {
  [ ! -f "$DB_FILE" ] && return 0
  python - "$DB_FILE" <<'PY' || return 0
import sqlite3, sys
con = sqlite3.connect(sys.argv[1])
row = con.execute("SELECT name FROM sqlite_master WHERE type='table' AND name='daily_engagement'").fetchone()
sys.exit(0 if row else 1)
PY
  return 1
}
if needs_gen; then
  echo "[entrypoint] Generating synthetic HCM data at $DB_FILE..."
  rm -f "$DB_FILE" "$DB_FILE-wal" "$DB_FILE-shm"
  python scripts/generate_mock_data.py --db "$DB_FILE"
else
  echo "[entrypoint] Reusing existing HCM database at $DB_FILE"
fi
exec "$@"
```

### frontend/Dockerfile (verbatim)
```dockerfile
# syntax=docker/dockerfile:1
FROM node:20-alpine AS build
WORKDIR /app
COPY package.json package-lock.json* ./
RUN npm ci || npm install
COPY . .
RUN npm run build:prod

FROM nginx:1.27-alpine AS serve
COPY nginx.conf /etc/nginx/conf.d/default.conf
COPY --from=build /app/dist/analytics-chat-frontend/browser /usr/share/nginx/html
EXPOSE 80
HEALTHCHECK --interval=30s --timeout=3s CMD wget -qO- http://localhost/ >/dev/null || exit 1
CMD ["nginx", "-g", "daemon off;"]
```

### frontend/nginx.conf (verbatim)
```nginx
server {
    listen 80;
    server_name _;
    root /usr/share/nginx/html;
    index index.html;
    location / { try_files $uri $uri/ /index.html; }
    location /api/ {
        proxy_pass http://backend:8080;
        proxy_http_version 1.1;
        proxy_set_header Connection '';
        proxy_buffering off;
        proxy_cache off;
        proxy_read_timeout 3600s;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    }
}
```
`backend/.dockerignore` and `frontend/.dockerignore` exclude `data/`, `*.db*`,
`__pycache__`, `.venv*`, `node_modules`, `dist`, `.angular`.

### k3d-config.yaml (verbatim)
```yaml
apiVersion: k3d.io/v1alpha5
kind: Simple
metadata:
  name: analytics
servers: 1
agents: 0
image: rancher/k3s:v1.30.2-k3s1
ports:
  - port: 8081:80
    nodeFilters:
      - loadbalancer
```

### k8s manifests
- `00-namespace.yaml`: Namespace `analytics-chat`.
- `10-backend-configmap.yaml`: ConfigMap `backend-config` with
  `LLM_BASE_URL=http://host.k3d.internal:1234/v1`, `LLM_MODEL=qwen/qwen3.5-9b`,
  `LLM_TEMPERATURE=0.4`, `LLM_REQUEST_TIMEOUT=240`,
  `HOST_AGENT_URL=http://host.k3d.internal:9900`, `APP_HOST=0.0.0.0`,
  `APP_PORT=8080`, `DB_PATH=/app/data/analytics.db`,
  `DB_APP_PATH=/app/data/app.db`, `CORS_ORIGINS=http://localhost:8081`.
- `11-backend-deployment.yaml`: Deployment `backend`, `replicas:1`, image
  `analytics-backend:local` (`imagePullPolicy: IfNotPresent`), container port
  8080, `envFrom` the ConfigMap + optional `secretRef: backend-secrets`. Add
  `hostAliases: [{ip: "192.168.65.254", hostnames: ["host.k3d.internal"]}]` (Docker
  Desktop host-gateway, so LM Studio stays reachable across cluster restarts).
  Resources requests `50m/96Mi`, limits `500m/256Mi`. Liveness+readiness httpGet
  `/api/health:8080`. Mount PVC `backend-data` at `/app/data`.
- `12-backend-service.yaml`: ClusterIP Service `backend`, port 8080→8080.
- `13-backend-pvc.yaml`: PVC `backend-data`, ReadWriteOnce, 1Gi (k3s local-path).
- `20-frontend-deployment.yaml`: Deployment `frontend`, image
  `analytics-frontend:local`, port 80, requests `25m/32Mi` limits `200m/128Mi`,
  readiness httpGet `/:80`.
- `21-frontend-service.yaml`: ClusterIP Service `frontend`, port 80→80.
- `30-ingress.yaml`: Ingress `analytics-chat`, path `/` Prefix → frontend:80
  (annotation `ingress.kubernetes.io/ssl-redirect: "false"`).
- `31-frontend-nodeport.yaml`: OPTIONAL NodePort Service `frontend-nodeport`
  nodePort 30080 (not in kustomization).
- `09-backend-secret.example.yaml`: TEMPLATE Secret `backend-secrets` (Opaque,
  stringData) with placeholder `SESSION_SECRET`, `FIELD_ENCRYPTION_KEY`,
  `LLM_API_KEY`. Real file `09-backend-secret.yaml` is gitignored.
- `kustomization.yaml`: `namespace: analytics-chat`, lists resources 00,10,11,12,
  13,20,21,30 (NOT 31, NOT the secret).

### Makefile targets
`cluster-up` (`k3d cluster create --config k3d-config.yaml`), `cluster-down`,
`start`/`stop` (k3d cluster start/stop + rollout status), `build` (docker build
both images `analytics-backend:local`, `analytics-frontend:local`), `import`
(`k3d image import ... -c analytics`), `deploy` (`kubectl apply -k k8s/`), `up`
(=build import deploy), `down`, `restart` (rollout restart both), `status`,
`logs`, `seed-local` (`python scripts/generate_mock_data.py`), `metrics-agent`
(venv + run host-agent on :9900), `docs` (Sphinx), `test`/`test-backend`
(`pytest`)/`test-frontend` (`npm run test:ci`), plus Robot `regression*` targets.
Vars: `CLUSTER=analytics NAMESPACE=analytics-chat`.

### .github/workflows/regression.yml
Two jobs on push(main)/PR: (1) `backend-tests` — setup Python 3.12, install
`requirements.txt`+`requirements-dev.txt`, `cd backend && pytest -q`. (2)
`e2e-regression` — `docker compose up -d --build`, poll `/api/health`, seed
users (`docker compose exec -T backend python -m scripts.seed_ci`), install Robot
+ Chromium (`rfbrowser init`), run `robot --exclude llm tests/robot`, upload the
report artifact, `docker compose down -v`.

### .gitignore
Ignore `__pycache__/ *.pyc .venv*/ venv/`, `backend/docs/_build/ backend/data/`,
`*.db *.db-wal *.db-shm .env`, `node_modules/ frontend/dist/ .angular/`,
`CREDENTIALS.local.md k8s/09-backend-secret.yaml`, `.DS_Store *.log`,
`.venv-robot/ tests/robot/results*/`.

---

## 3. BACKEND BLUEPRINTS (file-by-file)

All handlers are `aiohttp` coroutines `(web.Request) -> web.Response`. Three
shared aiosqlite connections are stored on `app[...]`: `db` (analytics.db),
`app_db` (app.db), `ingest_db` (ingest.db). `row_factory = aiosqlite.Row`;
pragmas `journal_mode=WAL`, `foreign_keys=ON` on app.db.

### app/config.py
Frozen `@dataclass Settings` sourcing every value from env via
`_env(key, default)`. Fields (env → default): `host`(APP_HOST=`0.0.0.0`),
`port`(APP_PORT=8080), `llm_base_url`(LLM_BASE_URL=`http://localhost:1234/v1`),
`llm_model`(LLM_MODEL=`qwen3.5-9b`), `llm_api_key`(LLM_API_KEY=`lm-studio`),
`llm_temperature`(0.4), `llm_request_timeout`(240),
`host_agent_url`(HOST_AGENT_URL=`http://host.k3d.internal:9900`),
`db_path`(`./data/analytics.db`), `db_app_path`(`./data/app.db`),
`db_ingest_path`(`./data/ingest.db`), `ingest_max_upload_mb`(25),
`ingest_max_rows`(200000), `upstream_health_url`(""),
`upstream_fail_threshold`(3), `ingest_auto_fallback`(bool from
INGEST_AUTO_FALLBACK, default true), `cors_origins`
(`http://localhost:8081,http://localhost:4200`), `retention_enabled`(true),
`retention_interval_hours`(6), `session_secret`(""), `field_encryption_key`("").
Export singleton `settings = Settings()`.

### app/runtime_config.py
`RuntimeConfig` holds runtime-mutable `llm_base_url`, `llm_model`,
`llm_temperature` (init from settings). `.update(*, ...)` (clamps temperature to
[0,2], strips strings), `.as_dict()`, `.defaults()` (the startup values). Export
singleton `runtime`. `EDITABLE = ("llm_base_url","llm_model","llm_temperature")`.

### app/db.py
`DB_KEY="db"`. `QUERYABLE_TABLES` = the 9 analytics tables set:
`{locations, cost_centers, job_families, job_profiles, supervisory_orgs, workers,
daily_metrics, daily_engagement, daily_attendance}`. `init_db`/`close_db`
on_startup/on_cleanup hooks opening/closing the analytics.db connection (WAL,
synchronous=NORMAL). `run_query(conn, sql, params=()) -> list[dict]`.

### app/store.py — app.db schema + all persistence (largest file)
Contains `SCHEMA` (executescript) with these tables **verbatim in intent**:

```sql
CREATE TABLE IF NOT EXISTS conversations (
    id TEXT PRIMARY KEY, title TEXT NOT NULL,
    user_id INTEGER REFERENCES users(id),  -- owner; NULL only pre-backfill
    created_at REAL NOT NULL, updated_at REAL NOT NULL);

CREATE TABLE IF NOT EXISTS messages (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    conversation_id TEXT NOT NULL REFERENCES conversations(id) ON DELETE CASCADE,
    role TEXT NOT NULL, content TEXT NOT NULL, reasoning TEXT,
    reasoning_tokens INTEGER DEFAULT 0, content_tokens INTEGER DEFAULT 0,
    thinking_ms INTEGER DEFAULT 0, feedback TEXT, created_at REAL NOT NULL);
CREATE INDEX IF NOT EXISTS idx_messages_conv ON messages(conversation_id, id);

CREATE TABLE IF NOT EXISTS generation_log (
    id INTEGER PRIMARY KEY AUTOINCREMENT, conversation_id TEXT, user_id INTEGER,
    created_at REAL NOT NULL, model TEXT, reasoning_tokens INTEGER,
    content_tokens INTEGER, duration_seconds REAL, tokens_per_sec REAL,
    had_payload INTEGER, error TEXT);
CREATE INDEX IF NOT EXISTS idx_genlog_time ON generation_log(created_at);

CREATE TABLE IF NOT EXISTS insights (
    id INTEGER PRIMARY KEY AUTOINCREMENT, created_at REAL NOT NULL,
    digest TEXT NOT NULL, facts_json TEXT NOT NULL);

CREATE TABLE IF NOT EXISTS app_settings (key TEXT PRIMARY KEY, value TEXT NOT NULL);

CREATE TABLE IF NOT EXISTS users (
    id INTEGER PRIMARY KEY AUTOINCREMENT, username TEXT UNIQUE NOT NULL,
    password_hash TEXT NOT NULL, role TEXT NOT NULL,
    must_change_password INTEGER DEFAULT 0, disabled INTEGER DEFAULT 0,
    created_at REAL NOT NULL);

CREATE TABLE IF NOT EXISTS user_scopes (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    user_id INTEGER NOT NULL REFERENCES users(id) ON DELETE CASCADE,
    scope_type TEXT NOT NULL, scope_value TEXT NOT NULL);

CREATE TABLE IF NOT EXISTS data_loads (
    id INTEGER PRIMARY KEY AUTOINCREMENT, source_type TEXT NOT NULL,
    filename TEXT, file_format TEXT, status TEXT NOT NULL,
    row_count INTEGER DEFAULT 0, error_count INTEGER DEFAULT 0, label TEXT,
    details_json TEXT, created_by INTEGER REFERENCES users(id),
    created_at REAL NOT NULL, activated_at REAL);
CREATE INDEX IF NOT EXISTS idx_loads_time ON data_loads(created_at);

CREATE TABLE IF NOT EXISTS system_alerts (
    id INTEGER PRIMARY KEY AUTOINCREMENT, created_at REAL NOT NULL,
    severity TEXT NOT NULL, category TEXT NOT NULL, message TEXT NOT NULL,
    details_json TEXT, dedup_key TEXT, acknowledged_at REAL,
    acknowledged_by INTEGER REFERENCES users(id));
CREATE INDEX IF NOT EXISTS idx_alerts_open ON system_alerts(acknowledged_at, created_at);

CREATE TABLE IF NOT EXISTS retention_policies (
    id INTEGER PRIMARY KEY AUTOINCREMENT, table_name TEXT UNIQUE NOT NULL,
    max_age_days INTEGER NOT NULL, enabled INTEGER NOT NULL DEFAULT 1,
    last_run REAL, last_pruned INTEGER NOT NULL DEFAULT 0);
```

Module constants:
- `APP_DB_KEY="app_db"`.
- `RETENTION_TABLES = {"generation_log":"created_at","messages":"created_at",
  "conversations":"updated_at","insights":"created_at"}`.
- `RETENTION_DEFAULTS = {"generation_log":90,"messages":180,"conversations":180,
  "insights":30}`.
- `ALERT_SEVERITIES=("info","warning","critical")`.
- `ALERT_CATEGORIES=("schema","data_quality","llm","storage","retention","auth",
  "audit","source")`.
- `ACTIVE_SOURCE_SETTING="active_data_source"`, `GENERATED_SOURCE="generated"`.

Idempotent migration runner: `schema_migrations(version PK, applied_at)`, a
`MIGRATIONS` list, `_add_column_if_missing`, migrations `001_message_feedback`,
`002_conversation_owner` (adds `conversations.user_id`, backfills legacy rows to
the first active admin, creates the `(user_id, updated_at)` index — index lives
in the migration, not SCHEMA, since on upgraded DBs the column doesn't exist
until it runs), and `003_genlog_user` (adds `generation_log.user_id`).
`init_app_db` runs SCHEMA + migrations + `seed_retention_policies` (INSERT OR
IGNORE defaults). Provide async functions (all commit; best-effort semantics):
`upsert_conversation(conv_id, title, user_id=None)` (stamps the owner at
creation; ON CONFLICT deliberately never touches user_id — ownership is
immutable), `add_message(...)->id` (also bumps conversation.updated_at),
`set_message_feedback`, `delete_conversation`, ownership helpers
`conversation_owner(conv_id) -> (exists, owner)` / `message_owner(message_id) ->
(exists, owner)` (via message→conversation join) and the pure gate
`can_access(owner, user)` (owner-or-admin; a NULL legacy owner is admin-only,
never public), `log_generation(**fields)` (includes user_id),
`list_conversations(user_id=None)` (with message_count subquery; user_id filters
to that owner, None = admin view of all), `get_conversation`
(convo incl. user_id + ordered messages), `recent_logs`, `logs_summary` (aggregate totals +
error_rate% + avg_tokens_per_sec + payload_rate% + feedback up/down counts +
recent series reversed to chronological), `triage_facts` (rich aggregate over
generation_log: totals, errors, stalls where content_tokens=0, avg/max reasoning,
avg content, tps min/avg/max, avg duration, chart emit rate, feedback counts,
last 5 distinct errors, last 5 downvoted message excerpts), `save_insight`,
`latest_insight`. Users: `_user_dict` (bool-casts, **pops password_hash**),
`get_user_by_id`, `get_user_credentials` (includes hash — login path only),
`create_user`, `count_users`, `set_user_password`, `list_users` (attaches each
user's scopes), `set_user_role`, `set_user_disabled`, `count_active_admins`,
`add_user_scope` (idempotent), `list_user_scopes`, `delete_user_scope`. Alerts:
`raise_alert(sev,cat,msg,*,details,dedup_key)` — if dedup_key set and an
**unacknowledged** alert with that key exists, no-op (returns None); else insert.
`list_alerts(*, include_acked, severity, category, limit=200)`,
`acknowledge_alert`, `alert_counts` (open by severity). Retention:
`seed_retention_policies`, `list_retention_policies`, `get_retention_policy`,
`update_retention_policy`, `prune_older_than(conn,table,date_column,cutoff,*,
batch_size=1000)` — **assert `RETENTION_TABLES[table]==date_column`** before the
f-string delete (no injection surface), batched deletes by rowid;
`record_retention_run`. Data loads: `get_active_source`/`set_active_source` (via
app_settings), `record_load`, `list_loads`, `get_load`, `delete_load`,
`update_load_details` (merge into details_json), `update_load_counts`,
`set_load_status(*, activated=False)`. Settings: `load_app_settings`/
`save_app_settings` (upsert).

### app/crypto.py — field encryption (Fernet)
`_PREFIX="fernet:"`. Optional import of `cryptography.fernet`. `_cipher()` returns
a Fernet if `settings.field_encryption_key` is set+valid else None (no-op when
absent). `encryption_active()`, `generate_key()`, `encrypt(value)` →
`"fernet:<token>"` (or plaintext unchanged if no key / already tagged),
`decrypt(value)` (passes through untagged; raises if tagged but no key).

### app/auth.py — identity core (stdlib only)
`SESSION_COOKIE="wa_session"`, `SESSION_TTL_SECONDS=7*24*3600`,
`PUBLIC_PATHS={"/api/health","/api/auth/login"}`, `ROLES=("admin","analyst",
"viewer")`. Passwords stored as `"<alg>:<salt-hex>:<digest-hex>"` where alg is
`scrypt` (n=2**14,r=8,p=1) preferred, `pbkdf2` (sha256, 600_000 iters) fallback.
`hash_password`, `verify_password` (hmac.compare_digest). Sessions: HMAC-signed
token `"<user_id>.<expiry>.<sig>"` with `sha256`; `set_session_secret`,
`issue_session`, `verify_session`→uid|None (checks sig + expiry). `@web.middleware
auth_middleware`: pass OPTIONS + PUBLIC_PATHS + non-`/api`; else read cookie →
`verify_session` → load user (401 if missing/disabled) → set `request["user"]`.
`require_role(*roles)` decorator (403 if role not allowed).

### app/scopes.py — application-layer RLS
`ScopeFilter(clause_template, params)` with `.clause(alias="w")` (formats
`{alias}`), `.unrestricted`. `ALLOW_ALL=ScopeFilter("1=1",[])`,
`DENY_ALL=ScopeFilter("0=1",[])`. `expand_org_subtrees(analytics_conn, org_ids)`
via a **RECURSIVE CTE** over `supervisory_orgs.parent_org_id`. `build_filter(
expanded_org_ids, regions)` → parameterized fragment:
`({alias}.supervisory_org_id IN (?..) OR {alias}.location_id IN (SELECT id FROM
locations WHERE region IN (?..)))`; no grants ⇒ DENY_ALL. `scope_filter_for(user,
app_conn, analytics_conn)`: admin ⇒ ALLOW_ALL; else read `user_scopes`, split into
org ids + regions, expand orgs, build filter. Default-deny for non-admins with no
grants.

### app/pii.py — role-based masking at serialization
`initials(name)` → "J. D."; `hide(_)` → None. `POLICY = {"admin":{}, "analyst":
{"full_name":initials}, "viewer":{"full_name":initials, "base_salary":hide,
"badge_in_time":hide, "badge_out_time":hide}}`. Unknown/missing role → viewer
(most restrictive). `mask_row`, `mask_rows(rows, role)`.

### app/metrics.py — in-memory app metrics singleton
`MetricsCollector` tracks `requests_total`, reasoning/content token totals,
`active_streams`, `last_generation` (tokens + seconds + tokens_per_sec of the
last completed stream), and a current-generation live view. Methods
`start_stream/record(kind)/end_stream/snapshot()`. Export `metrics`.

### app/llm_client.py — streaming OpenAI-compatible client
Talks to `{runtime.llm_base_url}/chat/completions`. `LLMError`. `stream_chat(
messages) -> AsyncIterator[(kind,text)]` where kind is `"reasoning"` (reads delta
`reasoning_content` OR `reasoning`) or `"content"`; parses SSE `data: {json}` lines
until `[DONE]`. `stream_with_tools(messages, tools)` — same, `tool_choice:auto`,
accumulates `tool_calls` argument fragments and yields a final `("tool_call",
{"id","name","arguments"})`. `complete(messages) -> str` (non-stream, returns
`choices[0].message.content`). Auth header `Bearer {settings.llm_api_key}`, total
timeout `settings.llm_request_timeout`.

### app/prompts.py — system prompt + payload contract
`SCHEMA_DESCRIPTION` describes all 9 HCM tables + the discoverable people-analytics
correlations. `ANALYTICS_CONTRACT` teaches THREE inline block formats the frontend
parses as **data, never eval**:
- `<analytics_payload>` — `{type: "bar"|"line"|"pie", title, x_label, y_label,
  series:[{label,value}]}`, ≤20 points, prose before it.
- `<analytics_dashboard>` — `{title, data:[flat row objs], charts:[{type:
  "bar"|"line", dimension, measure, agg:"sum"|"avg"|"count", title}]}`, 15–60 rows.
- `<analytics_map>` — `{title, value_label, points:[{city,value}]}`; **cities must
  come from the fixed 28-city list** (coords resolved client-side).
`SYSTEM_PROMPT = f"..."` combining them + instruction to CALL the `query_workforce`
tool for real numbers, keep reasoning brief for small talk. `build_messages(
history)` prepends the system prompt. `build_tool_narration_messages(question,
result)` — minimal prompt to write ONE sentence over real rows (no chart).
`INSIGHTS_SYSTEM` + `build_insights_messages(facts)` for the reliability digest.

### app/analytics_query.py — the grounded tool (allow-listed aggregates)
Model NEVER writes SQL. `DIMENSIONS` dict maps keys → (SQL expr over the `w` CTE,
label, is_band): `region, country, work_mode, job_family, management_level,
status, worker_type, comp_grade`, plus banded `engagement_band, calls_band,
lateness_band` (CASE expressions over `avg_engagement`/`avg_calls`/`avg_late`).
`MEASURES` maps keys → (aggregate, label): `headcount`(COUNT(*)),
`avg_performance`, `avg_engagement`, `avg_calls`, `avg_minutes_late`,
`avg_salary`. `_PIE_DIMS={"work_mode","status","worker_type"}`. `_WORKER_CTE` — a
per-worker CTE joining workers→job_profiles→job_families→locations with correlated
subaverages of `daily_engagement.calls_completed`, `digital_engagement_score`, and
`daily_attendance.minutes_late (on_site=1)`, gated by `WHERE {scope}`.
`tool_definition()` returns the OpenAI function schema `query_workforce(dimension
enum, measure enum)`. `run_tool(conn, dimension, measure, scope=ALLOW_ALL,
worker_cte=None)` validates against the allow-lists, builds `SELECT {dim} AS label,
{measure} AS value FROM w GROUP BY label HAVING value IS NOT NULL` (order: bands by
label, else value DESC), filters out None/"n/a", caps 25, returns `{title,
chart_type ("pie" if pie-dim+headcount else "bar"), x_label, y_label, series}`.
`worker_cte` lets a flat uploaded snapshot supply its own CTE template + leading
params (see ingest/serving).

### app/server.py — application factory
`create_app()`: `web.Application(middlewares=[auth_middleware],
client_max_size=(ingest_max_upload_mb+1)*1MB)`. Register EVERY route (see the
endpoint table in §5). Configure aiohttp-cors (wildcard if `cors_origins=="*"`
else per-origin, `allow_credentials=False`). on_startup order: `init_db`,
`init_app_db`, `init_ingest_db`, `load_persisted_settings`, `bootstrap_auth`,
`init_active_source`, `start_governance_worker`. on_cleanup:
`stop_governance_worker`, `close_db`, `close_app_db`, `close_ingest_db`.
`handle_health` → `{"status":"ok"}`. `main()` calls `web.run_app`. `__main__`.

### app/routes/auth.py
In-memory failed-login burst tracker (`deque` per username, window 300s, threshold
5) → raises a deduped `warning/auth` alert. `handle_login` (verify creds; set
HttpOnly SameSite=Lax session cookie; 401 on failure + record burst),
`handle_logout` (del cookie), `handle_me` (returns `request["user"]`),
`handle_change_password` (min length 8; verify current; set new hash).
`bootstrap_auth(app)` on_startup: resolve session secret (env `SESSION_SECRET`
wins; else load from app_settings decrypted; else generate `token_hex(32)`,
store encrypted); if `count_users()==0` seed `admin` with a random
`token_urlsafe(12)` password (`must_change_password=1`) and **log it once**.

### app/routes/chat.py — the SSE chat endpoint
`_sse(event,data)` frames `event: X\ndata: {json}\n\n`. POST `/api/chat` body
`{messages:[{role,content}], conversation_id, title}`. **Ownership guard before
any write**: conversation ids are client-generated, so check
`conversation_owner` + `can_access` and 404 (same as missing — no existence
leak) if the id belongs to someone else; the guard is enforcement, so it runs
OUTSIDE the best-effort suppress block. Stamp `user_id` on the upsert and on the
generation-log row. Then persist user message +
upsert conversation (best-effort). Open a `web.StreamResponse` with headers
`Content-Type: text/event-stream`, `Cache-Control: no-cache`, `X-Accel-Buffering:
no`. **Phase 1:** `stream_with_tools(build_messages(clean), [tool_definition()])`
— relay `reasoning`→SSE `reasoning` and `content`→SSE `token`; capture any
`tool_call`. **If tool called:** resolve the caller's scope (generated →
`scope_filter_for`; file source → `serving.file_scope_for` + FILE_WORKER_CTE),
run `run_tool`. On a result: **Phase 2** stream a one-sentence narration
(`build_tool_narration_messages`), then append a server-built
`<analytics_payload>` chart from the exact rows (guaranteed accurate). If no
result, fall back to a normal `stream_chat`. Persist assistant message FIRST (to
return its id) then emit SSE `done {message_id}`; on LLMError emit SSE `error` and
log. Always write a `generation_log` row (best-effort; never break the stream).
`had_payload` = payload/dashboard tags present. Uses `metrics.start_stream/record/
end_stream` and `active_analytics_conn(app)`/`source_kind(app)`.

### app/routes/analytics.py — direct data endpoints (all RLS + PII)
`request_scope(request)` resolves the caller's filter for the active source.
`_storage_snapshot()` sizes the db files + disk usage. Endpoints:
- `GET /api/summary` — worker count (scoped) + counts of supervisory_orgs,
  locations, job_profiles (file source → `serving.summary`).
- `GET /api/dataset/workers` — flat denormalized worker rows via a big JOIN
  (workers+job_profiles+job_families+supervisory_orgs+locations) with per-worker
  Viva/Brivo rollup subqueries (`avg_calls_per_day, avg_engagement_score,
  avg_after_hours_min, avg_minutes_late, on_site_rate_pct`) + `hire_year`; **WHERE
  {scope}**; masked by role. File source → `serving.dataset_rows`.
- `GET /api/tables/{table}` — first N (≤200) rows of an allow-listed table;
  worker-row tables (`workers, daily_engagement, daily_attendance`) are scoped;
  masked.
- `GET /api/trends/utilization` — avg utilization_pct/day over daily_metrics for
  orgs containing in-scope workers (file source → empty series).
- `GET /api/metrics` — `metrics.snapshot()` + host stats fetched from
  `settings.host_agent_url/host` (best-effort, `host_error` note) + storage.

### app/routes/conversations.py
`GET /api/conversations` (own conversations only; admins see all — support/
audit), `GET /api/conversations/{id}` and `DELETE /api/conversations/{id}`
(gated via `can_access`; non-owners get the **same 404 as a missing id** — no
existence leak; delete stays idempotent for never-existed ids), `POST
/api/messages/{id}/feedback` (rating in {up,down,null}; resolves message →
conversation → owner via `message_owner` before writing, 404 otherwise),
`GET /api/logs/recent`, `GET /api/logs/summary`.

### app/routes/insights.py
`GET /api/insights` (latest digest + facts, or nulls). `POST
/api/insights/generate` — gather `triage_facts`, if zero generations return a stub;
else `complete(build_insights_messages(facts))`, persist + return. This is the
read-only "triage agent".

### app/routes/settings.py
`GET /api/settings` (current + defaults + api_key_set + request_timeout).
`PUT /api/settings` `@require_role("admin")` — update `runtime`, persist to
app_settings, write an `info/audit` alert. `GET /api/llm/models` — proxy the LLM
server's `/v1/models` → `{reachable, models[], error}`. `load_persisted_settings`
on_startup applies saved overrides.

### app/routes/admin.py — everything `@require_role("admin")`
`_audit(request, message, details)` writes an `info/audit` alert (best-effort).
Retention: `GET /api/admin/retention`, `PUT /api/admin/retention/{table}`
(validate max_age_days≥1), `POST /api/admin/retention/run`. Alerts: `GET
/api/admin/alerts` (query include_acked/severity/category), `POST
/api/admin/alerts/{id}/ack`, `POST /api/admin/detectors/run` (runs detectors +
`check_upstream`). Users/scopes: `validate_user_change(...)` pure guard (no
self-lockout, keep ≥1 active admin), `GET/POST /api/admin/users`, `PATCH
/api/admin/users/{id}`, `POST /api/admin/users/{id}/reset-password`, `POST/DELETE
/api/admin/users/{id}/scopes[/{scope_id}]` (validate org/region exists in
analytics.db), `GET /api/admin/scope-options`. Ingestion: `GET
/api/admin/data-sources` (active + loads + upstream state), `GET
/api/admin/data/schema` (canonical fields), `POST /api/admin/data/upload`
(multipart; size-capped; reject `.xlsm`; → `receive_upload`), `POST
/api/admin/data/loads/{id}/apply` (validate mapping; mode replace|merge +
base_load_id), `POST /api/admin/data/loads/{id}/activate`, `POST
/api/admin/data/activate-generated`, `DELETE /api/admin/data/loads/{id}` (not the
active one). New/temp passwords are `secrets.token_urlsafe(9)`.

### app/governance/retention.py
`GOV_TASK_KEY`, day=86400, startup grace 30s. `run_due_retention(conn)` prunes each
enabled policy (cutoff = now − days*86400) via `prune_older_than`, records the run,
returns per-table summaries; one bad table never aborts the sweep.
`governance_cycle(app)` = retention (alert if pruned) + `run_detectors` +
`check_upstream`. `_loop` sleeps `retention_interval_hours` between cycles;
`start_governance_worker`/`stop_governance_worker` (skip if
`RETENTION_ENABLED=0`).

### app/governance/detectors.py
`run_detectors(app_conn, analytics_conn)` runs four `_safe`-wrapped detectors,
each raising a **deduped** alert: `_check_schema` (PRAGMA integrity_check on both
dbs + EXPECTED_ANALYTICS_TABLES present → critical/schema), `_check_orphans`
(workers w/ invalid org, engagement w/o worker → warning/data_quality),
`_check_storage` (disk ≥80% → warning/storage), `_check_llm` (error rate ≥25% over
last 50 gens, min 10 samples → warning/llm).

### app/ingest/ — the upload→map→apply→activate pipeline
- **schema.py**: `@dataclass Field(name,type,required,description)`;
  `CANONICAL_WORKER_FIELDS` (20 fields: `worker_id`(int,req), `full_name`(req),
  `region`(req), `country, city, job_family`(req), `management_level, worker_type,
  time_type, work_mode, status, comp_grade, base_salary, performance_rating,
  hire_date, supervisory_org, avg_calls_per_day, avg_engagement_score,
  avg_minutes_late, on_site_rate_pct`). Helpers `CANONICAL_BY_NAME`,
  `CANONICAL_FIELD_NAMES`, `REQUIRED_FIELD_NAMES`. `_clean_number` (strips $£€, comma,
  space, k/m suffix), `_coerce(value,ftype)`, `validate_rows(rows) ->
  (clean, errors)` coercing to types, enforcing required + **worker_id uniqueness**
  (first wins).
- **parsers.py**: no pandas. `SUPPORTED_FORMATS=("csv","xlsx","parquet")`,
  `ParseError`, `detect_format` (magic bytes `PAR1`→parquet, `PK`→xlsx, else
  extension), `parse(data,fmt)→(columns,rows)` using stdlib `csv`, `openpyxl`
  (`read_only=True, data_only=True` — never evaluates formulas), `pyarrow.parquet`.
  `profile(columns,rows,sample_size=10)` → `{columns,row_count,sample,
  inferred_types}`; `_infer_type`.
- **mapping.py**: `SYNONYMS` dict of normalized header → canonical field;
  `_norm`, `_match_field` (exact→1.0, synonym→0.85, substring→0.6). `heuristic_map`
  (deterministic; reserves first+last→`full_name` join_space).
  `validate_mapping(raw, columns)` keeps well-formed entries `{target, sources[],
  transform:"none"|"join_space", confidence}`. `llm_map(profile, completer) ->
  (mapping, "llm"|"heuristic")` (falls back on any failure). `apply_mapping(rows,
  mapping)` transforms every row in code.
- **llm_prompt.py**: `build_mapping_messages(profile, canonical_fields)` — strict
  JSON-only mapping request (send only a small sample).
- **loader.py**: `receive_upload(app,*,filename,data,created_by,completer=complete)`
  — parse, cap rows, profile, propose mapping, `record_load(status="pending")`,
  stash raw rows; returns proposal. `apply_mapping_to_load(app, load_id, mapping,*,
  mode="replace", base_load_id=None)` — apply mapping to stored raw rows, validate,
  stage; merge mode upserts into base (`merge_from_base`); update counts + status
  (`staged`|`failed`).
- **store.py** (`ingest_db`): `INGEST_SCHEMA` = flat `workers(load_id + the 20
  canonical columns)` + `raw_rows(load_id,row_index,data_json)`. `insert_workers`,
  `count_workers`, `delete_load_rows`, `merge_from_base(new,base)` (carry over base
  rows whose worker_id the new load lacks → standalone snapshot; base untouched),
  `insert_raw_rows`, `get_raw_rows`, `delete_raw_rows`.
- **serving.py**: file-source query shapes. `FILE_WORKER_CTE` (aliases flat columns
  to the tool's expected names, `WHERE load_id=? AND {scope}`). `DATASET_SQL`.
  `file_scope_filter`/`file_scope_for` — **region grants filter the `region`
  column; org-only grants fail closed**; admin unscoped. `dataset_rows`, `summary`,
  `table_preview`.
- **source.py**: active-source pointer (`generated` | `file:<id>`), cached in
  `app[ACTIVE_SOURCE_KEY]`. `init_active_source` (validates, falls back if
  dangling), `active_source`, `source_kind`, `active_analytics_conn` (returns
  ingest_db when file active else analytics.db), `activate_load` (supersede
  previous, flip pointer), `activate_generated` (rollback; previous → staged).
- **fallback.py**: `check_upstream(app, probe=_default_probe)` — probe
  `UPSTREAM_HEALTH_URL`; N consecutive failures → warning/source alert + (if
  `INGEST_AUTO_FALLBACK` and generated active) `_fallback_to_last_good` activates
  the most recent snapshot with rows; recovery → info/source alert (never
  auto-reverts). `upstream_state(app)` for the UI.

### scripts/generate_mock_data.py — synthetic HCM generator
CLI `--db --days=45 --seed=42`. Seeds `random` + `Faker`. Drops+recreates the 9
analytics tables (DDL below) and populates them relationally. **28-city list**
(city,country,region∈{EMEA,Americas,APAC},lat,lng — real coords). 8 job families;
6 profile levels (Associate→Director) with comp grades G2–G7 and base salaries.
Distributions: worker_type ~15% Contingent; time_type ~12% Part Time; status
90/6/4 Active/OnLeave/Terminated; work_mode 40/35/25 Hybrid/Onsite/Remote with
`ONSITE_PROBABILITY` and `ENGAGEMENT_MODE_BONUS`. Build Company→Division→Team org
tree (`parent_org_id`), CEO + division heads + team leads + ICs. **Each worker has
a hidden `quality` (0..1) trait** driving BOTH `performance_rating` (≈1.3+3.4·q+
noise, clamped 1..5) AND daily signals, so correlations are genuinely present:
calls/day↑→performance (+~0.95), lateness↑→performance (−~0.75), Remote skews
digital, Onsite skews to meetings. Emit `daily_metrics` per team/day and
`daily_engagement`+`daily_attendance` per active worker per **weekday**. Print
table counts.

Analytics DDL (drop+create) — reproduce these columns exactly:
```sql
CREATE TABLE locations (id INTEGER PRIMARY KEY, name TEXT, city TEXT, country TEXT,
    region TEXT, lat REAL, lng REAL);
CREATE TABLE cost_centers (id INTEGER PRIMARY KEY, name TEXT, function TEXT);
CREATE TABLE job_families (id INTEGER PRIMARY KEY, name TEXT);
CREATE TABLE job_profiles (id INTEGER PRIMARY KEY, job_family_id INTEGER REFERENCES
    job_families(id), name TEXT, management_level TEXT, comp_grade TEXT);
CREATE TABLE supervisory_orgs (id INTEGER PRIMARY KEY, name TEXT, parent_org_id
    INTEGER REFERENCES supervisory_orgs(id), location_id INTEGER REFERENCES
    locations(id), cost_center_id INTEGER REFERENCES cost_centers(id), level TEXT);
CREATE TABLE workers (id INTEGER PRIMARY KEY, full_name TEXT, job_profile_id INTEGER
    REFERENCES job_profiles(id), supervisory_org_id INTEGER REFERENCES
    supervisory_orgs(id), manager_id INTEGER REFERENCES workers(id), location_id
    INTEGER REFERENCES locations(id), cost_center_id INTEGER REFERENCES
    cost_centers(id), worker_type TEXT, time_type TEXT, work_mode TEXT, hire_date
    TEXT, status TEXT, comp_grade TEXT, base_salary INTEGER, performance_rating REAL);
CREATE TABLE daily_metrics (id INTEGER PRIMARY KEY, supervisory_org_id INTEGER
    REFERENCES supervisory_orgs(id), metric_date TEXT, headcount_active INTEGER,
    tickets_closed INTEGER, avg_handle_minutes REAL, satisfaction_score REAL,
    utilization_pct REAL, overtime_hours REAL);
CREATE TABLE daily_engagement (id INTEGER PRIMARY KEY, worker_id INTEGER REFERENCES
    workers(id), metric_date TEXT, emails_sent INTEGER, chats_sent INTEGER,
    meetings_attended INTEGER, meeting_hours REAL, calls_completed INTEGER,
    call_minutes INTEGER, focus_hours REAL, after_hours_minutes INTEGER,
    active_connected_hours REAL, digital_engagement_score REAL);
CREATE TABLE daily_attendance (id INTEGER PRIMARY KEY, worker_id INTEGER REFERENCES
    workers(id), metric_date TEXT, location_id INTEGER REFERENCES locations(id),
    on_site INTEGER, badge_in_time TEXT, badge_out_time TEXT, minutes_late INTEGER,
    on_site_hours REAL, access_denied_count INTEGER DEFAULT 0);
-- + indexes on workers(org/loc/mgr), daily_metrics(org,date), engagement/attendance(worker,date)
```

### scripts/seed_ci.py
Idempotently upserts `admin` (pass `ADMIN_PASS`, default `adminadmin`) and
`analyst` (pass `ANALYST_PASS`) with `must_change_password=0`, and grants the
analyst a region scope (`ANALYST_REGION`, default `EMEA`). Uses `hash_password`.

### host-agent/metrics_agent.py
Stdlib `http.server` on `:9900` (`METRICS_AGENT_PORT`), psutil-based. `GET
/host` (also `/metrics`, `/`) returns `{collected_at, agent_uptime_s, host:{cpu,
mem, swap, load_avg, disk}, lm_studio:{running, processes, rss_mb, cpu_percent},
docker:[{name,cpu_percent,mem_usage,mem_percent}]}`. Finds LM Studio processes by
name hints; runs `docker stats --no-stream --format` (locating the docker CLI).
CORS open, read-only, never crashes on a bad reading.

---

## 4. FRONTEND BLUEPRINTS

### models/ (TypeScript interfaces — match these shapes exactly)
- **chat.models.ts**: `Role='user'|'assistant'`; `ChatMessage {role, content,
  streaming?, error?, reasoning?, reasoningTokens?, contentTokens?,
  thinkingStartedAt?, thinkingEndedAt?, thinkingMs?, feedback?:'up'|'down',
  serverId?}`; `Conversation {id,title,messages,createdAt,loaded?}`;
  `ChartType='bar'|'line'|'pie'`; `SeriesPoint{label,value}`; `AnalyticsConfig
  {type,title?,x_label?,y_label?,series}`; `AggType='sum'|'avg'|'count'`;
  `DashboardChartDef {type:'bar'|'line',dimension,measure,agg?,title?}`;
  `DashboardConfig {title?,data:Array<Record<string,string|number>>,charts}`;
  `MapPoint{city,value}`; `MapConfig{title?,value_label?,points}`; discriminated
  `MessageSegment = {kind:'text',text} | {kind:'chart',config} |
  {kind:'dashboard',config} | {kind:'map',config}`.
- **worker.models.ts**: `WorkerRecord` (id, full_name, region, country, city,
  job_family, management_level, worker_type, time_type, status, comp_grade,
  base_salary, performance_rating, hire_date, hire_year, org_name, work_mode, and
  the nullable rollups avg_calls_per_day/avg_engagement_score/avg_after_hours_min/
  avg_minutes_late/on_site_rate_pct).
- **metrics.models.ts**: `AppMetrics, HostMetrics, StorageMetrics, MetricsResponse
  {app,host,host_error,storage}, LogsSummary` (match §3 metrics shapes).
- **insights.models.ts**: `TriageFacts` (all triage_facts keys), `InsightsResponse
  {digest,facts,generated_at}`.
- **settings.models.ts**: `LlmSettings {llm_base_url,llm_model,llm_temperature}`,
  `SettingsResponse {current,defaults,api_key_set,request_timeout}`,
  `ModelsResponse {reachable,models,error}`.
- **admin.models.ts**: `AlertSeverity`, `SystemAlert`, `AlertCounts`,
  `AlertsResponse`, `RetentionPolicy`, `Role='admin'|'analyst'|'viewer'`,
  `UserScope`, `ManagedUser` (+ transient draftType/draftValue), `ScopeOptions`,
  `DataLoad`, `DataSources`, `SchemaField`, `MappingEntry`, `UploadProposal`,
  `ApplyResult` (match the admin route responses).

### services/ (all use native `fetch`, relative `/api/...`, `cache:'no-store'`)
- **auth.service.ts**: `SessionUser`; `restore()` (`/api/auth/me`), `login`,
  `changePassword`, `logout`. Holds `user`.
- **chat.service.ts**: `StreamEvent` union `{token|reasoning|done|error}`.
  `stream(messages, onEvent, meta) -> AbortController` — POSTs `/api/chat` and
  reads the `ReadableStream`, splitting SSE frames on `\n\n`, dispatching by
  `event:` line. Uses fetch (not EventSource) because it POSTs history.
- **payload-parser.service.ts** (THE TRUST BOUNDARY — reproduce faithfully):
  `parse(raw): MessageSegment[]` scans for the three block tags in document order;
  emits prose between them; **stops at an unclosed tag (still streaming) so partial
  JSON never renders**; each block body is `JSON.parse`d (never eval) and validated
  against a fixed schema — `tryParseConfig` (type in {bar,line,pie}, normalized
  series of {string label, finite number value}, cap 50), `tryParseDashboard`
  (normalized rows keeping only string|number values, charts validated against
  sample row keys, agg in {sum,avg,count}, count needs no measure, cap 4 charts/
  1000 rows), `tryParseMap` (points {string city, finite value}, cap 60). Malformed
  → dropped or shown as text. Ship `payload-parser.service.spec.ts` covering:
  partial stream (no chart until closing tag), malformed JSON fallback, unknown
  chart type, invalid series points, oversized-series cap.
- **conversation.service.ts**: `list/get/delete` mapping stored messages →
  ChatMessage (snake→camel). **feedback.service.ts**: fire-and-forget POST rating.
  **dataset.service.ts**: `workers()`→WorkerRecord[]. **metrics.service.ts**:
  `fetchOnce`/`fetchSummary`. **insights.service.ts**: `getLatest`/`generate`.
  **settings.service.ts**: `get`/`save`/`models`. **admin.service.ts**: full
  Overwatch client (alerts, ackAlert, runDetectors, retention CRUD+run, users,
  scopeOptions, createUser, updateUser, resetPassword, add/removeScope,
  dataSources, dataSchema, uploadDataset(FormData), applyMapping(replace|merge),
  deleteLoad, activateLoad, activateGenerated).
- **samples.ts**: `resolveDemo(arg)` returns canned assistant text (with example
  `<analytics_payload>`/`<analytics_dashboard>`/`<analytics_map>` blocks) for the
  `/demo` slash-command previews. **fullscreen.service.ts**: singleton holding the
  content shown in the fullscreen overlay (`openChart/openDashboard/openMap/close`).

### data/
- **city-coords.ts**: `CITY_COORDS: Record<string,[lng,lat]>` for the 28 dataset
  cities (used by AnalyticsMap; coords resolved client-side, never trusted from the
  model).
- **world-geo.ts**: a bundled world topojson/geojson (or `world-atlas` import) for
  the d3-geo base map.

### app.component.ts — shell + conversation state owner
Standalone root. Login gate: `authState: 'checking'|'login'|'ready'` (renders
`<app-login>` until `/api/auth/me` resolves and no forced password change). Tron
banner with a collapsible nav; **pages** = `Console`(chat), `Telemetry`(metrics),
`Diagnostics`(aifeedback), `Grid`(insights), `Control`(settings), `Overwatch`
(admin-only). Owns `conversations`, `activeId`, streaming lifecycle (`onSend`
pushes user+placeholder assistant, calls `chat.stream`, re-enters NgZone on each
event since ReadableStream promises aren't zone-patched), `/demo` shortcut,
`onStop`, feedback (`onRate`), lazy history loading, delete-with-confirm,
autoscroll. New conversation id = `crypto.randomUUID()`.

### components/ (standalone; brief blueprints)
- **login**: username/password form; on success, if `must_change_password` show a
  change-password step; emits `(authed)`.
- **sidebar**: conversation list, new-chat, select, delete, collapse toggle.
- **message-bubble**: renders a ChatMessage — a collapsible "thinking" panel
  (reasoning + elapsed ms + live token counts), the answer parsed via
  `PayloadParserService` into interleaved text/chart/dashboard/map segments, and
  👍/👎 feedback buttons (emits `(rate)`).
- **prompt-input**: textarea composer, Enter-to-send, Stop button while `busy`.
- **analytics-chart**: D3 bar/line/pie renderer from `AnalyticsConfig`; a ⛶
  fullscreen button (→ FullscreenService).
- **analytics-dashboard**: crossfilter2-linked board — clicking a bar filters the
  shared flat dataset and every chart recomputes; supports sum/avg/count aggs.
- **analytics-map**: d3-geo world map plotting `MapPoint`s (coords from
  city-coords.ts).
- **fullscreen-overlay**: single app-root overlay rendering FullscreenService
  content (chart|dashboard|map) large; Esc/click-out closes.
- **metrics** (+ **sparkline**): live "Telemetry" — polls `/api/metrics` +
  `/api/logs/summary`, shows host CPU/mem/swap/load, LM Studio footprint, docker
  stats, token throughput with rolling neon sparklines, storage; banner prompting
  `make metrics-agent` if `host_error`.
- **ai-feedback** ("Diagnostics"): shows the latest LLM reliability digest + facts;
  a "generate" button (`/api/insights/generate`).
- **insights-dashboard** ("Grid"): the big DC.js-style crossfilter board over
  `/api/dataset/workers` — pies/row/bar charts, a hire-year brush, a location map,
  and correlation scatters (calls↔performance, lateness↔performance).
- **settings** ("Control"): runtime LLM model/endpoint/temperature; "Detect loaded
  models" (`/api/llm/models`); PUT is admin-only server-side (show read-only for
  non-admins).
- **admin-dashboard** ("Overwatch", admin-only) + **ingestion-panel**: alerts feed
  with ack + severity/category filters; retention policy editor + "run now";
  detector "run checks"; user/scope management (create/disable/role/reset-password/
  grant-revoke scopes with lockout guards); and the ingestion panel — upload a
  file, review the proposed column mapping (dropdowns from `/api/admin/data/schema`),
  apply (replace|merge), activate/rollback, upstream status.

### types/
- **crossfilter2.d.ts**, **world-atlas.d.ts**: minimal module declarations so the
  untyped libs compile under strict TS.

---

## 5. COMPLETE HTTP API (register all in server.py, in this order)

Public (no session): `GET /api/health`, `POST /api/auth/login`.
Authenticated: `POST /api/auth/logout`, `GET /api/auth/me`, `POST
/api/auth/change-password`, `POST /api/chat` (SSE), `GET /api/summary`, `GET
/api/metrics`, `GET /api/trends/utilization`, `GET /api/dataset/workers`, `GET
/api/tables/{table}`, `GET /api/conversations`, `GET /api/conversations/{id}`,
`DELETE /api/conversations/{id}`, `POST /api/messages/{id}/feedback`, `GET
/api/logs/recent`, `GET /api/logs/summary`, `GET /api/insights`, `POST
/api/insights/generate`, `GET /api/settings`, `PUT /api/settings` (admin), `GET
/api/llm/models`. Admin-only (`@require_role("admin")`): `GET /api/admin/retention`,
`PUT /api/admin/retention/{table}`, `POST /api/admin/retention/run`, `GET
/api/admin/alerts`, `POST /api/admin/alerts/{id}/ack`, `POST
/api/admin/detectors/run`, `GET /api/admin/users`, `POST /api/admin/users`, `PATCH
/api/admin/users/{id}`, `POST /api/admin/users/{id}/reset-password`, `POST
/api/admin/users/{id}/scopes`, `DELETE /api/admin/users/{id}/scopes/{scope_id}`,
`GET /api/admin/scope-options`, `GET /api/admin/data-sources`, `GET
/api/admin/data/schema`, `POST /api/admin/data/upload`, `POST
/api/admin/data/loads/{id}/apply`, `POST /api/admin/data/loads/{id}/activate`,
`POST /api/admin/data/activate-generated`, `DELETE /api/admin/data/loads/{id}`.

---

## 6. TESTS (backend pytest, asyncio_mode=auto)

Reproduce coverage for: mock-data integrity (referential integrity, org hierarchy,
value ranges, seed determinism, and that the embedded correlations actually exist);
SSE framing + the `_payload_block` contract + chat helpers; auth (salted hashing,
session signing/tampering/expiry); RLS (`build_filter`, recursive org-subtree
expansion, a chat-tool leak test proving tool output never contains a name); PII
masking per role; conversation ownership (two-user matrix across
list/get/delete/feedback/chat-append, owner immutability on conflicting upserts,
the `can_access` truth table incl. NULL-owner-is-admin-only, and the legacy
backfill migration with an idempotent re-run); alerts (dedup while open);
retention (`prune_older_than` guard +
batching); admin `validate_user_change` guards; and the full ingest pipeline
(parsers, heuristic+LLM mapping/validation, upload staging, flat-source serving +
file RLS, merge/upsert deltas, upstream fallback). Frontend Karma/Jasmine spec:
the payload-parser edge cases listed above.

---

## 7. FINAL NOTES FOR THE GENERATOR

- Keep everything domain-neutral/synthetic; no real company or vendor identifiers.
- The `<analytics_payload>` JSON is the ENTIRE trust boundary: parse as data,
  validate against the fixed schema, never `eval`.
- RLS + PII are enforced server-side at every query chokepoint; the UI hiding is
  convenience only.
- One backend pod ⇒ the governance worker is an in-process asyncio loop; the
  "run now" admin endpoints exist so a K8s CronJob could drive it if replicas grow.
- After generation: `pip install -r backend/requirements-dev.txt && cd backend &&
  python scripts/generate_mock_data.py && python -m pytest`; and `cd frontend &&
  npm i && npm run build && npm run test:ci`. Then `docker compose up --build` and
  open http://localhost:8081 (grab the first-boot admin password from
  `docker compose logs backend | grep "Temporary password"`).

END OF MASTER PROMPT.
