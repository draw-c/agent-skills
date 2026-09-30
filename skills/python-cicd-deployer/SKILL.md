---
name: python-cicd-deployer
description: Python CI/CD and deployment specialist for GitHub Actions, automated testing, security scanning, Docker, and release pipelines.
---

# Python CI/CD with GitHub Actions

> **Skill Level:** Professional / Production-Grade  
> **Scope:** Continuous Integration, Testing, Security Scanning, and Automated Deployment for Python projects using GitHub Actions.  
> **Last Updated:** 2026-08-26

---

## 1. Philosophy

Treat CI/CD as **production infrastructure**, not an afterthought. Every workflow should be:

- **Deterministic:** Pin versions. No floating dependencies in CI.
- **Fast:** Aggressive caching, parallel jobs, and minimal redundant work.
- **Secure:** OIDC over long-lived secrets. Least-privilege permissions.
- **Observable:** Clear job names, step outputs, and artifact retention.

---

## 2. Repository Structure

```
├── .github/
│   ├── workflows/
│   │   ├── ci.yml              # Main CI: lint, test, type-check
│   │   ├── security.yml        # SAST, dependency audit, secrets scan
│   │   ├── release.yml         # Tag-based PyPI publish + GitHub Release
│   │   └── docker.yml          # Multi-arch image build & push
│   ├── actions/
│   │   └── setup-python-env/   # Composite action (DRY)
│   │       ├── action.yml
│   │       └── README.md
│   └── dependabot.yml          # Automated dependency updates
├── src/
│   └── mypackage/
├── tests/
├── pyproject.toml
├── Dockerfile
└── README.md
```

---

## 3. Core Workflow: `ci.yml`

A single, comprehensive CI workflow that runs on every PR and push to `main`.

```yaml
name: CI

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]
  workflow_dispatch:

concurrency:
  group: ${{ github.workflow }}-${{ github.ref }}
  cancel-in-progress: true

permissions:
  contents: read

jobs:
  # ─────────────────────────────────────────────
  # Job 1: Lint & Format
  # ─────────────────────────────────────────────
  lint:
    name: Lint & Format
    runs-on: ubuntu-latest
    steps:
      - name: Checkout
        uses: actions/checkout@v4

      - name: Setup Python
        uses: actions/setup-python@v5
        with:
          python-version: "3.14"
          cache: "pip"

      - name: Install linting tools
        run: |
          python -m pip install --upgrade pip
          pip install ruff mypy

      - name: Ruff check
        run: ruff check src tests

      - name: Ruff format check
        run: ruff format --check src tests

      - name: Type check with mypy
        run: mypy src

  # ─────────────────────────────────────────────
  # Job 2: Test Matrix
  # ─────────────────────────────────────────────
  test:
    name: Test (Python ${{ matrix.python-version }})
    runs-on: ubuntu-latest
    needs: lint
    strategy:
      fail-fast: false
      matrix:
        python-version: ["3.10", "3.11", "3.12", "3.13", "3.14"]

    steps:
      - name: Checkout
        uses: actions/checkout@v4

      - name: Setup Python ${{ matrix.python-version }}
        uses: actions/setup-python@v5
        with:
          python-version: ${{ matrix.python-version }}
          cache: "pip"

      - name: Install dependencies
        run: |
          python -m pip install --upgrade pip
          pip install -e ".[dev]"

      - name: Run tests with pytest
        run: pytest --cov=mypackage --cov-report=xml --cov-report=term-missing -v

      - name: Upload coverage
        uses: codecov/codecov-action@v4
        with:
          files: ./coverage.xml
          fail_ci_if_error: false
          token: ${{ secrets.CODECOV_TOKEN }}

  # ─────────────────────────────────────────────
  # Job 3: Build Package
  # ─────────────────────────────────────────────
  build:
    name: Build Distribution
    runs-on: ubuntu-latest
    needs: test
    steps:
      - name: Checkout
        uses: actions/checkout@v4

      - name: Setup Python
        uses: actions/setup-python@v5
        with:
          python-version: "3.14"

      - name: Install build tools
        run: pip install build twine

      - name: Build wheel & sdist
        run: python -m build

      - name: Check distribution with twine
        run: twine check dist/*

      - name: Upload build artifacts
        uses: actions/upload-artifact@v4
        with:
          name: dist
          path: dist/
          retention-days: 5
```

---

## 4. Security Workflow: `security.yml`

Run on schedule (nightly) and on dependency changes.

```yaml
name: Security Audit

on:
  push:
    paths:
      - "pyproject.toml"
      - "requirements*.txt"
      - ".github/workflows/security.yml"
  pull_request:
    paths:
      - "pyproject.toml"
      - "requirements*.txt"
  schedule:
    - cron: "0 6 * * *"  # Daily at 06:00 UTC
  workflow_dispatch:

permissions:
  contents: read
  security-events: write

jobs:
  bandit:
    name: SAST (Bandit)
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-python@v5
        with:
          python-version: "3.14"
      - run: pip install bandit[toml]
      - run: bandit -r src -f json -o bandit-report.json || true
      - uses: actions/upload-artifact@v4
        with:
          name: bandit-report
          path: bandit-report.json

  pip-audit:
    name: Dependency Audit
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-python@v5
        with:
          python-version: "3.14"
      - run: pip install pip-audit
      - run: pip-audit --desc --format=json --output=pip-audit-report.json || true
      - uses: actions/upload-artifact@v4
        with:
          name: pip-audit-report
          path: pip-audit-report.json

  trufflehog:
    name: Secrets Scan
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
        with:
          fetch-depth: 0
      - uses: trufflesecurity/trufflehog@main
        with:
          path: ./
          base: ${{ github.event.repository.default_branch }}
          head: HEAD
          extra_args: --debug --only-verified
```

---

## 5. Release Workflow: `release.yml`

**Trusted Publishing (OIDC)** — no PyPI API tokens stored in GitHub secrets.

```yaml
name: Release

on:
  push:
    tags:
      - "v*.*.*"

permissions:
  contents: write
  id-token: write  # Required for OIDC to PyPI

jobs:
  build:
    name: Build Release Artifacts
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-python@v5
        with:
          python-version: "3.14"
      - run: pip install build
      - run: python -m build
      - uses: actions/upload-artifact@v4
        with:
          name: dist
          path: dist/

  publish-pypi:
    name: Publish to PyPI
    needs: build
    runs-on: ubuntu-latest
    environment:
      name: pypi
      url: https://pypi.org/p/mypackage
    steps:
      - uses: actions/download-artifact@v4
        with:
          name: dist
          path: dist/
      - uses: pypa/gh-action-pypi-publish@release/v1

  github-release:
    name: Create GitHub Release
    needs: publish-pypi
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: softprops/action-gh-release@v2
        with:
          generate_release_notes: true
          files: dist/*
```

### PyPI Trusted Publishing Setup

1. Go to [pypi.org/manage/account/publishing/](https://pypi.org/manage/account/publishing/)
2. Add a **pending publisher**:
   - **PyPI Project Name:** `mypackage`
   - **Owner:** `your-org`
   - **Repository name:** `your-repo`
   - **Workflow name:** `release.yml`
   - **Environment name:** `pypi`

---

## 6. Docker Workflow: `docker.yml`

Multi-arch builds with layer caching.

```yaml
name: Docker Build

on:
  push:
    branches: [main]
    tags: ["v*.*.*"]
  pull_request:
    branches: [main]

permissions:
  contents: read
  packages: write

env:
  REGISTRY: ghcr.io
  IMAGE_NAME: ${{ github.repository }}

jobs:
  build:
    name: Build & Push Image
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Set up QEMU
        uses: docker/setup-qemu-action@v3

      - name: Set up Buildx
        uses: docker/setup-buildx-action@v3

      - name: Log in to GHCR
        uses: docker/login-action@v3
        with:
          registry: ${{ env.REGISTRY }}
          username: ${{ github.actor }}
          password: ${{ secrets.GITHUB_TOKEN }}

      - name: Extract metadata
        id: meta
        uses: docker/metadata-action@v5
        with:
          images: ${{ env.REGISTRY }}/${{ env.IMAGE_NAME }}
          tags: |
            type=ref,event=branch
            type=semver,pattern={{version}}
            type=semver,pattern={{major}}.{{minor}}
            type=sha,prefix=,suffix=,format=short

      - name: Build and push
        uses: docker/build-push-action@v6
        with:
          context: .
          platforms: linux/amd64,linux/arm64
          push: ${{ github.event_name != 'pull_request' }}
          tags: ${{ steps.meta.outputs.tags }}
          labels: ${{ steps.meta.outputs.labels }}
          cache-from: type=gha
          cache-to: type=gha,mode=max
```

---

## 7. Composite Action: Reusable Setup

DRY your Python environment setup across workflows.

**`.github/actions/setup-python-env/action.yml`**

```yaml
name: "Setup Python Environment"
description: "Checkout, setup Python, install deps, and cache"

inputs:
  python-version:
    description: "Python version"
    required: false
    default: "3.14"
  install-extras:
    description: "Extras to install (e.g., dev,test)"
    required: false
    default: "dev"

runs:
  using: "composite"
  steps:
    - name: Setup Python
      uses: actions/setup-python@v5
      with:
        python-version: ${{ inputs.python-version }}
        cache: "pip"

    - name: Install dependencies
      shell: bash
      run: |
        python -m pip install --upgrade pip
        pip install -e ".[${{ inputs.install-extras }}]"
```

**Usage in a workflow:**

```yaml
    steps:
      - uses: actions/checkout@v4
      - uses: ./.github/actions/setup-python-env
        with:
          python-version: "3.14"
          install-extras: "dev,test"
```

---

## 8. `pyproject.toml` Essentials

```toml
[build-system]
requires = ["hatchling"]
build-backend = "hatchling.build"

[project]
name = "mypackage"
version = "0.1.0"
description = "Elite Python package"
readme = "README.md"
requires-python = ">=3.10"
license = { text = "MIT" }
classifiers = [
    "Programming Language :: Python :: 3",
    "License :: OSI Approved :: MIT License",
]
dependencies = [
    "pydantic>=2.0",
]

[project.optional-dependencies]
dev = [
    "pytest>=8.0",
    "pytest-cov>=5.0",
    "ruff>=0.6.0",
    "mypy>=1.10",
    "bandit[toml]>=1.7",
    "pip-audit>=2.7",
]

[tool.ruff]
target-version = "py310"
line-length = 100
select = ["E", "F", "I", "N", "W", "UP", "B", "C4", "SIM"]

[tool.ruff.format]
quote-style = "double"
indent-style = "space"

[tool.mypy]
python_version = "3.10"
strict = true
warn_return_any = true
warn_unused_ignores = true

[tool.pytest.ini_options]
testpaths = ["tests"]
addopts = "-ra -q --strict-markers"

[tool.coverage.run]
source = ["src/mypackage"]
branch = true

[tool.coverage.report]
exclude_lines = [
    "pragma: no cover",
    "if __name__ == .__main__.:",
    "raise NotImplementedError",
]
```

---

## 9. `Dockerfile` (Production-Ready)

```dockerfile
# ── Builder stage ──
FROM python:3.14-slim-bookworm AS builder

WORKDIR /build
RUN pip install --no-cache-dir build
COPY pyproject.toml README.md ./
COPY src ./src
RUN python -m build

# ── Runtime stage ──
FROM python:3.14-slim-bookworm AS runtime

ENV PYTHONDONTWRITEBYTECODE=1
ENV PYTHONUNBUFFERED=1
ENV PIP_NO_CACHE_DIR=1

RUN groupadd -r appgroup && useradd -r -g appgroup appuser

WORKDIR /app

COPY --from=builder /build/dist/*.whl ./
RUN pip install *.whl && rm *.whl

USER appuser

ENTRYPOINT ["python", "-m", "mypackage"]
```

---

## 10. Dependabot Configuration

**`.github/dependabot.yml`**

```yaml
version: 2
updates:
  - package-ecosystem: "pip"
    directory: "/"
    schedule:
      interval: "weekly"
      day: "monday"
      time: "09:00"
    open-pull-requests-limit: 10
    reviewers:
      - "your-team"
    labels:
      - "dependencies"
      - "python"

  - package-ecosystem: "github-actions"
    directory: "/"
    schedule:
      interval: "weekly"
    open-pull-requests-limit: 5
    labels:
      - "dependencies"
      - "github-actions"
---

## 11. Stealth Scraping & Anti-Bot CI/CD Safety

When deploying or testing projects with high-detection scraping pipelines (e.g., sportsbooks, betting odds, Cloudflare/DataDome protected endpoints):

1. **Never Hit Live Protected Targets in Raw CI Runners:**
   - GitHub Actions runner IP ranges (Azure datacenter ASN) are instantly flagged by Cloudflare Turnstile, DataDome, and Akamai.
   - Use mocked responses, recorded JSON/HTML fixtures, or mock servers in CI.
   - For integration testing against live endpoints, route requests through residential/mobile proxies or dedicated self-hosted runners.

2. **Lock Python Runtime for Stealth Binaries:**
   - Advanced stealth tools (`curl_cffi` for TLS/JA3 spoofing, `camoufox` for C++ Firefox engine spoofing, `nodriver`, `playwright-stealth`) rely on compiled C-extensions and custom browser builds.
   - Verify Python versions (e.g., Python 3.11) across local dev, CI, and Docker images to prevent broken binary wheels.

3. **Browser Engine Dependencies in Docker:**
   - Ensure the production Dockerfile installs requisite headless browser system dependencies (`libnss3`, `libgbm1`, `libxcomposite1`, `libasound2`, etc.) and runs `playwright install` or `camoufox fetch`.

4. **Secret Isolation for Proxies & Fingerprints:**
   - Residential/mobile proxy endpoints and API keys must always be stored in GitHub Secrets and never printed in debug logs.

---

## 12. Best Practices Checklist

| Area | Rule |
|------|------|
| **Versions** | Pin GitHub Actions to major versions (`@v4`) or SHA for maximum security. |
| **Caching** | Always use `actions/setup-python` with `cache: "pip"` or `cache-dependency-path`. |
| **Permissions** | Declare `permissions:` at job or workflow level. Default to `contents: read`. |
| **Secrets** | Never echo secrets. Use GitHub Environments + required reviewers for deployment jobs. |
| **Stealth Scrapers** | Mock protected sportsbook endpoints in CI; never leak proxy credentials or expose datacenter IPs. |
| **PyPI** | Use **Trusted Publishing (OIDC)**. Delete any legacy `PYPI_API_TOKEN` secrets. |
| **Artifacts** | Set `retention-days` on artifacts to avoid storage bloat. |
| **Concurrency** | Use `concurrency` to cancel stale runs on force-push. |
| **Matrix** | Use `fail-fast: false` in test matrices so all Python versions finish. |
| **Docker** | Build multi-arch (`amd64` + `arm64`). Install browser deps for Playwright/Camoufox. |
| **SAST** | Run Bandit + pip-audit on every dependency change and nightly. |

---

## 13. Quick Start Commands

```bash
# 1. Scaffold a new project
mkdir mypackage && cd mypackage
git init

# 2. Create the directory structure
mkdir -p .github/workflows .github/actions/setup-python-env src/mypackage tests

# 3. Copy the workflow files from this skill into place

# 4. Install locally for development
pip install -e ".[dev]"

# 5. Run the same checks CI will run
ruff check src tests
ruff format --check src tests
mypy src
pytest --cov=mypackage -v

# 6. Build the package locally
python -m build
twine check dist/*
```

---

## 14. Troubleshooting

| Symptom | Cause | Fix |
|---------|-------|-----|
| `pip cache` not hitting | Missing `cache-dependency-path` | Add `cache-dependency-path: pyproject.toml` |
| PyPI publish fails with 403 | Trusted Publishing not configured | Add pending publisher on PyPI.org |
| Docker build slow | No layer caching | Use `cache-from: type=gha` + `cache-to: type=gha` |
| Bandit fails on asserts | `assert` in production code | Use `bandit -s B101` or remove asserts |
| mypy errors on third-party libs | Missing stubs | Add `types-*` packages or `ignore_missing_imports = true` |

---

*End of Skill File. Copy the relevant sections into your repository and adapt `mypackage` to your project name.*
