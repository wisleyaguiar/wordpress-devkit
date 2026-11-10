# Design inputs and WordPress build decisions

Apply [the engineering contract](engineering-contract.md) first. Contents: design inputs, design inventory, design-to-WordPress map, design gaps, build approach, content model, plugin choice, environments, hosting requirements, sources.

## Design inputs

The design is the source of truth: do not redesign it. Refining keeps its identity; replacing it needs the user's explicit request.

| Input | Read it by | If it cannot be read |
|---|---|---|
| Images (PNG, JPG, WebP) | Viewing each screen | Ask for a smaller export or one screen at a time |
| PDF | Viewing its pages | Ask for one PNG per page |
| HTML and CSS export | Reading the files; colors, fonts and widths come from the CSS | Ask for the CSS or a style list |
| Figma or another design tool | A design tool connection, when the client already has one | Ask for one PNG per frame and the color and type styles (names and values) |
| Current site | Screenshots supplied by the user, or its public pages when the user allows it | Ask for screenshots of each page type at mobile and desktop width |
| Written spec | Reading it | Ask the questions in the briefing bank that it leaves open |

Do not log in to a live site, submit its forms or change anything on it.

## Design inventory

Work screen by screen and write the result into the plan.

1. List every screen and the page type it stands for (home, inner page, post, archive, contact, store page).
2. Mark the parts that repeat on every screen (header, footer, navigation) and the sections that repeat across screens.
3. Extract tokens: colors with their role (text, background, accent, border), font families, font sizes, spacing steps, content and wide widths, corner radius, shadows.
4. List components that show data or behave (sliders, tabs, filters, maps, counters, forms).
5. List the content each screen needs and whether it exists.
6. Record breakpoints shown in the design; when only desktop exists, say so.

## Design-to-WordPress map

| In the design | In WordPress (block theme) | Skill to build it, if installed |
|---|---|---|
| A whole screen | A template, for example `front-page.html`, `home.html`, `page.html`, `single.html`, `archive.html`, `search.html`, `404.html` | `wp-devkit-theme-development` |
| Header and footer | Template parts in the theme's `parts/` folder | `wp-devkit-theme-development` |
| Menu | The Navigation block, usually in the header part | `wp-devkit-theme-development` |
| A section that repeats | A pattern | `wp-devkit-theme-development` |
| Colors | `settings.color.palette` in `theme.json` | `wp-devkit-theme-development` |
| Font families and sizes | `settings.typography.fontFamilies` and `settings.typography.fontSizes` | `wp-devkit-theme-development` |
| Spacing steps | `settings.spacing.spacingSizes` or `settings.spacing.spacingScale` | `wp-devkit-theme-development` |
| Content and wide widths | `settings.layout.contentSize` and `settings.layout.wideSize` | `wp-devkit-theme-development` |
| A component with data or behavior | A custom block, or a core block with a pattern when core is enough | `wp-devkit-block-development` |
| Repeated structured content (team, services, locations) | A custom post type with fields, shown by a query or a block | `wp-devkit-acf-and-content-modeling` |
| A form | A form plugin or a form block; name where entries go | `wp-devkit-plugin-development` |
| Several languages | A translation plugin or a multisite network; decide in the plan | Plan decision |
| Store pages | WooCommerce pages and blocks | `wp-devkit-woocommerce-dev` |
| A page-builder design (Elementor) | Elementor templates and the kit; Theme Builder headers, footers and templates need Elementor Pro | `wp-devkit-elementor-development` |
| Layout, states and responsive behavior | Interface work on the built templates | `wp-devkit-ui-design` |

Home and front page differ: `front-page.html` is the site's front page; `home.html` is the posts index. When `front-page.html` exists it always serves the front page, whatever the front page setting says. That setting in Settings, Reading decides whether the posts index is the front page (latest posts) or a separate Posts page (a static page). Write the choice into the plan.

## Design gaps

Most designs leave these out. List each one with an owner and a decision:

- Mobile and tablet layouts for every screen.
- Hover, focus and active states; visible keyboard focus.
- Empty, loading, error and success states (search with no results, form errors, form sent).
- Long text, long names, missing images and translated text that is longer than the original.
- The 404 page, search results, archives, single posts and the privacy policy page.
- Cookie or consent notice, when tracking is used.
- Color contrast of text on its background.

## Build approach

| Choose | When | Avoid when |
|---|---|---|
| Block theme | New site, editors use the Site Editor, design fits blocks and patterns | The team must keep an existing classic theme or a builder |
| Child of an existing block theme | The design is close to a known block theme | The design needs a different structure |
| Classic or hybrid theme | An existing plugin stack or team depends on PHP templates | A new site with no such dependency |
| Elementor | The client already uses or wants Elementor and editors depend on it | Speed, long-term cost or license limits matter more than builder editing |
| Headless | A separate frontend team and a clear reason (several frontends, an app) | A brochure site or a small team |

Write the reason for the choice into the plan. Licence costs of commercial themes, builders and plugins go into the Plugins table.

## Content model

| Content | Use |
|---|---|
| A unique page (About, Contact) | Page |
| Dated articles or news | Post with categories or tags |
| Repeated items with the same fields (team, services, products, locations) | Custom post type with fields |
| A way to group items (service area, product type) | Taxonomy |
| A single value on an item (price, role, phone) | Field |

Prefer few types. Every custom type needs a template or a block to show it, and an editor who keeps it current.

## Plugin choice

- Start from the need, then check whether core already covers it.
- One plugin per need. Two plugins that do the same job conflict.
- Before naming a plugin, check: last update, the WordPress versions it states it was tested with, support activity, licence and cost, where it stores data, and what removing it would break.
- Do not install anything during planning. The plan lists needs and candidates; the user decides.

## Environments

| Environment | Purpose | Rule |
|---|---|---|
| Build | Where the site is made | Same PHP and WordPress versions as production |
| Staging | Final test before launch and before each later change | A faithful copy of production: same host type, database engine, caching and plugins |
| Production | The public site | Changes arrive only through the deployment path |

- WordPress Playground is good for a fast build or demo. It runs on SQLite, while WordPress core officially supports MySQL or MariaDB for production, so a Playground run never replaces a staging test. When the build runs on Playground, write SQLite in the plan's Database column.
- When the build runs on wp-env, Phase 0 creates a `.wp-env.json` that sets `phpVersion` and `mariadbVersion` to the production values and `core` to the production WordPress version (or `null` for the latest release), and maps the theme or plugin folder with `themes` or `plugins`. wp-env needs Docker, Node.js and git; that is a requirement of the chosen tool, not of this skill. Its default Docker runtime uses MariaDB; its Playground runtime uses SQLite. By default WordPress itself lives outside the project folder (unless `core` points to a local path), so the project root is not a site root.
- Choose a deployment path and write it down: Git with a deploy step, SFTP, the host's staging and push tools, or a migration with WP-CLI. A URL change after a move needs `wp search-replace`, run first with `--dry-run`.
- Take a backup of the database and the files before the first deploy. As a practice, restore it once on staging to prove it works. Write how to return to the last working version.
- Phase 0 ends with a test deploy of an empty theme to staging or production, so deployment problems appear at the start.
- When the hosting is already fixed, it is a constraint: the rest of the plan follows its versions and limits. When it is not, list what the site needs from a host and leave the choice to the user without naming a brand.

## Hosting requirements

Look up the current recommendation when planning: https://wordpress.org/about/requirements/. On the research date it recommended PHP 8.3 or greater, MariaDB 10.11 or greater or MySQL 8.0 or greater, and HTTPS support. The Hosting Handbook can be ahead of that page: on the same date it recommended PHP 8.4 or later for production. When the two differ, record both and plan for the newest version the host supports. Treat all these numbers as a dated snapshot; without web access, write them as unverified with the URLs. PHP extension details are in the Hosting Handbook.

## Sources

Research date: 2026-11-10.

- https://wordpress.org/about/requirements/
- https://make.wordpress.org/hosting/handbook/server-environment/ (PHP extensions; MySQL or MariaDB for production, SQLite experimental)
- https://developer.wordpress.org/block-editor/how-to-guides/themes/global-settings-and-styles/ (theme.json presets and layout settings)
- https://developer.wordpress.org/themes/templates/template-hierarchy/ (front page and home)
- https://wordpress.org/documentation/article/settings-reading-screen/ (front page setting)
- https://developer.wordpress.org/themes/templates/template-parts/ (template parts in `parts/`)
- https://developer.wordpress.org/themes/patterns/
- https://developer.wordpress.org/playground/developers/architecture/wordpress (Playground runs WordPress on SQLite)
- https://developer.wordpress.org/cli/commands/search-replace/
- https://developer.wordpress.org/block-editor/reference-guides/packages/packages-env/ (wp-env prerequisites, `.wp-env.json` fields, runtimes)
- https://developer.wordpress.org/advanced-administration/security/backup/
- https://elementor.com/help/theme-builder/ (Theme Builder is an Elementor Pro feature)
