# Repository Overview

This repository hosts the reusable GitHub Actions workflows for the `eingram-homelab` organization.

It also contains its own repository-local workflow files used for CI in this repository.

## Workflow Naming

Reusable workflows start with an underscore in the filename.

Examples:
- `_validate-workflow.yaml`
- `_terraform-lint.yaml`
- `_sonarqube-scan.yaml`

Workflow files that do not start with an underscore are repository-local entrypoint workflows for this repository's own CI behavior.

Current repository-local CI entrypoints:
- `f-branch-validate.yaml` for advisory validation on feature branch pushes
- `pr-validate.yaml` for merge-blocking validation on pull requests to `main`
- `release-reusable-workflows.yaml` for reusable workflow release tagging on pushes to `main`
- `release-actions.yaml` for action release tagging on pushes to `main`
- `sonarqube-smoke-test.yaml` for manual smoke testing of the reusable SonarQube workflow

Current reusable validation workflow:
- `_validate-workflow.yaml` contains the shared validation logic used by the repository-local entrypoints

## Branch Protection And Merge Expectations

This repository requires pull requests for merges to default branches. Do not assume direct pushes to the default branch are acceptable.

## Required CI Pattern

This repository has a required status check that the `gate` job must pass.

Because of that, CI workflows in this repository should end with a final job named `gate`.

The purpose of the `gate` job is to publish the final success or failure state after the real validation jobs complete. When updating or adding CI workflows, preserve this pattern so branch protection continues to work.

## Expected CI Behavior

When adding or modifying CI in this repository, preserve these expected behaviors.

### Feature Branch Validation

There should be a workflow that runs on pushes to feature branches and performs validation checks for workflow changes.

That workflow is implemented in `f-branch-validate.yaml`.

This is an advisory or soft check only. The workflow should still include a final `gate` job, but that `gate` job must always succeed even if validation fails.

### Pull Request Validation

There should be a workflow that runs on any pull request targeting `main` and performs the same validation checks.

That workflow is implemented in `pr-validate.yaml`.

This workflow is merge-blocking. If validation fails, the workflow's final `gate` job must fail so the required status check prevents merge.

### Main Branch Release Tagging

There should be separate workflows that run on pushes to `main` for reusable workflows and for actions.

These workflows are implemented in `release-reusable-workflows.yaml` and `release-actions.yaml`.

These workflows should determine semantic version bumps from commit message structure and create the appropriate tags for each changed reusable workflow or action.

Use commit message structure for version bumping with these rules:
- commits with `BREAKING CHANGE:` or `!:` imply a major bump
- commits starting with `feat:` or `feat(...):` imply a minor bump
- all other changes default to a patch bump

Workflow tags should be created per reusable workflow using the workflow filename as the prefix, with the leading underscore removed and the extension stripped.

Example:
- `_validate-workflow.yaml` -> `validate-workflow-v1.2.3`

Reusable workflow release tagging applies only to files in `.github/workflows/` whose filenames start with `_`.

Action release tagging applies to components under `.github/actions/`.

The shared reusable validation logic for these entrypoints lives in `_validate-workflow.yaml`.

The reusable SonarQube workflow lives in `_sonarqube-scan.yaml`. Use `sonarqube-smoke-test.yaml` as the fast manual validation path when changing that reusable workflow.

## Guidance For Agents

When implementing CI changes in this repository:

- Keep repository-local entrypoint workflows separate from reusable workflows.
- Preserve the final `gate` job pattern in all CI workflows.
- Treat feature branch validation as non-blocking.
- Treat pull request validation to `main` as blocking.
- Keep release tagging split by component type: reusable workflows in `release-reusable-workflows.yaml` and actions in `release-actions.yaml`.
