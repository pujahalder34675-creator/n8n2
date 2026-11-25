# Standalone CI audit

The standalone entry point is `.github/workflows/ci-main.yml`.
Legacy upstream workflows stay in the repository for reference and manual use.
Manual runs can still require the upstream runners and credentials below.
No workflow jobs, reusable input schemas, or setup protections were removed.

## Requested workflow checks

| File                                  | Runners                                                                                                                                                                                                                                                                  | Credentials and repository guards                                                                                                                                                                                                                                                                              | Standalone handling                                                                                                                                                                                                                                  |
| ------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `workflows/ci-master.yml`             | `blacksmith-4vcpu-ubuntu-2204`; `ubuntu-latest` for notifications; reusable jobs have their own runners                                                                                                                                                                  | Direct secret references: `CODECOV_TOKEN`, `QA_METRICS_WEBHOOK_PASSWORD`, `QA_METRICS_WEBHOOK_URL`, `QA_METRICS_WEBHOOK_USER`, `QBOT_SLACK_TOKEN`. It also passes inherited secrets to a reusable job. No repository guard protects the build.                                                                 | Manual legacy entry point. The standalone workflow replaces its automatic role.                                                                                                                                                                      |
| `workflows/ci-pull-requests.yml`      | `blacksmith-2vcpu-ubuntu-2204`, `blacksmith-4vcpu-ubuntu-2204`, and `blacksmith-8vcpu-ubuntu-2204` by default; most jobs select `ubuntu-latest` when `vars.RUNNER_PROVIDER == 'github'`. Cleanup still has a fixed Blacksmith runner. The result jobs use `ubuntu-slim`. | Direct secret references: `CODECOV_TOKEN`, `CURRENTS_RECORD_KEY`, `QA_METRICS_WEBHOOK_PASSWORD`, `QA_METRICS_WEBHOOK_URL`, `QA_METRICS_WEBHOOK_USER`, `QBOT_SLACK_TOKEN`. Ten reusable job calls inherit secrets. Docker preparation and Instance AI discovery use `github.repository == 'n8n-io/n8n'` guards. | Manual legacy entry point.                                                                                                                                                                                                                           |
| `workflows/test-linting-reusable.yml` | `blacksmith-4vcpu-ubuntu-2204` by default; `ubuntu-latest` when `vars.RUNNER_PROVIDER == 'github'`                                                                                                                                                                       | No direct secret reference or repository guard. It uses `useblacksmith/checkout` and the shared setup action.                                                                                                                                                                                                  | Kept as a reusable workflow. Set the runner variable before a manual legacy caller uses it.                                                                                                                                                          |
| `workflows/docker-build-smoke.yml`    | `ubuntu-slim` for the gate and notification; `blacksmith-4vcpu-ubuntu-2204` for amd64; `blacksmith-8vcpu-ubuntu-2204-arm` for arm64                                                                                                                                      | `DOCKER_USERNAME` and `DOCKER_PASSWORD` for DHI; `N8N_ASSISTANT_APP_ID` and `N8N_ASSISTANT_PRIVATE_KEY` to read the private `n8n-io/n8n-cloud` chart; `QBOT_SLACK_TOKEN` for failure messages. Fork PRs skip the build. No standalone repository guard covers the private chart step.                          | Manual only. Use the local Docker build for standalone work.                                                                                                                                                                                         |
| `workflows/build-windows.yml`         | `windows-latest`                                                                                                                                                                                                                                                         | No repository guard. `QBOT_SLACK_TOKEN` is optional and is used only when `notify_on_failure` is true.                                                                                                                                                                                                         | Manual and reusable triggers stay available.                                                                                                                                                                                                         |
| `actions/setup-nodejs/action.yml`     | Composite action; it uses the caller's runner                                                                                                                                                                                                                            | No upstream secret requirement or repository guard. It uses the built-in `github.token` during dependency installation. Blacksmith Docker setup runs only on Blacksmith runners. GitHub runners use the Buildx fallback when Docker caching is requested.                                                      | Unchanged. The action keeps SafeChain activation and checks, installer checksum verification, frozen-lockfile installation, exact Node/pnpm checks, and GitHub cache support. Its default Node pin is `26.13.0`; pnpm is read from `packageManager`. |

## Automatic validation kept

The Python workflows `ci-python.yml` and `test-evals-python.yml` keep their path-filtered push and PR triggers. They run on `ubuntu-latest`. Their optional Codecov upload skips when `CODECOV_TOKEN` is absent. Upload errors remain nonfatal.

The secret-free checks `ci-docs-backend.yml`, `ci-owners-validation.yml`, and `ci-terraform.yml` keep their existing validation triggers. The backend documentation check keeps its useful weekly validation. `ci-check-pr-title.yml` also stays unchanged; its existing branch filters cover `master` and `1.x`, not `main`.

The standalone workflow defines the checks for `main`. Reusable workflows keep every existing `workflow_call` input, output, and secret declaration.

## Standalone checks

`ci-main.yml` runs on pushes to `main`, pull requests to `main`, and manual dispatch.
It uses `ubuntu-latest`, Node `24.22.2`, and the root `pnpm@12.4.2` pin.
The shared setup action keeps its existing supply-chain checks and needs no organization secrets.
The three check jobs run the real workspace build, typecheck, lint and styles, backend unit tests, and frontend unit tests.
Turbo runs one task at a time. The default Node heap limit is 4096 MiB.
The frontend package sets its own 8192 MiB heap limit for build and typecheck.
Small private-repository runners can need more memory for these operations.
Optional coverage uploads, Slack messages, QA metrics, and registry pushes are not part of this workflow.

Every core CI run also parses all workflow YAML, lints both standalone workflows with a checksum-verified actionlint binary, validates Compose, and checks the local Dockerfile with BuildKit.

`ci-docker-local.yml` runs on manual dispatch and relevant Docker/build-input changes on `main` or its pull requests.
It builds the local image without a registry login or image push.
It checks application readiness, public settings, editor HTML, and native modules.
It saves logs and removes the test containers and volumes after the run.
The full source build can take time and needs adequate memory and disk space.

## Upstream automation made manual

The following 56 workflows now have `workflow_dispatch` instead of their automatic organization triggers. Existing dispatch inputs stay intact. `workflow_call` stays intact where present. No workflows were deleted.

### Legacy build and CI entry points

- `build-windows.yml`
- `ci-master.yml`
- `ci-pull-requests.yml`

### Image and release automation

- `build-base-image.yml`
- `build-benchmark-image.yml`
- `build-node-pc-image.yml`
- `build-v3-nightly.yml`
- `docker-build-push.yml`
- `docker-build-smoke.yml`
- `release-docs-backend.yml`
- `release-publish.yml`
- `release-storybook.yml`

### Private security sync and publishing

- `sec-publish-fix-1x.yml`
- `sec-publish-fix.yml`
- `sec-sync-bundle-branches.yml`
- `sec-sync-public-to-private.yml`

### Hosted integration tests and evaluation runs

- `encryption-upgrade-test.yml`
- `test-benchmark-destroy-nightly.yml`
- `test-benchmark-nightly.yml`
- `test-e2e-coverage-nightly.yml`
- `test-e2e-helm.yml`
- `test-e2e-infrastructure-reusable.yml`
- `test-e2e-pc-nightly.yml`
- `test-e2e-performance-reusable.yml`
- `test-evals-ai-release.yml`
- `test-evals-ai.yml`
- `test-get-n8n.yml`
- `test-image-efficiency.yml`
- `test-link-check-weekly.yml`
- `test-sbom-nightly.yml`

### Organization bots and maintenance

- `backport.yml`
- `ci-check-release-from-fork.yml`
- `ci-cla-check.yml`
- `ci-detect-new-packages.yml`
- `ci-instance-ai-evals.yml`
- `ci-owners-assign-reviewers.yml`
- `ci-owners-required-reviews.yml`
- `ci-pr-quality.yml`
- `ci-restrict-private-merges.yml`
- `clean-stale-branches.yml`
- `community-pr-close-stale.yml`
- `util-backport-bundle.yml`
- `util-cleanup-abandoned-release-branches.yml`
- `util-cleanup-pr-images.yml`
- `util-codespace-preview.yml`
- `util-hide-outdated-cubic-comments.yml`
- `util-notify-merge-queue-removal.yml`
- `util-notify-pr-status.yml`
- `util-probe-registry.yml`
- `util-refresh-cubic-schema.yml`
- `util-refresh-starter-scaffold.yml`
- `util-sync-master-to-3x.yml`
- `util-update-boundaries-baseline.yml`
- `util-update-credential-setupability.yml`
- `util-update-get-n8n-versions.yml`
- `util-update-node-popularity.yml`

These changes prevent the standalone repository from scheduling upstream image publishing, private security synchronization, hosted nightly tests, app-token bots, branch cleanup, and release maintenance. They also prevent upstream pull-request automation from waiting for unavailable private runners or credentials.

## Validation

All workflow files parse as YAML. For each edited workflow, the parsed content outside `on` is unchanged. Existing manual and reusable trigger schemas are unchanged. `git diff --check` passes. No `workflow_run` event remains, so the standalone CI cannot start an upstream automation chain after completion.

GitHub-hosted execution must still be checked in Actions after push. This audit does not claim that a manual upstream release or private-infrastructure workflow works without its original credentials.
