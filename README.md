# Github Actions

- GitHub Actions is a CI/CD platform integrated into GitHub.
- Workflows are defined in YAML under `.github/workflows/`.
- Runners execute jobs, and they can be GitHub-hosted or self-hosted.

## CI/CD

### CI

- CI stands for `Continuous Integration`.
- It is about automatically checking code changes whenever developers push commits or open pull requests.
- Common CI tasks:
  - running tests
  - linting code
  - checking formatting
  - building the project
- The goal is to catch issues early before code is merged.

### CD

- CD can mean `Continuous Delivery` or `Continuous Deployment`.
- It focuses on automating what happens after the code passes CI.
- Common CD tasks:
  - packaging the app
  - publishing artifacts
  - deploying to staging
  - deploying to production
- `Continuous Delivery` means the software is always ready to be deployed, but release may still need manual approval.
- `Continuous Deployment` means changes are deployed automatically once they pass all checks.

## Github Actions Building Blocks

- GitHub Actions is mainly built from `Workflows`, `Jobs`, and `Steps`.

### Workflow

- A workflow is the top-level automation file.
- It defines when the automation should run and what jobs it should execute.
- Workflows are triggered by `events` such as:
  - `push`
  - `pull_request`
  - `workflow_dispatch`

### Job

- A job is a group of steps that run on the same runner.
- Jobs run in parallel by default unless dependencies are defined with `needs`.
- Each job usually specifies:
  - a runner like `ubuntu-latest`
  - a list of steps

### Step

- A step is a single task inside a job.
- A step can:
  - run shell commands using `run`
  - use a reusable action using `uses`
- Steps in the same job run sequentially.
- Steps in the same job can share files through the workspace.
