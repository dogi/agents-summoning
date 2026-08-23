---
name: summoning
description: 'Summon another AI agent — CodeRabbit, Codex, Copilot, Devin, OpenHands, Jules, Claude Code, Dependabot — on a GitHub pull request or issue: pick the right agent for the job, compose the mention so it does what was asked and nothing more, and know what to expect back. Reviewers speak, Doers act, and an unleashed Doer mention defaults to commits on the branch. Use whenever asked to summon, tag, ping, or ask another bot or agent for a review or a fix — "get codex to review this", "ask coderabbit to re-review", "have openhands fix it", "label this for jules", "which agent should look at this?" — or when a summon went silent and someone asks why an agent did not respond, pushed unexpectedly, or committed under a different identity.'
---

# Summoning other AIs on PRs

**Reviewers** speak; **Doers** act — an unleashed Doer mention defaults to
commits on your branch. That's the one distinction to hold before casting
anything.

This skill has three layers. **The Grid** below is the timeless mechanics: how
each agent is summoned, what it does, and which file binds it — portable to any
repo these agents are installed on. The history — dated observations with
receipts, from the live experiments where each agent fact-checked its own grid
row — lives in **`NOTES.md`** next to this file. New evidence goes in the
notes; the grid changes only when behavior does, so read `NOTES.md` whenever a
grid row surprises you or you need the evidence behind a claim. The third
layer, **Connecting** (`references/connecting.md`), navigates a summoner
through what each agent needs installed and connected before a summon reaches
it at all — read it first on any fresh repo, and again whenever a summon goes
silent.

## The Grid

| Agent (identity) | Type | Summon | Pushes to your branch? | Standing instructions |
|---|---|---|---|---|
| **CodeRabbit** (`coderabbitai[bot]`) | Reviewer | auto on push (skips drafts & dependabot; auto-pauses after 5 reviewed commits) · `@coderabbitai review` / `full review` / `resolve` / `approve` (needs `reviews.request_changes_workflow: true`) / `fix ci [commit]` / `autofix [stacked pr]` / `resolve merge conflict` / `generate {docstrings, unit tests, sequence diagram}` / `configuration` / `pause`·`resume` / `help` — `ignore` only works in the PR **description**. Answers in minutes; chats in threads | only opt-in (`fix ci commit`, `autofix`, generators) | teachable per-repo **learnings** from PR discussion; config via `.coderabbit.yaml` — can ingest `CLAUDE.md` as code guidelines (`knowledge_base.code_guidelines`) |
| **Codex** (`chatgpt-codex-connector[bot]`) | Reviewer (+ cloud tasks) | `@codex review` · targeted: `@codex review for issues in <scope>` — 👀 ack in seconds, review in minutes, 👍 if clean. Auto-review on open/ready only if enabled in repo Codex settings | no — `@codex fix it` / `address that feedback` starts a cloud task that may update the PR **or** deliver a sibling branch/PR | `## Code Review Rules` in `AGENTS.md`; `codex` label, `*-codex/*` branches |
| **Copilot** (`Copilot` / `copilot-swe-agent`; Reviewers-UI reviews author as `copilot-pull-request-reviewer[bot]`) | Doer, instruction-following | `@copilot <ask>` (write-access users only) · Reviewers UI (Comment-only reviews — never approves, no auto re-review) · assign an issue (spawns its own `copilot/**` PR). Acks in ~30 s | **default on same-repository PRs** — mentions push to that branch; fork-origin PRs are unsupported; say "open a separate PR" to redirect | coding agent: `.github/copilot-instructions.md` + nearest `AGENTS.md`, with a root `CLAUDE.md`/`GEMINI.md` as the alternative (an `AGENTS.md` takes precedence and unbinds `CLAUDE.md`); code review: `AGENTS.md` but not `CLAUDE.md` |
| **Devin** (`devin-ai-integration[bot]`) | Doer, instruction-following | `@devin <ask>` — session link in ~10 s; one session **adopts** the PR, later mentions join it (no races from Devin). Reviews: **`/devin review`** triggers Devin Review in seconds and tolerates trailing text; the `@devin review` mention form goes **silent** with any extra text in the comment (receipts in `NOTES.md`), and "additional findings" are gated behind its web UI — a session can relay them | yes, unless leashed in the mention | Knowledge ingests `CLAUDE.md`/`AGENTS.md`; ⚠️ commit identity is a configurable "commit authoring mode" — audit via PR timeline, not `git log` |
| **OpenHands** (`openhands-ai[bot]`) | Doer, unleashable | `@openhands <ask>` · `openhands` label on an issue — "I'm on it!" + session link in ~10 s; **new session per mention**, and *any* mention (even "help") reads as "fix what's open" | yes — leash compliance is **mixed**: has both broken and honored explicit no-push leashes (see `NOTES.md`) | root `AGENTS.md` (auto-loaded memory) + skills from `.agents/skills/*/SKILL.md` (submodules, bootstrapped by `.openhands/setup.sh` — wiring in this repo's README); `.openhands/microagents/repo.md` also supported. All prompt-level, not a guardrail; commits authored `openhands` |
| **Jules** (`google-labs-jules[bot]`) | Issue-driven Doer | `jules` label on an issue (reliable) / Jules app. PR feedback only on **Jules-created PRs** via a submitted review (👀 ack per comment, then pushes fixes); acts on every review comment by default — opt-in Reactive Mode **narrows** that to explicit `@Jules` mentions. Mentions on foreign PRs go silent (no session owns the branch) | own Jules-created PRs only — including follow-up commits there on accepted review feedback (branch names vary: `jules-<slug>-<taskID>` on one repo, bare `<slug>-<taskID>` on another — don't key an audit on the prefix) | `AGENTS.md` + per-repo memory (no `CLAUDE.md` support); configurable commit-authoring modes — audit via PR timeline + the `Co-authored-by: google-labs-jules[bot]` trailer it leaves; quotas 15/100/300 tasks/day by plan |
| **Claude Code** (`claude[bot]` for reviews) | Session Doer + managed Reviewer | claude.ai/code session · `@claude review` **only where the repo has managed Code Review enabled and funded** — it bills $15–25/review against org overage credits, so check the repo's policy first (see `NOTES.md` for what one uncapped summon cost) | review: no, comments only; sessions: their own `claude/**` branch (session-ID suffix; first push with `git push -u`) | sessions read `CLAUDE.md`; can subscribe to PR events and drive to green; review model is server-side and undocumented, not admin-configurable |
| **Dependabot** (`dependabot[bot]`) | Scheduled | scheduled runs · `@dependabot rebase` / `recreate` / `ignore …` / `unignore …` / `show … ignore conditions` — on **its own PRs only** | own PRs | `.github/dependabot.yml` |

Config-dependent cells — CodeRabbit's chattiness and `approve` availability,
Codex auto-review, Claude Code managed review — come from the target repo's own
config (`.coderabbit.yaml`, Codex repo settings, claude.ai admin settings), not
from the vendor default. Read the repo's config before promising behavior.

## The surfaces — where a summon lands at all

The grid says how each agent behaves; this table says which page even reaches
it. Field-tested 2026-08-21, one summon per cell, receipts in `NOTES.md` —
and every ✖ assumes the agent is connected (`references/connecting.md`),
because not-connected is ✖ everywhere, silently.

| Agent | PR: review | PR: chat ask | Issue: mention | Issue: label / assign |
|---|---|---|---|---|
| **CodeRabbit** | ✔ auto + commands | ✔ answers in-thread | ✔ answers | — |
| **Codex** | ✔ `@codex review` | ⚠ needs the asker's connected Codex account (bounces with a signup link otherwise) | ✔ answered here, account-routed | — |
| **Copilot** | ✔ mention-ask · Reviewers UI (comment-only) | ✔ ~30 s | ✖ silent | ✔ assign → its own `copilot/**` PR |
| **Devin** | ✔ `/devin review` only | ✔ `@devin <ask>` session | ✖ silent, both forms | ✖ |
| **OpenHands** | ✔ session per mention | ✔ | ✔ | ✔ `openhands` label → doer session |
| **Jules** | ✖ (reviews only its own PRs) | ✖ | ✖ | ✔ `jules` label — and it means "implement this issue" |
| **Claude Code** | ✔ managed review where enabled **and funded** | — (sessions live at claude.ai/code, not in mentions) | — | — |
| **Dependabot** | ✔ commands on its own PRs | — | — | — (schedule-driven via config) |

## The Laws of Summoning

1. **Know the blast radius before you cast.** A Reviewer mention produces
   comments; a Doer mention produces commits on the PR's branch unless you say
   otherwise. If you only want an opinion from a Doer, leash it in the mention
   itself: *"comment only — do not push"*.
2. **A leash is a request, not a restraint.** No vendor documents a hard
   "never push" switch — every standing-rule mechanism (microagents, Knowledge,
   `AGENTS.md`) is prompt-level context, and OpenHands has both broken and
   honored explicit no-push leashes (`NOTES.md` has the receipts). The
   enforceable controls are GitHub-side: branch protection/rulesets and app
   write permissions.
3. **Backticks don't defuse a mention.** Writing `` `@openhands prepping` ``
   inside a code span still spawns a real (unleashed!) session — GitHub
   notifies on the handle regardless of code formatting. When *talking about*
   an agent, drop the `@` (write "openhands prepping"); use the live handle
   only when you mean to summon. This matters most when the one writing the
   comment is itself an agent: a status comment that quotes a summon *is* a
   summon.
4. **Match the summon to the agent's trigger surface.** Devin Review fires on
   the bare literal `@devin review` — leashes and focus instructions in that
   trigger comment suppress it rather than steer it. Jules ignores mentions on
   PRs it didn't create; the `jules` label on an issue is the reliable summon.
   Copilot mentions only work from write-access users. Any OpenHands mention —
   even "help" — reads as "fix what's open", and each mention spawns a fresh
   session, so mentions in quick succession race each other.
5. **Silence means not-connected, outage, quota, or wrong surface — not bad
   phrasing.** Before rewording and re-summoning, check first: is the agent
   even **installed and activated on this repo**
   (`references/connecting.md`)? An unconnected agent fails perfectly
   silently, and personal repos need their own grant even when the org has
   the app. Then: is the agent rate-limited
   (CodeRabbit free-OSS: 5 reviews/dev/hr rolling, banners tell you)? does it
   own this PR (Jules)? is the vendor down (Devin outages present as silence or
   "Failed to start a Devin session")? Retry later instead of rephrasing — a
   re-summon while limited only re-queues.
6. **Audit by PR timeline, not `git log` alone.** Devin and Jules have
   configurable commit-authoring modes: their commits can wear the requesting
   user's git identity in both the author and committer fields. The PR timeline
   is the attribution that always holds; the one git-log signal that can survive
   is a `Co-authored-by:` trailer (Jules leaves one on every commit) — so read
   the trailers, but never take their absence as proof no agent was involved. OpenHands
   double-posts under both the mentioning user's account and the bot.
7. **Mind the bill.** Managed reviewers can cost real money per summon
   (Claude Code managed review: $15–25/review against non-refundable org
   overage credits, and a spend cap does not stop a mid-process review — see
   `NOTES.md`). When free reviewers cover the same ground, prefer them.
8. **One summon, one job.** Scope targeted asks (`@codex review for issues in
   <file>` is honored precisely). Don't stack multiple agent mentions in one
   comment — GitHub notifies every mentioned handle (Law 3), so expect each to
   fire.

## Choosing the agent

- **Fresh review of a PR** — CodeRabbit reviews pushes automatically;
  `@coderabbitai review` forces one only when auto-reviews are paused (against
  an already-reviewed commit it replies "Already reviewed"). `@codex review`
  is the strongest single-finding reviewer on record (`NOTES.md`). For a
  scoped second opinion: `@codex review for issues in <scope>`.
- **Fix on the current branch** — `@copilot <ask>` (default pushes; ~30 s
  ack), `@devin <ask>` (session adopts the PR; later mentions join it), or
  `@openhands <ask>`. OpenHands' edge is that it will actually *run* things —
  ask it to **verify by running**, not just review; it has caught failures in
  the live environment that pure reviewers missed (`NOTES.md`).
- **Work from an issue** — assign the issue to Copilot (spawns its own
  `copilot/**` PR), or apply the `jules` / `openhands` label.
- **Dependency PRs** — `@dependabot rebase` / `recreate` / the
  `ignore`/`unignore` family, on Dependabot's own PRs only. The old merge
  commands are gone.
- **CI is red** — `@coderabbitai fix ci [commit]` (opt-in push; Pro-tier
  gating applies) or `@codex fix it` (cloud task that may update the PR or
  deliver a sibling branch/PR).

## Casting procedure

1. **Identify the job**: review, fix on this branch, issue-driven work, or a
   dependency-PR command. That picks the column of the grid you care about.
2. **Pick the agent** from the grid and the guide above, and confirm the page
   you're casting on actually reaches it (the surfaces table). For the
   config-dependent rows, check the target repo's config
   (`.coderabbit.yaml`, `@coderabbitai configuration` prints the live one)
   before promising behavior.
3. **Compose the mention.** Use the agent's exact trigger literal where it has
   one (`@devin review`, `@coderabbitai review`). For any Doer, state the leash
   explicitly when commits aren't wanted. Everywhere else in the comment,
   refer to agents without the `@` (Law 3).
4. **Post it** on the PR or issue. Remember Copilot mentions need a
   write-access author to fire at all.
5. **Watch for the ack** on the grid's timescale (seconds for 👀/session
   links, ~30 s for Copilot, minutes for reviews). No ack → Law 5: diagnose
   quota / ownership / outage before rewording.
6. **Verify the result by the PR timeline** (Law 6) — what was pushed, by
   which session, on which branch — and only then report back.

## Cross-cutting facts

- **No vendor documents a hard "never push" switch** — every standing-rule
  mechanism (microagents, Knowledge, `AGENTS.md`) is prompt-level context, and
  a leash in the mention (Law 1) is best-effort, not enforcement. The enforceable
  controls are GitHub-side — branch protection/rulesets and app/repository
  write permissions.
- **Copilot and Devin ingest `CLAUDE.md` directly** (Copilot's coding agent
  lists it first-party; Devin via Knowledge; CodeRabbit can read it too, as
  review guidelines via `knowledge_base.code_guidelines` in
  `.coderabbit.yaml`) — standing laws merged into `CLAUDE.md` bind those
  agents automatically. Two per-feature caveats: Copilot **code review** reads
  `AGENTS.md` but not `CLAUDE.md` (it still reads
  `.github/copilot-instructions.md` and path instructions), and for the coding
  agent the nearest `AGENTS.md` **takes precedence** over a root `CLAUDE.md`.
  Outside github.com the support matrix narrows: VS Code Chat honors
  `AGENTS.md` as the only agent-instruction filename (no
  `CLAUDE.md`/`GEMINI.md`); IDE code review reads no agent-instruction file at
  all. Jules reads only `AGENTS.md`. The cost of a real (non-symlink)
  `AGENTS.md` is that it silently replaces `CLAUDE.md` for Copilot's coding
  agent — a `.github/copilot-instructions.md` that points Copilot back at
  `CLAUDE.md` compensates.

## Vendor grimoires

Official references; the grid and `NOTES.md` record observed behavior where
the two differ.

- CodeRabbit: [review commands](https://docs.coderabbit.ai/reference/review-commands) · [configuration](https://docs.coderabbit.ai/reference/configuration) · [learnings](https://docs.coderabbit.ai/knowledge-base/learnings) — `@coderabbitai configuration` prints the repo's live config
- Codex: [GitHub integration](https://learn.chatgpt.com/docs/third-party/github)
- Copilot: [code review](https://docs.github.com/en/copilot/concepts/agents/code-review) · [changing existing PRs](https://docs.github.com/en/copilot/how-tos/use-copilot-agents/cloud-agent/make-changes-to-an-existing-pr) · [instruction-file support matrix](https://docs.github.com/en/copilot/reference/custom-instructions-support)
- Devin: [GitHub integration](https://docs.devin.ai/integrations/gh) (commit authoring modes) · [Knowledge](https://docs.devin.ai/product-guides/knowledge)
- OpenHands: [GitHub cloud](https://docs.openhands.dev/openhands/usage/cloud/github-installation) · [repo microagents](https://docs.openhands.dev/modules/usage/prompting/microagents-repo)
- Jules: [running tasks](https://jules.google/docs/running-tasks/) · [usage limits](https://jules.google/docs/usage-limits/) · [changelog](https://jules.google/docs/changelog/)
- Claude Code: [Code Review](https://code.claude.com/docs/en/code-review) (managed; admin settings at claude.ai/admin-settings/claude-code)
- Dependabot: [comment commands](https://docs.github.com/en/code-security/reference/supply-chain-security/dependabot-pull-request-comment-commands)

## Notes and connecting

The dated evidence behind the grid lives in `NOTES.md` — read it when a grid
row surprises you, and append new dated observations there rather than editing
the grid; the grid changes only when behavior does.
`references/connecting.md` is the per-agent checklist for getting a summon to
land at all — what to install, how to verify, and what "not connected" looks
like (perfect silence).

Adopting this spellbook in another repo? Take the grid, the laws, and the
vendor links — then grow your own notes.
