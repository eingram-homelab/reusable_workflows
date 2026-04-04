# .github

Default community health files and shared reusable GitHub Actions workflows for the [eingram-homelab](https://github.com/eingram-homelab) organization.

## Reusable Workflows

Reusable workflows live in [`.github/workflows/`](.github/workflows/) and can be called from any repository in the organization using `workflow_call`.

### Usage

Reference a reusable workflow from another repo like this:

```yaml
jobs:
  my-job:
    uses: eingram-homelab/.github/.github/workflows/<workflow-name>.yaml@main
    with:
      input_name: value
    secrets:
      SECRET_NAME: ${{ secrets.SECRET_NAME }}
```

---

### `ansible-lint.yaml`

Runs `ansible-lint` against the checked-out repository.

**Inputs:**

| Input | Required | Default | Description |
|-------|----------|---------|-------------|
| `runner` | No | `ubuntu-latest` | Runner label to use |

**Example caller:**

```yaml
jobs:
  lint:
    uses: eingram-homelab/.github/.github/workflows/ansible-lint.yaml@main
```

---

### `run-ansible-playbook.yaml`

Installs Ansible and its dependencies, then runs a playbook from the `ansible-col-homelab` collection.

**Inputs:**

| Input | Required | Default | Description |
|-------|----------|---------|-------------|
| `playbook` | **Yes** | — | Playbook filename under `playbooks/` |
| `ansible_user` | **Yes** | — | SSH user for Ansible connections |
| `limit` | No | `''` | Ansible limit flag (e.g. `-l host1,host2`) |
| `extra_vars` | No | `''` | Extra vars as `key=value` pairs |
| `runner` | No | `arc-runners` | Runner label to use |

**Secrets:**

| Secret | Required | Description |
|--------|----------|-------------|
| `VAULT_TOKEN` | **Yes** | HashiCorp Vault token |
| `ANSIBLE_SSH_KEY` | **Yes** | Private SSH key for Ansible |

**Example caller:**

```yaml
jobs:
  run-playbook:
    uses: eingram-homelab/.github/.github/workflows/run-ansible-playbook.yaml@main
    with:
      playbook: new_build.yaml
      ansible_user: ansible
      limit: '-l host1.example.com'
    secrets: inherit
```

---

### `terraform-deploy.yaml`

Runs `fmt -check`, `init`, `validate`, and `apply` for a given Terraform project directory.

**Inputs:**

| Input | Required | Default | Description |
|-------|----------|---------|-------------|
| `project_dir` | **Yes** | — | Path to the Terraform project directory |
| `terraform_version` | No | `1.11.3` | Terraform version to install |
| `gcs_bucket` | No | `yc-srv1-tfstate` | GCS bucket for remote state |
| `runner` | No | `arc-runners` | Runner label to use |

**Secrets:**

| Secret | Required | Description |
|--------|----------|-------------|
| `VAULT_TOKEN` | **Yes** | HashiCorp Vault token |
| `GCP_STORAGE` | **Yes** | Base64-encoded GCS service-account credentials JSON |

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
