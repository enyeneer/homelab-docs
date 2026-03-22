# Karakeep Bookmark Manager Deployment and OpenAI Tagging

This page documents the deployment of a self-hosted Karakeep instance on `seranogenomics`, intended to replace a large and increasingly unwieldy collection of browser bookmarks and folders with a more searchable, cross-device bookmark system. :contentReference[oaicite:1]{index=1}

## Summary

The goal of this phase was to move away from managing thousands of links in normal browser bookmarks and bookmark folders, and instead stand up a dedicated self-hosted bookmark manager that would be easier to search, organize, and access across devices. :contentReference[oaicite:2]{index=2}

The work centered on:

- deploying Karakeep as its own standalone Docker Compose stack
- standing up the required supporting services:
  - `web`
  - `chrome`
  - `meilisearch`
- exposing the app on port `3000`
- importing an existing bookmark collection from a browser HTML export
- later enabling OpenAI-based auto-tagging through a dedicated OpenAI Platform project and API key
- using the service both on LAN and remotely through Tailscale on `seranogenomics` :contentReference[oaicite:3]{index=3}

## Main Goal

Set up a self-hosted Karakeep instance on `seranogenomics` so bookmark management would be more scalable, searchable, and usable across devices than ordinary browser bookmark folders. :contentReference[oaicite:4]{index=4}

## Environment

### Host
- Hostname: `seranogenomics`
- Stack path: `/opt/karakeep-app`
- Role: Docker host for the Karakeep stack :contentReference[oaicite:5]{index=5}

### Stack Design
Karakeep was deployed as a standalone Docker Compose stack rather than being folded into another application group. The stack used three services:

- `web`
- `chrome`
- `meilisearch` :contentReference[oaicite:6]{index=6}

### Published Access
- web UI exposed on port `3000` :contentReference[oaicite:7]{index=7}

### Persistence
Persistent Docker named volumes were used for:

- Karakeep app data
- Meilisearch data :contentReference[oaicite:8]{index=8}

## Starting State

Before Karakeep, bookmark management relied on ordinary browser bookmarks and bookmark folders. That was becoming cumbersome because the number of saved links was large, search and discovery were weak, and the approach did not feel very scalable across devices. The SG server already existed as a Docker host, making it a natural place to deploy Karakeep as a dedicated stack. :contentReference[oaicite:9]{index=9}

## Problems Addressed

This work addressed several related problems:

- replacing a large browser-bookmark/folder system with something more capable
- deploying Karakeep cleanly as a dedicated stack on `seranogenomics`
- setting up the required helper services for the app
- importing an existing bookmark collection from browser-exported HTML
- later adding OpenAI-based AI enrichment without mixing it into other projects
- making the service practical to access remotely through Tailscale :contentReference[oaicite:10]{index=10}

## Changes Made

## 1. Created a Dedicated Karakeep Stack

A standalone Karakeep stack was created under:

- `/opt/karakeep-app` :contentReference[oaicite:11]{index=11}

### Result
The bookmark manager was kept separate from other application groups and easier to reason about operationally. :contentReference[oaicite:12]{index=12}

## 2. Deployed the Required Compose Services

The stack was deployed with three services:

- `web`
- `chrome`
- `meilisearch` :contentReference[oaicite:13]{index=13}

### Result
The application and its supporting services came up in the expected structure for a working Karakeep deployment. :contentReference[oaicite:14]{index=14}

## 3. Published the Web UI

Karakeep was exposed on:

- `3000:3000` :contentReference[oaicite:15]{index=15}

### Result
The web UI became available locally on the homelab network and later usable remotely through Tailscale on the SG host. :contentReference[oaicite:16]{index=16}

## 4. Added Persistent Storage

Persistent Docker named volumes were used for both the application and search backend.

### Volumes
- `data:/data`
- `meilisearch:/meili_data` :contentReference[oaicite:17]{index=17}

### Result
Karakeep data and Meilisearch state were stored persistently rather than living only inside ephemeral containers. :contentReference[oaicite:18]{index=18}

## 5. Imported Existing Browser Bookmarks

An existing bookmark collection was imported from a browser-exported HTML file. :contentReference[oaicite:19]{index=19}

### Result
The service became immediately useful instead of starting from an empty library. :contentReference[oaicite:20]{index=20}

## 6. Added OpenAI-Based Auto-Tagging Later

OpenAI enrichment was not enabled immediately. A dedicated OpenAI Platform project and API key were created later for Karakeep-specific AI tagging. :contentReference[oaicite:21]{index=21}

### Result
AI tagging could be added without mixing the integration into unrelated projects or credentials. :contentReference[oaicite:22]{index=22}

## What Worked

The following parts of the deployment worked successfully:

- the Karakeep stack came up on `seranogenomics`
- the `web`, `chrome`, and `meilisearch` services all existed as expected
- the web container reported healthy status
- health checks returned `200`
- bookmark import from a browser HTML export worked
- remote access became practical through Tailscale
- OpenAI-based auto-tagging was added later through a separate OpenAI project and API key :contentReference[oaicite:23]{index=23}

## What Did Not Work

No major breakage stands out from this deployment. The main limitation was simply that OpenAI auto-tagging was not enabled immediately, because OpenAI Platform billing and API access had not yet been set up at first. The earliest version of the stack was therefore more basic, with AI enrichment added only after the core app was already working. :contentReference[oaicite:24]{index=24}

## Validation

### Compose Service Check

```bash
docker compose config --services
```

Expected service list:

```text
chrome
meilisearch
web
```

This confirmed that the Compose stack had the expected three-service structure. :contentReference[oaicite:25]{index=25}

### Image Snapshot

```bash
docker compose images
```

Observed image set:

```text
CONTAINER                    REPOSITORY                         TAG
karakeep-app-chrome-1        gcr.io/zenika-hub/alpine-chrome   124
karakeep-app-meilisearch-1   getmeili/meilisearch              v1.13.3
karakeep-app-web-1           ghcr.io/karakeep-app/karakeep     release
```

This matched the expected app, browser helper, and search backend layout. :contentReference[oaicite:26]{index=26}

### Persistent Volume Notes

Persistent volumes recorded in the deployment included:

- `karakeep-app_data`
- `karakeep-app_meilisearch`

Both were noted as created on `2025-12-12`. :contentReference[oaicite:27]{index=27}

## Environment Variables Worth Preserving

Useful environment variable names present in the `.env` included:

- `KARAKEEP_VERSION`
- `NEXTAUTH_SECRET`
- `MEILI_MASTER_KEY`
- `NEXTAUTH_URL`
- `OPENAI_API_KEY`
- `INFERENCE_TEXT_MODEL`
- `EMBEDDING_TEXT_MODEL`
- `INFERENCE_ENABLE_AUTO_TAGGING`
- `INFERENCE_ENABLE_AUTO_SUMMARIZATION`
- `DISABLE_SIGNUPS` :contentReference[oaicite:28]{index=28}

## Ending State

By the end of this phase, Karakeep was running on `seranogenomics` as its own Docker Compose stack in `/opt/karakeep-app`, with the web UI exposed on port `3000` and supported by `chrome` and `meilisearch`. Persistent volumes were in place for both Karakeep data and Meilisearch data. The bookmark collection had been imported from a browser HTML export, the service was usable on LAN and remotely via Tailscale, and OpenAI-based auto-tagging was later added through a dedicated OpenAI Platform project created specifically for Karakeep. :contentReference[oaicite:29]{index=29}

## Lessons Learned

- a dedicated self-hosted bookmark manager scales better than large browser bookmark folders once the collection gets big
- keeping Karakeep in its own Compose stack makes the deployment easier to understand and maintain
- persistent storage for both app data and search data matters from day one
- importing from an exported browser HTML file makes migration practical
- adding AI enrichment later is a clean path if the core app is already working
- using a dedicated OpenAI project for one service keeps credentials and usage easier to reason about
- Tailscale makes remote access practical without needing to redesign the whole access model :contentReference[oaicite:30]{index=30}

## Follow-Up Items

- document the full Karakeep workflow in `homelab-docs`, including import, export, and day-to-day usage
- document Tailscale-based remote access more explicitly
- monitor OpenAI tagging usage and cost over time
- consider a backup and export strategy for Karakeep data and Meilisearch data
- optionally document any browser extension or capture workflow if one becomes standardized later :contentReference[oaicite:31]{index=31}

## Commands and Details Worth Preserving

### Host and Path

```text
seranogenomics
/opt/karakeep-app
```

### Core Compose Services

```text
web
chrome
meilisearch
```

### Compose Image Notes

```text
ghcr.io/karakeep-app/karakeep:${KARAKEEP_VERSION:-release}
gcr.io/zenika-hub/alpine-chrome:124
getmeili/meilisearch:v1.13.3
published port: 3000:3000
```

### Volume Notes

```text
data:/data
meilisearch:/meili_data
```

### Access Notes

```text
- Initially used on LAN
- Later accessed remotely via Tailscale on SG
- Bookmark import was done from a browser-exported HTML file
- OpenAI project/API key was created later specifically for Karakeep auto-tagging
```

## Status

Completed as a successful standalone Karakeep deployment, with documentation, backup strategy, and long-term usage monitoring still remaining.

This one came from the uploaded summary you just shared. 
