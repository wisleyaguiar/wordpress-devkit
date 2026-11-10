# Pulling production down and pushing changes up

Contents: the direction rule; before a pull; pull with SSH and WP-CLI; uploads; local safety; personal data; wp-env; hosts without SSH; what goes up; failure symptoms; sources.

Never run any of this from a review. Each step runs against a real environment only as a separately authorized task. The production side of a pull is read-only: export and copy, never write there.

## The direction rule

| What | Direction | Why |
|---|---|---|
| Database and uploads | Down, once the site is live: production to staging or local | Production holds the real content, orders, form entries and users |
| Code (theme, own plugins, mu-plugins) | Up: git to staging to production, through the deployment path | The repository is the source of code, not a server's disk |
| `wp-config.php` | Never moves | Each environment keeps its own database credentials, salts and constants |

After launch, never import a local or staging database into production. Production keeps writing after the pull (orders, form entries, comments, users, posts), and a full import deletes all of it. The rule holds from the first real write on production. Before that, at the first launch, the build database may go once to an empty production as a staged move: produce one transformed dump per target URL with `wp search-replace --export` (see `search-replace-and-migrations.md`), import the staging dump on staging and check it, then import the production dump on production after a backup.

## Before a pull

- Identify both ends by a deliberate fact (`wp option get home` on each), not by a folder or alias name. Define production as an alias in `wp-cli.yml` (`@prod`) so the target is explicit; see `environment-safety-and-recovery.md`.
- Check out the code production runs (the deployed tag or commit), so the copied database meets matching code.
- Back up the local database first (`wp db export`): the import replaces it.
- Match the table prefix: `wp db prefix` on production must equal `$table_prefix` in the local `wp-config.php`, or the site looks freshly installed after the import.
- Decide which personal data may leave production (see Personal data).
- Export outside peak hours: dumping a large database adds load to production.

## Pull with SSH and WP-CLI

1. Set the local safety constants (see Local safety) in the local config before the import, so no request runs with production settings.
2. Export from production to a file outside the repository and outside every web root. `--single-transaction` is passed to mysqldump and gives a consistent InnoDB dump without locking:
   `wp @prod db export - --single-transaction > ~/site-dumps/prod-$(date +%F).sql`
   Check the file is not empty and its last line starts with `-- Dump completed`.
3. Import locally: `wp db import ~/site-dumps/prod-<date>.sql`. Until the guards are on, add `--skip-plugins --skip-themes` to each WP-CLI command, so production plugins with live keys do not boot (mu-plugins still load, and the skipped plugins' hooks do not run).
4. Replace the URL before opening the site in a browser, or WordPress redirects to production. Dry run first, then the same command without `--dry-run`:
   `wp search-replace 'https://www.example.com' 'http://localhost:8888' --skip-columns=guid --all-tables-with-prefix --skip-plugins --skip-themes --dry-run --report-changed-only`
   Builder JSON with escaped slashes needs the checks in `search-replace-and-migrations.md`.
5. Apply the database-level guards from Local safety, still with the load flags, then flush: `wp cache flush`, `wp rewrite flush`, and regenerate builder CSS (Elementor: `wp-devkit-elementor-development`).
6. Verify: `wp option get home` and `wp option get siteurl` show the local URL, you can log in, pages show their images, and no email left the machine.
7. Delete the dump once the copy works, or keep it encrypted. It holds password hashes, email addresses and API keys stored in options.

## Uploads

- Copy down with rsync, dry run first. The trailing slashes copy the folder contents rather than the folder:
  `rsync -av --dry-run user@host:/path/to/site/wp-content/uploads/ wp-content/uploads/`
  then the same command without `--dry-run`. Never swap source and destination: that sends local files to production.
- Large libraries: copy only recent year folders (`uploads/2026/`), or add a local web server rule that sends requests for missing upload files to the production URL.
- Generated folders inside uploads (builder CSS, cache plugin output) can be skipped and rebuilt locally.

## Local safety

Apply before the first page load of the copy:

| Risk | Guard |
|---|---|
| Plugins behave as on production | Set `WP_ENVIRONMENT_TYPE` to `local` in the local config; when unset, WordPress reports `production` |
| Scheduled jobs copied from production run locally (renewals, newsletters, syncs) | Set `DISABLE_WP_CRON` to `true`; plugin job queues may run by other means, so also apply the next row |
| Real email, payments, CRM calls or webhooks | Before logging in, switch those plugins to test keys or deactivate them (`wp plugin deactivate <slug> --skip-plugins --skip-themes`, so their deactivation hooks do not call live services); send mail to a local mail catcher |
| Search engines index the copy | `wp option update blog_public 0` |
| Accounts are production's after the import | Set a local-only password for your own account with `wp user update`; never reuse a real one |

## Personal data

A production database holds customer names, email addresses, addresses and orders. Under laws such as the LGPD (Brazil) or the GDPR (European Union), a copy on a laptop is still processing of that data. The owner decides; record the decision where the project keeps its rules. In order of preference:

1. Leave out tables the task does not need, such as order, form-entry and log tables (`wp db export` with `--exclude_tables=`).
2. Or anonymize names and email addresses right after the import, before anyone works on the copy.
3. Keep any dump encrypted at rest and delete it after use. Never commit a `.sql` file.

## wp-env

- Use the Docker runtime: it has MySQL and `wp-env run`. The Playground and PHP runtimes use SQLite, so a MySQL dump does not import as it is.
- Run WP-CLI inside it with `npx wp-env run cli wp <command>`. Its working folder is the WordPress root.
- The dump must sit in a folder the container sees: add one under `mappings` in `.wp-env.json` and keep it out of git.
- wp-env defines `WP_HOME` and `WP_SITEURL` as `http://localhost:8888` (with the configured port). Use that URL as the search-replace target. Set `WP_ENVIRONMENT_TYPE` and `DISABLE_WP_CRON` in the `config` field.

## Hosts without SSH

- Database: export structure and data from the host panel (phpMyAdmin or the host's backup tool) or with a backup plugin.
- Uploads: SFTP, or a zip from the host's file manager.
- Then import and replace URLs locally with WP-CLI as above. Never replace URLs with a SQL `REPLACE` or a text editor on the dump: serialized values break (see `search-replace-and-migrations.md`).
- A migration plugin that replaces URLs itself must state that it handles serialized data. Check its result with a counted search afterwards.

## What goes up

| Change | How it reaches production |
|---|---|
| The whole build database, at the first launch | Once, as the staged move in The direction rule; never again after the first real write on production |
| Theme, own plugins, mu-plugins | Git, then the deployment path, staging first; automation belongs to `wp-devkit-ci-cd-and-release-engineering` |
| Third-party plugin and core updates | Update on staging and test, then update production after a backup |
| Settings, menus, pages and roles stored in the database | Repeatable steps: a reviewed WP-CLI script or a versioned upgrade routine, run on staging, then on production after a backup (`wp-devkit-migration-upgrade-review`) |
| Builder layouts (Elementor and others keep them in the database) | Export the template as JSON, import it on staging and check it, then production; or redo the edit on staging (`wp-devkit-elementor-development`) |
| New media | Upload it on production, or copy only the new files up |
| Content written locally | Recreate or import it item by item, never through a database import |

## Failure symptoms

| Symptom | Evidence | Fix direction |
|---|---|---|
| The local copy redirects to production | URL replace not run, or no local `WP_HOME` | Run the search-replace; check `wp option get home` |
| The site looks freshly installed after the import | Table prefix differs | Match `$table_prefix` to `wp db prefix` from production |
| Images missing locally | Uploads not copied | Copy uploads, or add the missing-file rule |
| Customers received email from the copy | Mail or cron ran before the guards | Stop, deactivate mail and job plugins, tell the owner |
| Recent orders or form entries are gone from production | A database was imported into production | Restore production from its latest verified backup (`environment-safety-and-recovery.md`) |
| The import fails on wp-env | PHP or Playground runtime (SQLite) | Use the Docker runtime |

## Sources

Research date: 2026-11-10.

- Running commands remotely (aliases, `--ssh`): https://make.wordpress.org/cli/handbook/guides/running-commands-remotely/
- `wp db export`: https://developer.wordpress.org/cli/commands/db/export/ ; `wp db import`: https://developer.wordpress.org/cli/commands/db/import/ ; `wp db prefix`: https://developer.wordpress.org/cli/commands/db/prefix/
- `wp search-replace`: https://developer.wordpress.org/cli/commands/search-replace/
- `wp option update`: https://developer.wordpress.org/cli/commands/option/update/
- `wp plugin deactivate`: https://developer.wordpress.org/cli/commands/plugin/deactivate/ ; `wp user update`: https://developer.wordpress.org/cli/commands/user/update/
- `wp_get_environment_type()`: https://developer.wordpress.org/reference/functions/wp_get_environment_type/
- `DISABLE_WP_CRON`: https://developer.wordpress.org/plugins/cron/hooking-wp-cron-into-the-system-task-scheduler/
- `@wordpress/env` (runtimes, `run`, `mappings`, `config`, default constants): https://developer.wordpress.org/block-editor/reference-guides/packages/packages-env/
- rsync (`-a`, `-v`, `--dry-run`, trailing slash): https://man7.org/linux/man-pages/man1/rsync.1.html
- LGPD, Brazil, Lei 13.709: https://www.planalto.gov.br/ccivil_03/_ato2015-2018/2018/lei/l13709.htm

Notes: the direction rule, the guards and the order of steps are operational practice, not documented requirements. Plugin job queues that run outside WP-Cron, and URL handling in migration plugins, vary by plugin: check the installed version.
