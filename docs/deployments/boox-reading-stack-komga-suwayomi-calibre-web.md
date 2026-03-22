# BOOX Reading Stack: Komga, Suwayomi, and Calibre-Web

This page documents the setup of a self-hosted reading stack for a BOOX Go 7 BW using Komga, Suwayomi, and Calibre-Web, designed to be reachable over LAN and Tailscale in a way that parallels the rest of the homelab media stack.

## Summary

The goal of this phase was to stand up a clean self-hosted reading stack for manga, comics, and books, using the same general philosophy as the existing YAMS and Jellyfin setup.

The work centered on:

- finding and validating the NAS-backed storage path already mounted on `seranogenomics`
- deciding to keep the reading services in a separate stack rather than mixing them into YAMS
- deploying Komga, Calibre-Web, and Suwayomi under a dedicated `/opt/reading` Docker Compose stack
- making BOOX app workflows practical for both comics/manga and books
- fixing Calibre-Web library issues
- fixing BOOX connectivity and authentication issues
- tuning Suwayomi output and update behavior for Komga compatibility
- carrying the new stack cleanly through the 5.x network renumbering

## Main Goal

Stand up a self-hosted reading stack for a BOOX Go 7 BW that would:

- host manga and comics cleanly
- expose books through OPDS
- work over Tailscale
- fit into the homelab in a clean, maintainable way
- use NAS-backed storage instead of ad hoc local paths

## Environment

### Existing Homelab Baseline
- main server: `seranogenomics`
- existing media stack: YAMS under `/opt/yams`
- YAMS configs bind-mounted under `/opt/yams/config/<service>`
- NAS archive share mounted on `seranogenomics` via CIFS at `/mnt/nas-archive`
- BOOX already connected to the network through Tailscale

### Reading Stack Design
A new dedicated reading stack was created under:

- `/opt/reading`

with config paths standardized to:

- `/opt/reading/config/<service>`

### Primary Services
- Komga
- Suwayomi
- Calibre-Web

### BOOX Client Apps
- Mihon for manga/comics via Komga
- KOReader for books via Calibre-Web OPDS

## Starting State

At the beginning of this phase:

- the homelab already had a working Docker and YAMS pattern
- the NAS archive share was already mounted on the main server
- the BOOX was already on Tailscale
- there was no dedicated self-hosted reading stack yet
- there was no finalized workflow for manga, comics, or books on the BOOX

## Problems Addressed

This work covered several connected setup and usability problems:

- identifying the correct NAS mount path from the CLI
- deciding whether to create a separate reading stack or mix it into YAMS
- Calibre-Web rejecting the library path because no valid Calibre `metadata.db` existed yet
- deciding on a sane BOOX app workflow
- getting Suwayomi output into a format Komga could index cleanly
- disabling Suwayomi smart or intelligent update behavior that was not wanted
- fixing BOOX connectivity and login issues when accessing Komga remotely
- reducing friction in KOReader OPDS navigation

## Changes Made

## 1. Confirmed the NAS Mount Path on the Main Server

The first step was confirming the real NAS-backed path already in use by the homelab.

### Confirmed Path
- `/mnt/nas-archive`

### Result
This became the storage base for the reading stack.

## 2. Chose a Separate Reading Stack Instead of Mixing Into YAMS

A design decision was made to keep the reading services separate.

### Decision
Use a dedicated stack under:

- `/opt/reading`

rather than mixing reading services into the existing YAMS stack.

### Result
The reading setup stayed cleaner and easier to reason about.

## 3. Created NAS-Backed Reading Directories

New directories were created on the NAS-backed share for the reading workflow.

### Created Paths
- `/mnt/nas-archive/manga`
- `/mnt/nas-archive/comics`
- `/mnt/nas-archive/books/CalibreLibrary`

### Result
The reading stack had persistent storage paths aligned with the rest of the homelab.

## 4. Deployed the Reading Stack

A new Docker Compose stack was deployed in `/opt/reading`.

### Services
- Komga on port `25600`
- Calibre-Web on port `8083`
- Suwayomi on port `4567`

### Result
The three core reading services came up cleanly and formed the basis of the BOOX workflow.

## 5. Standardized Config Layout

The new stack was aligned with the existing YAMS layout style.

### Config Convention
- `/opt/reading/config/komga`
- `/opt/reading/config/calibre-web`
- `/opt/reading/config/suwayomi`

### Result
The reading stack matched the same general maintenance pattern already used elsewhere in the homelab.

## 6. Added a Reading Stack Restart Helper

A simple CLI helper was added for convenience.

### Helper Command
- `reading restart`

### Result
The whole stack could be restarted with one short command, matching the spirit of the existing homelab tooling.

## 7. Fixed Calibre-Web Library Path Requirements

Calibre-Web initially failed with an invalid database path error.

### Root Cause
Calibre-Web requires a real Calibre library with a valid:

- `metadata.db`

### Fix
Calibre on Windows was used to write to the NAS-backed library path so the proper Calibre metadata database existed.

### Result
Calibre-Web could successfully use the library and expose it over OPDS.

## 8. Tuned Suwayomi for Komga Compatibility

Suwayomi was configured to behave more like a clean content feeder for Komga.

### Changes
- set downloads to save as **CBZ**
- enabled auto-download of new chapters
- identified and disabled **Smart/Intelligent updates**

### Result
Suwayomi output became more compatible with Komga indexing and less likely to behave in unwanted ways.

## 9. Established the BOOX App Workflow

A practical per-content-type client model was chosen.

### Manga / Comics
- Mihon -> Komga source

### Books
- KOReader -> Calibre-Web OPDS

### Result
The BOOX ended up with a workable split workflow instead of trying to force one app to do everything.

## 10. Fixed BOOX Connectivity and Login Issues

The initial BOOX -> Komga path had a couple of early failures.

### Problems
- Tailscale being off caused timeouts
- Mihon received `401 Unauthorized` until Komga credentials were corrected

### Result
Once Tailscale was active and credentials were correct, BOOX access to Komga worked as intended.

## 11. Reduced KOReader Friction

KOReader OPDS navigation was still somewhat clunky.

### Change
A corner gesture was bound in KOReader to reduce the number of steps needed to return to the catalog or OPDS flow.

### Result
The book-reading flow became noticeably less annoying.

## 12. Carried the Stack Through Network Renumbering

The reading stack also had to survive the move from legacy 7.x and 8.x addresses to the 5.x LAN.

### Renumbering Applied
- `seranogenomics`: `192.168.7.5` -> `192.168.5.7`
- `nas`: `192.168.8.2` -> `192.168.5.8`

### Result
Paths, references, and mounts were updated and the reading stack remained functional after the renumbering.

## What Worked

The following parts of the setup worked well:

- Docker images pulled and containers started cleanly
- Komga successfully indexed Suwayomi-downloaded content from the NAS-backed directory
- Mihon -> Komga worked on the BOOX after fixing Tailscale and credentials
- streaming manga and comics from Komga to Mihon was acceptable performance-wise
- Calibre on Windows successfully created a proper Calibre library with `metadata.db`
- Calibre-Web exposed the library successfully over OPDS
- KOReader gesture binding reduced friction in the OPDS workflow
- the stack continued working after the 5.x renumbering

## What Did Not Work

A few issues came up during the buildout:

### Initial BOOX -> Komga Access
The first BOOX connection attempts failed because:

- Tailscale was off, causing timeouts
- Komga credentials were wrong, causing `401 Unauthorized`

### Calibre-Web Library Validation
Calibre-Web would not accept the library path until a real Calibre library with `metadata.db` existed.

## Validation

### Confirm NAS Mount Path

```bash
findmnt -t cifs
df -hT /mnt/nas-archive
ls -lah /mnt/nas-archive
```

These checks confirmed the existing NAS-backed path used by the homelab.

### Create Reading Directories

```bash
mkdir -p /mnt/nas-archive/manga
mkdir -p /mnt/nas-archive/comics
mkdir -p /mnt/nas-archive/books/CalibreLibrary
```

### Permission Sanity Check

```bash
touch /mnt/nas-archive/manga/.perm_test && rm /mnt/nas-archive/manga/.perm_test
```

This confirmed the mounted NAS path was writable.

### Determine Existing YAMS Compose Layout

```bash
docker inspect -f 'WORKDIR={{ index .Config.Labels "com.docker.compose.project.working_dir" }}' jellyfin
docker inspect -f 'COMPOSE_FILES={{ index .Config.Labels "com.docker.compose.project.config_files" }}' jellyfin
docker inspect -f '{{range .Mounts}}{{println .Type .Source "->" .Destination}}{{end}}' jellyfin
```

These checks were used to mirror the established homelab layout pattern.

### Deploy the Reading Stack

```bash
cd /opt/reading
docker compose up -d
```

### Verify Calibre-Web Library Access

```bash
docker exec -it calibre-web ls -lah /books
docker exec -it calibre-web ls -lah /books/CalibreLibrary
```

These checks confirmed Calibre-Web could see the expected library path.

### Verify Komga View of Suwayomi Downloads

```bash
docker exec -it komga ls -lah /data/manga/_suwayomi-downloads | head
```

This confirmed Komga could see the Suwayomi-fed directory.

### Reading Restart Helper

```bash
sudo tee /usr/local/bin/reading >/dev/null <<'EOF'
#!/usr/bin/env bash
set -euo pipefail
if [[ "${1:-}" != "restart" ]]; then
  echo "Usage: reading restart"
  exit 1
fi
cd /opt/reading
docker compose down
docker compose up -d
EOF
sudo chmod +x /usr/local/bin/reading
```

### Renumbering Sanity Check

```bash
findmnt /mnt/nas-archive
```

This was part of verifying that the reading stack still pointed at the correct NAS-backed path after renumbering.

## Ending State

By the end of this phase:

### Reading Stack
- `/opt/reading` was up and running
- Komga was available on `:25600`
- Calibre-Web was available on `:8083`
- Suwayomi was available on `:4567`

### Storage
- reading storage was NAS-backed under `/mnt/nas-archive`
- Suwayomi downloads were stored as CBZ under a manga path on the NAS-backed share
- Komga was configured to read from the relevant NAS-backed manga directory
- the books library existed at `/mnt/nas-archive/books/CalibreLibrary`

### BOOX Workflow
- manga and comics workflow:
  - Mihon -> Komga
- books workflow:
  - KOReader -> Calibre-Web OPDS

### Network State
- `seranogenomics` was on `192.168.5.7`
- `nas` was on `192.168.5.8`
- the reading stack remained functional after the IP renumbering

## Lessons Learned

- keeping the reading stack separate from YAMS made the design easier to understand
- Calibre-Web expects a real Calibre library, not just an arbitrary folder
- streaming manga and comics from Komga is good enough that local-download assumptions can often be relaxed
- BOOX connectivity problems are sometimes just Tailscale state or credential problems, not deeper stack failures
- Suwayomi output settings matter if Komga is expected to index the results cleanly
- small UX tweaks like KOReader gesture bindings can matter a lot for day-to-day usability
- carrying new services through renumbering is much easier when paths and compose layouts are already clean

## Follow-Up Items

- optionally tune Komga library performance settings, especially if hashing or page-dimension analysis are not needed
- optionally restart Komga automatically after Suwayomi's nightly update window if a more hands-off new-chapter flow is desired
- optionally separate manually managed manga from Suwayomi downloads to avoid duplicates
- continue treating the reading stack as its own small service group rather than stuffing it into YAMS without a strong reason

## Status

Completed as a successful standalone reading-stack deployment, with only optional polish and workflow tuning remaining.
