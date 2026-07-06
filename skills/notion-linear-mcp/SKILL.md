---
name: notion-linear-mcp
description: >
  How to drive the Notion and Linear connectors for Forgewurks tenant work — a standalone
  snapshot that needs no platform checkout. Use when filing or updating work and you need the
  actual tool mechanics: creating/updating a Linear issue in YOUR private team (save_issue
  fields, labels, relations), or writing to YOUR private Notion teamspace (schema DDL, both-way
  relations, the formula-dialect trap, page-write value formats). Also covers WHERE work goes
  (all work items → your Linear team; Notion = your private teamspace KB), publishing to the
  Forgewurks Labs community surface, and Notion write discipline. Trigger phrases: "file /
  create / update a Linear issue", "add this to Notion", "which teamspace", "log a decision /
  parameter / claim", "where do I track this", "set up a Notion database / relation / formula",
  "publish this", "share with the community", "make this public".
metadata:
  version: 1.2.0
  issued: 2026-07-06
  source: https://github.com/forgewurks-labs/claude-skills
---

# Notion & Linear connector guidelines (tenant snapshot)

> **This is a curated snapshot, not the source of truth.** The canonical standards live with
> the operator (the platform/standards repos). This copy is published at
> https://github.com/forgewurks-labs/claude-skills — if a rule here seems wrong or stale,
> don't patch it locally: reinstall the latest from that repo, and flag it to Robert if the
> problem survives. Same discipline as everything below: promote forward from the source,
> never fork the rules.
>
> **Freshness check:** the frontmatter above carries `issued`. If it is more than 90 days
> old, mention that to the user once per session and suggest reinstalling from the repo.

## Your compartment (routing — read this first)

The workspaces are **compartmentalized**. You have a **private Linear team** and a **private
Notion teamspace**; every write of yours goes to your own compartment.

- **Resolve your compartment live at the start of a session** — never hardcode IDs or names.
  Linear: `list_teams` (you only see teams you belong to — the private team that isn't a shared
  surface is yours). Notion: `notion-get-teams` / `search` for your teamspace's home page.
  Your visibility **is** the permission boundary: if you can't see a team or teamspace, it isn't
  yours to write to, and that's correct.
- **The public community surface is a publish target, never a default.** The teamspace
  `Forgewurks - Closed Beta Sandbox` (its home page is titled "HQ") and the Linear team
  `Forgewurks Labs` are **public** community surfaces — write there only on the user's explicit,
  deliberate choice to publish (see "The community surface" below). Nothing proprietary or
  financial goes there, ever.
- **Never relate across teamspaces.** Your teamspace carries its **own full mirror** of the
  database set. Relations stay internal to it — a relation into another teamspace's databases
  (including any operator/platform teamspace you happen to see) crosses the boundary the
  compartments exist to draw.
- **Routing rule of thumb:** *what to do next* → Linear (your team, always — software **and**
  hardware). *A committed value/decision/claim* → Notion (your teamspace). *The derivation /
  rationale / code* → your repo. *Genuinely open material you choose to share* → the community
  surface, on an explicit publish.

## What you should see on each plane — and what to do if you see more

A correctly-provisioned participant sees **exactly** the surfaces below, no others. At the start
of a session, enumerate what's actually visible — Linear `list_teams`, Notion `notion-get-teams`
(plus a `search` for teamspace home pages), GitHub the orgs/repos you can reach — and match it
against this table. Each surface is one of three kinds:

| Plane | Surface | Kind | Meaning |
|---|---|---|---|
| Linear | Your private team | **Yours** | Default write target — all your work items. |
| Linear | `Forgewurks Labs` team | **Public** | Community surface; publish-only, never a default. |
| Notion | Your private teamspace | **Yours** | Default write target — your KB mirror. |
| Notion | `Forgewurks - Closed Beta Sandbox` (home page titled "HQ") | **Public** | Community surface; publish-only. |
| GitHub | Granted `forgewurks-labs` repos | **Public** | Community repos; access is per-repo, granted by the operator. |

Everything else is **operator-internal** and you should not be able to see it — the operator's
platform team/teamspace (anything named "Forgewurks Platform" / "Forgewurks - Platform"), the
`forgewurks-platform` GitHub org, and any **other participant's** private team or teamspace. The
whole point of the compartments is that these are invisible to you.

**If you can see any surface that isn't in the table above — an operator/platform team or
teamspace, a `forgewurks-platform` repo, another participant's private compartment, or anything
you can't confidently classify as Yours or Public — stop and tell the user to report it to
Robert.** Do not write to it, do not read from it, and do not assume it's fine just because it's
visible: a surface you can see that isn't on this list is a **provisioning leak**, not a new
capability. Flagging it is how a misconfigured grant gets caught at first session instead of going
unnoticed.

## The community surface — publishing & interacting (opt-in)

Alongside the private compartments sits the public **Forgewurks Labs** surface, on all three
planes:

- **Notion:** the `Forgewurks - Closed Beta Sandbox` teamspace — it carries its own full DB
  mirror, same shape as yours.
- **Linear:** the `Forgewurks Labs` team — work items on open/community projects.
- **GitHub:** the `forgewurks-labs` org — community-visible project repos. Access there is
  granted per org/repo by the operator; contribution flows through normal GitHub issues/PRs on
  the repos you can see.

Rules of the surface:

- **Publishing is a deliberate human act.** Never route a write there yourself — publish only
  when the user explicitly chooses to, and if it's ambiguous whether something is meant to be
  public, ask. Treat a publish as irreversible: public content may be cached or indexed even if
  deleted later.
- **What belongs there:** your **own**, genuinely open material — community/demo content, open
  project pages, work items on shared community projects. **What never goes there:** anything
  proprietary, financial, operator-internal, or belonging to another participant — and nothing
  from your private compartment that hasn't been explicitly cleared for publication.
- **Interaction is encouraged — and always each participant's choice.** Published projects are
  an invitation to engage: comment on Sandbox pages, relate your Sandbox pages to others'
  Sandbox pages, file or discuss issues on `Forgewurks Labs` projects, open issues/PRs on labs
  repos. Interaction stays **inside** the public surface — the no-cross-teamspace rule is
  unchanged, so Sandbox pages relate only to Sandbox pages, never into your (or anyone's)
  private teamspace.
- **Same mechanics as below** — resolve the Sandbox's / labs team's live IDs first; the
  `save_issue`, schema-DDL, and page-write semantics are identical.

## Connectors prerequisite

Filing into Linear/Notion needs the **connectors authenticated** once via **`/mcp`** (Settings →
Connectors). If a Notion or Linear action fails for lack of a connector, walk the user through
enabling it in `/mcp` rather than failing silently. Read/write is enforced at each product's
**permission layer** (view / comment / edit) — view-only means you can query but not write.

## Linear — mechanics

- **Workspace shape:** your private team, one project per repo/tool. File into the matching
  project; `list_projects` scoped to your team to see what exists.
- **Tools:** read `list_*` / `get_*`; write `save_issue` (create **and** update), `save_project`,
  `save_comment`, `save_document`, `create_issue_label`.
- **`save_issue`:** `title` + `team` (yours) required to create. Pass `id` (e.g. `ABC-47`)
  **only** to update — omitting it always creates, so never pass it on a create. `description`
  is Markdown sent **directly** (real newlines, no escaped `\n`); mention users with
  `@displayName`. `priority`: 0 none / 1 urgent / 2 high / 3 medium / 4 low. The field is
  **`assignee`** (not `assigneeId`). `labels` = array of label **names**. Relations
  (`relatedTo` / `blockedBy` / `blocks` / `duplicateOf`) take identifiers like `["ABC-36"]` and
  are append-only; `parentId` makes a sub-issue. Batch independent creates in parallel.
- **Labels:** `list_issue_labels` before using one — reuse the team's existing names; create new
  ones deliberately with `create_issue_label`, not ad-hoc variants of existing ones.
- **Conventions:** search before you create (your team is the single backlog — relate/update,
  don't duplicate); self-contained issues (title reads as an action, description cites the
  concrete source); cite the source rather than pasting rationale that will drift.
- **Git loop:** each issue exposes a `gitBranchName`; cite the issue key (e.g. `ABC-47`) in the
  PR and Linear auto-links it.

## Notion — mechanics

- **Layout:** your teamspace carries its **own full DB mirror** under its home page. DBs are
  grouped **Things** (Projects, Products, Equipment, Materials, Components, Sites, Customers) /
  **Statements** (Decisions, Docs, Parameters) / **Reference** (Quantities, …). `fetch` your
  teamspace's DBs to resolve current IDs first; **relations stay internal to your teamspace**.
- **Tools:** read `search` / `fetch` / `get-users` / `query-database-view`; schema
  `create-database` / `update-data-source` / `create-view`; data `create-pages` / `update-page` /
  `move-pages` / `create-comment`.
- **database vs data source:** schema + rows live on the **data source** (`collection://<uuid>`,
  shown in `<data-source url="…">` from `fetch`). `create-database` / `update-data-source` /
  `query-database-view` want the **data source** id — not a database URL.
- **Schema = SQL DDL:** `CREATE TABLE` / `ADD COLUMN` / `ALTER COLUMN "X" SET <type>` / `RENAME` /
  `DROP`. Columns double-quoted, option values single-quoted. Types include `TITLE`, `RICH_TEXT`,
  `NUMBER [FORMAT 'dollar']`, `SELECT(...)`, `STATUS`, `FORMULA('…')`, `RELATION(...)`,
  `ROLLUP(...)`, `UNIQUE_ID [PREFIX 'X']`. Select options are a **fixed vocabulary** — seed in
  the DDL, extend deliberately. (`status` type ≠ `select` type — pick deliberately.)
- **Relations resolve both ways:** `RELATION('<target_ds>', DUAL 'SyncedName')` authored on
  **one** side auto-creates the back-relation — author each pair exactly once (both sides =
  duplicates). Self-relations use the `DUAL 'name' 'id'` form. Run schema mutations that share a
  back-relation target **sequentially**, not in parallel.
- **Formula dialect ≠ the in-app editor:** the API **rejects `let`/`lets` and list literals**
  (`[…]`, so no `join`/`filter` over a literal) — they exist only in the Notion UI editor and
  fail with `validation_error`. What works: `prop`, `if`, `format`, `replaceAll` (RE2 regex),
  `empty`, `and`/`or`, `==`/`!=`, string `+`. Build list-like output by inline concatenation,
  then strip the leading separator with `replaceAll(x, "^ ", "")`. **You cannot read a formula's
  rendered value back through the connector** (it returns an opaque `formulaResult://` handle) —
  verify formula output in the Notion UI.
- **Writing pages:** parent `data_source_id` (DB row) or `page_id` (normal page); `properties`
  is a name→value map keyed to the schema. Value formats: date → `date:{prop}:start` / `:end` /
  `:is_datetime`; number → a real number (not a string); checkbox → `"__YES__"` / `"__NO__"`;
  person → a JSON-array **string** of user IDs (resolve with `get-users {user_id:"self"}`);
  relation → JSON array of page URLs; a property literally named `id`/`url` needs a
  `userDefined:` prefix. Page bodies are **Notion-flavored Markdown** — read the
  `notion://docs/enhanced-markdown-spec` resource rather than guessing; don't repeat the title
  in the body.
- **Gotcha — Cloudflare WAF:** payloads containing shell-command strings (e.g. `python -m …`)
  get blocked before reaching Notion. Reword the command as prose; chunk very large page bodies
  if a create bounces.

## Notion write discipline (what keeps the KB trustworthy)

1. **One home per fact.** Derivation stays in the repo; the committed *value* others consume
   lives in Notion. No fact has two homes — it's not a mirror.
2. **Skeptical by default.** Migrated knowledge enters **Unverified**. `Source` (who said it) is
   provenance, never evidence. **Verified** is earned and needs a recorded `Basis` (link, data,
   experiment). Rank is not a basis.
3. **Classify correctly.** A *decision* ("we will ship X", true-by-authority) → Decisions. A
   *factual claim* ("X is feasible", true-or-false regardless of who said it) → Docs with
   status. Don't dress a claim as a decision.
4. **Numbers are first-class.** A contestable value others depend on is a **Parameter** (Value +
   Unit + Quantity + Source/Basis/Status), not a sentence. Definitional/decided numbers (a SKU,
   a committed size) are plain properties.
5. **Search before you create** (scope the search to the data source) — promote-and-point,
   don't duplicate.
6. **Don't add a database** without clearing the four-gate test out loud (distinct kind / own
   identity / cross-functional demand / no worse alternative).

## The invariant (both trackers)

The repo is the **system of record** for rationale / derivations / schemas; Linear and Notion
are **projections** that own live status / schedule. Distill resolved decisions back into the
repo, point the repo at the tracker — **never bidirectionally sync.**
