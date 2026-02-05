# Ansible Role: Python

[![CI](https://github.com/pluggero/ansible-role-python/actions/workflows/ci.yml/badge.svg)](https://github.com/pluggero/ansible-role-python/actions/workflows/ci.yml) [![Ansible Galaxy downloads](https://img.shields.io/ansible/role/d/pluggero/python?label=Galaxy%20downloads&logo=ansible&color=%23096598)](https://galaxy.ansible.com/ui/standalone/roles/pluggero/python)

An Ansible Role that installs a basic configuration of Python.

## Requirements

None.

## Role Variables

Available variables are listed below, along with default values (see `defaults/main.yml`):

```yaml
python_version: "x.x.x"
```

The version of Python to install can be defined in the variable `python_version`.

```yaml
python_install_method: "dynamic"
```

The method used to install Python can be defined in the variable `python_install_method`.
The following methods are available:

- `source`: Installs Python from source
- `package`: Installs Python from the package manager of the distribution
  - **NOTE**: This method installs the latest version available in the package manager and not the version defined in `python_version`.
- `dynamic`: Installs Python from package manager if available in the correct version, otherwise installs from source

```yaml
python_tools: []
```

Optional list of Python tools/packages to install in isolated virtual environments. Each tool is installed in its own venv and made available via wrapper scripts.

```yaml
python_dev_auto_activate: true
```

When enabled (default), automatically activates the development virtual environment for interactive Python sessions. This means when you run `python3` or `python` without arguments or with the `-i` flag, all `importable` packages are automatically available without needing to manually source the activation script. Set to `false` to require manual activation via `source ~/.local/bin/activate-python-dev`.

Each tool entry supports:

- `package`: The Python package name (required)
- `version`: Version to install, use "latest" for the newest version (required)
- `executables`: List of command-line executables to create wrapper scripts for (optional, can be omitted for library-only packages)
- `install_scope`: Installation scope - "user" or "system" (optional, defaults to "user")
  - **user**: Tools installed in `~/.local/venvs`, wrappers in `~/.local/bin` (per-user)
  - **system**: Tools installed in `/opt/python-tools/venvs`, wrappers in `/usr/local/bin` (available to all users, requires sudo)
- `venv_name`: Name of the virtual environment to use (optional, defaults to package name)
  - Multiple tools can share the same venv by using the same `venv_name`
  - All packages with the same `venv_name` are installed together in a single pip command for proper dependency resolution
- `importable`: Make package importable in Python REPL/scripts (optional, defaults to false)
  - When `true`, package is additionally installed in a shared development virtual environment
  - With `python_dev_auto_activate: true` (default), packages are automatically available in interactive Python sessions
  - Manual activation available via: `source ~/.local/bin/activate-python-dev` (user) or `source /usr/local/bin/activate-python-dev` (system)
  - Compatible with `executables` - package can be both a CLI tool and importable library
- `tool_dependencies`: OS-specific system packages required by the tool (optional)

**Example - Basic usage:**

```yaml
python_tools:
  - package: "black"
    version: "22.3.0"
    executables: ["black"]
  - package: "frida-tools"
    version: "latest"
    executables: ["frida", "frida-ps", "frida-trace"]
    tool_dependencies:
      Alpine: ["npm"]
```

**Example - System-wide installation:**

```yaml
python_tools:
  # User-scoped tool (default)
  - package: "black"
    version: "latest"
    install_scope: "user"
    executables: ["black"]

  # System-wide tool (available to all users)
  - package: "ansible-lint"
    version: "latest"
    install_scope: "system"
    executables: ["ansible-lint"]
```

**Example - Multiple packages in one venv:**

```yaml
python_tools:
  - package: "objection"
    version: "1.11.0"
    venv_name: "mobile-security"
    install_scope: "user"
    executables: ["objection"]
  - package: "frida-tools"
    version: "13.7.1"
    venv_name: "mobile-security"
    install_scope: "user"
    executables: ["frida", "frida-ps", "frida-trace"]
```

In this example, both `objection==1.11.0` and `frida-tools==13.7.1` will be installed in the same virtual environment named "mobile-security" using a single `pip install objection==1.11.0 frida-tools==13.7.1` command, ensuring proper dependency resolution.

**Example - Importable packages for Python development:**

```yaml
python_tools:
  # CLI tool that's also importable
  - package: "black"
    version: "22.3.0"
    install_scope: "user"
    executables: ["black"]
    importable: true

  # Library-only package (no CLI executables)
  - package: "requests"
    version: "2.31.0"
    install_scope: "user"
    importable: true

  - package: "pandas"
    version: "latest"
    install_scope: "system"
    importable: true
```

After running the playbook, packages are automatically available in interactive Python sessions (when `python_dev_auto_activate: true`, which is the default):

```bash
# Just run python - packages are automatically available!
python3
>>> import black
>>> import requests
>>> import pandas

# Also works with python command
python
>>> import requests

# Works when running Python scripts that need imports
python3 my_script.py  # imports work if script uses the packages
```

**Manual activation** (when `python_dev_auto_activate: false` or for shell integration):

```bash
# For user-scoped packages
source ~/.local/bin/activate-python-dev

# For system-scoped packages
source /usr/local/bin/activate-python-dev
```

**How auto-activation works:**
- When `python_dev_auto_activate: true`, wrapper scripts are created at `~/.local/bin/python3` (user) or `/usr/local/bin/python3` (system)
- The wrappers check for **both** user and system dev venvs and combine them using PYTHONPATH
- If both scopes have importable packages, all packages from both user (`~/.local/venvs/python-dev`) and system (`/opt/python-tools/venvs/python-dev`) are available
- Automatic activation occurs for interactive sessions (no arguments or `-i` flag)
- For non-interactive use (running scripts), the wrapper uses system Python without modifications
- `~/.local/bin` is prepended to PATH (via `/etc/profile.d/python_path.sh`), so user wrappers take precedence over system wrappers

**Note:** System-scoped tools require the playbook to run with `become: true` (sudo privileges).

## Dependencies

None.

## Example Playbook

```yaml
- hosts: all
  roles:
    - pluggero.python
```

## License

MIT / BSD

## Author Information

This role was created in 2025 by Robin Plugge.
