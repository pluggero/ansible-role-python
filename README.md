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

