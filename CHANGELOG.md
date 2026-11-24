# Changelog

All notable changes to this project will be documented in this file.

Each release has a `## [version] - YYYY-MM-DD` section. The Release workflow publishes that section as the release notes and announces it in Teams, and fails if the section is missing. Add the entry in the pull request that makes the change.

## Unreleased

### Deprecated

- This workflow is deprecated in favour of the Atlantis Terraform PR automation pattern (AVS `avm/ptn/lz/atlantis`). Existing tags keep working, only security fixes until the repository is archived on 2027-02-22 (#PR)

## [v1.0.10] - 2026-04-27

Pull request: [#17](https://github.com/innofactororg/terraform-action/pull/17).

### Fixed

- **Infracost baseline scope** ([#17](https://github.com/innofactororg/terraform-action/pull/17), [0c67ad6](https://github.com/innofactororg/terraform-action/commit/0c67ad6)) — the infracost baseline was previously keyed
  per-PR (`definition-plan_{repo}_PR{number}`), meaning every new PR started
  from zero and showed absolute cost rather than a cost delta. The baseline is
  now keyed per-environment, derived from `state_name` (e.g.
  `test/terraform.tfstate` → `test/terraform-infracost.json`) and stored in the
  same state container. This gives a true cost delta for every PR: "what does
  this change add or remove from what is currently deployed?"

### Changed

- **Infracost baseline updated on apply, not on plan** ([#17](https://github.com/innofactororg/terraform-action/pull/17), [0c67ad6](https://github.com/innofactororg/terraform-action/commit/0c67ad6)) — a new "Update
  infracost baseline" step runs after a successful `apply` and uploads the
  post-apply breakdown as the environment baseline. Plan runs only read the
  baseline; they no longer write it. This ensures the baseline always reflects
  deployed infrastructure, not a speculative plan.

## [v1.0.9] - 2026-04-27

Pull requests: [#13](https://github.com/innofactororg/terraform-action/pull/13), [#15](https://github.com/innofactororg/terraform-action/pull/15), [#16](https://github.com/innofactororg/terraform-action/pull/16).

### Fixed

- **Cross-org secret passing** ([#13](https://github.com/innofactororg/terraform-action/pull/13), [8102c46](https://github.com/innofactororg/terraform-action/commit/8102c46)) — replaced `secrets: inherit` in `bootstrap.yml`,
  `terraform-init.yml`, and `terraform-job.yml` with explicit secret lists.
  GitHub cannot re-export inherited secrets across organization boundaries;
  callers from outside `innofactororg` would silently receive empty secrets.

- **State blob subdirectory creation** ([#13](https://github.com/innofactororg/terraform-action/pull/13), [8102c46](https://github.com/innofactororg/terraform-action/commit/8102c46)) — `terraform-init.yml` now runs
  `mkdir -p "$(dirname ...)"` before downloading a state blob whose
  `state_name` contains a subdirectory prefix (e.g. `test/terraform.tfstate`).

- **`var_path` resolution after `cd`** ([#13](https://github.com/innofactororg/terraform-action/pull/13), [8102c46](https://github.com/innofactororg/terraform-action/commit/8102c46)) — the terraform step changes directory
  to `src/<code_path>` before running. `var_path` is now resolved from the
  workspace root (`${start_path}/src/<var_path>`) so it is not accidentally
  re-rooted under the new working directory.

- **`terraform show -json` stderr isolation** ([#13](https://github.com/innofactororg/terraform-action/pull/13), [8102c46](https://github.com/innofactororg/terraform-action/commit/8102c46)) — plan JSON is now generated
  inside the terraform step (while providers are still initialized) with stderr
  redirected to a separate log file and validated with `jq empty`, preventing
  provider warnings from corrupting the JSON consumed by infracost.

- **`containers_list` in backup instance policy** ([#13](https://github.com/innofactororg/terraform-action/pull/13), [8102c46](https://github.com/innofactororg/terraform-action/commit/8102c46)) — `terraform-init.yml`
  correctly patches `containers_list` in the BlobBackupDatasourceParameters
  JSON (not the deprecated `containers_backup_friendly_names` field).

- **`jq -c` + quoted echo in terraform-init.yml** ([#13](https://github.com/innofactororg/terraform-action/pull/13), [8102c46](https://github.com/innofactororg/terraform-action/commit/8102c46)) — compact `jq` output and
  a properly quoted `echo` prevent newlines from being mangled in the
  `bvi_request` JSON payload.

### Changed

- **Infracost: blob-storage run-to-run baseline** ([#15](https://github.com/innofactororg/terraform-action/pull/15), [6d4225d](https://github.com/innofactororg/terraform-action/commit/6d4225d)) — the infracost step now
  stores the absolute cost breakdown (`infracost-abs.json`) in the plan blob
  container after each plan run, and downloads the previous run's breakdown as
  the `--compare-to` baseline on the next run. This replaces the previous
  approach of re-running `infracost breakdown` against the git `main` checkout,
  which produced false cost deltas on repos with multiple `.tfvars` files
  (infracost auto-detects each file as a separate project, causing spurious
  add/remove rows). Using `--path=plan.json` for both runs guarantees
  consistent project names and accurate deltas.

- **`var_path` input description** ([#13](https://github.com/innofactororg/terraform-action/pull/13), [8102c46](https://github.com/innofactororg/terraform-action/commit/8102c46)) — clarified that `var_path` is relative to
  the repository root (same convention as `code_path`), and that it defaults
  to `code_path` (tfvars alongside .tf files). For multi-environment repos,
  set it to the environment-specific subdirectory.

## [v1.0.8] - 2024-04-23

### Added

- `terraform_version` input ([#12](https://github.com/innofactororg/terraform-action/pull/12))

## [v1.0.7] - 2024-04-23

### Changed

- Update actions for Node 20 ([#11](https://github.com/innofactororg/terraform-action/pull/11))

### Fixed

- Ensure the state file is created if it does not exist on a `/plan` comment ([#10](https://github.com/innofactororg/terraform-action/pull/10))

## [v1.0.6] - 2024-04-23

### Fixed

- Wrong name of the terraform output and error file ([#9](https://github.com/innofactororg/terraform-action/pull/9))
- Wrong variable name for the resource group in "Establish resource group" ([#9](https://github.com/innofactororg/terraform-action/pull/9))

## [v1.0.5] - 2023-11-27

### Fixed

- Remove `typeset -l` for path variables ([#8](https://github.com/innofactororg/terraform-action/pull/8))

## [v1.0.4] - 2023-11-27

### Added

- More tests for terraform log files ([#7](https://github.com/innofactororg/terraform-action/pull/7))

## [v1.0.3] - 2023-11-27

### Fixed

- Touch the log file ([#6](https://github.com/innofactororg/terraform-action/pull/6))

## [v1.0.2] - 2023-08-09

### Fixed

- Release action, terraform-init, terraform-job and README after the repo rename ([#5](https://github.com/innofactororg/terraform-action/pull/5))

### Changed

- Bump size-label-action to v0.5.0 ([#5](https://github.com/innofactororg/terraform-action/pull/5))

## [v1.0.1] - 2023-06-09

### Added

- `require_approved_review_for_apply` ([#4](https://github.com/innofactororg/github-workflow-terraform-ops/pull/4))
- Job number in `comment_when_start` ([#4](https://github.com/innofactororg/github-workflow-terraform-ops/pull/4))
- Exit 1 for `deny_plan` and `deny_apply` ([#4](https://github.com/innofactororg/github-workflow-terraform-ops/pull/4))

### Fixed

- Finding the backup vault principalId ([#1](https://github.com/innofactororg/github-workflow-terraform-ops/pull/1))
- Improve the error logs that can be downloaded from the artifacts after a job has completed ([#4](https://github.com/innofactororg/github-workflow-terraform-ops/pull/4))

## [v1.0.0] - 2023-06-06

No notes were recorded for this release (originally published as `github-workflow-terraform-ops`). Commits: "Initial commit" (1f3dedc) and "Release v1" (fdabe62).
