# Bun Library Build Action

Build a bun/npm package using the [Bun](https://bun.sh/) package manager.

## Inputs

| Input | Description | Required | Default |
|-------|-------------|----------|---------|
| `path` | Path to the package to build | Yes | - |

## Usage

### Using Latest Version (@main)
Best for testing the latest features. May receive breaking changes.

```yaml
- name: Build library
  uses: tardy-sh/actions/bun/build@main
  with:
    path: './packages/my-library'
```

### Using Stable Version (@v1)
Recommended for production. Stable with semantic versioning.

```yaml
- name: Build library
  uses: tardy-sh/actions/bun/build@v1
  with:
    path: './packages/my-library'
```

## Complete Workflow Example

```yaml
name: Build and Test

on:
  push:
    branches: [ main ]
  pull_request:
    branches: [ main ]

jobs:
  build:
    name: Build Package
    runs-on: ubuntu-latest

    steps:
      - name: Checkout repository
        uses: actions/checkout@v4

      - name: Build library
        uses: tardy-sh/actions/bun/build@v1
        with:
          path: '.'

      - name: Upload build artifacts
        uses: actions/upload-artifact@v4
        with:
          name: build-output
          path: |
            dist/
            build/
```

## Advanced Usage

### Build Multiple Packages (Monorepo)

```yaml
jobs:
  build:
    name: Build All Packages
    runs-on: ubuntu-latest
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
```

### Build and Test

```yaml
- name: Build library
  uses: tardy-sh/actions/bun/build@v1
  with:
    path: '.'

- name: Run tests
  working-directory: '.'
  run: bun test
```

## Requirements

- Your `package.json` must contain a `build` script
- Example `package.json`:
  ```json
  {
    "name": "my-package",
    "scripts": {
      "build": "bun build src/index.ts --outdir dist"
    }
  }
  ```

## Notes

- The action automatically installs Bun if not already available
- Dependencies are installed using `bun install` before building
- Build output location depends on your build script configuration
