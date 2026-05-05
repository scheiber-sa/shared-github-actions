# shared-github-actions

Repository serving shared GitHub Actions and Workflows for all projects of the organization.

---

## Actions

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

### Run Sonar Scan

**Path:** `scheiber-sa/shared-github-actions/sonar-scan@master`

Runs Sonarqube code quality scan using Scanwise. Always checks out the repository with `fetch-depth: 0`.

#### Inputs

| Name                     | Description                                                                                             | Required | Default       |
| ------------------------ | ------------------------------------------------------------------------------------------------------- | -------- | ------------- |
| `sonar-source-path`      | Path to the source code for Sonar scan                                                                  | false    | `src`         |
| `reports-scopes`         | JSON array of report scopes                                                                             | false    | `["overall"]` |
| `reports-extensions`     | JSON array of report file extensions                                                                    | false    | `["html"]`    |
| `reports-retention-days` | Number of days to retain reports                                                                        | false    | `7`           |
| `new-code-n-days`        | Period for new code analysis                                                                            | false    | `3d`          |
| `coverage-artifact`      | Name of the coverage artifact to download. When set, downloads it to `/tmp/coverage/` before scanning. | false    | –             |

#### Usage

```yaml
jobs:
  code-quality:
    runs-on: ubuntu-latest
    steps:
      - name: Run Sonar Scan
        uses: scheiber-sa/shared-github-actions/sonar-scan@master
        with:
          coverage-artifact: coverage-report
```

---

### Release

**Path:** `scheiber-sa/shared-github-actions/release@master`

Validates a semver tag, generates release notes from git history and package metadata, and creates a GitHub Release via `softprops/action-gh-release`.

> **Requires** `permissions: contents: write` on the calling job and a prior `actions/checkout@v6` step with `fetch-depth: 0`.

#### Inputs

| Name               | Description                                                                                              | Required | Default |
| ------------------ | -------------------------------------------------------------------------------------------------------- | -------- | ------- |
| `package-artifact` | Name of the package artifact to attach to the release. When omitted, the release has no file attachment. | false    | –       |

#### Outputs

| Name            | Description                                          |
| --------------- | ---------------------------------------------------- |
| `tag`           | The validated release tag (e.g. `v1.2.3`)            |
| `is_prerelease` | Whether the release is a prerelease (`true`/`false`) |
| `url`           | URL of the created GitHub Release                    |

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
      - name: Checkout code
        uses: actions/checkout@v6
        with:
          fetch-depth: 0

      - name: Release
        uses: scheiber-sa/shared-github-actions/release@master
        with:
          package-artifact: dist-package
```

---

### Visual Tests

**Path:** `scheiber-sa/shared-github-actions/visual-tests@master`

Runs Playwright visual tests against a locally served Storybook. Handles Playwright binary caching, snapshot caching, serving, and CTRF report publishing. The caller is responsible for checkout, `npm install`, and building Storybook.

> **Requires** `GITHUB_TOKEN` env on the calling step (for the CTRF reporter).

#### Inputs

| Name                | Description                                             | Required | Default           |
| ------------------- | ------------------------------------------------------- | -------- | ----------------- |
| `storybook-path`    | Path to the built Storybook output to serve             | false    | `storybook-static` |
| `snapshots-command` | Command to initialize snapshots when no cache exists    | true     | –                 |
| `test-command`      | Command to run visual tests                             | true     | –                 |

#### Usage

```yaml
jobs:
  visual-tests:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout code
        uses: actions/checkout@v6

      - name: Setup and install
        uses: scheiber-sa/shared-github-actions/npm-install@master
        with:
          node-version: "24.x"

      - name: Build Storybook
        run: npm run build-storybook

      - name: Run visual tests
        uses: scheiber-sa/shared-github-actions/visual-tests@master
        with:
          snapshots-command: npm run visual-test:snapshots-all
          test-command: npm run visual-test
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
```

---

### Deploy Storybook to GitHub Pages

**Path:** `scheiber-sa/shared-github-actions/deploy-storybook@master`

Uploads a pre-built Storybook directory as a Pages artifact and deploys it. The caller is responsible for checkout, `npm install`, and building Storybook.

> **Requires** `permissions: pages: write` and `id-token: write` on the calling job.

#### Inputs

| Name             | Description                                   | Required | Default            |
| ---------------- | --------------------------------------------- | -------- | ------------------ |
| `storybook-path` | Path to the built Storybook output to deploy  | false    | `storybook-static` |

#### Outputs

| Name       | Description                          |
| ---------- | ------------------------------------ |
| `page_url` | URL of the deployed GitHub Pages site |

#### Usage

```yaml
jobs:
  deploy-storybook:
    runs-on: ubuntu-latest
    if: startsWith(github.ref, 'refs/tags/v')
    environment:
      name: github-pages
      url: ${{ steps.deploy.outputs.page_url }}
    permissions:
      pages: write
      id-token: write
    steps:
      - name: Checkout code
        uses: actions/checkout@v6

      - name: Setup and install
        uses: scheiber-sa/shared-github-actions/npm-install@master
        with:
          node-version: "24.x"

      - name: Build Storybook
        run: npm run build-storybook

      - name: Deploy Storybook
        id: deploy
        uses: scheiber-sa/shared-github-actions/deploy-storybook@master
        with:
          storybook-path: dist/storybook
```

---

### Publish to GitHub Packages

**Path:** `scheiber-sa/shared-github-actions/publish@master`

Configures npm for the `@scheiber-sa` GitHub Packages registry and publishes the `dist/` directory. The caller is responsible for checkout and building.

> **Requires** `NODE_AUTH_TOKEN` env on the calling step and `permissions: packages: write` on the job.

#### Inputs

| Name           | Description          | Required | Default  |
| -------------- | -------------------- | -------- | -------- |
| `node-version` | Node.js version to use | false  | `latest` |

#### Usage

```yaml
jobs:
  publish:
    runs-on: ubuntu-latest
    if: startsWith(github.ref, 'refs/tags/v')
    permissions:
      packages: write
    steps:
      - name: Checkout code
        uses: actions/checkout@v6

      - name: Build library
        run: npm run build

      - name: Publish to GitHub Packages
        uses: scheiber-sa/shared-github-actions/publish@master
        with:
          node-version: "24.x"
        env:
          NODE_AUTH_TOKEN: ${{ secrets.GITHUB_TOKEN }}
```
