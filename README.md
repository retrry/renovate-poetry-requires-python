# poetry manager does not extract `project.requires-python`

Three projects, each with `requires-python = ">=3.12,<3.13"` and one dependency (`requests`):

- `poetry-project/`: `[project]` only, `poetry-core` backend, `poetry.lock`.
- `poetry-mixed-project/`: `[project]` plus `[tool.poetry.dependencies]`, `poetry-core` backend, `poetry.lock`.
- `hatch-project/`: `[project]` only, `hatchling` backend, no lock file.

Run:

    LOG_LEVEL=debug npx renovate@<version> --platform=local --dry-run=extract

Manager that handles each file, and whether a `python` dependency is extracted:

| Project | 41.75.0 | 44.186.7 |
|---|---|---|
| `poetry-project/` | pep621, yes | poetry, no |
| `poetry-mixed-project/` | poetry, no | poetry, no |
| `hatch-project/` | pep621, yes | poetry, no |

With `RENOVATE_ENABLED_MANAGERS='["pep621"]'`, 44.186.7 extracts it from all three.

Relocking, with Poetry 2.5.1 (after changing `requires-python` in `poetry-project/`):

    poetry update --lock --no-interaction python  # exit 1: "The following packages are not dependencies of this project: python"
    poetry check --lock                           # exit 1: "pyproject.toml changed significantly since poetry.lock was last generated"
    poetry lock --no-interaction                  # exit 0, locked versions unchanged
