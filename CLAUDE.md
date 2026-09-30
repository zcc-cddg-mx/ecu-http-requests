# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository Purpose

This is a **Bruno API client workspace** containing HTTP request collections for Zurich Seguros LATAM (Ecuador and Mexico). It serves as a centralized API testing and documentation tool for multiple backend services.

## Workspaces

| Workspace | Path | Notes |
|-----------|------|-------|
| Ecuador | `bruno/ecu-workspace/` | Active — 897 requests across 12 collections |
| Mexico | `bruno/mx-workspace/` | In progress — no collections yet |

## Bruno Workspace Structure

### Ecuador (`bruno/ecu-workspace/`)

Opened directly in the Bruno desktop app (or via the Bruno CLI `bru`).

```
bruno/ecu-workspace/
├── workspace.yml          # Workspace definition listing all collections
├── environments/
│   └── GLOBAL.yml         # Shared environment variables (server URL, auth tokens)
└── collections/
    ├── Arizona/            # 486 requests — claims, accounting, FNOL, reinsurance
    ├── OV/                 # 258 requests — office virtual portal APIs
    ├── Ensurance/          # 48  requests — insurance management
    ├── Puntos de Venta/    # 40  requests — point-of-sale integrations
    ├── Incidentes/         # 31  requests — incident management
    ├── AUTORITY/           # 19  requests — authority/auth service
    ├── Gilbert y Bolona/   # 13  requests — partner integrations
    ├── Compliance/         #  1  request  — compliance checks
    ├── AZURE/              # 27  .yml files — Azure DevOps & N8N automation
    ├── VOC/                # 24  .yml files — accounting/financial queries
    ├── CRYSTAL_REPORTS/    #  4  .yml files — reporting endpoints
    └── INSPEKTOR/          #  3  .yml files — inspection service
```

Collections using `.bru` files are the primary format. Collections under `AZURE/`, `VOC/`, `CRYSTAL_REPORTS/`, and `INSPEKTOR/` use OpenAPI `.yml` format instead.

## Running Requests

**Bruno CLI** (headless, for CI or scripting):
```bash
# Run a single request
bru run "bruno/ecu-workspace/collections/Arizona/path/to/request.bru" --env GLOBAL

# Run an entire collection
bru run "bruno/ecu-workspace/collections/Arizona" --env GLOBAL --recursive

# Run with output
bru run "bruno/ecu-workspace/collections/OV" --env GLOBAL --recursive --reporter json
```

**Bruno VS Code Extension**: El workspace está preconfigurado en `.vscode/settings.json` apuntando a `bruno/ecu-workspace/`. Instalar la extensión Bruno en VS Code carga el workspace automáticamente.

**Bruno Desktop App**: Abrir la carpeta `bruno/ecu-workspace/` como workspace, seleccionar el entorno `GLOBAL` y ejecutar requests individualmente o por colección.

## .bru File Format

Each `.bru` file defines one HTTP request:

```
meta {
  name: Request Name
  type: http
  seq: 1
}

post {
  url: {{server}}/endpoint
  body: json
  auth: bearer
}

params:query {
  page: 0
  pageSize: 20
}

headers {
  Content-Type: application/json
  x-tenantid-1: ec
}

auth:bearer {
  token: {{token}}
}

body:json {
  {
    "field": "value"
  }
}

script:post-response {
  var json = res.getBody();
  bru.setGlobalEnvVar("token", json.body.accessToken);
}
```

Key points:
- Template variables use `{{varName}}` syntax — resolved from the active environment (`GLOBAL.yml`)
- `script:pre-request` and `script:post-response` blocks run JavaScript
- Post-response scripts often extract tokens and store them with `bru.setGlobalEnvVar()`
- Auth tokens are frequently chained: a login request sets `token`, subsequent requests reference `{{token}}`
- `{% response 'body', 'req_id', '$.path' %}` syntax references values from a previous response

## Environment Variables (GLOBAL.yml)

The `GLOBAL` environment defines:
- `server` — base URL for the preprod portal (`https://preprodoficinavirtual.zurichseguros.com.ec/nuevoportal/`)
- `token` — bearer token, typically populated dynamically by running a login request first
- `token-autority-2` — secondary token for the AUTORITY service

**Workflow**: Run the Login request in the relevant collection first to populate `token` before running authenticated requests.

## Adding New Requests

1. Create a `.bru` file in the appropriate collection folder
2. Use `{{server}}` for the base URL and `{{token}}` for auth
3. Set `seq:` to control ordering within a collection folder
4. If the request is the first in an auth chain, add a `script:post-response` block to extract and store the token
5. Register the collection in `workspace.yml` if adding a new top-level collection
