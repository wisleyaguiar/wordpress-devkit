---
description: Plan a WordPress site from a brief or a finished design, with environments and phased build tasks
argument-hint: "[idea-brief-or-design-path]"
---

Follow **wp-devkit-site-planning** and its engineering contract.

**Target**: $ARGUMENTS (if empty, start the briefing in the current working directory).

Use the skill's "Implementation workflow". Resume an existing plan (the path given, `site-plan/SITE-PLAN.md`, or a `SITE-PLAN.md` found in the project), and never overwrite an existing plan without asking. Ask before writing, stop at each gate and write only the plan file. Do not build, install or publish anything. Building is not part of this command: each build task needs its own request after the plan is approved.
