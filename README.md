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

Optional list of Python tools/packages to install in isolated virtual environments. Each tool is installed in its own venv and made available via wrapper scripts in `~/.local/bin`. Tools are automatically added to the system PATH for all users.

Each tool entry supports:

- `package`: The Python package name (required)
- `version`: Version to install, use "latest" for the newest version (required)
- `executables`: List of command-line executables to create wrapper scripts for (required)
- `venv_name`: Name of the virtual environment to use (optional, defaults to package name)
  - Multiple tools can share the same venv by using the same `venv_name`
  - All packages with the same `venv_name` are installed together in a single pip command for proper dependency resolution
- `tool_dependencies`: OS-specific system packages required by the tool (optional)

Example:

```yaml
python_tools:
  - package: "frida-tools"
    version: "latest"
    executables: ["frida", "frida-ps", "frida-trace"]
    tool_dependencies:
      Alpine: ["npm"]
  - package: "black"
    version: "22.3.0"
    executables: ["black"]
```

Example with multiple packages in one venv:

```yaml
python_tools:
  - package: "objection"
    version: "1.11.0"
    venv_name: "mobile-security"
    executables: ["objection"]
  - package: "frida-tools"
    version: "13.7.1"
    venv_name: "mobile-security"
    executables: ["frida", "frida-ps", "frida-trace"]
```

In this example, both `objection==1.11.0` and `frida-tools==13.7.1` will be installed in the same virtual environment named "mobile-security" using a single `pip install objection==1.11.0 frida-tools==13.7.1` command, ensuring proper dependency resolution.

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
