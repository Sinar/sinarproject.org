---
name: plone-css-theming
description: Use when customizing CSS, styles, or the look of this Plone 6 site (sinarproject.org) — adding custom CSS/JS via plone.resource bundles, or working with a Diazo theme (theme/manifest.cfg, rules.xml, index.html, production-css). Reference implementation: plonetheme.kaerumy (https://github.com/kaerumy/plonetheme.kaerumy).
---

# Plone CSS / Theme Customization (sinarproject.org)

Plone 6.1.5 site. Static assets are served by `plone.resource`. Pick a
mechanism by scope:

| Scope | Mechanism |
|---|---|
| Extra CSS/JS files, default Barceloneta layout stays | resource bundle (Workflow A) |
| Completely different layout/structure | Diazo theme (Workflow B, plonetheme.kaerumy pattern) |

## Project facts

- `sinarproject.customizations` already publishes static files:
  `browser/configure.zcml` registers
  `<plone:static name="sinarproject.customizations" type="plone" directory="static" />`,
  so anything under
  `src/sinarproject.customizations/src/sinarproject/customizations/browser/static/`
  is served at `++plone++sinarproject.customizations/<path>`.
- Plone 6 does **not** compile or merge bundles at runtime: the
  `resources`/`compile`/`weight` keys of `IBundleRegistry` are deprecated
  and there is no Resources control panel. Custom bundles point directly
  at the final `.css`/`.js` file.
- Reference Diazo theme package: `plonetheme.kaerumy`
  (https://github.com/kaerumy/plonetheme.kaerumy — a Clean Blog/Bootstrap
  3 theme for Plone 6, in production on kaeru.my with the same Plone
  6.1.5). Its layout is the template to copy for Workflow B.
- Everything under `src/` is a git checkout via mr.developer
  (`auto-checkout = *`, `always-checkout = true`). Edit the git working
  copy, then re-run buildout. Never edit `eggs/` — buildout clobbers it.

## Workflow A — custom CSS/JS bundle (no theme change)

1. Create the file, e.g.
   `src/sinarproject.customizations/src/sinarproject/customizations/browser/static/css/custom.css`
   (the directory currently holds only `.gitkeep`).
2. Register the bundle in the GenericSetup profile — new file
   `src/sinarproject.customizations/src/sinarproject/customizations/profiles/default/registry/bundles.xml`
   (every `*.xml` in `profiles/default/` is applied automatically):

   ```xml
   <?xml version="1.0" encoding="utf-8"?>
   <registry>
     <records interface="plone.base.interfaces.IBundleRegistry"
              prefix="plone.bundles/sinar-custom">
       <value key="enabled">True</value>
       <value key="csscompilation">++plone++sinarproject.customizations/css/custom.css</value>
       <value key="load_async">False</value>
       <value key="load_defer">False</value>
     </records>
   </registry>
   ```

   For JavaScript use `jscompilation` instead of `csscompilation`.
3. Registry records are only written when the profile is applied, so
   **reinstall the add-on** after adding records:
   ZMI → Site → Add-ons → uninstall `sinarproject.customizations` →
   install it again.
4. Restart the instance (`bin/instance fg`) — `plone.resource` caches
   static directories.
5. Verify:
   - `http://localhost:8080/++plone++sinarproject.customizations/css/custom.css`
   - `<link data-bundle="sinar-custom">` in the rendered page head.
6. When removing a bundle later, add `profiles/uninstall/registry/bundles.xml`:

   ```xml
   <registry>
     <delete interface="plone.base.interfaces.IBundleRegistry"
             prefix="plone.bundles/sinar-custom" />
   </registry>
   ```

## Workflow B — Diazo theme (plonetheme.kaerumy pattern)

### B1. Adopt the plonetheme.kaerumy package

In the site `buildout.cfg`:

```ini
[sources]
plonetheme.kaerumy = git https://github.com/kaerumy/plonetheme.kaerumy

[instance]
eggs =
    ...
    plonetheme.kaerumy
```

`auto-checkout = *` checks it out to `src/plonetheme.kaerumy` for edits.
Run `.venv/bin/buildout`, then install the `plonetheme.kaerumy` add-on in
the ZMI. Its `profiles/default` then applies: the browser layer, the
Diazo theme activation (`theme.xml`), and the `kaerumy-core` bundle that
keeps the Plone 6 edit bar / member tools styled (barceloneta CSS).

### B2. Site-local theme inside sinarproject.customizations

Same layout under the customizations package:

```
src/sinarproject/customizations/theme/
    manifest.cfg
    rules.xml
    index.html          # static skeleton the theme renders into
    css/  js/  fonts/  img/
    preview.png
    tinymce-templates/  # optional TinyMCE content templates
```

Register it in the package `configure.zcml`:

```xml
<plone:static directory="theme" type="theme" name="sinarproject-customizations" />
```

(served at `++theme++sinarproject-customizations/...`) and enable it via
`profiles/default/theme.xml`:

```xml
<theme>
  <name>sinarproject-customizations</name>
  <enabled>true</enabled>
</theme>
```

### manifest.cfg

```ini
[theme]
title = ...
description = ...
doctype = <!DOCTYPE html>
rules = /++theme++sinarproject-customizations/rules.xml
prefix = /++theme++sinarproject-customizations
preview = preview.png
enabled-bundles = <plone.bundles names to keep active under this theme>

production-css = ++theme++sinarproject-customizations/css/main.css
development-css = ++theme++sinarproject-customizations/css/main.css
tinymce-content-css = /++theme++sinarproject-customizations/css/main.css
```

- `production-css` is injected on every page — this is the theme
  stylesheet, and it comes last in the cascade.
- `enabled-bundles` controls which `plone.bundles/*` registry bundles are
  rendered while this theme is active.
- `tinymce-content-css` styles the TinyMCE WYSIWYG preview.

### rules.xml — key patterns (see the reference theme for full examples)

```xml
<theme href="index.html"/>
<notheme if="$ajax_load" />
<notheme css:if-not-content="#visual-portal-wrapper" />

<!-- CSS ordering: Plone core bundle goes BEFORE the theme stylesheets
     so the theme's own CSS wins equivalent rules -->
<before css:theme="head link[rel='stylesheet']:first-of-type" css:content="head link[data-bundle='kaerumy-core']" />
<after css:theme-children="head" css:content="head link:not([data-bundle='kaerumy-core'])" />
```

- `<copy attributes="*" css:content="body" css:theme="body" />` keeps
  body id/classes (e.g. `userrole-anonymous`) for per-section styling.
- The Plone edit bar (`#edit-bar`) must be replaced only for
  `.userrole-authenticated` and dropped for anonymous users.
- The reference theme ships a small `theme/css/plone6.css` to compensate
  for Bootstrap 5 markup coming from Plone 6 vs the theme's Bootstrap 3
  skeleton; `rules.xml` also rewrites nav classes with `xsl:template`.
- Plone 6 status messages need `alert alert-*` classes added in
  `rules.xml` to pick up Bootstrap styling (see the reference).

### Changing theme CSS

1. Edit CSS in the git checkout
   (`src/plonetheme.kaerumy/src/plonetheme/kaerumy/theme/css/...` or the
   package's own `theme/css/`), commit.
2. Re-run `.venv/bin/buildout` when adding new files (the checkout is
   linked into the develop egg).
3. Restart the instance — `plone.resource` caches the directory.

## Gotchas

- Never edit files under `eggs/`; buildout overwrites them.
- Restart the instance after any static-file change (resource cache).
- A Diazo theme replaces the whole `<head>`/`<body>`; unstyled Plone
  widgets (edit bar, member tools) are a bundle problem — keep a
  barceloneta bundle active under the theme (the `kaerumy-core` pattern)
  and order stylesheets in `rules.xml`.
- TinyMCE content templates are plain HTML files listed in the
  `plone.templates` registry record
  (`profiles/default/registry/tinymce.xml`); in Plone 6 the TinyMCE
  template plugin is not part of the core bundle, so the record only
  takes effect where the plugin exists.
- New CSS that targets content-type-specific markup should go in the
  theme (or bundle) and be verified with the `userrole-anonymous` and
  `userrole-authenticated` body classes.
