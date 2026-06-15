# `run_pep723`

`run_pep723` is a wrapper script designed to run tools or execute Python scripts in a temporary, clean virtual environment managed by [`uv`](https://github.com/astral-sh/uv).

It automatically detects Python scripts containing [PEP 723](https://peps.python.org/pep-0723/) metadata (inline script metadata), exports their declared dependencies, installs them into the temporary virtual environment, runs the target script or tool, and then automatically cleans up the virtual environment upon completion.

## Features

- **[PEP 723](https://peps.python.org/pep-0723/) Inline Metadata Support**: Automatically scans all `.py` files in the current working directory, extracts dependencies from any files using PEP 723 format (e.g., `# /// script`), and installs/syncs them.
- **On-the-Fly Environment Creation**: Dynamically creates a virtual environment (`.venv`) using `uv` if one doesn't exist. If `.venv` already exists, the script exits with an error unless `--with-existing-venv` is specified.
- **Custom Tool Execution**: Run any specific script or CLI tool within the environment, and optionally pre-install other tools using the `--with` flag (e.g., `ruff`, `black`).
- **Safety First**: Verifies that the workspace does not contain a `pyproject.toml` file to prevent modifying or messing with existing project environments.
- **Automatic Cleanup**: If `.venv` was created by this script, it automatically cleans up and deletes it on exit. If an existing `.venv` was used via `--with-existing-venv`, it is preserved.
- **Quiet Execution by Default**: Suppresses diagnostic logs and `uv` setup/progress output so that only the output of the executed script or tool is displayed on success. In case of failure, `uv` logs are printed for debugging. Use `--with-verbose-uv` to force verbose output.

## Prerequisites

- [uv](https://github.com/astral-sh/uv) (fast Python package installer and resolver) must be installed and available in your `PATH`.

## Usage

```bash
run_pep723 [options] <script_or_tool> [args...]
```

### Options

- `-h, --help`: Show the help message and exit.
- `--with <tool>`: Install and run the specified tool (can be specified multiple times).
- `--with-existing-venv`: Use the existing `.venv` if it exists, and do not delete it on exit.
- `--with-verbose-uv`: Show verbose output from `uv` commands and the script.

### Behavior

1. **Project Check**: Confirms that there is no `pyproject.toml` in the current directory. If there is, it exits with an error to avoid conflicting with existing project settings.
2. **Existing Environment Check**: Checks if a virtual environment `.venv` already exists. If it does, exits with an error unless `--with-existing-venv` is specified.
3. **Virtual Environment Setup**: Runs `uv venv` to create a virtual environment if `.venv` doesn't exist.
4. **Dependency Scanning & Syncing**: Scans all `*.py` files in the current directory for PEP 723 metadata (`# /// script`). If found, runs `uv export --script <file>` to extract dependencies and pipes them to `uv pip sync` to populate the environment.
5. **Execution**: Uses `uv run` to execute the specified script or tool with any passed arguments.
6. **Clean up**: Deletes the virtual environment `.venv` directory upon exit if it was created by this script.

## Examples

### Running a script with [PEP 723](https://peps.python.org/pep-0723/) metadata
If you have a script `script.py` with inline metadata:
```python
# /// script
# requires-python = ">=3.11"
# dependencies = [
#     "requests",
# ]
# ///
import requests
print("Hello World!")
```
Run it like this:
```bash
run_pep723 script.py arg1 arg2
```

### Running tools without installing them globally
You can run tools like `ruff` or `black` on your files using the `--with` flag:
```bash
run_pep723 --with ruff ruff check script.py
```

Using multiple tools together:
```bash
run_pep723 --with black --with ruff black --check script.py
```

### Running with an existing virtual environment
To use an existing `.venv` and preserve it after running:
```bash
run_pep723 --with-existing-venv script.py arg1 arg2
```

## License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.
