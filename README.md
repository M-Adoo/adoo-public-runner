# adoo-public-runner

Public GitHub-hosted runner shell for trusted release scripts stored in private
`M-Adoo/*` repositories.

This repository intentionally contains only the workflow wrapper. It does not
store private source snapshots, build outputs, release assets, artifacts, or
build logs.

## Workflow Contract

`trusted-private-script.yml` accepts only `workflow_dispatch` input:

- `repo`: bare private repository name under `M-Adoo`
- `ref`: 40-character commit SHA to check out
- `script`: repository-relative trusted script path
- `matrix`: JSON array of matrix objects
- `input`: JSON payload written to a temporary file
- `request_id`: caller-generated run identifier

The workflow validates the repository and script path, checks out the private
repository with `persist-credentials: false`, runs the trusted script once in a
`verify` phase, then runs the matrix in a `build` phase. `GH_TOKEN` is injected
only for the build phase that uploads release assets. stdout/stderr are
redirected to runner-local temporary files, and the workspace and temporary logs
are removed in an `always()` cleanup step.

The private release script is responsible for deleting the workflow run after it
has verified the private release assets.
