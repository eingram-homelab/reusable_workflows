# Repository Overview

This repository hosts the reusable GitHub Actions workflows for the `eingram-homelab` organization.

It also contains its own repository-local workflow files used for CI in this repository.

## Workflow Naming

Reusable workflows start with an underscore in the filename.

Examples:
- `_validate-workflow.yaml`
- `_terraform-lint.yaml`

Workflow files that do not start with an underscore are repository-local entrypoint workflows for this repository's own CI behavior.

## Branch Protection And Merge Expectations

This repository requires pull requests for merges to default branches. Do not assume direct pushes to the default branch are acceptable.

## Required CI Pattern

This repository has a required status check that the `gate` job must pass.

Because of that, CI workflows in this repository should end with a final job named `gate`.

The purpose of the `gate` job is to publish the final success or failure state after the real validation jobs complete. When updating or adding CI workflows, preserve this pattern so branch protection continues to work.
