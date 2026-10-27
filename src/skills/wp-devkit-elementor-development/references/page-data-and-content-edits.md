# Page data, content edits, templates and page building

Contents: where Elementor stores a page; choosing the edit method; backups; scoped text and link edits; URL changes; templates and the library; global widgets; kits and global styles; translations; building a page from a design; failure symptoms.

Researched 2026-10-27. Sources: [Data structure](https://developers.elementor.com/docs/data-structure/general-structure/), [Page content](https://developers.elementor.com/docs/data-structure/page-content/), [Widget element](https://developers.elementor.com/docs/data-structure/widget-element/), [Page settings](https://developers.elementor.com/docs/data-structure/page-settings/), [Document meta keys](https://github.com/elementor/elementor/blob/main/core/base/document.php), [Kit option](https://github.com/elementor/elementor/blob/main/core/kits/manager.php), [wp elementor replace-urls](https://developers.elementor.com/docs/cli/replace-urls/), [wp elementor flush-css](https://developers.elementor.com/docs/cli/flush-css/), [Library sync](https://developers.elementor.com/docs/cli/library-sync/), [wp search-replace](https://developer.wordpress.org/cli/commands/search-replace/). Parts adapted from Mekko Digital elementor-skills (MIT, see `third-party-notices.md`).

## Where a page lives

- The documented export format has `title`, `type`, `version`, `page_settings` and `content`. On a site, the content tree is stored in the post meta `_elementor_data` (a JSON string) and the page settings in `_elementor_page_settings`. `_elementor_edit_mode` = `builder` marks the post as built with Elementor. Confirm the keys on the installed version with `wp post meta list <id> --fields=meta_key`.
- Each element has `id`, `elType` (`container`, `section`, `column`, `widget`), `widgetType` for widgets, `isInner`, `settings` and `elements`. `settings` is an empty array when nothing is set and an object otherwise. Control IDs are the setting keys (`title` on `heading`, `text` on `button`).
- The atomic editor stores elements with `e-*` types and a typed prop shape. Do not write classic settings into an atomic element or the reverse; check which editor built the page.
- Theme Builder parts, popups, saved sections and the kit are posts of type `elementor_library` with their own `_elementor_data`. A visible header or footer is usually a template, not the page.

Minimal content tree (one container with a heading):

```json
[
  {
    "id": "6af611eb",
    "elType": "container",
    "isInner": false,
    "settings": [],
    "elements": [
      {
        "id": "6a637978",
        "elType": "widget",
        "widgetType": "heading",
        "isInner": false,
        "settings": { "title": "Welcome", "align": "center" },
        "elements": []
      }
    ]
  }
]
```

## Choose the edit method

| Change | Method |
|---|---|
| Layout, styling, adding or removing elements, applying a template | Elementor editor (by hand or browser automation). It validates settings and writes CSS and revisions. |
| A few text, link or image changes on one known post | Editor, or a scoped script on that post after a backup |
| A domain or path change across the site | `wp elementor replace-urls <old> <new>` after a database backup |
| A new page from a design | Editor or an MCP page-building tool; a scripted tree only when settings keys are introspected (see below) |
| Code-defined output | A widget or dynamic tag (`custom-widgets.md`) instead of hand-edited data |

Editor URL: `/wp-admin/post.php?post=<id>&action=elementor`. Editor UI labels change between versions; describe the goal, not button positions.

## Back up before any data write

```sh
wp post meta get <id> _elementor_data > elementor-data-<id>.json
wp post meta get <id> _elementor_page_settings --format=json > elementor-settings-<id>.json
```

Check the backup parses (`jq . elementor-data-<id>.json`). For site-wide changes, `wp db export` first. Elementor also keeps its data on revisions; list them with `wp post list --post_type=revision --post_parent=<id> --fields=ID,post_date` and check that a revision carries `_elementor_data` on the installed version before counting on it as a backup. Restoring through the editor's History panel is the safer path.

## Scoped text and link edits

Avoid `wp search-replace "Old" "New" wp_postmeta`. It touches every post, every revision and every template that contains the string, including global widgets used elsewhere. It also often misses: the JSON stores `/` as `\/` and non-ASCII characters (accents in Portuguese, Spanish or French text) may be stored as `é`-style escapes. Inspect the raw meta before choosing a search string.

Edit one post by decoding, changing and re-encoding the JSON:

```php
<?php
// Run with: wp eval-file edit-heading.php <post_id> "<old text>" "<new text>"
// Writes a single post; take the backup above first.
$post_id  = (int) $args[0];
$old_text = (string) $args[1];
$new_text = (string) $args[2];
$raw      = get_post_meta( $post_id, '_elementor_data', true );
$data     = is_string( $raw ) ? json_decode( $raw, true ) : $raw;
if ( ! is_array( $data ) ) {
	WP_CLI::error( 'No readable _elementor_data on this post.' );
}
$count = 0;
$walk  = function ( array &$elements ) use ( &$walk, &$count, $old_text, $new_text ) {
	foreach ( $elements as &$element ) {
		if ( isset( $element['settings']['title'] ) && $element['settings']['title'] === $old_text ) {
			$element['settings']['title'] = $new_text;
			++$count;
		}
		if ( ! empty( $element['elements'] ) ) {
			$walk( $element['elements'] );
		}
	}
};
$walk( $data );
if ( 1 !== $count ) {
	WP_CLI::error( "Expected 1 match, found {$count}. Nothing written." );
}
update_post_meta( $post_id, '_elementor_data', wp_slash( wp_json_encode( $data ) ) );
WP_CLI::success( 'Updated 1 heading.' );
```

- The script changes one settings key on purpose. Other widgets use other keys (`text` on button, `editor` on text editor); read them from the post's JSON or the widget's controls and adjust the key.
- `update_post_meta()` unslashes its value, so slash the encoded JSON with `wp_slash()`. Without it the backslashes are lost: `Olá` is saved as the text `Olu00e1`, and an escaped quote (`\"`) breaks the JSON.
- Do not pipe JSON through `wp post meta update <id> _elementor_data "$JSON"` without testing on a throwaway post: whether the installed WP-CLI slashes the value before saving decides whether escapes survive.
- Elementor's own document save path (`\Elementor\Plugin::$instance->documents->get( $id )`) also handles revisions and CSS; check that the method exists and which capability it needs on the installed version before using it from WP-CLI (`--user=<admin>`).
- After the write, run `wp elementor flush-css` (add `--regenerate` to rebuild files now) and purge page caches for that URL.

## URL changes

`wp elementor replace-urls <old> <new> [--force]` updates Elementor data, which stores escaped URLs that a plain `wp search-replace` misses. Run core `wp search-replace` for the rest of the database separately, with `--dry-run` first. Both are site-wide writes: backup, staging first.

## Templates and the library

- List saved templates: `wp post list --post_type=elementor_library --fields=ID,post_title,post_status`. The meta `_elementor_template_type` holds the type (page, section, header, footer, single, archive, popup, loop item and others; check the values present with `wp post meta list`).
- Export and import JSON templates through Templates > Saved Templates, or with the `wp elementor library` commands this install lists in `wp help elementor library`. Imported templates can carry form actions with secrets and custom code; review them before import.
- Copying `_elementor_data` and `_elementor_page_settings` from one post to another duplicates the layout, but element IDs repeat across pages and Pro conditions do not follow. Prefer the editor's "Save as Template" and insert, or a duplication plugin, unless a script is the task.
- Theme Builder conditions are stored separately from the template content. After moving templates between sites, re-save the conditions and check them in Theme Builder.

## Global widgets

A Pro global widget references one library template; editing it changes every page that uses it. Find them with `wp post list --post_type=elementor_library --meta_key=_elementor_template_type --meta_value=widget --fields=ID,post_title`. A text change that should affect one page belongs in that page, not in the global widget. Check whether the installed version still offers global widgets or replaces them with a newer reusable element before advising either.

## Kits and global styles

Global colors, fonts, theme style and site settings live in the active kit, an `elementor_library` post referenced by the `elementor_active_kit` option (check with `wp option get elementor_active_kit`). Changing a global color in the kit restyles every element that uses it. Kit export and import exist as `wp elementor kit` commands on recent versions; list them with `wp help elementor kit`. Import overwrites site settings: staging only.

## Translations

WPML and Polylang keep a separate post, and so a separate `_elementor_data`, per language. A fix applied to one language is not applied to the others. Templates may also be translated per language.

## Build a page from a design

1. Read the design (image, Figma export, HTML, written spec) and write a section tree first: containers with direction, width, gap, alignment and background; widgets inside; responsive intent; repeated content. Share the plan before building.
2. Map each block to a stock widget. Confirm free or Pro with `free-vs-pro-matrix.md` and that it is registered on this install. Prefer free widgets and flag Pro dependencies. Build a custom widget only when no stock widget fits, and verify it before placing it.
3. Build in the editor or through a connected MCP page builder. When writing the tree by script, get the real setting keys by introspection (`\Elementor\Plugin::instance()->widgets_manager->get_widget_types( 'heading' )->get_controls()`); keys differ between versions.
4. Verify render (`deploy-and-verify.md`), flush CSS, then compare a screenshot of the page with the design and iterate.

Shapes to respect in a scripted tree: responsive values use suffixed keys (`align`, `align_tablet`, `align_mobile`); advanced keys are underscore-prefixed (`_element_id`, `_css_classes`, `_margin`, `_padding`); media controls hold an object with `id` and `url` from an attachment in the Media Library, not a bare URL; dynamic bindings live in the element's `__dynamic__` settings map; every element ID must be unique within the page.

## Failure symptoms

| Symptom | Likely cause |
|---|---|
| Change visible in the editor, not on the site | Stale CSS or page cache; see `render-debugging-and-caches.md` |
| Text changed on many pages at once | Edited a global widget, a template or the kit |
| Editor will not open the page after a script edit | Invalid JSON or lost backslashes in `_elementor_data`; restore the backup |
| Fix applied on one language only | Separate translated posts |
| Images gone after a scripted build | Media setting written as a bare URL or attachment missing |
