# Github Actions for Releasing Python Packages to PYPI

1. Add `pre-release.yml` to `.github/workflows`. This workflow is triggered on tag creation and publishes to `tests.pypi`:

```yml
name: Publish to Test.PyPI
on:
  push:
    # when a new tag has been pushed to repo
    tags:
      - '*.*.*'

jobs:
  package_pre_release:
    name: Publish Package to Test PyPI
    runs-on: ubuntu-latest
    strategy:
      matrix:
        python-version: [3.11]

    steps:
      - name: Checkout Repository
        uses: actions/checkout@v4

      - name: Setup Python 3.11
        uses: actions/setup-python@v4
        with:
          python-version: ${{ matrix.python-version }}

      - name: Install Pipx & Poetry
        run: |
          python3 -m pip install --user pipx
          python3 -m pipx ensurepath
          pipx install poetry

      - name: Configure Test.PYPI as source to publish to
        run: |
          poetry config repositories.testpypi https://test.pypi.org/legacy/
          poetry config pypi-token.testpypi ${{ secrets.TEST_PYPI_PASSWORD }}

      - name: Publish package
        run: poetry publish --build -r testpypi
```

- Github Secret Used: `TEST_PYPI_PASSWORD`- API token generated on test.pypi.org

2. Add `release.yml` to `.github/workflows`. Triggered on Github Releases and publishes to `pypi`:

```yml
name: Publish to PyPI
on:
  release:
    types: [published]

jobs:
  package_release:
    name: Publish Package to PyPI
    runs-on: ubuntu-latest
    strategy:
      matrix:
        python-version: [3.11]

    steps:
      - name: Checkout Repository
        uses: actions/checkout@v4

      - name: Setup Python 3.11
        uses: actions/setup-python@v4
        with:
          python-version: ${{ matrix.python-version }}

      - name: Install Pipx & Poetry
        run: |
          python3 -m pip install --user pipx
          python3 -m pipx ensurepath
          pipx install poetry

      - name: Configure PYPI as source to publish to
        run: |
          poetry config pypi-token.pypi ${{ secrets.PYPI_PASSWORD }}

      - name: Publish package
        run: poetry publish --build

```

- Github Secret Used: `PYPI_PASSWORD` - API token generated on pypi.org

3. Update repo's readme with procedure for running a deployment

```md
1. In `pyproject.toml` *bump* the version number `*.*.*`

2. Create a [git tag](https://git-scm.com/book/en/v2/Git-Basics-Tagging) with the new version number `*.*.*` you specified in `pyproject.toml`.

3. Push the newly created tag `git push origin *.*.*` to the repository. This will trigger the `pre-release.yml` github workflow to publish our package to `test.pypi`. The pre-release can be seen [here](https://test.pypi.org/project/getdat/) for testing. Install with:
```bash
-> python3.11 -m pip install --index-url https://test.pypi.org/simple/ getdat --extra-index-url https://pypi.org/simple beautifulsoup4 requests click
```
- *Note*: `--extra-index-url` option is pulling dependencies from `pypi.org` and not `test.pypi.org` though our package is coming in from `test.pypi.org`. Make sure to add all dependencies from `[tool.poetry.dependencies]` in `pyproject.toml` (except python) before running this command.

4. *Create* a [release](https://www.toolsqa.com/git/github-releases/) on github. Make sure to select `Tags` from the toggle menu. Select the latest tag (highest version number). Name the release `v*.*.*`. Make sure the version number in `pyproject.toml` syncs up with the release version. *Click* `Publish release`. This will kick off our `release.yml` workflow to publish our package to `pypi`. The release can be seen [here and installed](https://pypi.org/project/getdat/) for production use. Install with:
```bash
-> pipx install getdat
```

```



