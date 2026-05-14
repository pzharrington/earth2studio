# Earth2Studio – Agents Guide

See `CLAUDE.md` for the authoritative project rules, coding conventions, and skill references.

## Cursor Cloud specific instructions

### Environment Overview

Earth2Studio is a Python-based AI inference toolkit for weather/climate. The dev environment
uses Python 3.12 (system) with a virtualenv at `/workspace/.venv`.

### Key Gotchas

1. **Custom uv index unreachable**: The `pyproject.toml` defines an `ort-cuda-13-nightly` index
   (`aiinfra.pkgs.visualstudio.com`) that is blocked by network egress restrictions. All `uv`
   commands (including `uv sync`, `uv venv --seed`, `uv pip install`) **must** be run from
   `/tmp` (not `/workspace`) so that uv does not read the project's `[[tool.uv.index]]` config.
   Use: `cd /tmp && uv pip install --python /workspace/.venv/bin/python <packages>`.

2. **PATH setup**: After installing uv via pip, add it to PATH:
   `export PATH="/home/ubuntu/.local/bin:$PATH"`

3. **No GPU available**: This VM does not have a CUDA GPU. Tests parametrized with `cuda:0`
   will fail. Use `-k "cpu"` to filter CPU-only tests. The `warp-lang` package warns about
   missing NVIDIA driver but works in CPU mode.

4. **Git-pinned dependencies**: Several packages must match the lockfile pins to avoid
   compatibility issues:
   - `warp-lang==1.11.1` (not latest — newer versions break `nvidia-physicsnemo`)
   - `nvidia-physicsnemo` from git rev `369a15782b9922d7694d49a00bfaf8960b9be429`
   - `torch-harmonics` from git rev `a632ca748a12bd9f74dbc1e00653317810991f74`
   - `earth2grid` from git rev `11dcf1b0787a7eb6a8497a3a5a5e1fdcc31232d3`

5. **pre-commit hooks**: Full `pre-commit install --install-hooks` fails because markdownlint
   needs Node.js which can't be downloaded. Run linters directly instead:
   - `.venv/bin/ruff check earth2studio/`
   - `.venv/bin/black --check earth2studio/`
   - `.venv/bin/mypy earth2studio/`
   - `.venv/bin/python test/_license/header_check.py`

### Running Tests

```bash
# Activate the venv
source /workspace/.venv/bin/activate

# Core tests (CPU only)
pytest test/io test/run test/utils/test_coords.py test/utils/test_imports.py -k "cpu" --timeout=120

# Perturbation tests
pytest test/perturbation -k "cpu" --timeout=120

# Statistics tests
pytest test/statistics -k "cpu" --timeout=120

# License check
python test/_license/header_check.py
```

### Running Linting

```bash
source /workspace/.venv/bin/activate
ruff check earth2studio/
black --check earth2studio/
mypy earth2studio/
```

### Services

This is a library package, not a web application. There is no dev server to start.
The `serve` module (FastAPI + Redis) is optional and requires Redis + GPU for meaningful use.
For library development, just import `earth2studio` and use it directly.
