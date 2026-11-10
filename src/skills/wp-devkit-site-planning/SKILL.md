---
name: wp-devkit-site-planning
description: "Plan a new WordPress site before building it: client briefing, a short brainstorm of directions, build, staging and production environments, sitemap, content model, design tokens, plugin needs and phased build tasks in SITE-PLAN.md. Also turns a finished design (image, PDF, HTML, Figma export or the current site) into a WordPress build plan, and reviews an existing plan. Building belongs to the theme, block, ACF, WooCommerce and Elementor skills."
---

# WordPress site planning

Read `references/engineering-contract.md` first. Review requests stay read-only: read the plan and report gaps without editing it. The planning path writes only the plan file (`site-plan/SITE-PLAN.md` by default). It never writes WordPress code, installs software, changes a site or publishes anything. Building starts only after the user approves the plan, as separate tasks.

## Applicability

This skill applies without WordPress being present: in an empty folder, in a folder with design material, or beside an existing site. This section overrides the stop rule in contract item 11. A review request with no plan to read is the only stop: report that no `SITE-PLAN.md` was found in the project and offer the planning path.

## Works with any agent

- Needs only reading files, viewing images, asking questions and writing Markdown. Do not install packages, servers, extensions or other skills, and do not depend on skills from other plugins.
- Ask with the client's structured question tool when it has one; otherwise ask in plain text and stop until the user answers.
- Name sibling skills only as hints ("if installed"). Write every task so that a person, or an agent without that skill, can still do it.

| Limit | Do this |
|---|---|
| Cannot open a URL, PDF or image | Ask for a screenshot, an export or the text pasted into the chat |
| No design tool connection | Ask for one PNG or PDF per screen and the colors and fonts (or the CSS) |
| No web access for current requirements | Mark the item unverified and keep the official URL |
| Cannot write files | Print the plan in the reply |
| Nobody answers (unattended run) | Continue with labelled assumptions and stop at `Status: draft` |

## Inputs

What the user wants and why, who the site is for, the material supplied (brief, content, brand, design files, current site), whether a plan exists and its `Status:`, hosting, domain and staging already owned, the tool the site will be built with (for example wp-env, a local app, Docker, Playground or the host's staging), deadline and budget limits, and who will edit the site. Read the supplied material before asking; never ask what it already answers.

## Boundaries

- Inventory of an inherited site or codebase: `wp-devkit-site-audit-and-onboarding`.
- Building the theme, templates, patterns and `theme.json`: `wp-devkit-theme-development`. Layouts and UI states: `wp-devkit-ui-design`. Custom blocks: `wp-devkit-block-development`. Content types and fields: `wp-devkit-acf-and-content-modeling`. Store: `wp-devkit-woocommerce-dev`. Elementor sites: `wp-devkit-elementor-development`. Decoupled frontends: `wp-devkit-headless-and-wpgraphql`.
- Environments and deployment: `wp-devkit-playground-development`, `wp-devkit-wpcli-and-ops`, `wp-devkit-ci-cd-and-release-engineering`.
- Not covered: installing WordPress, buying hosting, writing final copy, producing images or mockups, and legal advice.

## Modes

| Mode | Use when | Read |
|---|---|---|
| From scratch | The user has an idea, a brief or nothing written | `references/briefing-and-plan-template.md` |
| From a design | The user supplies a design, or wants the look of the current site carried over | `references/design-and-build-decisions.md`, then the template |
| Plan review | A plan exists and the user wants it checked | Code Review Workflow below |

A current site is one more design input: use screenshots, or read its public pages only when the user allows it. Do not log in to or change a live site.

## Size

Classify from signals, not opinion. When signals conflict, take the larger size.

| Size | Signals | Effect |
|---|---|---|
| Small | Up to about six pages, one language, no store, no member area, no integrations | Gates 1 and 2 merge; fewer question rounds |
| Medium | Blog or catalog with structured content, forms tied to other systems, or two languages | All gates |
| Large | Store, member area, bookings, three or more languages, external systems, or migration of existing content | All gates; risks and environments in more detail |

## Implementation workflow

The planning path. `Status:` values are fixed: `briefing-confirmed`, `direction-chosen`, `draft`, `approved`.

1. Intake. Look for the plan at the path the user gives, else `site-plan/SITE-PLAN.md`, else search the project, hidden folders such as `.spec/` included, for a file named `SITE-PLAN.md`. If it exists, read `Status:` and resume at the next step; never overwrite it without asking. List the supplied material and set the size. The `site-plan/` folder keeps the plan and any design material (`site-plan/design/`) apart from the code; the user may choose another path at Gate 1. If the folder looks like a site root (`wp-load.php`, `wp-content/` or `wp-config.php`), ask where to save the plan, so client data is not served publicly.
2. Brief. Ask at most five questions per round from the question bank, most decisive first; round one always covers hosting, domain, staging and the build tool. Record facts as facts, guesses as assumptions and unknowns as open questions; never invent client data. Gate 1: show the brief summary, get confirmation, then write the plan file with `Status: briefing-confirmed`.
3. Direction. From scratch: offer two or three directions, each with a sitemap, a two-sentence visual thesis with allowed patterns, a build approach and trade-offs. Rewrite any thesis that would fit every site in the sector. From a design: the design is the direction; inventory it and list its gaps instead. Gate 2: the user chooses; set `direction-chosen`.
4. Decisions. Build approach, content model, plugins by need (core features first), environments (build, staging, production and the path between them), editing roles, SEO, privacy law by jurisdiction (an open question for the client), accessibility target and performance budget. Look up current hosting requirements at the official source.
5. Plan. Fill the template in the user's language; keep section headings, `Status:` values and skill names in English. Phase 0 is always a plan task for after approval, never something the planner runs: set up the environments, send a test deploy to staging or production and write the project context file (`AGENTS.md`). Every task has a plain description, an acceptance check and a ready request. Set `draft`.
6. Gate 3. Show the summary, the open questions and the risks, then wait. On approval set `approved` and add a History line. Allow at most two rounds of corrections; after that, list what is still disputed and let the user decide.
7. Next step. Point to Phase 0 and its ready request.

## Code Review Workflow

The plan review path. Read-only: do not edit the plan or any other file.

1. Read the plan (the path given, else `site-plan/SITE-PLAN.md`, else a `SITE-PLAN.md` found in the project) and the material it names. Do not log in to or change any site the plan names.
2. Check the structure against the template: every section present, a valid `Status:` value, History kept.
3. Check that facts, assumptions and open questions are separate and that no client fact is invented.
4. Check that each decision has a reason and fits the brief: a store needs payment, tax and shipping decisions; two languages need a translation decision; a form needs a destination and a privacy note.
5. Check the environments: build, staging and production defined, versions matched, deployment path and rollback stated, Phase 0 present with a test deploy and the context file task.
6. Check every task: plain description, acceptance check and a ready request that works without any tool the user lacks.
7. Check the launch checklist and, for a design-based plan, the gaps list.

## Output Format

Reply in the user's language. When the request gives no clue (only a command or a path), use the language of the plan's own text, not of its fixed English headings.

Planning: the current `Status:`, what changed in the plan file, the questions or choices waiting for the user and the next step. Say whether the plan was written to a file or printed, and give its path.

Plan review writes no files; give the result in the reply. The result first, then findings. Each finding gives the plan section (quote its heading), the gap or contradiction, why it would block or mislead the build, and the smallest correction. Security severities, CWE identifiers and file:line do not apply to plan text. Keep unverified items and questions for the client in a separate list.

## Acceptance checks

- No plan file before Gate 1 (an unattended run writes one and stops at `draft`), and no `approved` without an explicit approval from the user.
- Every task in Build phases has an acceptance check and a ready request stating that the build is authorized by the approved plan.
- Phase 0 exists and names the build, staging and production environments, the test deploy and the project context file.
- Facts, assumptions and open questions are separate; nothing in the brief is invented.
- Running this skill never required installing anything.

## References

- `references/briefing-and-plan-template.md`: read for the question bank, question rounds, the direction format, the `SITE-PLAN.md` template, ready requests, the `AGENTS.md` model and the review checklist.
- `references/design-and-build-decisions.md`: read for design inputs and inventory, the design-to-WordPress map, design gaps, the build approach, the content model, plugin choice and environments.
