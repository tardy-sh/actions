# Bun Library Publish Action

Publish a bun/npm package to a registry using the [Bun](https://bun.sh/) package manager.

## Inputs

| Input | Description | Required | Default |
|-------|-------------|----------|---------|
| `path` | Path to the built package to publish | Yes | - |
| `registry_url` | Registry URL | No | `https://npm.pkg.github.com` |
| `token` | Authentication token for the registry | Yes | - |
| `dry_run` | Run publish in dry-run mode (test without publishing) | No | `false` |

## Usage

### Using Latest Version (@main)
Best for testing the latest features. May receive breaking changes.

```yaml
- name: Publish package
  uses: tardy-sh/actions/bun/publish@main
  with:
    path: './packages/my-library'
    token: ${{ secrets.GITHUB_TOKEN }}
```

### Using Stable Version (@v1)
Recommended for production. Stable with semantic versioning.

```yaml
- name: Publish package
  uses: tardy-sh/actions/bun/publish@v1
  with:
    path: './packages/my-library'
    token: ${{ secrets.GITHUB_TOKEN }}
```

## Complete Workflow Example

```yaml
name: Build and Publish

on:
  push:
    tags:
      - 'v*'

jobs:
  publish:
    name: Build and Publish Package
    runs-on: ubuntu-latest
    permissions:
      contents: read
      packages: write

    steps:
      - name: Checkout repository
        uses: actions/checkout@v4

      - name: Build library
        uses: tardy-sh/actions/bun/build@v1
        with:
          path: '.'

      - name: Publish to GitHub Packages
        uses: tardy-sh/actions/bun/publish@v1
        with:
          path: '.'
          token: ${{ secrets.GITHUB_TOKEN }}
```

## Setup Guide

### 1. Configure package.json

For GitHub Packages, your `package.json` must include:

```json
{
  "name": "@your-username/your-package",
  "version": "1.0.0",
  "publishConfig": {
    "registry": "https://npm.pkg.github.com"
  }
}
```

**Important Notes:**
- Package name MUST be scoped (e.g., `@your-username/package-name`)
- The scope should match your GitHub username or organization
- The `publishConfig.registry` field is required for GitHub Packages

### 2. Create GitHub Token

For publishing to GitHub Packages:

1. Go to GitHub Settings → Developer settings → Personal access tokens → Tokens (classic)
2. Click "Generate new token (classic)"
3. Give it a descriptive name (e.g., "Package Publishing")
4. Select the following scopes:
   - `write:packages` - Upload packages to GitHub Package Registry
   - `read:packages` - Download packages from GitHub Package Registry (optional)
   - `repo` - Only if publishing from a private repository
5. Click "Generate token" and copy the token immediately

**Alternative:** Use `${{ secrets.GITHUB_TOKEN }}` in workflows, which is automatically provided by GitHub Actions with appropriate permissions when you configure the workflow permissions.

### 3. Configure Repository Secrets

If using a Personal Access Token (PAT):

1. Go to your repository → Settings → Secrets and variables → Actions
2. Click "New repository secret"
3. Name: `NPM_TOKEN` (or any name you prefer)
4. Value: Paste your token from step 2
5. Click "Add secret"

Then use it in your workflow:
```yaml
- name: Publish package
  uses: tardy-sh/actions/bun/publish@v1
  with:
    path: '.'
    token: ${{ secrets.NPM_TOKEN }}
```

**Recommended:** Use `${{ secrets.GITHUB_TOKEN }}` instead, which requires no manual setup:
```yaml
jobs:
  publish:
    permissions:
      contents: read
      packages: write
    steps:
      - name: Publish package
        uses: tardy-sh/actions/bun/publish@v1
        with:
          path: '.'
          token: ${{ secrets.GITHUB_TOKEN }}
```

## Advanced Usage

### Dry Run (Test Publishing)

Test your publish configuration without actually publishing:

```yaml
- name: Test publish configuration
  uses: tardy-sh/actions/bun/publish@v1
  with:
    path: '.'
    token: ${{ secrets.GITHUB_TOKEN }}
    dry_run: 'true'
```

### Custom Registry

Publish to a different npm registry:

```yaml
- name: Publish to custom registry
  uses: tardy-sh/actions/bun/publish@v1
  with:
    path: '.'
    registry_url: 'https://registry.npmjs.org'
    token: ${{ secrets.NPM_TOKEN }}
```

### Publish Multiple Packages (Monorepo)

```yaml
jobs:
  publish:
    name: Publish Packages
    runs-on: ubuntu-latest
    permissions:
      packages: write
      contents: read
    strategy:
      matrix:
        package:
          - packages/core
          - packages/utils
          - packages/ui

    steps:
      - name: Checkout repository
        uses: actions/checkout@v4

      - name: Build ${{ matrix.package }}
        uses: tardy-sh/actions/bun/build@v1
        with:
          path: ${{ matrix.package }}

      - name: Publish ${{ matrix.package }}
        uses: tardy-sh/actions/bun/publish@v1
        with:
          path: ${{ matrix.package }}
          token: ${{ secrets.GITHUB_TOKEN }}
```

## Troubleshooting

### "publishConfig not found in package.json"

If you see this warning when using GitHub Packages:
1. Add `publishConfig` to your `package.json` as shown in the setup guide
2. Ensure your package name is scoped (starts with `@`)

### Permission Denied Errors

Ensure your workflow has the correct permissions:
```yaml
permissions:
  contents: read
  packages: write
```

### Authentication Failures

- Verify your token has the `write:packages` scope
- For `GITHUB_TOKEN`, ensure workflow permissions are configured
- Check that the registry URL matches your publishConfig

## Notes

- The action automatically installs Bun if not already available
- Authentication is configured automatically via `.npmrc`
- GitHub Packages requires scoped package names
- The package version in `package.json` must be unique for each publish
