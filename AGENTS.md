# sinarproject.org — Agent Quick Reference

Plone 6.0.15 CMS project. Buildout-driven, Python 3.12, GPLv2.

## Boot

```
bin/buildout          # bootstrap + install
bin/instance fg       # dev server, port 8080, login admin:admin
bin/zopepy            # interactive Plone shell
bin/update_locale     # regenerate i18n catalogs
```

## Initialize project

1. Ensure `uv` is installed
2. Check for `.venv` virtual environment for the supported Python version (3.12)
   - If `.venv` exists → activate it
   - If `.venv` doesn't exist → create it with `uv venv --python 3.12` then activate
3. Install recommended build tool versions from Plone 6.0.15:
   - `zc.buildout = 4.1.4`
   - `wheel = 0.45.1`
   - `setuptools = 75.8.2`
4. Run `bin/buildout` to bootstrap the project

## Deployment mode

```
bin/buildout -c deployment.cfg
# starts ZEO server (8100) + instance as ZEO client (8090)
```

## Source layout

All packages are namespace packages under `src/`, managed by `mr.developer` from git:

| Package | Content / Purpose |
|---|---|
| `sinarproject.customizations` | Main customization glue (browserlayer, registry, views, z3c.jbot overrides) |
| `sinar.activity` | Activity / ProjectActivity content types |
| `sinar.article` | Article content type |
| `sinar.citation` | Citation content type |
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

- **`mr.developer` auto-checks out every package** (`auto-checkout = *`, `always-checkout = true`). Never edit checked-out packages directly — make changes in the git working copy and re-run buildout.
- **`eea.facetednavigation`** appears in `.installed.cfg` and `buildout.cfg` — it's an extra dependency beyond the base Plone install.
- **`sinar.article` and `sinar.organization`** are on the `plone-6-update` branch. Other packages target `main`.
- **`sinar.miscbehavior`, `sinar.organization`, `sinar.project`, `collective.vocabularies.iso`** declare compatibility with Plone 4.3/5.x — verify before assuming Plone 6-only behavior.
- Lint/format: `isort`, `flake8`, `black`. Run per-package: `cd src/<pkg> && tox -e lint` or `tox -e black-check`.

## Commit messages

- Author: check current git user via `git config user.name`
- Follow [Conventional Commits](https://github.com/conventional-commits/conventionalcommits.org/blob/master/content/v1.0.0/index.md) with multi-paragraph bodies (summary, blank line, explanation, blank line, attribution)
- If AI-assisted, add the attribution line at the end using the format `Assisted-by: <AGENT_NAME>:<MODEL_VERSION>` (see [Linux Kernel AI Attribution](https://github.com/torvalds/linux/blob/master/Documentation/process/coding-assistants.rst))
