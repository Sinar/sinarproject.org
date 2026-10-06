# sinarproject.org

Buildout for the sinarproject.org website, based on the Sinar organization's
Plone packages.

- Plone 6.1.5 (see `extends` in `buildout.cfg`)
- Python 3.12
- `zc.buildout` + `mr.developer` (content packages are git-checked-out into `src/`)

## Requirements

- [uv](https://docs.astral.sh/uv/) for managing the build virtualenv
- `git` with access to the `Sinar` GitHub org (the default buildout clones over SSH)

## Setup

```sh
uv venv --python 3.12
source .venv/bin/activate

# build tool versions must match the Plone 6.1.5 versions.cfg pins
uv pip install zc.buildout==4.2.0 setuptools==81.0.0 wheel==0.47.0

buildout
```

## Development

```sh
bin/instance fg     # dev server at http://localhost:8080, login admin:admin
bin/zopepy          # interactive Plone shell
bin/update_locale   # regenerate i18n catalogs
```

## Deployment

```sh
bin/buildout -c deployment.cfg
```

Starts a ZEO server on port 8100 and runs the instance as a ZEO client on
port 8090.

## Layout

- `buildout.cfg` — default (development) buildout
- `deployment.cfg` — ZEO deployment buildout
- `src/` — package working copies managed by `mr.developer` (edit upstream in each package's git repo, not in place)
