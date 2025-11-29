# Biome Lint Action

Run [Biome](https://biomejs.dev/) linter with GitHub reporter for seamless integration with GitHub's code review interface.

## Inputs

| Input | Description | Required | Default |
|-------|-------------|----------|---------|
| `path` | Path to lint | No | `.` (current directory) |
| `config_path` | Path to Biome config file | No | (uses default config discovery) |

## Usage

### Using Latest Version (@main)
Best for testing the latest features. May receive breaking changes.

```yaml
- name: Lint with Biome
  uses: tardy-sh/actions/biome-lint@main
  with:
    path: '.'
```

### Using Stable Version (@v1)
Recommended for production. Stable with semantic versioning.

```yaml
- name: Lint with Biome
  uses: tardy-sh/actions/biome-lint@v1
  with:
    path: '.'
```

## Complete Workflow Example

```yaml
name: Lint Code

on:
  push:
    branches: [ main, develop ]
  pull_request:
    branches: [ main ]

jobs:
  lint:
    name: Run Biome Linter
    runs-on: ubuntu-latest

    steps:
      - name: Checkout repository
        uses: actions/checkout@v4

      - name: Run Biome lint
        uses: tardy-sh/actions/biome-lint@v1
        with:
          path: '.'
```

## Advanced Usage

### Custom Config Path
If your Biome configuration is in a non-standard location:

```yaml
- name: Lint with custom config
  uses: tardy-sh/actions/biome-lint@v1
  with:
    path: 'src'
    config_path: 'config/biome.json'
```

### Lint Specific Directory
To lint only a specific subdirectory:

```yaml
- name: Lint source directory only
  uses: tardy-sh/actions/biome-lint@v1
  with:
    path: 'src'
```

## Notes

- The action uses `biome ci --reporter=github` which formats output for GitHub's annotation system
- Linting errors will appear as annotations in pull request reviews
- The action installs Biome globally via npm if not already available
