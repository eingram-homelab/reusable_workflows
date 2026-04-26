# reusable-workflows

Shared reusable GitHub Actions workflows for the [eingram-homelab](https://github.com/eingram-homelab) organization.

## Reusable Workflows

Reusable workflows live in [`.github/workflows/`](.github/workflows/) and are prefixed with `_` to distinguish them from workflows that run directly in this repository. They can be called from any repository in the organization using `workflow_call`.

### Usage

Reference a reusable workflow from another repo like this:

```yaml
jobs:
  my-job:
    uses: eingram-homelab/reusable-workflows/.github/workflows/<workflow-name>.yaml@main
    with:
      input_name: value
    secrets:
      SECRET_NAME: ${{ secrets.SECRET_NAME }}
```

---

## Ansible

### `_ansible-lint.yaml`

Runs `ansible-lint` against the checked-out repository.

**Inputs:**

| Input | Required | Default | Description |
|-------|----------|---------|-------------|
| `repository` | No | calling repo | Repository containing Ansible playbooks and roles |
| `ref` | No | PR head branch | Branch or ref to check out |

---

### `_ansible-run-playbook.yaml`

Runs a playbook from the `ansible-col-homelab` collection.

**Inputs:**

| Input | Required | Default | Description |
|-------|----------|---------|-------------|
| `playbook` | **Yes** | — | Playbook to run |
| `ansible_user` | **Yes** | — | Ansible user for the playbook run |
| `limit` | **Yes** | — | Ansible limit for the playbook run |
| `extra_vars` | No | `''` | Extra variables for the playbook run |

**Secrets:**

| Secret | Required | Description |
|--------|----------|-------------|
| `VAULT_TOKEN` | **Yes** | HashiCorp Vault token |

---

### `_build-ansible-ee.yaml`

Builds and pushes an Ansible Execution Environment image to a container registry.

**Inputs:**

| Input | Required | Default | Description |
|-------|----------|---------|-------------|
| `ee_dir` | **Yes** | — | Path to the EE definition directory |
| `ee_name` | **Yes** | — | Name for the EE image |
| `ee_version` | **Yes** | — | Version tag for the EE image |
| `registry` | No | `gitea.local.lan` | Container registry to push to |
| `registry_user` | No | `eingram` | Username for the container registry |
| `runner` | No | `self-hosted` | Runner to use |

**Secrets:**

| Secret | Required | Description |
|--------|----------|-------------|
| `REGISTRY_PASSWORD` | **Yes** | Password for the container registry |

---

## Packer

### `_packer-validate.yaml`

Runs `packer fmt -check`, `packer init`, and `packer validate` against all changed `.pkr.hcl` files.

**Inputs:**

| Input | Required | Default | Description |
|-------|----------|---------|-------------|
| `repository` | No | calling repo | Repository containing Packer templates |
| `ref` | No | PR head branch | Branch or ref to check out |

**Secrets:**

| Secret | Required | Description |
|--------|----------|-------------|
| `VAULT_TOKEN` | **Yes** | Vault token for resolving `vault()` locals |

---

### `_packer-build.yaml`

Runs `packer init` and `packer build` against all changed `.pkr.hcl` files, then calls `_vsphere-verify-template.yaml` and `_packer-publish-template.yaml`.

**Inputs:**

| Input | Required | Default | Description |
|-------|----------|---------|-------------|
| `repository` | No | calling repo | Repository containing Packer templates |
| `ref` | No | PR head branch | Branch or ref to check out |

**Secrets:**

| Secret | Required | Description |
|--------|----------|-------------|
| `VAULT_TOKEN` | **Yes** | Vault token for build secrets |
| `GCP_STORAGE` | **Yes** | Base64-encoded GCP service account key for Terraform template verification |

**Outputs:**

| Output | Description |
|--------|-------------|
| `TMPL_NAME` | Timestamped vSphere template name from the build manifest |

---

### `_packer-publish-template.yaml`

Deletes the previous vCenter template and renames the newly built timestamped template to the canonical name using `scripts/publish_template.ps1`.

**Inputs:**

| Input | Required | Default | Description |
|-------|----------|---------|-------------|
| `TMPL_NAME` | **Yes** | — | Timestamped template name from packer build |
| `repository` | No | `eingram-homelab/pkr-vsphere` | Repository containing `publish_template.ps1` |
| `ref` | No | `main` | Branch or ref to check out |

**Secrets:**

| Secret | Required | Description |
|--------|----------|-------------|
| `VAULT_TOKEN` | **Yes** | Vault token for vSphere credentials |

---

### `_vsphere-verify-template.yaml`

Spins up a test VM from the newly built template using Terraform to verify the template is functional.

**Inputs:**

| Input | Required | Default | Description |
|-------|----------|---------|-------------|
| `TMPL_NAME` | **Yes** | — | vSphere template name to verify |

**Secrets:**

| Secret | Required | Description |
|--------|----------|-------------|
| `VAULT_TOKEN` | **Yes** | Vault token for vSphere credentials |
| `GCP_STORAGE` | **Yes** | Base64-encoded GCP service account key for Terraform state |

---

## Terraform

### `_terraform-lint.yaml`

Runs `tflint` against all changed Terraform directories.

**Inputs:**

| Input | Required | Default | Description |
|-------|----------|---------|-------------|
| `repository` | No | calling repo | Repository containing Terraform code |
| `ref` | No | PR head branch | Branch or ref to check out |

---

### `_terraform-validate.yaml`

Runs `terraform init` and `terraform validate` against all changed Terraform directories.

**Inputs:**

| Input | Required | Default | Description |
|-------|----------|---------|-------------|
| `repository` | No | calling repo | Repository containing Terraform code |
| `ref` | No | PR head branch | Branch or ref to check out |

**Secrets:** `VAULT_TOKEN`, `GCP_STORAGE` (inherited from calling workflow)

---

### `_terraform-deploy.yaml`

Runs `terraform fmt -check`, `init`, `validate`, and `apply` for a given project directory.

**Inputs:**

| Input | Required | Default | Description |
|-------|----------|---------|-------------|
| `project_dir` | **Yes** | — | Path to the Terraform project directory |
| `terraform_version` | No | `1.11.3` | Terraform version to use |
| `gcs_bucket` | No | `yc-srv1-tfstate` | GCS bucket for remote state |
| `runner` | No | `arc-runners` | Runner label to use |

**Secrets:**

| Secret | Required | Description |
|--------|----------|-------------|
| `VAULT_TOKEN` | **Yes** | HashiCorp Vault token |
| `GCP_STORAGE` | **Yes** | Base64-encoded GCS service account credentials JSON |

---

### `_terraform-deploy-project.yaml`

Deploys a Terraform project and emits outputs (project name, type, Ansible limit/user, cluster info) for use by downstream jobs.

**Inputs:**

| Input | Required | Default | Description |
|-------|----------|---------|-------------|
| `repository` | No | calling repo | Repository containing Terraform code |
| `ref` | No | PR head branch | Branch or ref to check out |

**Secrets:** `VAULT_TOKEN`, `GCP_STORAGE` (inherited from calling workflow)

---

## Workflows

### `_validate-workflow.yaml`

Runs `yamllint` against workflow files and validates that all `workflow_call` inputs and secrets have descriptions and types.

**Inputs:**

| Input | Required | Default | Description |
|-------|----------|---------|-------------|
| `paths` | No | `.github/workflows/ workflow-templates/` | Space-separated paths to lint |
| `changed_files_patterns` | No | `**/.github/workflows/**, **/workflow-templates/**` | Glob patterns for changed-files detection |

---

## Workflow Templates

Starter workflow templates live in [`workflow-templates/`](workflow-templates/). They appear in the GitHub Actions "New workflow" UI for all repositories in the organization, under the `eingram-homelab` section.

| Template | Description |
|----------|-------------|
| `ansible-lint.yaml` | Lint Ansible code on push |
| `build-ansible-ee.yaml` | Build and push an Ansible EE image |
| `terraform-deploy.yaml` | Deploy a Terraform project on push to main |

**Example caller:**

```yaml
jobs:
  deploy:
    uses: eingram-homelab/.github/.github/workflows/terraform-deploy.yaml@main
    with:
      project_dir: projects/vsphere/my-project/
    secrets:
      VAULT_TOKEN: ${{ secrets.VAULT_TOKEN }}
      GCP_STORAGE: ${{ secrets.GCP_STORAGE }}
```

---

### `vsphere-tmpl-verify.yaml`

Deploys a test VM from a freshly built Packer template, verifies network connectivity, then destroys the VM.

**Inputs:**

| Input | Required | Default | Description |
|-------|----------|---------|-------------|
| `TMPL_NAME` | **Yes** | — | vSphere template name to verify |
| `terraform_version` | No | `1.11.3` | Terraform version to install |
| `runner` | No | `arc-runners` | Runner label to use |

**Secrets:**

| Secret | Required | Description |
|--------|----------|-------------|
| `VAULT_TOKEN` | **Yes** | HashiCorp Vault token |
| `GCP_STORAGE` | **Yes** | Base64-encoded GCS service-account credentials JSON |

**Example caller:**

```yaml
jobs:
  verify:
    uses: eingram-homelab/.github/.github/workflows/vsphere-tmpl-verify.yaml@main
    with:
      TMPL_NAME: ubuntu-22-04__20240101
    secrets:
      VAULT_TOKEN: ${{ secrets.VAULT_TOKEN }}
      GCP_STORAGE: ${{ secrets.GCP_STORAGE }}
```

---

### `build-ansible-ee.yaml`

Builds an Ansible Execution Environment image with `ansible-builder` and pushes it to the org's container registry.

**Inputs:**

| Input | Required | Default | Description |
|-------|----------|---------|-------------|
| `ee_dir` | **Yes** | — | Path to the EE definition directory |
| `ee_name` | **Yes** | — | Image name for the EE |
| `ee_version` | **Yes** | — | Image version tag |
| `registry` | No | `gitea.local.lan` | Container registry hostname |
| `registry_user` | No | `eingram` | Registry username |
| `runner` | No | `self-hosted` | Runner label to use |

**Secrets:**

| Secret | Required | Description |
|--------|----------|-------------|
| `REGISTRY_PASSWORD` | **Yes** | Password for the container registry |

**Example caller:**

```yaml
jobs:
  build-ee:
    uses: eingram-homelab/.github/.github/workflows/build-ansible-ee.yaml@main
    with:
      ee_dir: execution-environments/amd64/ee-homelab-default
      ee_name: ee-homelab-default
      ee_version: '1.2.0'
    secrets:
      REGISTRY_PASSWORD: ${{ secrets.GITEA_PASSWORD }}
```

---

### `create-release.yaml`

Creates a Git tag and a GitHub Release when a PR whose title contains `Release-vX.Y.Z` is merged.

**Inputs:**

| Input | Required | Default | Description |
|-------|----------|---------|-------------|
| `runner` | No | `ubuntu-latest` | Runner label to use |

**Example caller:**

```yaml
on:
  pull_request:
    types: [closed]

jobs:
  release:
    uses: eingram-homelab/.github/.github/workflows/create-release.yaml@main
```

---

## Workflow Templates

Starter workflow templates live in [`workflow-templates/`](workflow-templates/) and appear in the **Actions** tab of every repository in the organization under **"By eingram-homelab"**.

| Template | Description |
|----------|-------------|
| `ansible-lint` | Run Ansible lint on PRs to main |
| `terraform-deploy` | Deploy a Terraform project on push to main |
| `build-ansible-ee` | Build and push an Ansible Execution Environment |
