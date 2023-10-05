# sinarproject.org
Buildout for sinarproject.org website based on sinargo buildout and packages

## Installation and Buildout

### Ubuntu 22.04.2 LTS

Setup system dependencies:

```
sudo apt install build-essential python3-dev
```

### Setup venv and buildout environment Ubuntu 22.04.2

```
python3.8 -m venv venv
venv/bin/pip install setuptools==65.7.0 zc.buildout==3.0.1 wheel==0.38.4 \
plonecli
venv/bin/buildout bootstrap
```

### venv and buildout environment where base Python is newer than 3.8

Plone 5.2 currently works best on Python 3.8, where base version is not
Python 3.8, you will need to install custom version of Python 3.8 using
tools such as `pyenv`

Python module dependencies

```
apt install libssl-dev libsqlite3-dev libbz2-dev libncurses-dev \
libffi-dev libreadline-dev liblzma-dev
```

Setup venv using pyenv Python3.8 binary
```
git clone https://github.com/pyenv/pyenv.git ~/.pyenv
~/.pyenv/bin/install 3.8
~/.pyenv/versions/3.8.17/bin/python3.8 -m venv venv
```

### Buildout

Run buildout:

```
bin/buildout -vvv
```

Starting service in foreground debug mode:

```
bin/instance fg
```
