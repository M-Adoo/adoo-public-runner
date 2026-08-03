# adoo-public-runner

Public GitHub-hosted runner shell for trusted release scripts stored in private
`M-Adoo/*` repositories.

This repository intentionally contains only the workflow wrapper. Private
source snapshots and build logs remain runner-local. Matrix outputs are retained
as workflow artifacts for one day at most and are deleted with the workflow run
after the private release is verified.

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
`verify` phase, then runs the matrix in a `build` phase. Each build writes files
to `ADOO_RUNNER_OUTPUT`; the workflow uploads those directories as one-day
artifacts. A single `finalize` job downloads and merges the outputs, installs
Nix, checks out the complete private repository history with push credentials,
and runs the same trusted script with `ADOO_RUNNER_PHASE=finalize`.

The private PAT is not exposed to matrix build scripts. `GH_TOKEN` is injected
only for the finalizer that publishes release assets, branches, and tags.
stdout/stderr are redirected to runner-local temporary files, and each job
removes its workspace, outputs, and temporary logs in an `always()` cleanup
step.

The private release script is responsible for deleting the workflow run after it
has verified the published Release and Nix refs.
