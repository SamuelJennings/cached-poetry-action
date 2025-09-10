# Cached Poetry Action

Fast, reproducible Poetry setup for GitHub Actions with an in-project virtualenv and dependency caching.

## What it does

- Installs the requested Python and Poetry versions
- Configures Poetry to use an in-project virtualenv at `.venv`
- Restores and saves a cache of the `.venv` based on OS, Python, Poetry version, and `poetry.lock`
- Installs dependencies and the project via `poetry install`

## Why use it

- Speed: cache hits skip full dependency resolution and installation
- Isolation: `.venv` lives in the repo (no cross-job pollution)
- Simplicity: one step replaces multiple boilerplate steps in workflows

## Inputs

- `python-version` (string, default: `3.11`)
	- The Python version to set up via `actions/setup-python`
- `poetry-version` (string, default: `1.8.2`)
	- The Poetry version installed via `snok/install-poetry`

## Outputs

- None

## Usage

Basic usage (pin to a released tag, e.g., `v1`):

```yaml
steps:
	- uses: actions/checkout@v4

	- name: Set up Poetry env
		uses: SamuelJennings/cached-poetry-action@v1
		with:
			python-version: '3.11'
			poetry-version: '1.8.2'

	- name: Run tests
		run: poetry run pytest
```

Matrix example:

```yaml
strategy:
	matrix:
		python-version: ['3.10', '3.11', '3.12']

steps:
	- uses: actions/checkout@v4

	- name: Set up Poetry env
		uses: SamuelJennings/cached-poetry-action@v1
		with:
			python-version: ${{ matrix.python-version }}
			poetry-version: '1.8.2'

	- run: poetry run pytest -q
```

Docs build example:

```yaml
steps:
	- uses: actions/checkout@v4
	- uses: SamuelJennings/cached-poetry-action@v1
		with:
			python-version: '3.11'
	- run: poetry run sphinx-build -E -b html docs docs/_build
```

## Caching details

- Cache path: `.venv`
- Cache key includes: `${{ runner.os }}-py<version>-poetry<version>-<hash of poetry.lock>`
- On cache hit: dependency install step is skipped; project is still ensured installed
- On cache miss: a full `poetry install` runs and the cache is saved for next time

Notes:

- If you change Python or Poetry versions, the cache key changes automatically
- If you edit `poetry.lock`, the cache invalidates and dependencies are reinstalled

## Requirements

- A valid `pyproject.toml` and (ideally) `poetry.lock` at the repository root

## Versioning and pinning

- Use immutable, signed tags (recommended) such as `v1.0.0`
- Maintain a moving major tag like `v1` that points to the latest compatible v1.x release
- Example:
	- `git tag -a v1.0.0 -m "Initial release" && git push origin v1.0.0`
	- `git tag -fa v1 v1.0.0 -m "Update v1" && git push origin v1 --force`

## Troubleshooting

- Poetry not found: ensure the step ran and didn’t fail early
- Cache never hits: verify `poetry.lock` exists and is committed
- Wrong Python picked up: check the `python-version` input and `actions/setup-python` logs

## License

MIT
