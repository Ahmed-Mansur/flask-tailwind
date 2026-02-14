# Local IDP (Backstage) Notes

This repo is a Backstage app (IDP) with a small example Flask + Tailwind service inside `flask-tailwind/`.

## What Was Changed For Local IDP Detection

The goal was: start the IDP locally and make it automatically discover the entities defined in this repo.

Changes made:

- Catalog locations: added local file locations so the catalog ingests the repo's entities on startup.
  - File: `app-config.yaml`
  - Added:
    - `catalog.locations` -> `../../catalog-info.yaml`
    - `catalog.locations` -> `../../flask-tailwind/catalog-info.yaml`

- Owner relation fix: the `flask-tailwind` component originally used an email as `spec.owner`, which Backstage treated as an entity reference to a missing group.
  - File: `examples/org.yaml`
  - Added group entity: `group:default/fa22-bcs-183` with `spec.profile.email: fa22-bcs-183@cuilahore.edu.pk`
  - Updated:
    - `catalog-info.yaml` -> `spec.owner: group:default/fa22-bcs-183`
    - `flask-tailwind/catalog-info.yaml` -> `spec.owner: group:default/fa22-bcs-183`

- Local dev hygiene: the Python virtualenv created for the Flask demo is not meant to be committed.
  - File: `.gitignore`
  - Added: `.venv/`

## How To Start The IDP Locally

This repo uses Yarn v4 (Berry). If `yarn` isn't available globally, you can run it via `npx`.

From the repo root:

```bash
npx --yes @yarnpkg/cli-dist@4.4.1 install
npx --yes @yarnpkg/cli-dist@4.4.1 start
```

Local URLs:

- Frontend: http://localhost:3000
- Backend: http://localhost:7007

## What The IDP Includes (Current Feature Set)

Enabled backend plugins/modules (see `packages/backend/src/index.ts` and `app-config.yaml`):

- Auth: `guest` provider (auto sign-in)
- Catalog: entity ingestion + entity pages
- Scaffolder: software templates + GitHub scaffolder module
- TechDocs: local publisher (for local dev)
- Search: catalog + techdocs collators; scheduled indexing tasks
- Permissions: permission backend + allow-all policy module
- Proxy: backend proxy plugin
- Kubernetes: plugin is present (requires kubernetes config to fully enable)
- Notifications + Signals: backend plugins are present

Local dev config highlights (see `app-config.yaml`):

- Database: in-memory SQLite (`better-sqlite3` with `:memory:`) so catalog state resets on restart.
- CORS: `http://localhost:3000` with credentials enabled.

## What It's Doing Right Now (When Running)

With `yarn start` running:

- Serves the frontend dev server on port 3000.
- Serves the backend on port 7007.
- Reads `catalog.locations` and ingests local entities (including this repo's `catalog-info.yaml` files and `examples/org.yaml`).
- Runs scheduled background tasks (e.g. catalog orphan cleanup and search indexing).
- Uses guest auth and issues a token for `user:development/guest`.

Common local warnings you may see in logs:

- Kubernetes plugin warns if no kubernetes config is set.
- Events bus endpoints may return 404 if the events backend isn't installed (signals/catalog will still work locally without it).

## How To Confirm This Repo Is Detected

In the UI (http://localhost:3000):

- Catalog -> Components should include:
  - `idp` (from `catalog-info.yaml`)
  - `flask-tailwind` (from `flask-tailwind/catalog-info.yaml`)
- Catalog -> Groups should include `fa22-bcs-183` (from `examples/org.yaml`).

## Production-Style Registration (GitHub)

For a real IDP setup, you typically register entities by URL rather than local file paths:

- Create -> Register Existing Component -> paste a URL to `catalog-info.yaml`
- Configure `integrations.github` and provide `GITHUB_TOKEN` (or other auth) as needed.
