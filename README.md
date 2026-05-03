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
- A workflow = set of jobs triggered by events.
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
- Common runners `ubuntu-latest`, `windows-latest`, `macos-latest` or self-hosted

### Step

- A step is a single task inside a job.
- A step can:
  - run shell commands using `run`
  - use a reusable action using `uses`
- Steps in the same job run sequentially.
- Steps in the same job can share files through the workspace.
- Steps share workspace but not environment by default unless `env` is set.

### Actions

- Reusable units of logic.
- Types:
  - Docker actions — packaged in containers.
  - JavaScript actions — fastest, native execution.
  - Composite actions — combine multiple shell/steps.
- Use public actions from GitHub Marketplace.

## Basic & Useless Workflow

```yaml
name: First Worflow
on: workflow_dispatch

jobs:
  first-job:
    runs-on: ubunut-latest
    steps:
      - name: Greetings
        runs: echo "Hello, There!"
```

## Reactjs Example

[Full Example Repo With Extended Workflow](https://github.com/moabdelazem/gh-actions-react-app)

```yaml
name: Running Tests
on: [push, pull_request]
jobs:
  run-tests:
    runs-on: ubunut-latest
    steps:
      - name: Checkout the code
        uses: actions/checkout@v5

      - name: Setup Nodejs
        uses: actions/setup-node@v6
        with:
          node-version: 22

      - name: Install Dependencies
        run: npm ci

      - name: Running Tests
        run: npm test
```

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
