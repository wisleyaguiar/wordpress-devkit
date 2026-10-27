# Custom widgets: classic, atomic and Pro bases

Contents: choose the base; add-on bootstrap; classic widget methods; controls; rendering and escaping; editor template; assets and frontend JS; skins and repeaters; Pro bases; atomic widgets; review checklist.

Researched 2026-10-27. Sources: [Add new widget](https://developers.elementor.com/docs/widgets/add-new-widget/), [Widget structure](https://developers.elementor.com/docs/widgets/widget-structure/), [Widget rendering](https://developers.elementor.com/docs/widgets/widget-rendering/), [Registering widgets](https://developers.elementor.com/docs/managers/registering-widgets/), [Frontend available controls](https://developers.elementor.com/docs/editor-controls/frontend-available/), [Deprecations](https://developers.elementor.com/docs/deprecations/), [Editor template syntax](https://github.com/elementor/elementor/blob/main/core/common/assets/js/common.js), [Elementor source](https://github.com/elementor/elementor). Pro base classes and atomic details are adapted from Mekko Digital elementor-skills (MIT, see `third-party-notices.md`), which read Elementor 4.1.x and Pro 4.0.4 source in 2026-06; class names and paths move between versions, so open the installed source before extending them.

## Choose the base

| Need | Extend |
|---|---|
| Classic widget (works in the classic editor and alongside atomic elements) | `\Elementor\Widget_Base` |
| Atomic editor element with an `e-` type | the atomic widget base in `modules/atomic-widgets/` of the installed Elementor (named `Atomic_Widget_Base` in 4.x source) |
| Pro query, carousel, theme element or WooCommerce widget | the Pro module base (see Pro bases below), only when Pro is a declared dependency |

Ask or infer the editor in use: the site's experiments and existing `_elementor_data` show whether pages use classic or `e-*` elements.

## Add-on bootstrap

```php
<?php
/**
 * Plugin Name: Example Elementor Widgets
 * Requires Plugins: elementor
 * Text Domain: example-widgets
 */

add_action(
	'elementor/widgets/register',
	function ( $widgets_manager ) {
		require_once __DIR__ . '/widgets/class-example-cta.php';
		$widgets_manager->register( new \Example\Widgets\Example_Cta() );
	}
);
```

- `elementor/widgets/register` replaces the deprecated `elementor/widgets/widgets_registered`, and `register()` replaces `register_widget_type()`; check the Deprecations page against the installed version.
- Custom categories: `elementor/elements/categories_registered` with `$elements_manager->add_category()`.
- The `Requires Plugins` header makes WordPress (6.5+) refuse activation without Elementor. Do not also hard-fail on `did_action( 'elementor/loaded' )` with a fatal; show an admin notice instead.

## Classic widget methods

- `get_name()`: unique slug stored as `widgetType` in `_elementor_data`. Renaming it orphans every saved instance.
- `get_title()`, `get_icon()`, `get_categories()`, `get_keywords()`.
- `register_controls()`: `start_controls_section()`, `add_control()`, `add_responsive_control()`, `add_group_control()`, `end_controls_section()`. Older code overrides `_register_controls()`, deprecated in favor of `register_controls()`.
- `render()`: PHP output for the frontend and the editor preview.
- `content_template()`: optional Backbone/Underscore template for instant editor preview. Without it the editor asks the server to render.

## Controls

- Types are `\Elementor\Controls_Manager` constants: TEXT, TEXTAREA, NUMBER, SWITCHER, SELECT, SELECT2, CHOOSE, COLOR, MEDIA, ICONS, GALLERY, REPEATER, WYSIWYG, CODE, URL, HIDDEN, RAW_HTML, HEADING, DIVIDER, DATE_TIME and others.
- Group controls (typography, border, background, box shadow, image size and more) via `add_group_control( \Elementor\Group_Control_Typography::get_type(), [...] )`.
- `condition` (AND of values) and `conditions` (nested `relation` and `terms`) only show or hide controls in the editor. They do not validate saved data; `render()` must still handle any value.
- `'dynamic' => [ 'active' => true ]` lets users bind a dynamic tag to the control.
- `'frontend_available' => true` exposes the value to the widget's frontend JS handler.
- `selectors` write CSS from control values into the generated CSS file. Values pass through into CSS, so restrict free-text controls used in selectors.

## Rendering and escaping

- Read `$this->get_settings_for_display()`, not `get_settings()`, so dynamic tags resolve.
- Escape at output: `esc_html()` for text, `esc_url()` for URLs, `esc_attr()` for attributes, `wp_kses_post()` for WYSIWYG. A control value is user input from anyone who can edit with Elementor; on multisite or with the Role Manager that may not be an administrator.
- For URL controls use `$this->add_link_attributes( 'link', $settings['link'] )` and `$this->print_render_attribute_string( 'link' )`, which handle target, nofollow and custom attributes. Check both methods exist on the installed version.
- Guard optional settings (`$settings['title'] ?? ''`); a missing key after an update is a common blank-render cause.

```php
<?php
namespace Example\Widgets;

class Example_Cta extends \Elementor\Widget_Base {
	public function get_name(): string {
		return 'example_cta';
	}

	public function get_title(): string {
		return esc_html__( 'Example CTA', 'example-widgets' );
	}

	public function get_categories(): array {
		return array( 'general' );
	}

	protected function register_controls(): void {
		$this->start_controls_section( 'content', array( 'label' => esc_html__( 'Content', 'example-widgets' ) ) );
		$this->add_control(
			'title',
			array(
				'label'   => esc_html__( 'Title', 'example-widgets' ),
				'type'    => \Elementor\Controls_Manager::TEXT,
				'dynamic' => array( 'active' => true ),
			)
		);
		$this->add_control(
			'link',
			array(
				'label' => esc_html__( 'Link', 'example-widgets' ),
				'type'  => \Elementor\Controls_Manager::URL,
			)
		);
		$this->end_controls_section();
	}

	protected function render(): void {
		$settings = $this->get_settings_for_display();
		$title    = $settings['title'] ?? '';
		if ( '' === $title ) {
			return;
		}
		if ( ! empty( $settings['link']['url'] ) ) {
			$this->add_link_attributes( 'link', $settings['link'] );
			echo '<a ';
			$this->print_render_attribute_string( 'link' );
			echo '>' . esc_html( $title ) . '</a>';
			return;
		}
		echo '<h3>' . esc_html( $title ) . '</h3>';
	}

	protected function content_template(): void {
		?>
		<# if ( settings.title ) { #>
			<h3>{{ settings.title }}</h3>
		<# } #>
		<?php
	}
}
```

## Editor template

`{{ }}` escapes, `{{{ }}}` prints raw HTML, `<# #>` runs JS. Use `{{{ }}}` only for values already sanitized for HTML. Keep the template's markup equal to `render()`; a mismatch shows one thing in the editor and another on the site.

## Assets and frontend JS

- Register scripts and styles on `wp_enqueue_scripts` (and the Elementor frontend register hooks if needed), then return their handles from `get_script_depends()` and `get_style_depends()`. Elementor enqueues them only on pages that use the widget. Enqueuing globally works but costs every page.
- Frontend handlers attach on `elementor/frontend/init` with `elementorFrontend.hooks.addAction( 'frontend/element_ready/<name>.default', callback )` or the handler classes documented for the installed version. Code must handle the editor re-rendering the element.
- With improved asset loading, the asset list is saved with the page. A widget added by code after the last save may not enqueue until the page is saved again.

## Skins and repeaters

- A skin is a swappable render layer: extend `\Elementor\Skin_Base`, implement `get_id()`, `get_title()` and `render()`, and register it from the widget's `register_skins()` (older code: `_register_skins()`) with `$this->add_skin( new My_Skin( $this ) )`.
- Repeater: build a `new \Elementor\Repeater()`, add controls to it, then add a REPEATER control with `'fields' => $repeater->get_controls()` and a `title_field` such as `'{{{ item_text }}}'`. In `render()` loop `$settings['list']`; in the template use `_.each( settings.list, function( item ) { ... } )`. Escape item values like any other setting.

## Pro bases

Only extend these when Elementor Pro is a declared requirement; they are internal classes without a stability promise.

- A Pro base widget class adds Pro categories and promotion handling.
- The posts module base provides query and pagination controls and the `elementor/query/{query_id}` filter for custom query arguments. Using that filter from a stock Posts or Loop Grid widget is often enough; no new widget is needed.
- The carousel module base configures Swiper slides. The theme elements base places widgets in the Pro theme elements category. Both classes were named plain `Base` in Pro 4.0 source; import them with an alias.
- WooCommerce widget bases exist for product and archive widgets; store behavior goes to `wp-devkit-woocommerce-dev`.

## Atomic widgets

The atomic editor (Elementor 4.x, gated by experiments) changes fast. Treat these points as leads and read the installed `modules/atomic-widgets/` source.

- The element type starts with `e-`. The class defines a typed props schema (`define_props_schema()` with `*_Prop_Type::make()` types such as string, number, boolean, link, image, size, classes) and atomic controls in sections.
- Output comes from a Twig template next to the class (a template trait) or an overridden `render()`. A missing template file can render nothing without a PHP error.
- `register_controls()` and `content_template()` are classic-only. Do not mix classic settings into an atomic element.
- A saved value that no longer matches the schema can be dropped on load. Changing a prop type needs a migration plan like any data change.
- Registration still goes through the widgets manager on `elementor/widgets/register`; confirm on the installed version.

## Review checklist

- Unique, stable `get_name()`; no rename without migrating saved data.
- `get_settings_for_display()` in `render()`; every sink escaped; `{{{ }}}` justified.
- `content_template()` matches `render()` or is absent.
- Controls used in `selectors` cannot inject arbitrary CSS from untrusted editors.
- Assets declared through dependencies, not enqueued on every page.
- Works without Pro unless Pro is declared; no fatal when Elementor is inactive.
- Registration and render proven with `deploy-and-verify.md`.
