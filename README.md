# Foundry VTT Module Actions

GitHub Actions for building, testing, and releasing Foundry VTT modules.

## Actions Included

- **Release Action** (`action.yml`): Build and release modules with automatic zip creation
- **Testing Action** (`test-action.yml`): Run tests, linting, and build validation

---

## Release Action

A GitHub Action for building and releasing Foundry VTT modules with automatic zip creation and artifact upload.

## Features

- 🏗️ **Automatic building** with configurable build commands
- 📦 **Smart zip creation** with configurable file inclusion
- 🚀 **Release artifact upload** of both module.json and module.zip
- 🏷️ **Version extraction** from git release tags
- ⚙️ **Environment injection** for build-time customization

## Usage

### Basic Usage

```yaml
name: Release Module

on:
  release:
    types: [published]

jobs:
  build:
    runs-on: ubuntu-latest
    permissions:
      contents: write
    steps:
      - name: Checkout
        uses: actions/checkout@v4

      - name: Release module
        uses: rayners/foundry-module-actions/release@v1
        with:
          module-files: 'module.json styles/ templates/ languages/'
```

### Advanced Usage

```yaml
name: Release Module

on:
  release:
    types: [published]

jobs:
  build:
    runs-on: ubuntu-latest
    permissions:
      contents: write
    steps:
      - name: Checkout
        uses: actions/checkout@v4

      - name: Release module
        uses: rayners/foundry-module-actions/release@v1
        with:
          node-version: '20'
          build-command: 'npm run build:prod'
          working-directory: 'build'
          module-files: 'module.json scripts/ styles/ templates/ languages/ assets/'
          zip-name: 'my-module.zip'
```

## Inputs

| Input | Description | Required | Default |
|-------|-------------|----------|---------|
| `node-version` | Node.js version to use | No | `18` |
| `build-command` | Build command to run | No | `npm run build` |
| `working-directory` | Directory containing built files | No | `dist` |
| `module-files` | Files/directories to include in zip | Yes | `module.json styles/ templates/ languages/` |
| `zip-name` | Name of the zip file | No | `module.zip` |

## Outputs

| Output | Description |
|--------|-------------|
| `version` | Version number extracted from git tag |
| `zip-path` | Path to the created module zip file |

## How It Works

1. **Setup**: Installs Node.js and dependencies
2. **Version Extraction**: Gets version from the git tag that triggered the release
3. **Build**: Runs your build command with environment variables:
   - `MODULE_VERSION`: Version without 'v' prefix
   - `GH_PROJECT`: GitHub repository name
   - `GH_TAG`: Full git tag name
4. **Zip Creation**: Creates module.zip with specified files plus README.md and LICENSE
5. **Artifact Upload**: Uploads both module.json and module.zip to the GitHub release

## Environment Variables During Build

Your build process can access these environment variables:

- `MODULE_VERSION`: Version number (e.g., `1.0.0`)
- `GH_PROJECT`: Repository name (e.g., `rayners/my-module`)
- `GH_TAG`: Git tag (e.g., `v1.0.0`)

These are commonly used to inject version numbers and URLs into module.json during build.

## Module Structure Requirements

Your module should:
- Have a `package.json` with appropriate build scripts
- Build output should go to the configured working directory (default: `dist/`)
- Include `module.json` in the build output
- Optionally include `README.md` and `LICENSE` (automatically added if present)

## Example Build Scripts

### Vite (TypeScript/SCSS)
```json
{
  "scripts": {
    "build": "vite build"
  }
}
```

### Rollup (TypeScript)
```json
{
  "scripts": {
    "build": "rollup -c"
  }
}
```

### Simple Copy (JavaScript/CSS)
```json
{
  "scripts": {
    "build": "mkdir -p dist && cp -r src/* dist/"
  }
}
```

## Permissions

The workflow needs `contents: write` permission to upload artifacts to the release:

```yaml
jobs:
  build:
    runs-on: ubuntu-latest
    permissions:
      contents: write
```

## Version Tagging

Create releases with semantic version tags:
- ✅ `v1.0.0`, `v0.1.0`, `v2.1.3`
- ❌ `v1.0.0-alpha`, `1.0.0`, `release-1.0.0`

The action extracts the version number and provides it to your build process.

---

## Testing Action

A GitHub Action for running comprehensive tests, linting, and build validation for Foundry VTT modules.

### Features

- 🧪 **Automated testing** with configurable test commands
- 🔍 **Code linting** and format checking
- 🏗️ **Build validation** to ensure module compiles
- 📊 **Test coverage** reporting (optional)
- 📋 **Summary reporting** with pass/fail status
- 🚫 **Fail-fast behavior** to block bad PRs

### Basic Usage

```yaml
name: Test Module

on:
  push:
    branches: [ main ]
  pull_request:
    branches: [ main ]

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout
        uses: actions/checkout@v4

      - name: Run tests
        uses: rayners/foundry-module-actions/test@v1
```

### Advanced Usage

```yaml
name: Test Module

on:
  push:
    branches: [ main ]
  pull_request:
    branches: [ main ]

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout
        uses: actions/checkout@v4

      - name: Run comprehensive tests
        uses: rayners/foundry-module-actions/test@v1
        with:
          node-version: '20'
          test-command: 'npm run test:ci'
          lint-command: 'npm run lint:strict'
          coverage: true
```

### Testing Action Inputs

| Input | Description | Required | Default |
|-------|-------------|----------|---------|
| `node-version` | Node.js version to use | No | `18` |
| `test-command` | Test command to run | No | `npm run test:run` |
| `lint-command` | Lint command to run | No | `npm run lint` |
| `build-command` | Build command to run | No | `npm run build` |
| `format-check-command` | Format check command | No | `npm run format:check` |
| `coverage` | Generate coverage report | No | `false` |
| `coverage-command` | Coverage command to run | No | `npm run test:coverage` |

### Testing Action Outputs

| Output | Description |
|--------|-------------|
| `test-result` | Result of test execution (success/failure) |
| `lint-result` | Result of lint execution (success/failure) |
| `build-result` | Result of build execution (success/failure) |

### Required Package.json Scripts

Your module should have these npm scripts (or customize the commands):

```json
{
  "scripts": {
    "test:run": "vitest run",
    "test:coverage": "vitest run --coverage", 
    "lint": "eslint src",
    "format:check": "prettier --check \"src/**/*.{ts,js}\"",
    "build": "rollup -c"
  }
}
```

## License

MIT License - see LICENSE file for details.