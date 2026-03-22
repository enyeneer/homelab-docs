# Infrastructure Files

This folder contains sanitized infrastructure files from the homelab, primarily Docker Compose and environment examples.

The goal is to document how services are structured and deployed without exposing secrets, private credentials, or overly brittle personal details.

Redacted information is generally replaced with "..."

## Purpose

This folder exists to make the homelab more reproducible and easier to understand from an infrastructure point of view.

It is intended to show:

- how standalone stacks are organized
- how selected services are composed and wired together
- which configuration patterns are used repeatedly
- how the public-facing examples differ from the live private configs

## Expected Contents

Examples of files that belong here:

- Docker Compose files for standalone stacks
- sanitized compose overrides
- `.env.example` files
- notes about required variables, mounts, ports, and dependencies

## What Should Be Published

Good candidates for this folder include:

- standalone reading stack Compose files
- Karakeep Compose files
- selected sanitized YAMS custom Compose snippets
- example environment files with placeholder values
- small helper notes explaining service dependencies

## Notes on Accuracy

These files are public documentation examples, not guaranteed exact mirrors of the current live environment.

The live homelab may evolve faster than the public examples, especially where:

- secrets are removed
- values are generalized
- paths are simplified
- service definitions are trimmed for clarity

## Related Documentation

For architecture, service descriptions, incident history, and operational notes, see the main documentation under `docs/`.
