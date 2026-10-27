# Deploy to staging and verify

Contents: safety; reach the site (WP-CLI, Docker, MCP); place the code; assert registration (widget, dynamic tag, form action, theme location); render check; flush; Elementor commands; reading the results.

Researched 2026-10-27. Sources: [Elementor CLI](https://developers.elementor.com/docs/cli/), [wp elementor flush-css](https://developers.elementor.com/docs/cli/flush-css/), [Registering widgets](https://developers.elementor.com/docs/managers/registering-widgets/), [Registering dynamic tags](https://developers.elementor.com/docs/managers/registering-dynamic-tags/), [Add new form action](https://developers.elementor.com/docs/form-actions/add-new-action/), [Registering theme locations](https://developers.elementor.com/docs/themes/registering-locations/), [wp eval](https://developer.wordpress.org/cli/commands/eval/), [Must-use plugins](https://developer.wordpress.org/advanced-administration/plugins/mu-plugins/). The loop and the assertions are adapted from Mekko Digital elementor-skills (MIT, see `third-party-notices.md`), which validated the free snippets on Elementor 4.1.1 with WordPress 6.9, PHP 8.3 and WP-CLI 2.12 and wrote the Pro snippets from Pro 4.0.4 source. Internal accessors can change; a fatal from a snippet means "check the installed source", not "the extension is broken".

## Safety

Staging or local only. Confirm the target is not production before writing files, activating plugins or creating posts. `wp eval` loads WordPress with all active and must-use code: run it only on a copy you trust. A review does not run these steps.

## Reach the site

- WP-CLI on the host: run from the WordPress root, or pass `--path=<root>`.
- Docker: `docker exec <container> wp <command>` when the image ships WP-CLI; otherwise `docker exec <container> sh -c 'cd /var/www/html && wp <command> --allow-root'`. Copy code in with `docker cp ./example-widgets <container>:/var/www/html/wp-content/plugins/example-widgets`. Find the container with `docker ps`.
- An MCP server that exposes file writes, WP-CLI or PHP evaluation: use its tools; the PHP assertions below stay the same, only the transport changes.
- None available: give the user the files and the exact commands, and stop. Do not guess a runtime.

## Place the code

- One file: `wp-content/mu-plugins/<slug>.php` loads without activation. Good for a quick check of one widget or tag.
- A real add-on: `wp-content/plugins/<slug>/`, then `wp plugin activate <slug>`. Use this for anything that will ship.
- Confirm load: `wp plugin list --fields=name,status,version` and the PHP error log.

## Assert registration

Replace `NAME` with the extension's `get_name()` slug (lowercase letters, digits, `_` and `-` only; it is pasted into PHP). Each prints one word so the result is easy to check. Add `--allow-root` inside containers that run as root.

Widget:

```sh
wp eval '$w = \Elementor\Plugin::instance()->widgets_manager->get_widget_types( "NAME" ); echo $w ? "REGISTERED " . $w->get_title() : "MISSING", PHP_EOL;'
```

Dynamic tag:

```sh
wp eval '$i = \Elementor\Plugin::instance()->dynamic_tags->get_tag_info( "NAME" ); echo $i ? "REGISTERED " . $i["class"] : "MISSING", PHP_EOL;'
```

Pro form action (the registrar's `get()` without an argument returns all actions):

```sh
wp eval '$r = \ElementorPro\Modules\Forms\Module::instance()->actions_registrar; echo $r->get( "NAME" ) ? "REGISTERED" : "MISSING have: " . implode( ",", array_keys( $r->get() ) ), PHP_EOL;'
```

Pro theme location:

```sh
wp eval '$l = \ElementorPro\Modules\ThemeBuilder\Module::instance()->get_locations_manager()->get_locations(); echo isset( $l["NAME"] ) ? "REGISTERED" : "MISSING have: " . implode( ",", array_keys( $l ) ), PHP_EOL;'
```

`MISSING` means the hook did not run for this extension: wrong hook name, plugin not active, a fatal while loading, or a Pro-only extension on a site without Pro.

## Render check

Proves `render()` runs and prints the expected output. It creates a draft page, renders it without a browser, and deletes it.

```php
<?php
// Run with: wp eval-file render-check.php NAME MARKER
// Pass widget settings that make the widget print MARKER.
$name   = (string) $args[0];
$marker = (string) $args[1];
$data   = array(
	array(
		'id'       => 'a1b2c3d',
		'elType'   => 'container',
		'settings' => array(),
		'elements' => array(
			array(
				'id'         => 'd4e5f6a',
				'elType'     => 'widget',
				'widgetType' => $name,
				'settings'   => array( 'title' => $marker ),
				'elements'   => array(),
			),
		),
	),
);
$post_id = wp_insert_post(
	array(
		'post_title'  => 'render check',
		'post_status' => 'draft',
		'post_type'   => 'page',
	)
);
if ( ! $post_id || is_wp_error( $post_id ) ) {
	WP_CLI::error( 'INSERT_FAIL' );
}
update_post_meta( $post_id, '_elementor_edit_mode', 'builder' );
update_post_meta( $post_id, '_elementor_data', wp_slash( wp_json_encode( $data ) ) );
$html = \Elementor\Plugin::instance()->frontend->get_builder_content_for_display( $post_id );
wp_delete_post( $post_id, true );
if ( false === strpos( (string) $html, $marker ) ) {
	WP_CLI::error( 'RENDER_FAIL length=' . strlen( (string) $html ) );
}
WP_CLI::success( 'RENDER_OK length=' . strlen( $html ) );
```

- `RENDER_FAIL` with a large length: the widget rendered but did not print the marker with these settings. Length 0 or a PHP fatal: `render()` failed; go to `render-debugging-and-caches.md`.
- The settings key (`title` here) must match the widget's control IDs.
- Check the page in a browser too: the editor preview, the frontend, and the console.

## Flush

- `wp elementor flush-css` after control or style changes; `--regenerate` to rebuild files now.
- `wp cache flush` when a persistent object cache is active.
- Re-save pages that should load new widget assets.

## Elementor commands

Elementor documents these command groups: system info, flush CSS, replace URLs, update DB, library sync, connect, disconnect, import and import-dir, kit import and export, experiments status, activate and deactivate; Pro adds license activate and deactivate and clearing Theme Builder conditions. Exact names differ between versions and between `wp elementor` and `wp elementor-pro`; list them with `wp help elementor` and `wp help elementor-pro` on the target. No Elementor command places a widget, builds a layout or registers an extension: that is code or `_elementor_data`.

## Reading the results

| Result | Meaning |
|---|---|
| `REGISTERED` and `RENDER_OK` | The extension loads and renders with the tested settings on this version |
| `REGISTERED`, `RENDER_FAIL` | Loads, but `render()` output differs or fails |
| `MISSING` | Not loaded: hook, activation, fatal or Pro dependency |
| Snippet fatal on `\ElementorPro\...` | Pro inactive or accessor renamed; read the installed source |

A pass covers the tested settings, version and experiments. Test the editor preview and at least one page that uses the extension before calling the work done.
