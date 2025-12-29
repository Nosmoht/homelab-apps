# Repository Guidelines

## Project Structure & Module Organization
- `app-of-apps.yaml` bootstraps Argo CD using the “app of apps” pattern and points at `apps/`.
- `apps/` contains Argo CD `Application` manifests, one per component (e.g., `cert-manager.yaml`).
- `base/` holds versioned Kustomize bases for upstream components (e.g., `base/argocd/v2.14.8/`).
- `overlays/homelab/` contains environment-specific overlays, organized by component.
  - Argo CD applications should always point to `overlays/homelab/<component>`.

## Build, Test, and Development Commands
- `kustomize build overlays/homelab/<component>` renders manifests for a component locally.
- `kubectl apply -k overlays/homelab/<component>` applies an overlay to a cluster.
- `kubectl diff -k overlays/homelab/<component>` previews changes before applying.
- `kubectl apply -f app-of-apps.yaml` bootstraps Argo CD with all apps.

## Coding Style & Naming Conventions
- YAML uses 2-space indentation; keep keys sorted logically and grouped by resource.
- Component folders use kebab-case; version directories follow `vX.Y.Z`.
- App manifests in `apps/` are named `<component>.yaml` and should match the Argo CD `Application` name.

## Testing Guidelines
- No automated tests are defined in this repository.
- Validate changes by rendering manifests (`kustomize build ...`) and, if possible, dry-running apply (`kubectl apply --dry-run=client -k ...`).

## Commit & Pull Request Guidelines
- Commit messages follow Conventional Commits with short scopes (e.g., `feat: Piraeus Operator v2.8.1`, `fix: Node Feature Discovery ...`).
- PRs should describe the change, list affected components, and note any required cluster actions (sync, restart, or manual steps).
- If updating a base version, include the exact upstream version in the commit/PR text.

## Configuration Notes
- Keep environment-specific changes in `overlays/homelab/` and avoid modifying upstream base manifests unless necessary.
- Prefer Argo CD syncs for rollout; apply directly only for bootstrapping or troubleshooting.
 - When upgrading components, add a new `base/<component>/vX.Y.Z/` and update the overlay to match the latest stable release.
