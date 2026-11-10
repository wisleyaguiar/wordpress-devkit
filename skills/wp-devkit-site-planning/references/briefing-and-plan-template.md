# Briefing, directions and the plan template

Apply [the engineering contract](engineering-contract.md) first. Contents: question rounds, question bank, direction format, plan template, ready requests, review checklist, sources.

## Question rounds

- Ask at most five questions per round. Start with the questions that change the most decisions: goal, audience, must-have features, hosting.
- Skip any question the supplied material already answers, and say where the answer came from.
- Offer choices when the user is unsure ("a list of services, or one page per service?"). Keep an "I don't know yet" option; record it as an open question, not as a guess.
- After each round, restate what was learned in three to six lines before asking the next round.
- Small sites usually need two rounds, medium three, large four or more. Stop asking when the remaining unknowns do not change the plan; list them as open questions.
- Never fill a gap with invented client data (names, numbers, prices, addresses, testimonials). Use a labelled assumption or an open question.

## Question bank

Round marks: R1 asks in the first round, R2 later, "if" only when the earlier answer makes it relevant.

| Topic | Questions | Round |
|---|---|---|
| Goal | What should the site make visitors do (call, buy, book, sign up, read)? How will you know it works? | R1 |
| Audience | Who are the main visitors? What do they need to find in the first minute? Which devices do they mostly use? | R1 |
| Must-have features | Blog or news, contact or quote form, store, bookings, member area, events, downloads, multiple languages? | R1 |
| Hosting and environments | Do you already have hosting and a domain? Who manages them? Is there a staging site? Who will publish changes? | R1 |
| Build tool | Where will the site be built: wp-env, a local app, Docker, WordPress Playground or the host's staging? Is it already installed? | R1 |
| Current site | Is there a site today? What must be kept (pages, URLs, content, rankings, forms)? | R1 |
| Pages and content | Which pages are certain? Is the content written, or does it still need to be written? Who supplies photos? | R2 |
| Structured content | Does any content repeat with the same fields (team, services, products, cases, locations)? | R2 |
| Brand | Logo, colors, fonts, tone of voice, brand rules? Sites you like or dislike, and why? | R2 |
| Editing | Who will edit the site, and how comfortable are they with WordPress? What must they never be able to break? | R2 |
| Integrations | Email marketing, CRM, payment, analytics, chat, maps, social feeds? | R2 |
| SEO | Main search terms, local business listing, existing rankings to protect? | R2 |
| Privacy and law | Which country or countries do visitors come from? Do you collect personal data, use tracking or cookies, or send newsletters? | R2 |
| Accessibility | Is there a required standard or customer requirement (for example WCAG 2.2 AA)? | R2 |
| Performance | Any traffic peaks, large media, or visitors on slow mobile connections? | if |
| Store | Products or services, how many, physical or digital, shipping regions, taxes, payment methods? | if |
| Languages | Which languages, all content or part of it, one domain or several? | if |
| Members | What do members get, who approves them, paid or free? | if |
| Timeline and budget | Launch date, fixed events, budget range for paid plugins, themes and hosting? | R2 |
| Maintenance | Who updates WordPress and plugins after launch? How often are backups needed? | R2 |

Privacy answers go into the plan as open questions for the client or their adviser. The plan names the law to check (for example LGPD in Brazil or GDPR in the EU) but does not conclude what the law requires.

## Direction format

Offer two or three directions. Each one is short:

```text
Direction A: <name>
Sitemap: <top-level pages, one line>
Visual thesis: <sentence on color, type, spacing and image style>. <sentence on what it must feel like and what it must avoid>.
Allowed patterns: <three to six section types, for example full-width photo hero, three-card services row, testimonial strip>
Build approach: <block theme | child of a block theme | classic or hybrid theme | Elementor | headless> and why
Trade-offs: cost, ease of editing, speed, maintenance
```

Guessability test: if the thesis would fit any site in the same sector, rewrite it with something specific to this client (their place, method, audience or material).

From a design, skip the directions: the design is the direction. Present its inventory and gaps instead and ask the user to confirm what to fill in.

## Plan template

Save it as `site-plan/SITE-PLAN.md` unless the user chose another path; supplied design files can go in `site-plan/design/`. Write the plan in the user's language. Keep the headings, the `Status:` values and the skill names in English so the plan can be resumed and reviewed. Replace every value in angle brackets.

```markdown
# Site plan: <site name>

Status: <briefing-confirmed | direction-chosen | draft | approved>
Size: <small | medium | large>
Mode: <from scratch | from a design>

## Summary
<three to five lines: who the site is for, what it must make them do, how it will be built>

## Brief
### Facts
- <confirmed by the user or the supplied material, with the source>
### Assumptions
- <a working guess, labelled, to confirm>

## Direction
<the chosen direction, or the design inventory summary>
Rejected: <other directions and the reason>

## Sitemap
- <page>
  - <child page>

## Pages and templates
| Page or template | Purpose | Sections | Data shown |
|---|---|---|---|

## Content model
| Content | Type (page, post, custom post type, taxonomy, field) | Fields | Who edits |
|---|---|---|---|

## Design tokens
| Token | Value | theme.json key |
|---|---|---|

## Plugins
| Need | Core feature or plugin | Why | Cost | Removal plan |
|---|---|---|---|---|

## Environments
| Environment | Where | PHP | WordPress | Database | Who has access |
|---|---|---|---|---|---|
| Build | | | | | |
| Staging | | | | | |
| Production | | | | | |
Deployment path: <how code and content move from build to staging to production>
Rollback: <how to return to the last working version>

## Build phases
### Phase 0: environments
- [ ] 0.1 <set up the build environment with the production versions>
  Acceptance: <check>
  Request: <ready request>
- [ ] 0.2 <send a test deploy of an empty theme to staging or production>
  Acceptance: <the test page loads over HTTPS on the target>
  Request: <ready request>
- [ ] 0.3 <keep the plan folder (site-plan/ by default) out of every deploy and package>
  Acceptance: <the deployed files and any release archive contain no plan folder>
  Request: <ready request>
### Phase 1: <name>
- [ ] 1.1 <task>
  Acceptance: <check>
  Request: <ready request>

## Launch checklist
- [ ] HTTPS works on every URL and redirects from HTTP
- [ ] Backups of the database and files run, and one restore was tried on staging
- [ ] Site email (forms, password resets) is delivered
- [ ] Caching is on and logged-in pages are not cached
- [ ] Domain and DNS point to production; old URLs redirect
- [ ] Privacy policy page is published and linked from every page
- [ ] "Discourage search engines" is unchecked in production and checked on staging
- [ ] Staging is also behind a password, because search engines may ignore that request
- [ ] Security, performance and accessibility reviews were run

## Risks
| Risk | Effect | Mitigation |
|---|---|---|

## Open questions
- <question> (owner: <who answers>)

## History
- <date>: <what changed and who approved>
```

## Ready requests

Each task carries one request that a person can paste into their agent. It names the task, the skill as a hint and the authorization:

```text
Use wp-devkit-theme-development, if installed, to implement task 1.2 of site-plan/SITE-PLAN.md
on the build environment. This is a new build authorized by the approved plan.
Run the task's acceptance check and report what you could not run.
```

- Send it as a normal message. In Claude, the short commands such as `/wp-devkit:theme` only scan; `/wp-devkit:design` is the one short command that implements.
- Without the named skill, the task description and the acceptance check must still be enough to do the work.
- Some sibling skills stop with `Not used in this project` while their technology is absent: WooCommerce, ACF or SCF, Elementor, WPGraphQL and Playground Blueprints. The task that first brings one of them in (installing the plugin, creating the first Blueprint) is written in plain words with no skill hint. Later tasks, once it is present, name the skill.
- Phase 0 tasks set up the build environment in plain words (a local app, WordPress Playground or the host's staging) and name `wp-devkit-wpcli-and-ops` for moves and URL changes and `wp-devkit-ci-cd-and-release-engineering` for automated deploys.
- Launch tasks name `wp-devkit-security-review`, `wp-devkit-performance-review` and `wp-devkit-wcag-review`.

## Review checklist

| Check | Finding when |
|---|---|
| Structure | A template section is missing, `Status:` has another value, or History is empty after an approval |
| Evidence | A client fact has no source, or an assumption is written as a fact |
| Fit | A must-have feature from the brief has no page, content type, plugin or task |
| Decisions | A build approach, plugin or environment has no reason |
| Environments | Build and production versions differ, there is no staging, no deployment path or no rollback |
| Phase 0 | Missing, it has no test deploy, or nothing keeps the plan folder out of deploys |
| Tasks | A task has no acceptance check, no ready request, or depends on a tool the user does not have |
| First use | The task that first brings in WooCommerce, ACF or SCF, Elementor, WPGraphQL or a Playground Blueprint names the skill for that technology, which stops while it is absent |
| Store, languages, forms | Payment, tax, shipping, translation or form destination decisions are missing |
| Launch | The launch checklist was removed or shortened without a reason |
| Gaps | A design-based plan has no gaps list, or a gap has no owner |

## Sources

Research date: 2026-11-10.

- https://wordpress.org/documentation/article/settings-reading-screen/ (search engine visibility setting)
- https://wordpress.org/documentation/article/settings-privacy-screen/ (privacy policy page tool)
- https://developer.wordpress.org/advanced-administration/security/backup/ (backups)
- https://www.w3.org/WAI/planning/ (planning and managing accessibility, laws and policies)
- https://www.w3.org/TR/WCAG22/ (WCAG 2.2)
- https://www.planalto.gov.br/ccivil_03/_ato2015-2018/2018/lei/l13709.htm (LGPD, Brazil, Lei 13.709 of 14 August 2018)
- https://eur-lex.europa.eu/eli/reg/2016/679/oj (GDPR, European Union; unverified: EUR-Lex returned no content to the research tools)
