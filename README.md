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
- It is stored as a YAML file inside `.github/workflows/`.
- It defines:
  - the event that triggers the workflow
  - the jobs that should run
  - optional environment variables, permissions, and defaults
- A repository can have multiple workflows.
- Each workflow should usually represent one automation purpose, such as:
  - testing
  - deployment
  - release automation

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

## Workflow Structure

- A workflow usually contains these main parts:
  - `name`: the workflow name shown in GitHub
  - `on`: the event or events that trigger the workflow
  - `jobs`: the jobs that should run

### Basic Example

```yaml
name: CI

on: [push, pull_request]

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - run: echo "Run tests here"
```

## Events

- Events are actions that happen on GitHub and can trigger a workflow.
- They are defined in the `on` key.
- A workflow can listen to one event or many events.

### Common Events

#### `push`

- Runs when commits are pushed to a branch or tag.
- Commonly used for:
  - CI checks
  - builds
  - deployments from specific branches

#### `pull_request`

- Runs when pull request activity happens, such as opening, updating, or reopening a PR.
- Commonly used for:
  - testing code before merge
  - review checks
  - policy enforcement

#### `workflow_dispatch`

- Allows a workflow to be triggered manually from the GitHub UI.
- Useful for:
  - manual deployments
  - rerunning special jobs
  - workflows that need user input

#### `schedule`

- Runs on a schedule using cron syntax.
- Useful for:
  - nightly tests
  - cleanup tasks
  - recurring reports

#### `workflow_call`

- Lets one workflow be reused by another workflow.
- Useful for sharing common automation logic across workflows.

### Event Filters

- Events can be filtered so workflows only run in specific cases.
- Common filters include:
  - `branches`
  - `branches-ignore`
  - `tags`
  - `paths`
  - `paths-ignore`

### Filter Example

```yaml
on:
  push:
    branches:
      - main
    paths:
      - "src/**"
  pull_request:
    branches:
      - main
```

### Notes About Events

- `push` is usually for branch or tag updates.
- `pull_request` is usually for validating incoming changes before merge.
- `workflow_dispatch` is the easiest way to run a workflow manually.
- Choosing the right event helps avoid unnecessary workflow runs.

## Events Deep Dive

### How The `on` Key Works

- The `on` key tells GitHub Actions when a workflow should start.
- It can be written in different forms:
  - a single event
  - a list of events
  - an object with filters

#### Single Event

```yaml
on: push
```

#### Multiple Events

```yaml
on: [push, pull_request]
```

#### Event With Filters

```yaml
on:
  push:
    branches:
      - main
```

### `push` Deep Dive

- The `push` event runs when commits are pushed to a branch or tag.
- It is best for:
  - branch-based CI
  - deployments from specific branches
  - tag-based release flows

#### Example

```yaml
on:
  push:
    branches:
      - main
      - develop
```

#### Notes

- If you push to `main`, the workflow runs only if `main` matches the filter.
- `push` does not mean "a pull request was created". It only reacts to pushed commits.

### `pull_request` Deep Dive

- The `pull_request` event runs when activity happens on a pull request.
- Common activities include:
  - opening a PR
  - synchronizing new commits to a PR
  - reopening a PR
- This event is best for validating code before merge.

#### Example

```yaml
on:
  pull_request:
    branches:
      - main
```

#### Notes

- In `pull_request`, the `branches` filter refers to the target branch of the PR, not the source branch.
- This is important because a PR from `feature/login` into `main` matches `branches: [main]`.

### `workflow_dispatch` Deep Dive

- `workflow_dispatch` allows manual workflow runs from the GitHub Actions tab.
- It is useful when automation should not run on every code change.
- It can also accept inputs.

#### Example

```yaml
on:
  workflow_dispatch:
    inputs:
      environment:
        description: "Where to deploy"
        required: true
```

#### Notes

- This is commonly used for manual deployments or admin tasks.
- Inputs can be accessed in the workflow through contexts.

### `schedule` Deep Dive

- `schedule` runs workflows automatically based on cron syntax.
- It is useful for repeated background automation.

#### Example

```yaml
on:
  schedule:
    - cron: "0 0 * * *"
```

#### Notes

- Cron is written in UTC.
- This event is useful for nightly builds, dependency checks, and cleanup jobs.

### `workflow_call` Deep Dive

- `workflow_call` makes a workflow reusable from another workflow.
- It helps reduce duplication when many workflows share the same logic.

#### Example

```yaml
on:
  workflow_call:
```

#### Notes

- Reusable workflows are useful for shared CI pipelines across repositories or teams.

### Branch And Path Filters

- Filters help reduce unnecessary runs.
- They make workflows more precise.

#### Branch Filter Example

```yaml
on:
  push:
    branches:
      - main
      - develop
```

#### Path Filter Example

```yaml
on:
  push:
    paths:
      - "src/**"
      - "package.json"
```

#### Important Detail

- If both `branches` and `paths` are used, both conditions must match for the workflow to run.

### Choosing The Right Event

- Use `push` when you want automation after commits are pushed.
- Use `pull_request` when you want to validate changes before merge.
- Use `workflow_dispatch` when the workflow should be started manually.
- Use `schedule` for recurring automation.
- Use `workflow_call` when you want reusable workflow logic.

## Github Actions Contexts

- Contexts are built-in objects that provide information during a workflow run.
- They are accessed with expressions like `${{ github.ref }}`.
- Contexts are useful in:
  - `if` conditions
  - environment variables
  - action inputs
  - job and step settings

### Why Contexts Matter

- Contexts make workflows dynamic.
- They let the workflow behave differently depending on the branch, event, runner, matrix value, or earlier step outputs.

### Common Contexts

#### `github`

- Contains details about the repository, workflow run, and triggering event.
- Examples:
  - `${{ github.repository }}`
  - `${{ github.ref }}`
  - `${{ github.actor }}`
  - `${{ github.event_name }}`

#### `env`

- Contains environment variables defined in the workflow, job, or step.
- Example:
  - `${{ env.APP_NAME }}`

#### `vars`

- Contains configuration variables defined in GitHub settings.
- Example:
  - `${{ vars.NODE_VERSION }}`

#### `secrets`

- Contains sensitive values like tokens and API keys.
- Example:
  - `${{ secrets.MY_TOKEN }}`
- Secrets should be used for sensitive data and are masked in logs.

#### `job`

- Contains information about the current job.
- Example:
  - `${{ job.status }}`

#### `steps`

- Contains outputs and results from previous steps in the same job.
- Example:
  - `${{ steps.build.outputs.version }}`

#### `runner`

- Contains information about the runner machine.
- Example:
  - `${{ runner.os }}`

#### `matrix`

- Contains the current value from a matrix strategy.
- Example:
  - `${{ matrix.node }}`

### Example

```yaml
jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - name: Print branch
        run: echo "${{ github.ref }}"

      - name: Print runner OS
        run: echo "${{ runner.os }}"
```

### Notes

- `${{ ... }}` is the GitHub Actions expression syntax.
- Contexts are evaluated by GitHub Actions before the shell runs the command.
- Step outputs are accessed through the `steps` context.
