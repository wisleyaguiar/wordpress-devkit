---
name: wp-devkit-elementor-development
description: "Build, debug or review Elementor and Elementor Pro sites. Use for safe page and template edits in _elementor_data, custom widgets, dynamic tags, Pro form actions and Theme Builder locations, blank or stale renders and CSS caches, free vs Pro checks and proving an extension works on a staging site."
---

# Elementor pages and extensions

Read `references/engineering-contract.md` first. Review requests stay read-only: assess, explain and recommend without edits, `_elementor_data` or option writes, `wp elementor flush-css|replace-urls|update db|library import|kit import`, cache purges, plugin activation, or running untrusted code. Use the implementation path only when the request authorizes changes.

## Applicability

Confirm that Elementor is active (`wp plugin list --fields=name,status,version` shows `elementor`), that posts are built with it (`_elementor_edit_mode` = `builder`), or that the code hooks Elementor (`elementor/widgets/register`, `\Elementor\Widget_Base`). If not, report `Not used in this project` and stop. Pro-only parts (forms, Theme Builder, popups, loop, Pro dynamic tags) apply only when `elementor-pro` is active; say which parts apply.

## Inputs to establish

Elementor and Elementor Pro versions; active experiments (the atomic editor and its `e-*` elements, container, element caching, asset loading); active theme and whether Theme Builder replaces its header, footer or templates; third-party Elementor add-ons; target environment (production, staging, local) and backup status; post or template IDs involved; cache layers (page cache, CDN, object cache); multisite; translation plugins that copy Elementor data per language; ACF or Secure Custom Fields when dynamic tags read fields.

Inspect project evidence before asking. State material assumptions. Version-sensitive details in the references were checked against the sources they cite; confirm them on the installed version before relying on them.

## Route the task

| Symptom or goal | Read |
|---|---|
| Change text, links or images on a page or template; copy or apply a template; global widgets, kits, global colors and fonts; build a page from a design | `references/page-data-and-content-edits.md` |
| Create or extend a widget, its controls, `render()` or `content_template()`, skins, an atomic `e-*` widget, widget assets | `references/custom-widgets.md` |
| New dynamic tag, Pro form "Actions After Submit" action, Theme Builder location | `references/dynamic-tags-forms-and-theme-builder.md` |
| Blank editor, preview or frontend; old styles; missing widget JS; broken after an update; damaged `_elementor_data` | `references/render-debugging-and-caches.md` |
| Is a widget or feature free or Pro; does it exist on this install | `references/free-vs-pro-matrix.md` |
| Install scaffolded code on staging and prove it registered and renders; WP-CLI, Docker or MCP transport | `references/deploy-and-verify.md` |

Hand off: ACF or Secure Custom Fields field design and values behind Pro ACF dynamic tags to `wp-devkit-acf-and-content-modeling`; WooCommerce Builder widgets with store behavior to `wp-devkit-woocommerce-dev`; theme templates, child themes and Hello-based themes to `wp-devkit-theme-development`; plugin lifecycle and packaging of an add-on to `wp-devkit-plugin-development`; XSS or authorization defects with a reachable path to `wp-devkit-security-review`; measured whole-site slowness to `wp-devkit-performance-review`; bulk content operations to `wp-devkit-wpcli-and-ops`; layout and visual design to `wp-devkit-ui-design`.

## Code Review Workflow

1. Name the surface: page content, templates and kit, custom extension code, or render and cache behavior. Check versions and experiments first; the classic and atomic editors use different data shapes and code paths.
2. For extensions, trace registration hook, class base, control definitions, `render()` and `content_template()` output, assets and dependencies. Check escaping at each sink and that `render()` reads `get_settings_for_display()`.
3. For content, read `_elementor_data` as JSON for the affected post only. Identify shared sources (global widgets, kit, Theme Builder templates, translations) before attributing a change to one page.
4. For render problems, place the failure (editor, preview, frontend) before touching caches. Cache regeneration that hides a defect is not a fix.
5. For Pro features, confirm Pro is active and that the feature is registered on this install; do not rely on the static matrix alone.
6. Classify by reachable impact. Escaping gaps in a widget only an administrator can configure still need the actor and path named.

A finding needs a reachable path and impact. A style preference is INFO. Missing evidence (no staging copy, unknown Pro version, unknown experiments) is a valid outcome: say what is missing and how to get it.

## Implementation workflow

1. Define the expected behavior, the smallest affected boundary and the target environment. Work on staging or local; production content edits need explicit authorization and a backup.
2. For content, back up the affected meta, edit one post through the editor or a scoped script, and keep the JSON valid. Never run broad `wp search-replace` on `wp_postmeta` for page text.
3. For code, use the documented registration hooks and base classes for the installed version; escape output; keep secrets out of exported templates.
4. Prove registration and render with the checks in `references/deploy-and-verify.md`, then regenerate CSS with `wp elementor flush-css` when styles changed.
5. Run the acceptance checks; report unavailable checks as unexecuted. Cache purges on shared hosting and production deploys need their own scope.

## Search Patterns for Quick Detection

Read-only leads from the project root; matches are candidates, not findings. Exit 0 means a match, 1 none, 2 an error.

```sh
rg -n -g '*.php' -g '!vendor' -g '!node_modules' -e 'elementor/widgets/register|elementor/widgets/widgets_registered|Widget_Base|Atomic_Widget_Base|elementor/elements/categories_registered' .
rg -n -g '*.php' -g '!vendor' -g '!node_modules' -e 'elementor/dynamic_tags/register|DynamicTags\\(Data_)?Tag|elementor_pro/forms/actions/register|Action_Base|elementor/theme/register_locations|elementor_theme_do_location' .
rg -n -g '*.php' -g '!vendor' -g '!node_modules' -e '->get_settings\(|\{\{\{|echo\s+\$settings|_elementor_data|_elementor_page_settings' .
rg -n -g '*.php' -g '*.js' -g '!vendor' -g '!node_modules' -e 'elementor/frontend/init|elementorFrontend\.hooks|elementor/widget/render_content|elementor/frontend/the_content' .
```

No match does not prove absence (add-on frameworks, generated code, content-only sites). `{{{ }}}` in `content_template()` prints unescaped; judge it at the sink, not by the match.

## Acceptance checks

- Versions: `wp plugin get elementor --field=version` and, when present, `wp plugin get elementor-pro --field=version`; `wp help elementor` lists the commands this install supports.
- Registration: the `wp eval` assertions in `references/deploy-and-verify.md` print `REGISTERED` for each widget, tag, form action or location.
- Render: a disposable draft renders the expected marker headlessly; the page loads in the editor, the preview and the frontend without console errors or PHP log entries.
- Content edits: the backup exists, `_elementor_data` parses as JSON after the change, only the intended post changed, and the page shows the new content after `wp elementor flush-css`.
- Escaping: a control value with `<script>` and a quote renders inert in `render()` and in the editor preview.
- Static: `phpcs` WordPress standards and project PHPStan, if configured.

A pass proves the tested pages, versions and experiments. It does not prove other translations, cached copies at a CDN, untested templates, or behavior with a different Elementor or Pro version.

## Output Format

Review: result and scope first. Each confirmed finding: severity per the contract, file:line or post ID and element ID, actor, trigger, reachable path, impact, confidence, minimal remediation, regression check. Candidates and missing evidence in a separate list.

Implementation: changed boundaries, backups taken, commands run with exit status, registration and render results, caches regenerated, checks not run, residual risk, rollback.

## References

- `references/page-data-and-content-edits.md`: read for `_elementor_data`, safe content edits, templates, global widgets, kits and page building.
- `references/custom-widgets.md`: read for classic and atomic widget scaffolds, controls, rendering and assets.
- `references/dynamic-tags-forms-and-theme-builder.md`: read for dynamic tags, Pro form actions and Theme Builder locations.
- `references/render-debugging-and-caches.md`: read for blank or stale output, cache layers and damaged data.
- `references/free-vs-pro-matrix.md`: read to answer free versus Pro questions.
- `references/deploy-and-verify.md`: read to install and prove an extension on staging.
- `references/third-party-notices.md`: license notice for material adapted from Mekko Digital elementor-skills.
