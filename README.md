# shared-github-actions

Repository serving shared GitHub Actions and Workflows for all projects of the organization.

---

## Actions

### Deploy Storybook to GitHub Pages

**Path:** `scheiber-sa/shared-github-actions/deploy-storybook@main`

Builds and deploys Storybook to GitHub Pages.

#### Inputs

| Name              | Description                                          | Required | Default               |
| ----------------- | ---------------------------------------------------- | -------- | --------------------- |
| `checkout`        | Specifies if this action should checkout the code    | false    | `true`                |
| `path`            | Specifies the path of the static assets after build  | false    | `dist/storybook`      |
| `install_command` | Specifies the command to run the installation        | false    | `npm ci`              |
| `build_command`   | Specifies the command to run for the build           | false    | `npm run build-storybook` |

#### Outputs

| Name       | Description          |
| ---------- | -------------------- |
| `page_url` | The URL of the page  |

#### Usage

```yaml
jobs:
  deploy:
    runs-on: ubuntu-latest
    permissions:
      pages: write
      id-token: write
    steps:
      - name: Deploy Storybook
        uses: scheiber-sa/shared-github-actions/deploy-storybook@main
        with:
          path: dist/storybook
```

### Run Sonar Scan

**Path:** `scheiber-sa/shared-github-actions/sonar-scan@main`

Runs Sonarqube code quality scan using Scanwise.

#### Inputs

| Name                     | Description                                                            | Required | Default                        |
| ------------------------ | ---------------------------------------------------------------------- | -------- | ------------------------------ |
| `checkout`               | Specifies if this action should checkout the code                      | false    | `true`                         |
| `sonar-source-path`      | Path to the source code for Sonar scan                                 | false    | `src`                          |
| `reports-scopes`         | JSON array of report scopes                                            | false    | `["overall"]`                  |
| `reports-extensions`     | JSON array of report file extensions                                   | false    | `["html"]`                     |
| `reports-retention-days` | Number of days to retain reports                                       | false    | `7`                            |
| `new-code-n-days`        | Period for new code analysis                                           | false    | `3d`                           |
| `pre-scan-script`        | Script to run before scanning (restores coverage and validates it)    | false    | Coverage restoration script     |

#### Usage

```yaml
jobs:
  sonar:
    runs-on: ubuntu-latest
    steps:
      - name: Run Sonar Scan
        uses: scheiber-sa/shared-github-actions/sonar-scan@main
        with:
          sonar-source-path: src
```

### Setup Node and Install Dependencies

**Path:** `scheiber-sa/shared-github-actions/setup-node-and-dependencies@main`

Sets up Node.js and installs npm dependencies with support for private package registry and optional Python setup.

#### Inputs

| Name              | Description                                       | Required | Default |
| ----------------- | ------------------------------------------------- | -------- | ------- |
| `node-version`    | Node.js version to use                            | true     | –       |
| `github-token`    | GitHub token for accessing private packages       | true     | –       |
| `python`          | Whether to setup Python                           | false    | `false` |
| `python-version`  | Python version to use                             | false    | `3.10`  |

#### Usage

```yaml
jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - name: Setup Node and Install Dependencies
        uses: scheiber-sa/shared-github-actions/setup-node-and-dependencies@main
        with:
          node-version: "24.x"
          github-token: ${{ secrets.GH_TOKEN }}
          python: "true"
          python-version: "3.10"
```

---

## Workflows

### Deploy Storybook and Run Visual Tests

**Path:** `scheiber-sa/shared-github-actions/.github/workflows/deploy-storybook-and-visual-tests.yml@main`

Deploy Storybook to GitHub Pages and run Playwright visual tests with caching and test reporting.

#### Inputs

| Name                   | Description                                   | Required | Default                        |
| ---------------------- | --------------------------------------------- | -------- | ------------------------------ |
| `node-version`         | Node.js version to use                        | false    | `24.x`                         |
| `storybook-path`       | Path to the storybook build output            | false    | `storybook-static`             |
| `install-command`      | Command to install dependencies               | false    | `npm install`                  |
| `build-command`        | Command to build storybook                    | false    | `npm run build-storybook`      |
| `snapshots-command`    | Command to initialize snapshots               | false    | `npm run visual-test:snapshots-all` |
| `test-command`         | Command to run visual tests                   | false    | `npm run visual-test`          |
| `test-base-url`        | Base URL for visual tests                     | false    | `""`                           |
| `test-report-path`     | Path to test report JSON files                | false    | `./ctrf/*.json`                |
| `playwright-cache-path`| Path to cache Playwright binaries             | false    | `~/.cache/ms-playwright`       |
| `ci`                   | Set CI environment variable                   | false    | `true`                         |
| `update-snapshots`     | Force update snapshots                        | false    | `false`                        |

#### Usage

```yaml
on:
  push:
    branches: [develop]
  workflow_dispatch:
    inputs:
      update-snapshots:
        type: boolean
        default: false

jobs:
  deploy-test:
    uses: scheiber-sa/shared-github-actions/.github/workflows/deploy-storybook-and-visual-tests.yml@main
    with:
      test-base-url: "https://your-domain.github.io/your-project/"
      update-snapshots: ${{ github.event.inputs.update-snapshots }}
```

### Publish Package to GitHub Packages

**Path:** `scheiber-sa/shared-github-actions/.github/workflows/publish-design-system-package.yml@main`

Builds and publishes an npm package to GitHub Packages. Trigger this reusable workflow from your repository's release workflow.

#### Inputs

| Name                | Description                                      | Required | Default       |
| ------------------- | ------------------------------------------------ | -------- | ------------- |
| `node_version`      | The Node.js version to use                       | false    | `24`          |
| `scope`             | The npm scope for the package (e.g. `@your-org`) | true     | –             |
| `working_directory` | The working directory from which to publish      | false    | `dist`        |
| `install_command`   | The command to install dependencies              | false    | `npm ci`      |
| `build_command`     | The command to build the library                 | false    | `npm run build` |

#### Usage

```yaml
on:
  release:
    types: [created]

jobs:
  publish:
    uses: scheiber-sa/shared-github-actions/.github/workflows/publish-design-system-package.yml@main
    with:
      scope: "@scheiber-sa"
      node_version: "24"
    secrets: inherit
```
