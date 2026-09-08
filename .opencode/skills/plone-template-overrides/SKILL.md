---
name: plone-template-overrides
description: Use when overriding or changing Plone page templates (.pt) on this site — z3c.jbot drop-in overrides in sinarproject.customizations/browser/overrides/ (full_view, leadimage, viewlet templates, listing templates), or finding which package provides a template.
---

# Plone Template Overrides (z3c.jbot)

Override any `.pt` template without forking the providing package: put a
drop-in file whose **filename is the template's dotted path** into

`src/sinarproject.customizations/src/sinarproject/customizations/browser/overrides/`

The directory is registered in `browser/configure.zcml`:

```xml
<browser:jbot
    directory="overrides"
    layer="sinarproject.customizations.interfaces.ISinarprojectCustomizationsLayer"
    />
```

New *views* (not overrides) belong in `sinarproject/customizations/views/`
(see `views/front_page.py` + `front_page.pt`).

## Critical: the jbot include must be the full one

`browser/configure.zcml` currently reads:

```xml
<include package="z3c.jbot" file="meta.zcml" />
```

This registers the `jbot` metadirective but does **not** install the
runtime patches. In z3c.jbot 3.x the overrides only take effect because
`z3c.jbot.patches` monkey-patches `PageTemplateFile.__get__`, five.pt's
`ViewPageTemplateFile.__get__` and CMF `FSObject.__of__` — and that
module is only imported by the package's full `configure.zcml`. With the
`meta.zcml`-only include, overrides are registered but **silently never
applied**.

Use the full include (this is what the working reference theme
`plonetheme.kaerumy` — https://github.com/kaerumy/plonetheme.kaerumy —
does in its `browser/configure.zcml`):

```xml
<include package="z3c.jbot" />
```

If an override you added doesn't show up, fix this line first.

## Naming convention

The override filename is the template's full **dotted name**: package
dots + subpath with `/` replaced by `.`. z3c.jbot matches on the exact
dotted name (`TemplateManager.registerTemplate`), no globs or partials.

| Original template (in the egg) | Override file in `browser/overrides/` |
|---|---|
| `plone/app/contenttypes/browser/templates/full_view_item.pt` | `plone.app.contenttypes.browser.templates.full_view_item.pt` |
| `plone/app/contenttypes/behaviors/leadimage.pt` | `plone.app.contenttypes.behaviors.leadimage.pt` |

Working examples of both (including full replacement markup for
`full_view_item.pt`) exist in the reference theme's `browser/overrides/`.

### How to find the dotted name of a template

1. Locate the file under `eggs/` and read where it's wired in ZCML:

   ```
   find eggs/<pkg>/ -name "<name>.pt"
   grep -rn "template=" eggs/<pkg>/<pkg>/.../configure.zcml
   ```

2. Or print the runtime filename from `bin/zopepy` and derive the name:

   ```
   bin/zopepy
   >>> from plone.app.contenttypes.browser.full_view import FullViewItem
   >>> print(FullViewItem.template.filename)
   /home/kaeru/src/plone/sinarproject.org/eggs/plone.app.contenttypes-4.0.10-py3.12.egg/plone/app/contenttypes/browser/templates/full_view_item.pt
   ```

   Strip everything up to and including the egg directory, then replace
   `/` with `.` → `plone.app.contenttypes.browser.templates.full_view_item.pt`.

   For viewlets use the viewlet class, e.g.
   `plone.app.contenttypes.behaviors.viewlets.LeadImageViewlet`.

## Workflow

1. Copy the original `.pt` from the egg as a starting point into
   `browser/overrides/` with the dotted filename.
2. Ensure the full `<include package="z3c.jbot" />` (above).
3. Run the package tests: `cd src/sinarproject.customizations && tox`.
4. Restart the instance (`bin/instance fg`) — the template manager reads
   the overrides directory at ZCML load time.
5. Verify the affected page in the browser (both anonymous and logged-in,
   since the browser layer is active for both).

## Activation requirements

- Overrides apply only to requests providing
  `ISinarprojectCustomizationsLayer` (registered in
  `profiles/default/browserlayer.xml`), i.e. the site must have the
  `sinarproject.customizations` add-on installed.
- Restart the instance after adding/removing override files.

## Notes

- `z3c.jbot = 3.1` is pinned in the site `buildout.cfg [versions]` and
  must stay: Plone 6.1.5 uses PEP 420 implicit `z3c` namespace packages;
  z3c.jbot 2.x is pkg_resources-style and would take over the namespace
  and break `z3c.form`/`z3c.pt` imports.
- jbot can also override **static resource files** published through
  `plone.resource`, using the same dotted-name filenames.
- When overriding a template that uses `metal` macros from
  `context/main_template/macros/master` (Plone 6 Barceloneta), keep the
  same macro usage unless the layout itself is being replaced.
- After an override changes visible markup, check the Diazo theme rules
  (`theme/rules.xml`) still select the expected selectors.
