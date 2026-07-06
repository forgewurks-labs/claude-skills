# Forgewurks Claude skills

Official distribution point for the Claude Code skills issued to Forgewurks closed-beta
participants. If you got a skill any other way (email attachment, base64 blob, a zip from
someone), don't install it — install from here. This repo is the source of truth for what
a current copy looks like: every release is versioned in git, and each skill's frontmatter
carries its `version`, `issued` date, and this repo's URL.

## Skills

| Skill | What it does |
| --- | --- |
| `notion-linear-mcp` | Routing + connector mechanics for your private Linear team and Notion teamspace, the community publish path, and the Notion write discipline. |

## Install

Steps are labeled **[you]** (run it yourself in a terminal) or **[agent]** (hand it to
Claude Code).

1. **[you] Clean up superseded copies, if present.** Earlier platform wiring installed two
   skills this repo supersedes: an emailed copy of `notion-linear-mcp`, and `org-context` —
   the *operator's* edition of the routing rules, which defaults writes to the operator's
   own team/teamspace instead of yours. Leaving `org-context` in place would misroute your
   work into the wrong compartment, which is why it must go. Neither existing is fine (a
   fresh machine has neither):

   ```sh
   [ -d ~/.claude/skills/notion-linear-mcp ] && rm -r ~/.claude/skills/notion-linear-mcp
   [ -d ~/.claude/skills/org-context ] && rm -r ~/.claude/skills/org-context
   ```

2. **[you] Install the skill:**

   ```sh
   npx skills add forgewurks-labs/claude-skills --skill notion-linear-mcp -g
   ```

   (If the installer prints a "PromptScript does not support global skill installation"
   line, that's expected and harmless — the skill still installs for Claude Code.)

3. **[you] Connect Notion and Linear**, once, inside Claude Code: run `/mcp` and
   authenticate both connectors (browser sign-in each).

4. **[agent] Verify routing.** Ask Claude: *"where do I track this?"* — it should default
   to your private team/teamspace, with the community surface only as a deliberate publish
   target.

5. **[agent] Verify write access (optional but recommended).** Ask Claude to create a test
   issue titled `onboarding write check` in your private Linear team, then delete it (or
   mark it canceled). This proves your connector has edit rights — better to learn now than
   the first time you file real work.

## Updating

Rerun the install command (step 2). Skills are static snapshots — when the canonical
standards change, a new version lands here and the old copy on your machine does not
update itself. Claude will nudge you if its copy's `issued` date is more than 90 days old.

Don't hand-edit an installed skill. If something in it looks wrong or stale, reinstall
from here first; if the problem survives, email Rob.

## Provenance

These skills are derived by the operator from the canonical platform/standards repos and
published here for participants who don't carry a platform checkout. The derivation
process and source of truth live on the operator side; this repo is the publish surface.
