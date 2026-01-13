# CI/CD Best Practices for KRL Repositories

This document outlines best practices to prevent common CI/CD failures across all KRL repositories.

## Table of Contents
1. [Dependency Management](#dependency-management)
2. [Workflow Configuration](#workflow-configuration)
3. [Security Scanning](#security-scanning)
4. [Disk Space Management](#disk-space-management)
5. [Testing Best Practices](#testing-best-practices)

---

## 1. Dependency Management

### Problem
Missing dependencies cause test failures with `ModuleNotFoundError`.

### Solutions

#### Always declare dependencies explicitly in `pyproject.toml`

```toml
[project]
dependencies = [
    "numpy>=1.24.0",  # If using numpy anywhere
    "geoalchemy2>=0.14.0",  # If using GIS features
    "krl-types~=0.2.0",  # KRL internal dependencies
]

[project.optional-dependencies]
test = [
    "pytest>=7.4.0",
    "pytest-asyncio>=0.21.0",
    "pytest-cov>=4.1.0",
    "pytest-timeout>=2.2.0",  # If using --timeout flag
]
```

#### Check for transitive dependencies

```bash
# After adding imports, verify dependencies are declared
python -c "import your_new_module"
pip list | grep -i module_name
```

#### CI Workflow Best Practice

```yaml
- name: Install dependencies
  run: |
    pip install --upgrade pip
    pip install -e ".[test]"  # Install with test dependencies
    pip install -e ".[dev]"   # For dev workflows
```

---

## 2. Workflow Configuration

### Problem
Workflows failing with 0s duration due to incorrect trigger configuration.

### Solutions

#### Only trigger workflows on appropriate events

```yaml
# ❌ BAD - Will fail on push events
on:
  pull_request:
    paths:
      - 'app/**/*.py'

# ✅ GOOD - Explicitly define when to run
on:
  pull_request:
    paths:
      - 'app/**/*.py'
  workflow_dispatch:  # Allow manual triggering
```

#### Validate YAML syntax

```bash
# Use yamllint or GitHub Actions VS Code extension
yamllint .github/workflows/*.yml

# Or use a online YAML validator
```

#### Common YAML pitfalls

```yaml
# ❌ BAD - Multiline string breaks YAML parsing
script: |
  const comment = `This has a problem
  Because it contains colons: like this`

# ✅ GOOD - Use proper YAML multiline syntax
script: |
  const comment = 'Use single quotes or escape properly'
```

---

## 3. Security Scanning

### Problem
1,149 false positive secrets detected, blocking workflows.

### Solutions

#### Create `.gitleaksignore`

```
# Ignore test files and documentation
tests/**/*
examples/**/*
docs/**/*
*.md
*.example

# Ignore generated files
**/__pycache__/**/*
**/node_modules/**/*
**/venv/**/*
```

#### Configure `.gitleaks.toml` allowlist

```toml
[allowlist]
paths = [
    '''\.md$''',
    '''^tests/''',
    '''^docs/''',
    '''\.example$''',
]

regexes = [
    '''localhost''',
    '''127\.0\.0\.1''',
    '''test[_-]?(secret|password|token|key)''',
    '''fake[_-]?(secret|password|token|key)''',
    '''dummy[_-]?(secret|password|token|key)''',
    '''mock[_-]?(secret|password|token|key)''',
    '''test-.*-for-ci-only''',
]
```

#### Use proper test credentials

```python
# ❌ BAD
API_KEY = "sk_live_abcd1234"  # Looks like real key

# ✅ GOOD
API_KEY = "test-api-key-for-ci-only"
SECRET = "mock-secret-not-real"
```

---

## 4. Disk Space Management

### Problem
GitHub runners running out of disk space during builds.

### Solutions

#### Add disk space cleanup step to workflows

```yaml
jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - name: Free up disk space
        run: |
          sudo rm -rf /usr/share/dotnet
          sudo rm -rf /opt/ghc
          sudo rm -rf /usr/local/share/boost
          sudo rm -rf "$AGENT_TOOLSDIRECTORY"
          df -h

      - uses: actions/checkout@v4
      # ... rest of workflow
```

#### Use `--no-cache-dir` for pip installs

```yaml
- name: Install dependencies
  run: |
    pip install --upgrade pip
    pip install -r requirements.txt --no-cache-dir
    pip install -e . --no-cache-dir
```

#### Clean up after large operations

```yaml
- name: Build documentation
  run: |
    cd docs
    sphinx-build -b html . _build/html

- name: Clean up build artifacts
  run: |
    rm -rf docs/_build/.doctrees
    find . -type d -name '__pycache__' -exec rm -rf {} +
    find . -type f -name '*.pyc' -delete
```

---

## 5. Testing Best Practices

### Problem
Test failures due to missing test markers, timeouts, or environment issues.

### Solutions

#### Define pytest markers in `pyproject.toml`

```toml
[tool.pytest.ini_options]
markers = [
    "integration: Integration tests (deselect with '-m \"not integration\"')",
    "slow: Slow tests (deselect with '-m \"not slow\"')",
    "unit: Unit tests",
]
```

#### Use appropriate test commands

```yaml
# Unit tests only
- name: Run unit tests
  run: |
    pytest tests/ \
      -v \
      --tb=short \
      -m "not integration and not slow" \
      --cov=app \
      --cov-report=xml

# Integration tests with timeout
- name: Run integration tests
  run: |
    pytest tests/ \
      -v \
      -m "integration" \
      --timeout=60
```

#### Set up required services

```yaml
services:
  redis:
    image: redis:7-alpine
    ports:
      - 6379:6379
    options: >-
      --health-cmd "redis-cli ping"
      --health-interval 10s
      --health-timeout 5s
      --health-retries 5

  postgres:
    image: postgres:15-alpine
    env:
      POSTGRES_USER: test_user
      POSTGRES_PASSWORD: test_pass
      POSTGRES_DB: test_db
    ports:
      - 5432:5432
    options: >-
      --health-cmd pg_isready
      --health-interval 10s
```

---

## Checklist for New Workflows

- [ ] All dependencies declared in `pyproject.toml`
- [ ] Workflow triggers are correct (`pull_request`, `push`, etc.)
- [ ] YAML syntax is valid (no multiline string issues)
- [ ] Disk space cleanup added for large builds
- [ ] Test credentials use safe patterns (test-, mock-, dummy-)
- [ ] `.gitleaksignore` configured for test files
- [ ] Services (Redis, Postgres) properly configured
- [ ] Pytest markers defined and used correctly
- [ ] Timeouts set for long-running tests
- [ ] `--no-cache-dir` used for pip installs

---

## Quick Fix Commands

### Validate workflow locally
```bash
# Install act to run GitHub Actions locally
brew install act  # macOS
# or: sudo apt install act  # Linux

# Run workflow
act push -j test
```

### Check for missing dependencies
```bash
# Find all imports in Python files
grep -r "^import \|^from " app/ tests/ | sort -u

# Check if they're in pyproject.toml
cat pyproject.toml | grep -i package_name
```

### Test secret scanning
```bash
# Install gitleaks
brew install gitleaks

# Run scan locally
gitleaks detect --config .gitleaks.toml --verbose
```

### Check disk space usage
```bash
# In CI, add this to debug disk space issues
df -h
du -sh /usr/share/* | sort -h | tail -10
```

---

## Repository-Specific Fixes Applied

### krl-premium-backend
- ✅ Added `numpy>=1.24.0` to dependencies
- ✅ Added `geoalchemy2>=0.14.0` to dependencies
- ✅ Added `pytest-timeout>=2.2.0` to test dependencies

### krl-data-connectors
- ✅ Created `.gitleaksignore` for test files
- ✅ Updated `.gitleaks.toml` with comprehensive allowlist

### krl-model-zoo
- ✅ Added disk space cleanup to docs workflow
- ✅ Added `--no-cache-dir` to pip installs

### krl-frameworks
- ✅ Added disk space cleanup to all package-governance jobs
- ✅ Added `--no-cache-dir` to pip installs

---

## Monitoring and Maintenance

### Weekly Tasks
- Review failed workflow runs
- Update `.gitleaksignore` for new false positives
- Check for outdated dependencies

### Monthly Tasks
- Review disk space usage trends
- Update runner configurations if needed
- Audit security scan patterns

### Before Each Release
- Run all workflows manually
- Verify no ignored test failures
- Check security scan results

---

## Support

For issues or questions:
1. Check workflow run logs in GitHub Actions tab
2. Review this document for common solutions
3. Contact DevOps team for persistent issues

Last updated: 2026-01-12
