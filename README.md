# summoning — Claude Code plugin marketplace

A personal marketplace hosting `agents-summoning`: a spellbook for summoning
other AI agents (CodeRabbit, Codex, Copilot, Devin, OpenHands, Jules, Claude
Code, Dependabot) on GitHub pull requests and issues. Maintain the skill here
once; opt any project into it — including **Claude Code on the web / cloud**
sessions.

Harvested from
[open-learning-exchange/myplanet `docs/AGENT_SPELLBOOK.md`](https://github.com/open-learning-exchange/myplanet/blob/master/docs/AGENT_SPELLBOOK.md),
where the grid was fact-checked live by the agents themselves
([PR #15436](https://github.com/open-learning-exchange/myplanet/pull/15436),
2026-08-07/08; follow-up [PR #15499](https://github.com/open-learning-exchange/myplanet/pull/15499),
2026-08-09). The one distinction the skill exists to enforce: **Reviewers
speak; Doers act** — an unleashed Doer mention defaults to commits on your
branch.

The skill keeps the spellbook's layering:

| Layer | Where | Portability |
|---|---|---|
| **The Grid** — how each agent is summoned, what it does, which file binds it | `SKILL.md` | any repo these agents are installed on |
| **The Laws of Summoning** — blast radius, leashes, backtick trap, silence diagnosis, timeline auditing, cost | `SKILL.md` | portable |
| **Field notes** — dated observations with receipts | `NOTES.md` | grow your own per repo |
| **Connecting** — what each agent needs installed before a summon lands, and what "not connected" looks like | `references/connecting.md` | per account/repo |

## Structure

```text
SKILL.md                                 # the grid, the laws, choosing an agent, casting procedure
NOTES.md                                 # the history: dated evidence behind the grid (myplanet experiments)
references/
└── connecting.md                        # per-agent connection checklist: install, verify, silence tells
.claude-plugin/marketplace.json          # marketplace catalog
plugins/agents-summoning/
├── .claude-plugin/plugin.json           # plugin manifest
└── skills/summoning/
    ├── SKILL.md      -> ../../../../SKILL.md
    ├── NOTES.md      -> ../../../../NOTES.md
    └── references    -> ../../../../references
```

`SKILL.md` stays timeless — mechanics only, each claim pointing at `NOTES.md`
for its receipts. New evidence goes in `NOTES.md`; the grid changes only when
behavior does.

The repo root doubles as a skill directory so it works when mounted as a git
submodule (at `.agents/skills/agents-summoning/`); the three symlinks under
`plugins/` project the same files onto Claude Code's plugin path.

⚠️ **Those symlinks need `core.symlinks=true`.** Where Git runs without it —
Windows outside Developer Mode — the checkout writes them as plain files
containing the target path, and a loader will read the literal string
`../../../../SKILL.md` as the skill body instead of failing loudly. Check with
`git config core.symlinks` and
`test -L plugins/agents-summoning/skills/summoning/SKILL.md`.

## Hosting

This marketplace is hosted at `dogi/agents-summoning`. The
`.claude-plugin/marketplace.json` catalog lives at the repo root so Claude Code
can discover it when the repo is added as a marketplace.

## Use it in the terminal (CLI)

```text
/plugin marketplace add dogi/agents-summoning
/plugin install agents-summoning@summoning
/reload-plugins
```

Then invoke `/agents-summoning:summoning` — or just ask to "get codex to review
this", "ask coderabbit to re-review", "have openhands fix it", or "why didn't
jules respond" — the description auto-triggers it.

## Use it on Claude Code web / cloud

Cloud sessions can't see your local `~/.claude`, and user-scoped `enabledPlugins`
does **not** carry over. Declare the marketplace + plugin in the target repo's
`.claude/settings.json` (this file is part of the clone, so the cloud VM installs
the plugin at session start — needs network access to GitHub, which the default
allowlist covers):

```json
{
  "extraKnownMarketplaces": {
    "summoning": {
      "source": {
        "source": "github",
        "repo": "dogi/agents-summoning"
      }
    }
  },
  "enabledPlugins": {
    "agents-summoning@summoning": true
  }
}
```

Commit that to each repo where you want the skill available in web sessions.
The skill itself stays maintained here — bump `version` in `plugin.json` on
each release: with a `version` field present, existing installs only pick up
changes when it is bumped (fresh installs always fetch the `main` tip, and
omitting `version` entirely would switch update detection to commit SHAs).

## Use it from OpenHands

OpenHands auto-loads `.agents/skills/<name>/SKILL.md`. Add this repo as a
submodule in the target repo so the file is physically present:

```bash
git submodule add -b main https://github.com/dogi/agents-summoning.git .agents/skills/agents-summoning
```

On a fresh clone of the target repo the committed gitlink arrives
uninitialized — the skill is silently skipped until
`git submodule update --init --recursive` runs. Wire that into
`.openhands/setup.sh` so sessions bootstrap it before skill discovery.

Bump the pin after every merge here, or OpenHands keeps seeing the old revision
while Claude Code's marketplace fetch tracks this repo's `main` tip:

```bash
git submodule update --remote --checkout -- .agents/skills/agents-summoning
```

then commit the gitlink (`--checkout` matters: it forces the pin even where the
target repo configures `merge`/`rebase` update modes, and the submodule sits on
a detached HEAD). Bonus: `.agents/skills` is also one of Copilot's project
skill directories (alongside `.github/skills` and `.claude/skills`), so the
same submodule serves Copilot's coding agent — provided its checkout
initializes submodules, which a plain clone does not: wire
`git submodule update --init --recursive` into Copilot's setup steps the same
way `.openhands/setup.sh` does it for OpenHands.

## Use it from Jules

Jules: see [references/connecting.md](references/connecting.md#loading-the-spellbook-into-jules).
