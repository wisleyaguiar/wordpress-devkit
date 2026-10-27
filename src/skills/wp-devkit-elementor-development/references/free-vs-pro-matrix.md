# Elementor free versus Pro matrix

Contents: how to answer; live inventory (the authority); free elements; free core widgets; free modules; free atomic widgets; Pro widgets; Pro WooCommerce widgets; Pro feature gates; Pro form actions; Pro dynamic tag groups; notes.

Researched 2026-10-27. Sources: [Elementor source](https://github.com/elementor/elementor), [Elementor Pro features](https://elementor.com/pro/), [Mekko Digital elementor-skills](https://github.com/Mekko-Digital/elementor-skills). The rows below are a snapshot made by Mekko Digital (MIT, see `third-party-notices.md`) from Elementor 4.1.x and Elementor Pro 4.0.4 source, published 2026-06-11. Widgets are added, renamed and moved between tiers; the live inventory on the target site wins over this table.

## How to answer

1. Match the user's term against the widget name or the feature gates below.
2. Reply with the tier (free, Pro, Pro plus WooCommerce), the module, and any caveat (deprecated variant, experiment gate, license feature). Quote the row.
3. If the term is missing, say so. Do not guess a tier; check the live inventory or the installed source (`modules/*/widgets/`).
4. Prefer free widgets when building and flag every Pro dependency to the user.

## Live inventory (the authority)

```sh
wp eval 'echo implode( PHP_EOL, array_keys( \Elementor\Plugin::instance()->widgets_manager->get_widget_types() ) ), PHP_EOL;'
wp plugin list --fields=name,status,version | grep -i elementor
```

Run on the target site (or a copy) with and without Pro active to see what Pro adds. Experiments change the list.

Rows use the form `name | tier | module`.

## Free — elements (containers, not widgets)

section | free | core (legacy row-style section)
column | free | core (legacy column inside section)
container | free | core (flexbox/grid container, classic editor)
repeater | free | core (internal; used by widgets with sub-items)
e-div-block | free | atomic-widgets
e-flexbox | free | atomic-widgets
e-tabs | free | atomic-widgets
e-tabs-menu | free | atomic-widgets
e-tab | free | atomic-widgets
e-tabs-content-area | free | atomic-widgets
e-tab-content | free | atomic-widgets
e-form | free+pro gate | atomic-widgets (requires Pro active + experiment `e_pro_atomic_form`)
e-form-success-message | free+pro gate | atomic-widgets (same gate)
e-form-error-message | free+pro gate | atomic-widgets (same gate)

## Free — core (includes/widgets)

accordion | free | core
alert | free | core
audio | free | core
button | free | core
common | free | core (internal base, not user-facing)
counter | free | core
divider | free | core
google_maps | free | core
heading | free | core
html | free | core
icon | free | core
icon-box | free | core
icon-list | free | core
image | free | core
image-box | free | core
image-carousel | free | core
image-gallery | free | core
inner-section | free | core (legacy section)
menu-anchor | free | core
progress | free | core
rating | free | core
read-more | free | core (theme-builder helper)
shortcode | free | core
sidebar | free | core
social-icons | free | core
spacer | free | core
star-rating | free | core
tabs | free | core
testimonial | free | core
text-editor | free | core
toggle | free | core
video | free | core
wp-widget-* | free | core (WordPress widget bridge)

## Free — modules

contact-buttons | free | floating-buttons
floating-bars-var-1 | free | floating-buttons
link-in-bio | free | link-in-bio
nested-accordion | free | nested-accordion
nested-tabs | free | nested-tabs
text-path | free | shapes

## Free — atomic 4.0 (Elementor 4.x editor)

e-button | free | atomic-widgets
e-divider | free | atomic-widgets
e-heading | free | atomic-widgets
e-image | free | atomic-widgets
e-paragraph | free | atomic-widgets
e-self-hosted-video | free | atomic-widgets
e-svg | free | atomic-widgets
e-youtube | free | atomic-widgets
(Container/flexbox/tabs/form atomic types are elements, listed in the Elements section above — not widgets.)

## Pro

animated-headline | pro | animated-headline
blockquote | pro | blockquote
call-to-action | pro | call-to-action
media-carousel | pro | carousel
reviews | pro | carousel
testimonial-carousel | pro | carousel
code-highlight | pro | code-highlight
countdown | pro | countdown
flip-box | pro | flip-box
contact-buttons-var-1 | pro | floating-buttons
contact-buttons-var-3 | pro | floating-buttons
contact-buttons-var-4 | pro | floating-buttons
contact-buttons-var-5 | pro | floating-buttons
contact-buttons-var-6 | pro | floating-buttons
contact-buttons-var-7 | pro | floating-buttons
contact-buttons-var-8 | pro | floating-buttons
contact-buttons-var-9 | pro | floating-buttons
contact-buttons-var-10 | pro | floating-buttons
floating-bars-var-2 | pro | floating-buttons
floating-bars-var-3 | pro | floating-buttons
(contact-buttons-var-2 is absent in source)
form | pro | forms
login | pro | forms
gallery | pro | gallery
global | pro | global-widget
hotspot | pro | hotspot
template | pro | library
link-in-bio-var-2 | pro | link-in-bio
link-in-bio-var-3 | pro | link-in-bio
link-in-bio-var-4 | pro | link-in-bio
link-in-bio-var-5 | pro | link-in-bio
link-in-bio-var-6 | pro | link-in-bio
link-in-bio-var-7 | pro | link-in-bio
loop-carousel | pro | loop-builder
loop-grid | pro | loop-builder
taxonomy-filter | pro | loop-filter
lottie | pro | lottie
mega-menu | pro | mega-menu
nav-menu | pro | nav-menu
nested-carousel | pro | nested-carousel
off-canvas | pro | off-canvas
paypal-button | pro | payments
stripe-button | pro | payments
portfolio | pro | posts
posts | pro | posts
price-list | pro | pricing
price-table | pro | pricing
progress-tracker | pro | progress-tracker
search | pro | search
share-buttons | pro | share-buttons
slides | pro | slides
facebook-button | pro | social
facebook-comments | pro | social
facebook-embed | pro | social
facebook-page | pro | social
table-of-contents | pro | table-of-contents
archive-posts | pro | theme-builder
archive-title | pro | theme-builder
page-title | pro | theme-builder
post-content | pro | theme-builder
post-excerpt | pro | theme-builder
post-featured-image | pro | theme-builder
post-title | pro | theme-builder
site-logo | pro | theme-builder
site-title | pro | theme-builder
author-box | pro | theme-elements
breadcrumbs | pro | theme-elements
post-comments | pro | theme-elements
post-info | pro | theme-elements
post-navigation | pro | theme-elements
search-form | pro | theme-elements
sitemap | pro | theme-elements
video-playlist | pro | video-playlist

### Pro — WooCommerce (requires WooCommerce plugin)

wc-add-to-cart | pro | woocommerce
wc-archive-products | pro | woocommerce (current)
woocommerce-archive-products | pro | woocommerce (deprecated)
wc-categories | pro | woocommerce
wc-elements | pro | woocommerce (single-product elements)
wc-products | pro | woocommerce (current)
woocommerce-products | pro | woocommerce (deprecated)
wc-single-elements | pro | woocommerce
woocommerce-archive-description | pro | woocommerce
woocommerce-breadcrumb | pro | woocommerce
woocommerce-cart | pro | woocommerce
woocommerce-category-image | pro | woocommerce
woocommerce-checkout-page | pro | woocommerce
woocommerce-menu-cart | pro | woocommerce
woocommerce-my-account | pro | woocommerce
woocommerce-notices | pro | woocommerce
woocommerce-product-add-to-cart | pro | woocommerce
woocommerce-product-additional-information | pro | woocommerce
woocommerce-product-content | pro | woocommerce
woocommerce-product-data-tabs | pro | woocommerce
woocommerce-product-images | pro | woocommerce
woocommerce-product-meta | pro | woocommerce
woocommerce-product-price | pro | woocommerce
woocommerce-product-rating | pro | woocommerce
woocommerce-product-related | pro | woocommerce
woocommerce-product-short-description | pro | woocommerce
woocommerce-product-stock | pro | woocommerce
woocommerce-product-title | pro | woocommerce
woocommerce-product-upsell | pro | woocommerce
woocommerce-purchase-summary | pro | woocommerce


## Pro feature gates (not widgets)

- Theme Builder (header, footer, single, archive, search results, 404, loop item) | pro | theme-builder
- Display Conditions for theme templates | pro | theme-builder
- Popup Builder | pro | popup
- Custom CSS per element | pro | custom-css
- Custom Attributes | pro | custom-attributes
- Custom Code (site-wide head and body) | pro | custom-code
- Role Manager | pro | role-manager
- Motion effects (mouse, scroll) | pro | motion-fx
- Sticky effects | pro | sticky
- Scroll snap | pro | scroll-snap
- Page transitions | pro | page-transitions
- Forms (widget, actions, integrations) | pro | forms
- Dynamic tags for ACF, Pods, Toolset and advanced post fields | pro | dynamic-tags
- Loop Builder, Loop Grid, Loop Carousel, Taxonomy Filter | pro | loop-builder, loop-filter
- WooCommerce Builder | pro | woocommerce
- Mega Menu | pro | mega-menu
- Off-Canvas | pro | off-canvas

## Pro form submit actions

Found in the Pro forms module actions folder in the snapshot: email, email2, redirect, webhook, mailchimp, mailerlite, mailpoet, mailpoet3, drip, convertkit, getresponse, activecampaign, slack, discord, activity-log, cf7db. All Pro.

## Pro dynamic tag groups

Built-in Pro groups: author, post, comments, site, archive, media, action, woocommerce. ACF, Pods and Toolset tags register only when the field plugin is detected and the license feature is present. For Secure Custom Fields, see `dynamic-tags-forms-and-theme-builder.md`.

## Notes

- Free `link-in-bio` ships one variant; Pro adds more variants.
- Free floating buttons ship `contact-buttons` and `floating-bars-var-1`; Pro adds the rest.
- `inner-section` and `sidebar` are legacy. The atomic editor uses `e-flexbox` and `e-div-block` instead of section, column and container.
- `section`, `column` and `container` are elements, not widgets.
- Older WooCommerce widget names (`woocommerce-products`, `woocommerce-archive-products`) remain for saved data; new pages use the `wc-` names in the snapshot.
