# sinarproject.org — Agent Quick Reference

Plone 6.1.5 CMS project. Buildout-driven, Python 3.12, GPLv2.

## Boot

```
.venv/bin/buildout    # run buildout (bin/buildout exists once generated)
bin/instance fg       # dev server, port 8080, login admin:admin
bin/zopepy            # interactive Plone shell
bin/update_locale     # regenerate i18n catalogs
```

## Initialize project

1. Ensure `uv` is installed
2. Check for `.venv` virtual environment for the supported Python version (3.12)
   - If `.venv` exists → activate it
   - If `.venv` doesn't exist → create it with `uv venv --python 3.12` then activate
3. Install build tool versions matching the Plone 6.1.5 `versions.cfg` pins
   (versions must match exactly, or buildout aborts with a `VersionConflict`):
   - `zc.buildout = 4.2.0`
   - `setuptools = 81.0.0`
   - `wheel = 0.47.0`
   `uv pip install zc.buildout==4.2.0 setuptools==81.0.0 wheel==0.47.0`
4. Run `.venv/bin/buildout` (generates `bin/buildout` for subsequent runs)
5. Remove the namespace `.pth` file from the venv (re-run this after any
   `uv venv` recreation or `zc.buildout` reinstall):
   `rm .venv/lib/python3.12/site-packages/zc.buildout-*-nspkg.pth`
   The `zc.buildout` wheel ships this file, and `site` executes it at every
   Python startup, pre-registering `sys.modules['zc']` with only the venv's
   own `zc` portion. That shadows the pkg_resources-style `zc` namespace
   packages in the buildout eggs (`zc.lockfile`, `zc.recipe.egg`, ...), so
   any generated script that imports one of them before `pkg_resources` is
   loaded breaks — most visibly `bin/zeo` with
   `ModuleNotFoundError: No module named 'zc.lockfile'`.

## Deployment mode

```
.venv/bin/buildout -c deployment.cfg
# builds the ZEO server (127.0.0.1:8100) + instance as ZEO client (8080)
# also pins plone.recipe.zeoserver = 4.0.1 and checks out
# sinar.article from the plone-6-update branch

bin/zeo start       # start the ZEO server (daemon)
bin/zeo status      # check ZEO server state
bin/instance fg     # start the instance (ZEO client) on port 8080
```

The instance uses `shared-blob = on`: on a single machine it shares
`var/blobstorage` with the ZEO server instead of keeping a separate
blob cache (a non-shared client would force the `zeocache` layout and
crash on the existing `bushy` blob directory).

## Source layout

All packages are namespace packages under `src/`, managed by `mr.developer` from git:

| Package | Content / Purpose |
|---|---|
| `sinarproject.customizations` | Main customization glue (browserlayer, registry, views, z3c.jbot overrides) |
| `sinar.activity` | Activity / ProjectActivity content types |
| `sinar.article` | Article content type |
| `sinar.indicators` | M&E indicators |
| `sinar.miscbehavior` | Extra behaviors for content types |
| `sinar.opportunities` | Opportunity content type |
| `sinar.organization` | Organization content type |
| `sinar.project` | Folderish Project content type |
| `sinar.resource` | Resource content type |
| `collective.vocabularies.iso` | ISO country/currency vocabularies |

## Testing changes

After making changes, verify nothing breaks:

1. Run per-package tests: `cd src/<pkg> && tox`
2. Start the dev server and watch for errors: `bin/instance fg`

The instance will log any import errors, missing dependencies, or configuration issues on startup. Check the console output for failures before proceeding.

## Gotchas

- **The `zc.buildout` wheel breaks ZEO if its `nspkg.pth` stays in the venv.**
  `.venv/lib/python3.12/site-packages/zc.buildout-*-nspkg.pth` must be removed
  (see step 5 in "Initialize project"). Without it, `bin/zeo` dies with
  `ModuleNotFoundError: No module named 'zc.lockfile'`: the `.pth` pre-registers
  `sys.modules['zc']` at interpreter startup, hiding the `zc.lockfile` egg's
  pkg_resources namespace. Egg scripts like `bin/zeo` or `bin/zopepy` survive
  because they import `pkg_resources` first (which repairs the namespace);
  `parts/zeo/bin/runzeo` does not, so only the ZEO server is affected.
- **`mr.developer` auto-checks out every package** (`auto-checkout = *`, `always-checkout = true`). Never edit checked-out packages directly — make changes in the git working copy and re-run buildout.
- **`eea.facetednavigation`** appears in `.installed.cfg` and `buildout.cfg` — it's an extra dependency beyond the base Plone install.
- **`sinar.article`** is checked out from the `plone-6-update` branch in `deployment.cfg`; the default buildout uses `main` for all packages (`collective.vocabularies.iso` uses its default `master` branch).
- **Stale classifiers:** some packages (e.g. `sinar.project` declares Plone 5.2 / Python 2.7 in setup.py) have outdated metadata, but their `main` branches work on Plone 6.1.5 — judge by running the build/tests, not by reading classifiers.
- **Build tool pins:** the venv must run the exact `zc.buildout`/`setuptools`/`wheel` versions pinned by the Plone release `versions.cfg`, or buildout aborts with a `VersionConflict`.
- Lint/format: `isort`, `flake8`, `black`. Run per-package: `cd src/<pkg> && tox -e lint` or `tox -e black-check`.

## Commit messages

- Author: check current git user via `git config user.name`
- Follow [Conventional Commits](https://github.com/conventional-commits/conventionalcommits.org/blob/master/content/v1.0.0/index.md) with multi-paragraph bodies (summary, blank line, explanation)
- Do **not** include AI attribution (e.g. `Assisted-by:`) lines in commit messages
