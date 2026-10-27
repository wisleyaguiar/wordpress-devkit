# Dynamic tags, Pro form actions and Theme Builder locations

Contents: dynamic tags (Tag vs Data_Tag, categories, groups, registration, ACF and SCF); Pro form submit actions (methods, settings, secrets, side effects); Theme Builder locations (registration, output, conditions); review checklist.

Researched 2026-10-27. Sources: [Registering dynamic tags](https://developers.elementor.com/docs/managers/registering-dynamic-tags/), [Dynamic tag data](https://developers.elementor.com/docs/dynamic-tags/dynamic-tags-data/), [Dynamic tag categories](https://developers.elementor.com/docs/dynamic-tags/dynamic-tags-categories/), [Dynamic tag rendering](https://developers.elementor.com/docs/dynamic-tags/dynamic-tags-rendering/), [Add new form action](https://developers.elementor.com/docs/form-actions/add-new-action/), [Form action structure](https://developers.elementor.com/docs/form-actions/action-structure/), [Registering theme locations](https://developers.elementor.com/docs/themes/registering-locations/), [Displaying locations](https://developers.elementor.com/docs/themes/displaying-locations/), [Migrating themes with hooks](https://developers.elementor.com/docs/themes/migrating-themes-with-hooks/), [Dynamic tags manager source](https://github.com/elementor/elementor/blob/main/core/dynamic-tags/manager.php), [Secure Custom Fields](https://developer.wordpress.org/secure-custom-fields/). Internal details are adapted from Mekko Digital elementor-skills (MIT, see `third-party-notices.md`), read from Elementor 4.1.x and Pro 4.0.4 source in 2026-06; confirm in the installed source.

## Dynamic tags

Pick the base by output shape:

- `\Elementor\Core\DynamicTags\Tag`: `render()` echoes a string. Use for text-like output.
- `\Elementor\Core\DynamicTags\Data_Tag`: `get_value( array $options = [] )` returns data. Needed when the consuming control expects a value array, such as an image (`[ 'id' => ..., 'url' => ... ]`), a URL, a color or a gallery.

Required methods: `get_name()` (slug stored in the element's `__dynamic__` map), `get_title()`, `get_group()`, `get_categories()`. Optional `register_controls()` adds settings to the tag popover.

Categories are `\Elementor\Modules\DynamicTags\Module::*_CATEGORY` constants (TEXT, URL, NUMBER, IMAGE, GALLERY, MEDIA, COLOR, POST_META and others on the installed version). They decide which controls accept the tag: a TEXT tag does not appear on a media control. A tag can declare several categories.

```php
<?php
add_action(
	'elementor/dynamic_tags/register',
	function ( $dynamic_tags_manager ) {
		$dynamic_tags_manager->register_group( 'example', array( 'title' => esc_html__( 'Example', 'example-widgets' ) ) );
		require_once __DIR__ . '/tags/class-example-support-phone.php';
		$dynamic_tags_manager->register( new \Example\Tags\Support_Phone() );
	}
);
```

- `elementor/dynamic_tags/register` replaces the deprecated `elementor/dynamic_tags/register_tags`.
- Escape inside `render()`. Tags render for every visitor of the page, so a tag that prints post meta, user data or request input is an output sink like any template.
- Do not expose private data (another user's meta, draft content, options) just because an editor can pick the tag. Check the capability or the post status the tag reads.
- In the atomic editor, classic tags are adapted by a converter; a failed conversion drops the binding. New atomic-only tags follow the atomic dynamic prop types in the installed source.

### ACF and Secure Custom Fields

Elementor Pro ships ACF dynamic tags (and Pods and Toolset tags) that register only when the field plugin is detected and the license allows the feature. Secure Custom Fields (SCF) is the WordPress.org fork of ACF with ACF-style APIs. Whether Pro detects SCF and lists its fields depends on what both installed versions check: confirm with `wp eval 'var_dump( class_exists( "ACF" ) );'`, then open a text control's dynamic picker and look for the ACF group. If the group is missing, a small custom `Data_Tag` that calls `get_field()` is a supported fallback. Field design and value formatting go to `wp-devkit-acf-and-content-modeling`.

## Pro form submit actions

Pro only; the Forms module must be active. Extend `\ElementorPro\Modules\Forms\Classes\Action_Base`:

- `get_name()`: ID stored in the form's `submit_actions` setting.
- `get_label()`: label in "Actions After Submit".
- `register_settings_section( $widget )`: optional controls. Wrap them in a section and gate them with `'condition' => [ 'submit_actions' => $this->get_name() ]` so they show only when the action is selected.
- `run( $record, $ajax_handler )`: runs on a valid submission. Read `$record->get( 'fields' )` and `$record->get( 'form_settings' )`. Report problems through `$ajax_handler` methods documented for the installed version.
- `on_export( $element )`: remove API keys and secrets from the element before template export, and return the element.

```php
<?php
add_action(
	'elementor_pro/forms/actions/register',
	function ( $form_actions_registrar ) {
		require_once __DIR__ . '/forms/class-crm-action.php';
		$form_actions_registrar->register( new \Example\Forms\Crm_Action() );
	}
);
```

- Older code registers actions with `add_form_action()` on the Forms module. Use the documented registrar hook instead, and check the installed Pro before calling the old method removed.
- Fields are keyed by their field ID; labels are for display. Treat every field value as untrusted input: validate and sanitize before sending it on or storing it.
- Secrets: store keys in options or constants, not in the widget settings when possible. If the settings must hold a key, strip it in `on_export()`, set `'render_type' => 'none'` and turn off the control's AI button (`'ai' => [ 'active' => false ]`).
- External calls: short timeout on `wp_remote_post()`, check the response, log failures without logging personal data. A slow provider makes every submission slow.
- Spam checks (honeypot, reCAPTCHA, Akismet) run before actions; do not rely on them as authorization for privileged side effects. A public form must not create users with roles, publish content or change settings without its own checks.
- A provider with OAuth and remote field mapping uses the Pro integration base and its panel requests; read the installed Mailchimp or ActiveCampaign handlers as reference.
- A custom form field type registers through the Forms module's fields registrar, a separate hook.

## Theme Builder locations

Pro only. Core locations already registered: `header`, `footer`, `archive`, `single`. A theme supports them with `$manager->register_all_core_location()` on `elementor/theme/register_locations`; condition types such as single post, search results or 404 are not locations.

Register a new location and print it from the theme:

```php
<?php
add_action(
	'elementor/theme/register_locations',
	function ( $manager ) {
		$manager->register_location(
			'before_content',
			array(
				'label'           => esc_html__( 'Before Content', 'example-theme' ),
				'multiple'        => false,
				'edit_in_content' => false,
				'hook'            => 'example_theme_before_content',
			)
		);
	}
);
```

- Registration prints nothing. The theme must fire the hook (`do_action( 'example_theme_before_content' )`) or print the location with the documented fallback: `if ( ! function_exists( 'elementor_theme_do_location' ) || ! elementor_theme_do_location( 'before_content' ) ) { get_template_part( 'template-parts/before-content' ); }`.
- `remove_hooks` lists callbacks removed from the hook once a template renders, to suppress the theme's default markup.
- `multiple` allows several templates in the location. `edit_in_content` should be false for locations outside `the_content`, such as sidebars.
- Templates bind to the location through Display Conditions. After a migration, check conditions in Theme Builder; Pro lists a WP-CLI command to clear conditions (`wp help elementor-pro` or `wp help elementor`, depending on version).
- Hello Elementor and other Elementor-ready themes already register the core locations. A child theme or a block theme needs its own registration; theme structure goes to `wp-devkit-theme-development`.

## Review checklist

- Hooks are the current registrar hooks, not deprecated ones.
- Tag and action output escaped; private data not exposed to anyone who can pick a tag.
- Form actions validate fields, use timeouts, keep secrets out of exports and logs.
- Theme locations fire their hook and degrade when no template applies.
- Pro-only code checks that Pro is active and does not fatal without it.
- Registration proven with `deploy-and-verify.md`.
