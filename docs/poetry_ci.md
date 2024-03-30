---
category: Continuous Integration
---

# Python - Poetry CI Github Action

The below instructions assume that you are already using poetry and have `pyproject.toml`.

```yaml
# ./github/workflows/ci.yml
name: CI

on:
  push:
    # On merge into master
    branches:
      - master # or main
  # On pull requests
  pull_request:
  # manually trigger workflow
  workflow_dispatch:

jobs:
  run_test_suite:
    name: Run Test Suite, Style Guide, & Code Checks
    runs-on: ubuntu-latest

    steps:
      - name: Checkout Repository
        uses: actions/checkout@v4

      - name: Setup Python 3.12
        uses: actions/setup-python@v4
        with:
          python-version: 3.12

      - name: Install Pipx & Poetry
        run: |
          python3 -m pip install --user pipx
          python3 -m pipx ensurepath
          pipx install poetry

      - name: Install Dependencies & Test Dependencies
        run: poetry install --no-interaction --only=main,dev,tests

      - name: Style Guide Check & Code Check
        run: poetry run pre-commit run -a

      - name: Run Pytest & Code Coverage
        run: poetry run pytest --cov-config=.coveragerc --cov=. tests/
```

- Assumes [pytest](https://docs.pytest.org/en/7.4.x/) and [precommit](https://pre-commit.com/) has been installed by [poetry](https://python-poetry.org/)

```bash
-> poetry add pytest pytest-mock pytest-cov --group tests
-> poetry add pre-commit --group dev
```

---

Starter `.pre-commit-config.yaml` file. Place in root directory of project

```yaml
repos:
  - repo: https://github.com/pre-commit/pre-commit-hooks
    rev: v4.5.0
    hooks:
      - id: check-yaml
      - id: check-docstring-first
      - id: end-of-file-fixer
      - id: trailing-whitespace
  - repo: https://github.com/psf/black
    rev: 24.3.0
    hooks:
      - id: black
  - repo: https://github.com/pycqa/isort
    rev: 5.13.2
    hooks:
      - id: isort
  - repo: https://github.com/python-poetry/poetry
    rev: 1.8.2 # add poetry version here
    hooks:
      - id: poetry-check
      - id: poetry-lock
      - id: poetry-install
```

Starter `.coveragerc` file. Place in root directory of project

```.coveragerc
[run]
omit = tests/*

[report]
fail_under = 90
```

After adding these files locally initialize pre-commit and push to remote repo

```bash
-> pre-commit install
-> pre-commit run -a
# git commit and push to repo
```
