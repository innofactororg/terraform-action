# Terraform workflow

A reusable workflow to perform Terraform operations like plan and apply.

This workflow is designed to trigger on either a `pull request` or a
pull request `comment`.

When triggered by a `pull request`, a terraform plan is created.

When triggered by a `comment`, the workflow will inspect the comment and
act on specific commands, as [listed below](#commands).

The comment can include more than one command, each on a separate line,
that will be executed in the order that they are specified.

To include additional text in the comment, add three number signs (###)
after the commands and on a separate line. The text will then be
preserved in the comment.

## Commands

<!-- no toc -->
* [/plan](#terraform-plan)
* [/apply](#terraform-apply)
* [/force-unlock](#terraform-force-unlock)
* [/graph](#terraform-graph)
* [/import](#terraform-import)
* [/state list](#terraform-state-list)
* [/state mv](#terraform-state-mv)
* [/state rm](#terraform-state-rm)
* [/state show](#terraform-state-show)
* [/untaint](#terraform-untaint)
* [/scan](#security-scan)

Each command can have additional arguments. The arguments must be placed
after the command, separated by a space, and must be valid terraform
arguments. The following arguments will not be passed on:

`-no-color`
`-input=false`
`-out=FILENAME`
`-auto-approve`
`-detailed-exitcode`

Example:

```text
/import module.csoc_solution.module.resource_groups["p-sec-mon"].azurerm_resource_group.rg /subscriptions/64499dc8-d437-45d7-8d2e-4910e88e17b2/resourceGroups/p-sec-mon
/import module.csoc_solution.module.resource_groups["p-sec-auto"].azurerm_resource_group.rg /subscriptions/64499dc8-d437-45d7-8d2e-4910e88e17b2/resourceGroups/p-sec-auto
/plan
/apply

### Comment

Import two existing resource groups before update and apply the plan.
```

## Usage

<!-- start usage -->
```yaml
name: Terraform
on:
  pull_request:
    types: [opened, synchronize]
    branches: [main]
    paths:
      - '**.tf'
      - '**.tfvars'
  issue_comment:
    types: [created]

jobs:
  terraform:
    name: 🔧 Terraform
    uses: innofactororg/github-workflow-terraform-ops/.github/workflows/bootstrap.yml@v1
    secrets:
      # The client secret of the service principal used for azure login.
      #
      # Required
      AZURE_AD_CLIENT_SECRET: ${{ secrets.AZURE_AD_CLIENT_SECRET }}

      # The infrastructure cost API key for calculating cost impact of the
      # terraform plan.
      # Retrieve the API key from <https://dashboard.infracost.io/>
      # under **Org Settings**.
      #
      # Required
      INFRACOST_API_KEY: ${{ secrets.INFRACOST_API_KEY }}

      # A private key from the GitHub App specified in library_01_app_id.
      # Optional
      LIBRARY_01_PRIVATE_KEY: ${{ secrets.GHW_TERRAFORM_PRIVATE_KEY }}

      # A private key from the GitHub App specified in library_02_app_id.
      #
      # Optional
      LIBRARY_02_PRIVATE_KEY: ${{ secrets.GHW_TERRAFORM_PRIVATE_KEY }}

      # A private key from the GitHub App specified in library_03_app_id.
      #
      # Optional
      LIBRARY_03_PRIVATE_KEY: ${{ secrets.GHW_TERRAFORM_PRIVATE_KEY }}

      # A private key from the GitHub App specified in library_04_app_id.
      #
      # Optional
      LIBRARY_04_PRIVATE_KEY: ${{ secrets.GHW_TERRAFORM_PRIVATE_KEY }}

      # A private key from the GitHub App specified in library_05_app_id.
      #
      # Optional
      LIBRARY_05_PRIVATE_KEY: ${{ secrets.GHW_TERRAFORM_PRIVATE_KEY }}
    with:
      # The terraform environment name.
      #
      # Required
      environment: hack4

      # The level in the CAF hierarchy where to deploy the resources.
      # Can be one of:
      #
      # level0 - Core platform automation.
      # level1 - Core platform governance.
      # level2 - Core platform connectivity.
      # level3 - Workload components delegated to platform operations teams.
      # level4 - Workload components delegated to application teams.
      #
      # Default: level4
      level: level4

      # The tenant ID in which the subscription exists.
      #
      # Required
      azure_tenant_id: d0d0d0d0-b93b-4f96-9e73-4ea6caa2f3b4

      # The client ID of the service principal used by terraform.
      #
      # This service principal must have permission to write to
      # the terraform state storage account.
      #
      # Required
      azure_ad_client_id: d0d0d0d0-4558-43bb-896a-008e763058bd

      # The blob name for the terraform state file.
      # It must be tfstate or end with .tfstate.
      #
      # Default: tfstate
      state_name: tfstate

      # The storage account container name for the terraform state.
      #
      # Default: tfstate
      state_container: sentinel

      # The Azure location for the required terraform state resources.
      #
      # Default: westeurope
      state_location: westeurope

      # The subscription of the terraform state management resources.
      #
      # Required
      state_subscription_id: d0d0d0d0-ed29-4694-ac26-2e358c364506

      # The subscription ID in which to deploy the resources.
      #
      # Required
      target_subscription_id: d0d0d0d0-ed29-4694-ac26-2e358c364506

      # The App ID of a GitHub App with content read access to the
      # repository specified in input variable library_01_repo.
      #
      # It is used, along with LIBRARY_01_PRIVATE_KEY, to create a token
      # that will be used to get the content of library_01_repo.
      #
      # Default: ''
      library_01_app_id: ${{ vars.GHW_TERRAFORM_APP_ID }}

      # The name of a repository with content needed by terraform.
      #
      # If specified, the repository will be checked out, before running
      # terraform, so it can be available for referencing in the
      # terraform source code or configuration files.
      #
      # Default: ''
      library_01_repo: innofactororg/azure-csoc-analytics-rules

      # The path where the repository, specified in library_01_repo
      # will be checked out to.
      #
      # The repository is checked out to src. If the library files
      # should be in a sub folder of the code (`.tf`) files, this path
      # must start with `src`.
      #
      # Default: ''
      library_01_path: src/csoc/analytic

      # The workflow can have up to 5 libraries (see description above).
      # For example, a second library:
      library_02_app_id: ${{ vars.GHW_TERRAFORM_APP_ID }}
      library_02_repo: innofactororg/azure-csoc-automation-rules
      library_02_path: src/csoc/automation

      # Path to terraform code (`.tf` files) to be executed. To specify
      # the repository root folder use a dot (.).
      #
      # Default: .
      code_path: .

      # Path to the configuration files. All `.tfvars` files in the
      # directory will be expanded.
      #
      # Default: .
      var_path: configuration

      # Require a CODEOWNERS file before allowing terraform apply.
      #
      # The check will fail if the repository don't have a CODEOWNERS file in
      # either the root, docs/, or .github/ directory.
      # About code owners: https://t.ly/8KUb
      #
      # The CODEOWNERS file is retrieved from the base branch of a
      # pull request (e.g. main), so it can be protected.
      #
      # Default: false
      require_codeowners_file_for_apply: false

      # Check that actor is code owner before allowing terraform apply.
      #
      # Without a CODEOWNERS file, everyone is considered a code owner.
      # The check will fail if the actor (the user that initiated this check)
      # is not a code owner.
      #
      # Default: true
      require_code_owner_for_apply: true

      # Check that a code owner has reviewed and approved the pull request.
      # The code owner can't be the user who opened the pull request.
      #
      # Note: This action will ignore emails and teams specified in CODEOWNERS file.
      #
      # Default: true
      require_code_owner_review_for_apply: true

      # Require a CODETEAMS file before allowing terraform apply.
      #
      # The check will fail if the repository don't have a CODETEAMS file in
      # either the root, docs/, or .github/ directory.
      #
      # The CODETEAMS file is retrieved from the base branch of a
      # pull request (e.g. main), so it can be protected.
      #
      # Default: false
      require_codeteams_file_for_apply: false

      # Check that a code team member has reviewed and approved the pull request.
      # The code team member can't be the user who opened the pull request.
      #
      # For the check to run, the repository must have a CODETEAMS file in
      # either the root, docs/, or .github/ directory of the repository.
      #
      # Default: true
      require_code_team_review_for_apply: true

      # Check that at least one approved review exist for the pull request.
      # The reviewer can't be the user who opened the pull request.
      #
      # Default: true
      require_approved_review: true

      # Check that the pull request mergable state is in one of the specified states.
      #
      # The value is a JSON-stringified list of one or more states:
      # clean
      # has_hooks
      # unstable
      # behind
      # blocked
      # dirty
      # draft
      #
      # Default: ["clean","has_hooks","unstable"]
      required_mergeable_state_for_apply: |-
        ["clean","has_hooks","unstable"]

      # The merge method to use after successful terraform apply.
      # Can be one of:
      #
      # merge - retain all of the commits.
      # squash - retain the changes but omit the individual commits.
      # rebase - move the commits to the tip of the target branch.
      # disable - turn off auto merge.
      #
      # Default: squash
      merge_method: squash

      # Prevent deleting the branch after merge.
      #
      # Default: false
      keep_branch_after_merge: false

      # The format to use for date and time in comments.
      # Corresponds to the locales parameter of the Intl.DateTimeFormat()
      # constructor. The default format, sv-SE, is yyyy-MM-dd HH:mm:ss.
      #
      # Default: sv-SE
      date_time_language_format: sv-SE

      # The time zone to use for time in comments. It is used to convert
      # from UTC time. Official list of time zones:
      # https://www.iana.org/time-zones.
      #
      # Default: Europe/Oslo
      time_zone: Europe/Oslo

      # The log verbosity. This affect terraform log output (TF_LOG) and
      # what information is generated in the workflow job log.
      # Can be one of:
      #
      # ERROR - dump github context etc. if workflow fail.
      # WARN - dump github context etc. if workflow fail.
      # INFO - always dump github context etc.
      # DEBUG - always dump github context etc.
      # TRACE - always dump github context etc.
      #
      # Default: ERROR
      log_severity: INFO
```
<!-- end usage -->

## Create GitHub App

This workflow require a GitHub App to handle some security checks and to
optionally get the content of up to five repositories.

1. Navigate to the organization account **Settings**.
1. In the left sidebar, click **Developer settings** and then **GitHub Apps**.
1. Click **New GitHub App**.
1. Fill inn the required information.
   * The home page URL can be set to the GitHub organization URL:
   * For example `https://github.com/organizations/innofactororg`.
1. Uncheck the **Active** box under **Webhook**.
1. Under **Permissions**:
   * Expand **Repository permissions** and select **Read-only** for **Administration** and **Contents**.
   * Expand **Organization permissions** and select **Read-only** for **Members**.
1. Click **Create GitHub App**.
1. Make a note of the App ID for use in the workflow, for example as value for **library_01_app_id**.
1. Under **Private keys**, click **Generate a private key**.
   * This will download a **.pem** file with the RSA private key.
   * Keep the **.pem** file secure.
1. In the left sidebar, click **Install App**.
1. Select the organization where it is needed.
1. Choose **All repositories**.
   * If choosing **Only select repositories**, include the library repositories and all repositories that use this workflow.
1. Create a secret at organization level:
   * Use the content of the **.pem** file to set the value of the secret.
1. Use the secret in the workflow, e.g. as value for **LIBRARY_01_PRIVATE_KEY** for example.
1. To add secrets and variables to use with the workflow:
   * Open organization account **Settings**.
   * Click **Secrets and variables** under **Security**.
   * Click **Actions**.
   * Add secret:
     * Secret name: **GHW_TERRAFORM_PRIVATE_KEY**, or use the name you prefer.
     * Secret value: Use the content of the **.pem** file.
   * Add secret:
     * Secret name: **INFRACOST_API_KEY**, or use the name you prefer.
     * Secret value: Use the **API key** from <https://dashboard.infracost.io/> under **Org Settings**.
   * Click **Variables** and add variable:
     * **GHW_TERRAFORM_APP_ID**.
   * Add repository secret:
     * Open the repository **Settings**.
     * Click **Secrets and variables** under **Security**.
     * Click **Actions**.
     * Add secret:
       * Secret name: **AZURE_AD_CLIENT_SECRET**, or use the name you prefer.
       * Secret value: Use the secret from the App registration in Azure Active Directory.

When using more libraries, apply the above for library_02_app_id,
library_02_repo, LIBRARY_02_PRIVATE_KEY up to LIBRARY_05_PRIVATE_KEY.

## Security scan

The scan command will generate a
[tfsec sarif file](https://github.com/aquasecurity/tfsec-sarif-action)
and upload the file to the job artifacts and to
[GitHub Advanced Security](https://docs.github.com/en/get-started/learning-about-github/about-github-advanced-security).

The job will fail if GitHub Advanced Security is not enabled in
the organization.

## Terraform plan

Generate a terraform plan based on the terraform (.tf) files and
configuration (.tfvars) included in the branch.

This command is automatically triggered by a pull request, including
new commits to an existing pull request.

To trigger it manually, comment on the pull request page and use
the following format:

`/plan [options]`

Example:

`/plan -destroy -target=module.csoc_solution.module.resource_groups["p-sec-mon"].azurerm_resource_group.rg`

### Terraform plan options

* `-destroy` - Creates a plan whose goal is to destroy all remote objects
that currently exist, leaving an empty terraform state.
* `-refresh-only` - Refresh-only mode creates a plan whose goal is only to
update the terraform state and any root module output values to match
changes made to remote objects outside of terraform. This can be useful
if you've intentionally changed one or more remote objects outside of the
usual workflow (e.g. while responding to an incident) and you now need to
reconcile terraform's records with those changes.
* `-refresh=false` - Disable the default behavior of synchronizing the
terraform state with remote objects before checking for configuration
changes. Can be `true` or `false`.
* `-replace=ADDRESS` - Instructs Terraform to plan to replace the resource
instance with the given address.
* `-target=ADDRESS` - Instructs Terraform to focus its planning efforts
only on resource instances which match the given address and on any objects
that those instances depend on.

## Terraform apply

This command executes the actions proposed in a saved terraform
plan (saved plan mode).

If the plan is successfully applied, the pull request will be merged with
it's target branch and the pull request branch will be deleted.
This behavior can be customized with the input parameters merge_method
and keep_branch_after_merge.

To trigger this command, comment on the pull request page and use
the following format:

`/apply`

## Terraform force-unlock

This command removes the lock on the state for the current configuration.
This will not modify the infrastructure.

To trigger this command, comment on the pull request page and use
the following format:

`/force-unlock LOCK_ID`

Example:

`/force-unlock 40e844a6-ebb0-fcfd-da85-5cce5bd1ec89`

## Terraform graph

Generate a visual representation of either a configuration or
execution plan.

The output of terraform graph is in the DOT format, which can be
converted to an image by making use of dot provided by
[GraphViz](https://graphviz.org/download/).

To trigger this command, comment on the pull request page and use
the following format:

`/graph [options]`

Example:

`/graph -type=apply`

### Terraform graph options

* `-type=TYPE` - Type of graph to output. Can be:
`plan`, `plan-destroy`, `apply`, `validate`, `input`, `refresh`.
* `-draw-cycles` - Highlight any cycles in the graph with colored edges.
This helps when diagnosing cycle errors.

The `-type` flag can be used to control the type of graph shown.
Terraform creates different graphs for different operations.
The default type is `plan` if type option is not given. If the `apply`
type is given, the saved plan file is passed as an argument.

## Terraform import

Import will find the existing resource from `ID` and import it into
terraform state at the given `ADDRESS`.

To trigger this command, comment on the pull request page and use
the following format:

`/import ADDRESS ID`

Example:

`/import module.csoc_solution.module.resource_groups["p-sec-mon"].azurerm_resource_group.rg /subscriptions/64499dc8-d437-45d7-8d2e-4910e88e17b2/resourceGroups/p-sec-mon`

## Terraform state list

List all resources in the state file matching the given addresses (if any).
If no addresses are given, all resources are listed.

To trigger this command, comment on the pull request page and use
the following format:

`/state list [options] [ADDRESS...]`

### Terraform state list options

`-id=id` - ID of resources to show.

## Terraform state mv

Terraform will look in the current state for a resource instance,
resource, or module that matches the given address, and if successful
it will move the remote objects currently associated with the source
to be tracked instead by the destination.

To trigger this command, comment on the pull request page and use
the following format:

`/state mv [options] SOURCE DESTINATION`

### Terraform state mv options

`-dry-run` - Report all of the resource instances that match the given
address without actually "moving" any of them.

## Terraform state rm

Terraform will search the state for any instances matching the given
resource address, and remove the record of each one so that Terraform
will no longer be tracking the corresponding remote objects.

To trigger this command, comment on the pull request page and use
the following format:

`/state rm [options] ADDRESS...`

### Terraform state rm options

`-dry-run` - Report all of the resource instances that match the given
address without actually "forgetting" any of them.

## Terraform state show

The command will show the attributes of a single resource in the state
file that matches the given address.

To trigger this command, comment on the pull request page and use
the following format:

`/state show ADDRESS`

## Terraform untaint

Terraform has a marker which it uses to track that an object might be
damaged and so a future terraform plan ought to replace it.

Terraform automatically marks an object as tainted if an error occurs
during a multi-step create action, because Terraform can't be sure that
the object was left in a fully-functional state.

If Terraform currently considers a particular object as tainted but
you've determined that it's actually functioning correctly and need not
be replaced, you can use terraform untaint to remove the taint marker
from that object.

This command will not modify any real remote objects, but will modify
the state in order to remove the tainted status.

To trigger this command, comment on the pull request page and use
the following format:

`/untaint [options] ADDRESS`

The ADDRESS argument is a terraform resource address identifying a
particular resource instance which is currently tainted.

Example:

`/untaint module.csoc_solution.module.resource_groups["p-sec-mon"].azurerm_resource_group.rg`

### Terraform untaint options

* `-allow-missing` - Succeed even if the resource is missing.

## License

The scripts and documentation in this project are released under the [MIT License](LICENSE).
