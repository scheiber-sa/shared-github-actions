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

---

## Workflows

### Publish Package to GitHub Packages

**Path:** `scheiber-sa/shared-github-actions/.github/workflows/publish-package.yml@main`

Builds and publishes an npm package to GitHub Packages. Trigger this reusable workflow from your repository's release workflow.

#### Inputs

| Name                | Description                                      | Required | Default       |
| ------------------- | ------------------------------------------------ | -------- | ------------- |
| `node_version`      | The Node.js version to use                       | false    | `20`          |
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
    uses: scheiber-sa/shared-github-actions/.github/workflows/publish-package.yml@main
    with:
      scope: "@scheiber-sa"
      node_version: "20"
    secrets: inherit
```
