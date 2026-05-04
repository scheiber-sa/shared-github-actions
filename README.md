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

| Name                     | Description                                                                                                     | Required | Default       |
| ------------------------ | --------------------------------------------------------------------------------------------------------------- | -------- | ------------- |
| `sonar-source-path`      | Path to the source code for Sonar scan                                                                          | false    | `src`         |
| `reports-scopes`         | JSON array of report scopes                                                                                     | false    | `["overall"]` |
| `reports-extensions`     | JSON array of report file extensions                                                                            | false    | `["html"]`    |
| `reports-retention-days` | Number of days to retain reports                                                                                | false    | `7`           |
| `new-code-n-days`        | Period for new code analysis                                                                                    | false    | `3d`          |
| `coverage-artifact`      | Name of the coverage artifact to download. When set, downloads it to `/tmp/coverage/` before scanning.        | false    | –             |

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

### Release

**Path:** `scheiber-sa/shared-github-actions/release@master`

Validates a semver tag, generates release notes from git history and package metadata, and creates a GitHub Release via `softprops/action-gh-release`.

> **Requires** `permissions: contents: write` on the calling job.

#### Inputs

| Name            | Description                                             | Required | Default        |
| --------------- | ------------------------------------------------------- | -------- | -------------- |
| `artifact-name` | Name of the package artifact to download                | false    | `dist-package` |
| `packages-path` | Directory where the package artifact will be downloaded | false    | `packages/`    |

#### Outputs

| Name           | Description                          |
| -------------- | ------------------------------------ |
| `tag`          | The validated release tag (e.g. `v1.2.3`) |
| `is_prerelease`| Whether the release is a prerelease (`true`/`false`) |
| `url`          | URL of the created GitHub Release    |

#### Usage

```yaml
jobs:
  release:
    runs-on: ubuntu-latest
    if: startsWith(github.ref, 'refs/tags/v')
    needs: package
    permissions:
      contents: write
    steps:
      - name: Release
        uses: scheiber-sa/shared-github-actions/release@master
```

---

### npm install

**Path:** `scheiber-sa/shared-github-actions/npm-install@master`

Sets up Node.js and runs `npm ci`. When `github-token` is provided, also configures npm to access the `@scheiber-sa` private registry on GitHub Packages.

#### Inputs

| Name           | Description                                                                             | Required | Default  |
| -------------- | --------------------------------------------------------------------------------------- | -------- | -------- |
| `node-version` | Node.js version to use                                                                  | false    | `latest` |
| `github-token` | GitHub token for accessing private packages. Enables `@scheiber-sa` registry when set. | false    | –        |
| `registry-url` | Registry URL to pass to `actions/setup-node`                                            | false    | –        |
| `scope`        | npm scope to pass to `actions/setup-node`                                               | false    | –        |

#### Usage

```yaml
jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout code
        uses: actions/checkout@v6

      - name: Install dependencies
        uses: scheiber-sa/shared-github-actions/npm-install@master
        with:
          node-version: "24.x"
          github-token: ${{ secrets.DESIGN_SYSTEM_TOKEN }}
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
