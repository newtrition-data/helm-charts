# Agent Guidelines (helm-charts)
This is a Helm chart repository (not an application). Typical work: edit `charts/typesense/**`, render/lint locally, then (when publishing) package the chart and regenerate `index.yaml` for GitHub Pages.

## Repo layout
- Chart source:
  - `charts/typesense/Chart.yaml`: chart metadata + versions
  - `charts/typesense/values.yaml`: default values (should render a working install)
  - `charts/typesense/templates/`: Kubernetes manifests (Helm Go templates)
  - `charts/typesense/templates/_helpers.tpl`: shared helpers (naming/labels)
  - `charts/typesense/templates/tests/test-connection.yaml`: Helm test hook pod
- Generated outputs (do not hand-edit):
  - `index.yaml`: Helm repository index used by GitHub Pages
  - `typesense-*.tgz`: packaged chart artifacts

## Build/lint/test commands
Requirements: Helm v3; for cluster checks/tests: `kubectl` + a reachable cluster.

Lint:
- `helm lint charts/typesense`
- With values (catches missing keys): `helm lint charts/typesense -f charts/typesense/values.yaml`
- Stricter lint (if you want CI-like behavior): `helm lint charts/typesense --strict`

Render manifests ("build"):
- `helm template my-release charts/typesense`
- Debug rendering: `helm template my-release charts/typesense --debug`
- Render one file: `helm template my-release charts/typesense --show-only charts/typesense/templates/statefulset.yaml`
- Override values: `helm template my-release charts/typesense -f /path/to/values.yaml` or `--set replicaCount=3`
- Quick sanity check of default values: `helm show values charts/typesense`

Install/upgrade (optional, cluster):
- `helm install my-release charts/typesense -n my-ns --create-namespace`
- `helm upgrade --install my-release charts/typesense -n my-ns`
- `helm upgrade --install my-release charts/typesense -n my-ns --dry-run --debug`

Tests (Helm test hooks): `charts/typesense/templates/tests/test-connection.yaml`
- Run all tests: `helm test my-release -n my-ns`
- Run a single test: `helm test my-release -n my-ns --filter test-connection`
- Debug failures:
  - `kubectl get pods -n my-ns -l "helm.sh/hook=test"`
  - `kubectl describe pod -n my-ns <pod-name>`
  - `kubectl logs -n my-ns <pod-name>`

Common debug loops:
- Render and inspect one resource: `helm template my-release charts/typesense --show-only charts/typesense/templates/service.yaml`
- Check computed values: `helm get values my-release -n my-ns` (after install)
- Compare live vs desired: `helm diff upgrade ...` (requires helm-diff plugin)

## Working Agreements for Agents

- Avoid destructive git operations (no force pushes, no hard resets).
- Do not commit real secrets; do not add credentials to `values.yaml`.
- If you change chart sources, keep generated artifacts (`typesense-*.tgz`, `index.yaml`) consistent with the bumped chart version.
- Prefer small, reviewable diffs; keep templates readable and deterministic.

## Local Dev Tips

- Prefer iterating via `helm template ... --show-only <template>` before installing.
- When touching anything under `charts/typesense/templates/`, run:
  - `helm lint charts/typesense -f charts/typesense/values.yaml`
  - `helm template my-release charts/typesense`

## Packaging and repo index (publishing)
If you change `charts/typesense/**`:
- Bump `charts/typesense/Chart.yaml:version`
- `helm package charts/typesense` (creates `typesense-<version>.tgz`)
- `helm repo index --url https://newtrition-data.github.io/helm-charts/ .` (updates `index.yaml`)

Versioning notes:
- `Chart.yaml:version` is SemVer for the chart package (bump on any chart/template/values change).
- `Chart.yaml:appVersion` tracks the upstream Typesense app version and should be quoted.

Generated files policy:
- Do not edit `index.yaml` by hand; regenerate it.
- Do not manually create or modify `typesense-*.tgz`; use `helm package`.

## Code style guidelines (YAML + Helm templates)
General:
- Keep templates deterministic (no cluster lookups); defaults in `values.yaml` should render a working install.
- Prefer readability over cleverness; keep changes easy to diff.
- Avoid huge inline template expressions; favor helpers for non-trivial logic.

Formatting:
- YAML uses 2-space indentation; avoid trailing whitespace; avoid extra blank lines at EOF.
- Keep key order stable within resources: `metadata` -> `spec` -> `template` -> `containers` -> `env` -> `resources`.
- Keep selectors/labels adjacent and consistent across resources.

Templating and reuse ("imports"):
- Reuse via helpers and `include` (not language imports). Put shared logic in `charts/typesense/templates/_helpers.tpl`.
- Prefer `include` over `template` so you can pipe to `nindent`, `quote`, etc.
- Use whitespace trimming `{{- ... -}}` to avoid stray blank lines.
- For structured values: `toYaml ... | nindent N`; for injected blocks: `include ... | nindent N`.
- Keep `if`/`range` blocks shallow; avoid big one-line YAML+template expressions.
- If you must build a comma-separated string, consider using `list` + `append` + `join` to keep it readable.

Values, types, and quoting:
- Treat `.Values` as untrusted input; guard optional sections with `if`.
- Keep value shapes consistent (maps for `resources`/`persistence`, booleans for feature flags).
- Kubernetes `env.value` is a string: use `| quote` when mapping booleans/numbers into env vars.
- Prefer configurable fields over hardcoded constants (ports, image tag, replica count, persistence).
- When rendering user-provided strings into YAML, `quote` unless you explicitly want YAML typing.

Secrets and sensitive config:
- Do not commit real secrets in `values.yaml` (placeholders are fine).
- Prefer `existingSecret` + `secretKeyRef` for secret-like values; otherwise use `required` to force user-provided values.
- Do not print secrets into ConfigMaps; use Secrets or secret refs.

Naming conventions:
- Resource names come from `{{ include "typesense.fullname" . }}`; add clear suffixes (`-service`, `-nodes`, `-test-connection`).
- Helper names are chart-namespaced (`typesense.*`) and must output DNS-safe names (use `trunc 63` + `trimSuffix "-"`).
- Keep selector labels stable (`typesense.selectorLabels`); changing them breaks upgrades.
- Prefer `typesense.labels` and `typesense.selectorLabels` helpers instead of hand-written label maps.

Validation and errors:
- Use `required` for must-provide values; use `fail` for invalid combinations. Messages should say what to set and where.
- Examples:
  - `{{ required "values.env.TYPESENSE_API_KEY is required" .Values.env.TYPESENSE_API_KEY }}`
  - `{{- if and .Values.persistence.enabled (not .Values.persistence.size) -}}{{ fail "persistence.enabled=true requires persistence.size" }}{{- end -}}`

Upgrade safety:
- Do not change label selectors, volumeClaimTemplate names, or StatefulSet identity fields lightly.
- If you must change a breaking field, document it in the PR and bump the chart version appropriately.

Kubernetes best practices:
- Keep `resources` configurable; avoid hardcoding.
- Avoid running as root unless required; if you must, document why and make it configurable.
- Use `readinessProbe`/`livenessProbe` where appropriate (and make them configurable if added).
- Ensure Services select the same labels used by workloads.

## Review Checklist (Before PR)

- `helm lint charts/typesense` passes (and preferably `--strict`).
- `helm template my-release charts/typesense` renders without errors.
- Chart `version` is bumped for any `charts/typesense/**` change.
- Generated artifacts updated when publishing: `typesense-*.tgz` and `index.yaml`.
- No secrets added; secret-like values use `existingSecret` or `required`.

## Cursor / Copilot rules
- No Cursor rules found in `.cursor/rules/` or `.cursorrules`.
- No Copilot instructions found in `.github/copilot-instructions.md`.
