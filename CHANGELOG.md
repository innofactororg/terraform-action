# Changelog

All notable changes to this project will be documented in this file.

## [v1.0.10] - 2026-04-27

### Fixed

- **Infracost baseline scope** — the infracost baseline was previously keyed
  per-PR (`definition-plan_{repo}_PR{number}`), meaning every new PR started
  from zero and showed absolute cost rather than a cost delta. The baseline is
  now keyed per-environment, derived from `state_name` (e.g.
  `test/terraform.tfstate` → `test/terraform-infracost.json`) and stored in the
  same state container. This gives a true cost delta for every PR: "what does
  this change add or remove from what is currently deployed?"

### Changed

- **Infracost baseline updated on apply, not on plan** — a new "Update
  infracost baseline" step runs after a successful `apply` and uploads the
  post-apply breakdown as the environment baseline. Plan runs only read the
  baseline; they no longer write it. This ensures the baseline always reflects
  deployed infrastructure, not a speculative plan.

---

## [v1.0.9] - 2026-04-27

### Fixed

- **Cross-org secret passing** — replaced `secrets: inherit` in `bootstrap.yml`,
  `terraform-init.yml`, and `terraform-job.yml` with explicit secret lists.
  GitHub cannot re-export inherited secrets across organization boundaries;
  callers from outside `innofactororg` would silently receive empty secrets.

- **State blob subdirectory creation** — `terraform-init.yml` now runs
  `mkdir -p "$(dirname ...)"` before downloading a state blob whose
  `state_name` contains a subdirectory prefix (e.g. `test/terraform.tfstate`).

- **`var_path` resolution after `cd`** — the terraform step changes directory
  to `src/<code_path>` before running. `var_path` is now resolved from the
  workspace root (`${start_path}/src/<var_path>`) so it is not accidentally
  re-rooted under the new working directory.

- **`terraform show -json` stderr isolation** — plan JSON is now generated
  inside the terraform step (while providers are still initialized) with stderr
  redirected to a separate log file and validated with `jq empty`, preventing
  provider warnings from corrupting the JSON consumed by infracost.

- **`containers_list` in backup instance policy** — `terraform-init.yml`
  correctly patches `containers_list` in the BlobBackupDatasourceParameters
  JSON (not the deprecated `containers_backup_friendly_names` field).

- **`jq -c` + quoted echo in terraform-init.yml** — compact `jq` output and
  a properly quoted `echo` prevent newlines from being mangled in the
  `bvi_request` JSON payload.

### Changed

- **Infracost: blob-storage run-to-run baseline** — the infracost step now
  stores the absolute cost breakdown (`infracost-abs.json`) in the plan blob
  container after each plan run, and downloads the previous run's breakdown as
  the `--compare-to` baseline on the next run. This replaces the previous
  approach of re-running `infracost breakdown` against the git `main` checkout,
  which produced false cost deltas on repos with multiple `.tfvars` files
  (infracost auto-detects each file as a separate project, causing spurious
  add/remove rows). Using `--path=plan.json` for both runs guarantees
  consistent project names and accurate deltas.

- **`var_path` input description** — clarified that `var_path` is relative to
  the repository root (same convention as `code_path`), and that it defaults
  to `code_path` (tfvars alongside .tf files). For multi-environment repos,
  set it to the environment-specific subdirectory.

---

## [v1] - 2024 (initial release)

Initial public release. Single-environment reusable workflow supporting
`/plan`, `/apply`, `/import`, `/state`, `/graph`, `/untaint`, `/scan`,
and `/force-unlock` PR comment commands with Azure blob-backed Terraform
state and infracost cost reporting.
</content>
</invoke>