# Python Installation

Official project: <https://www.python.org/>

On Ubuntu/Debian, the practical setup is to install the distro packages for Python, `venv`, and `pip`. If you want the `python` command in addition to `python3`, install `python-is-python3`.

## Install

```bash
sudo apt update
sudo apt install -y python3 python3-venv python3-pip python-is-python3
```

## Minimal baseline

Create a place for virtual environments:

```bash
mkdir -p ~/.venvs
```

Create and activate a test environment:

```bash
python -m venv ~/.venvs/test
source ~/.venvs/test/bin/activate
python -m pip --version
deactivate
```

## Verify

```bash
python --version
python3 --version
python -m pip --version
```
