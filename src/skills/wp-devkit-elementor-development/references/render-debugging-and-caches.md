# Render debugging, caches and damaged data

Contents: place the failure; read the data; cache layers in order; editor blank; preview blank; frontend only; damaged `_elementor_data`; atomic elements; Pro overriding free widgets; third-party filters; after an update.

Researched 2026-10-27. Sources: [wp elementor flush-css](https://developers.elementor.com/docs/cli/flush-css/), [wp elementor update db](https://developers.elementor.com/docs/cli/update-db/), [wp elementor system-info](https://developers.elementor.com/docs/cli/system-info/), [Page content](https://developers.elementor.com/docs/data-structure/page-content/), [Debugging in WordPress](https://developer.wordpress.org/advanced-administration/debug/debug-wordpress/), [Elementor source](https://github.com/elementor/elementor). Triage order and internal details are adapted from Mekko Digital elementor-skills (MIT, see `third-party-notices.md`), read from Elementor 4.1.x and Pro 4.0.4 source in 2026-06; option names and module paths can change, so confirm them on the installed version.

## Place the failure first

| Where it breaks | Look at |
|---|---|
| Editor canvas blank, panel works | Browser console for JS errors from the editor or an add-on; a classic `content_template()` that throws; an atomic value rejected by its schema |
| Preview iframe blank or error | PHP error log; the preview URL (`?elementor-preview=<id>`) opened in its own tab; the widget's `render()` |
| Preview fine, live page old or unstyled | Generated CSS, page cache, CDN, asset list saved with the page |
| Preview and live page both broken | `render()` bug, missing setting, unreadable `_elementor_data` |
| Only logged-out visitors see the problem | Page cache or CDN serving an old copy |
| Only with Pro active | A Pro module replacing or extending a free widget |

Record Elementor and Pro versions, active experiments (`wp elementor experiments` subcommands listed by `wp help elementor`), theme and add-ons before changing anything. `wp elementor system-info` prints the same report as the admin screen.

## Read the data

- `wp post meta get <id> _elementor_data | jq .` shows the tree. Look for a missing `widgetType`, `settings` of the wrong shape, or duplicate element IDs.
- `_elementor_edit_mode` must be `builder` for Elementor to render the post.
- `_elementor_css` holds metadata of the generated CSS file for the post; the CSS files themselves live under `wp-content/uploads/elementor/css/`.
- On versions with improved asset loading, the widget assets saved with the page live in another post meta (named `_elementor_page_assets` in the adapted notes); find it with `wp post meta list <id> --fields=meta_key`.
- A global element-cache key and TTL are stored as options on versions with element caching. Read their names in the installed `modules/element-cache/` source before changing them.

## Cache layers, in order

1. Save the page in the editor. This rewrites the post's CSS, asset list and element cache.
2. `wp elementor flush-css` removes generated CSS; `--regenerate` rebuilds files now instead of on the next visit; `--network` for multisite. The admin equivalent is Elementor > Tools > Regenerate CSS & Data.
3. Element cache: clear it from Tools or turn it off in Elementor settings while debugging. Re-enable after.
4. Object cache: `wp cache flush` when a persistent object cache is in use.
5. Page cache plugin, host cache and CDN: purge the affected URLs. CSS URLs carry a version query string; some CDNs ignore query strings.

Changing the CSS print method (external file or internal embedding) in Elementor's advanced settings forces a different path and helps tell a file-permission problem from a cache problem. Check that `wp-content/uploads/elementor/` is writable by the web server.

## Editor blank

- Console errors: note the failing element and the script that throws. Deactivate Elementor add-ons one at a time on staging.
- `elementor.elements.toJSON()` in the editor console shows whether the element loaded at all.
- In the network panel, the `elementor_ajax` request for `save_builder` or `get_document_config` shows server errors.
- Role and capability: the Role Manager (Pro) and `edit_with_elementor`-style checks can hide the editor for some users.
- Memory and PHP limits: `wp elementor system-info` shows the server environment. A fatal in `admin-ajax.php` appears only in the PHP log.

## Preview blank

- Enable `WP_DEBUG_LOG` on staging and reload the preview; fatals inside the iframe appear only in the log.
- In `render()`: unguarded array access, `foreach` over a non-array, functions that only exist on the frontend or in the admin.
- If markup exists but nothing shows, check CSS from a parent element (`display: none`, zero height) before blaming PHP.

## Frontend only

- Regenerate CSS (above), then purge page cache and CDN for that URL.
- A widget added by code after the page was last saved may be missing from the saved asset list; re-save the page.
- Experiments such as optimized markup, improved asset loading and element caching change output; toggle them on staging to isolate.
- Lazy-load, minify and combine settings in performance plugins often break widget JS; exclude Elementor frontend scripts and test.

## Damaged `_elementor_data`

- Back up the value before any repair: `wp post meta get <id> _elementor_data > damaged-<id>.json`.
- `jq . damaged-<id>.json` tells whether it parses. Typical causes: lost backslashes after a raw `update_post_meta` without `wp_slash()`, a search-replace that broke escapes, a byte-order mark or HTML entity inserted by a filter, double JSON encoding by an integration.
- Restore from the editor's History panel, from a revision that carries the meta, or from a backup. Check what revisions store on the installed version before relying on them.
- After a restore, flush CSS and test the editor, preview and frontend.

## Atomic elements

- `e-*` types are rendered by the atomic pipeline. A saved value that fails the props schema can be dropped silently; compare the element's props with the widget's schema.
- A missing Twig template next to the widget class can render nothing without an error.
- Atomic styles come from style definitions and land in the generated CSS; if the class is on the element but the rule is missing, regenerate CSS.
- Some atomic elements (the atomic form) need Pro and an experiment. Check `wp help elementor` for the experiments commands and list their state.

## Pro overriding free widgets

With Pro active, some free widgets are extended or replaced (posts, form, carousels; loop widgets override posts behavior). If the bug only happens with Pro, deactivate Pro on staging to confirm, then read the Pro module that owns the widget.

## Third-party filters

Search active plugins and the theme for filters that change output and can return empty strings:

```sh
rg -n -g '*.php' -e 'elementor/widget/render_content|elementor/frontend/the_content|elementor/widget/print_template|elementor/element/before_section_start|elementor/document/save/data|elementor/files/css' wp-content/plugins wp-content/themes
```

## After an Elementor update

- `wp elementor update db` runs pending data migrations; `--network` on multisite. Back up first and run on staging.
- Check the Deprecations page for removed hooks and methods used by custom code.
- Re-test custom widgets with the checks in `deploy-and-verify.md`; renamed internal Pro base classes are a common break.
