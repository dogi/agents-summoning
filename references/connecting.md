# Connecting — getting each agent to answer at all

A summon into a repo where the agent isn't installed fails **perfectly
silently** — no error, no reaction, indistinguishable from an outage
(field-tested 2026-08-21: five agents summoned on a personal repo before
activation; not one acknowledged; all five answered within minutes of the apps
being granted). So before Law 5's other diagnoses — quota, ownership, outage —
walk this checklist. Two facts frame all of it:

- **Org-level installs don't cover personal repos.** Each personal repo needs
  its own grant, per app.
- **The summon posts under the connecting user's account** when cast through an
  integration (e.g. Claude Code's GitHub tooling) — which is what satisfies
  Copilot's write-access rule, and why "who is the mentioning user" matters for
  every row below.

## Per-agent checklist

| Agent | Connect | Verify | Not connected looks like |
|---|---|---|---|
| **CodeRabbit** | install the [GitHub App](https://github.com/apps/coderabbitai) and grant this repo (free OSS tier exists; Pro trials show as "Plan: Pro Plus") | `@coderabbitai help` answers in ~1 min; a fresh PR gets a walkthrough comment | total silence |
| **Codex** | two lanes: **reviews** need the repo enabled in [Codex cloud settings](https://chatgpt.com/codex/cloud/settings); **chat asks** (`@codex <ask>`) additionally need the mentioning user's ChatGPT account connected to GitHub | `@codex review` → 👀 in seconds | reviews: silence · chat: a reply telling you to "create a Codex account and connect to github" (2026-08-21) |
| **Copilot** | Copilot coding agent enabled for the account; the mentioning user needs write access. Same-repository PRs only; **issue mentions go silent** (2026-08-21) — on issues, assign it instead | `@copilot <ask>` on a PR → 👀 then an answer in ~30 s | silence |
| **Devin** | install the Devin GitHub app and enable the repo at [app.devin.ai](https://app.devin.ai); **Devin Review is a separate activation** on top of the app | `/devin review` → "Starting Devin Review" in seconds (the slash form tolerates trailing text; bare `@devin review` is brittle with any extra text); `@devin <ask>` → session link in ~10 s | silence — and once connected, review findings hide behind the web UI: ask a session to relay them |
| **OpenHands** | install the [OpenHands Cloud app](https://app.all-hands.dev) and grant the repo | any mention → "I'm on it!" + session link in ~10 s | silence |
| **Jules** | connect the repo in the [Jules app](https://jules.google). Mentions do **not** summon it on either surface — silent on a PR and on an issue in the same test (2026-08-21); the `jules` label on an issue is the only summon that worked from GitHub in this test (the Jules app itself is the other surface), and it means "implement this issue". On a PR it **owns**, comments land — 👀, then follow-up commits | apply the `jules` label to an issue you actually want implemented → `google-labs-jules[bot]` acks on the issue within seconds, PR within minutes | silence |
| ↳ confirmed 2026-08-21 on this repo | issue #3 labeled `jules` → bot ack in 6 s, `jules-*` branch + [PR #4](https://github.com/dogi/agents-summoning/pull/4) in ~3 min. Audit by the **`Co-authored-by: google-labs-jules[bot]` trailer + task-ID suffix** — author/committer/PR-author all read as the human and the branch prefix is not dependable (`jules-*` here, bare `<slug>-<taskID>` on myplanet). Commit was empty; receipts in `NOTES.md` | repeat on a fresh issue | a Jules branch/PR appears within minutes |
| **Claude Code** | sessions via [claude.ai/code](https://claude.ai/code) with the GitHub connector granted this repo; managed `@claude review` needs enabling **and funding** at claude.ai/admin-settings/claude-code | a session can read and push; for managed review check the admin page before summoning (Law 7) | silence, or a "review skipped" reply at the spend limit |
| **Dependabot** | commit `.github/dependabot.yml` | scheduled PRs appear on the configured cadence | no PRs, ever |

## The connecting test

A repeatable probe run that turns the checklist above into a verdict. Cast it
on an **open PR you own**: one probe per agent, **each in its own comment**
(Law 8), every doer leashed — a passing run costs five comments and five
minutes and consumes no paid reviews (`@coderabbitai help` answers without
spending a review; Claude Code's paid reviewer is deliberately not probed).
The leashes make a clean run push nothing — but a leash is a request, not a
restraint (Law 2), and the triage table below anticipates exactly that break:
on a branch that can't absorb a surprise commit, have branch protection in
place before probing the doers.

> ⚠️ **A doer probe is a summon, not a ping.** The OpenHands and Copilot
> probes below don't just confirm connectivity — they **spawn real doer
> sessions** (OpenHands one per mention, per Law 4). Field-tested 2026-08-21:
> the `@openhands are you connected?` probe on PR #4 and on issue #3 each
> summoned a separate OpenHands session (distinct conversation IDs) — the
> summoner summoned itself. The leash held both times, but a leash is a
> request, not a restraint (Law 2), so run the doer probes on a branch that
> can absorb a surprise commit and expect a session per doer probe, not just
> a comment back. (See `NOTES.md` → "The connection probe IS the summon".)

| # | Probe (post verbatim, one comment each) | Connected looks like |
|---|---|---|
| 1 | `@coderabbitai help` | command-list reply within ~1 min |
| 2 | `@codex review` | 👀 within ~1 min; a review or 👍 follows |
| 3 | `@copilot are you connected? Comment only, do not push.` | 👀 in seconds, answer in ~30 s–2 min |
| 4 | `/devin review` | "Starting Devin Review" within ~1 min |
| 5 | `@openhands are you connected? Comment only — do not push anything.` | "I'm on it!" + session link in ~10 s, then an answer comment |

Close the window at **5 minutes** — every connected agent's first reaction
landed well inside that in the field test (2026-08-21). Three agents are
deliberately **not probeable by comment**: Jules (mentions are dead on every
surface, and its only summon — the `jules` label — means "implement this
issue"; verify in the Jules app instead), Claude Code's managed review (a
probe bills real money — Law 7; check claude.ai/admin-settings/claude-code),
and Dependabot (check `.github/dependabot.yml` and the repo's Insights tab).

### Triage

| Outcome inside the window | Diagnosis | Fix |
|---|---|---|
| Ack or answer | Connected ✔ | Record the baseline (below) |
| Perfect silence | Not installed, or installed for the org but not granted this repo — personal repos need their own grant | The checklist above; re-probe once after granting |
| Silence, but the app *is* installed | Rate limit (CodeRabbit posts banners — look for them), a brittle trigger form (Devin's `@`-review), or a surface the agent doesn't serve | The surfaces table in `SKILL.md`; re-probe with the exact form above, then Law 5 |
| Loud error ("Failed to start…", "unexpected error starting the job") | Outage — the kind that announces itself | Retry the identical probe once, later; never reword (Law 5) |
| A signup/connect bounce (Codex: "create a Codex account and connect to github") | The repo lane is connected; the *asker's* account is not | Connect the account it links; the review lane works meanwhile |
| The doer pushed despite the leash | Connected but ungoverned | Law 2 — branch protection and app write permissions, not better phrasing |

### Record the baseline

A passing probe per agent is worth a dated line in `NOTES.md` — what fired,
how fast, under what identity — so the next silence can be compared against a
known-good baseline instead of guesswork.

### Loading the spellbook into Jules

Jules runs every task in a fresh VM that clones the target repo, so the skill
has to be committed there and initialized before the task starts.

1. Add this repo as a submodule — the same path OpenHands and Copilot read:

```bash
git submodule add -b main https://github.com/dogi/agents-summoning.git .agents/skills/agents-summoning
```

2. Bridge it from the target repo's root `AGENTS.md`, the only instruction
   file Jules reads — it ignores `CLAUDE.md`. The submodule on its own is not
   a bridge:

```markdown
See `.agents/skills/agents-summoning/SKILL.md` for agent summoning rules.
```

3. Initialize the submodule from the **Initial Setup** script — Jules's
   counterpart to `.openhands/setup.sh` — configured in the Jules web UI at
   *Codebases → (repo) → Configuration → Initial Setup*
   ([docs](https://jules.google/docs/environment/)). A fresh task clone leaves
   the submodule directory empty, and no file committed to the repo can fix
   that:

```bash
git submodule update --init --recursive
```

4. Start a task with the `jules` label on an issue, or from the Jules web UI.
   A mention does not start one.

Without an Initial Setup script, Jules "will also refer to agents.md or your
readme.md file for hints to setup an environment on the fly"
([docs](https://jules.google/docs/environment/)) — hints for building the
environment, not a substitute for the files being present.

**Steering a running task.** Under Reactive Mode — enabled on the account
tested here — Jules acts only on comments that mention it
([changelog](https://jules.google/docs/changelog/2025-09-23/)), so a comment
without a mention proves nothing: a submitted review whose inline comment
wrote "jules" without the `@`, and a bare `/jules`, both drew no reaction,
exactly as that rule predicts. The mentions are the finding. On this repo on
2026-08-23, a mention **on the PR** was read — 👀 inside a minute — and
answered with a push that replayed the task's original commit, same tree and
same message; even "create an empty file at the repo root" came back that way,
and where someone had pushed to the branch meanwhile, the replay restored the
task's snapshot and read as a revert. A mention **on the originating issue**,
after the task had started, drew nothing at all. So no GitHub comment changed
what the running task produced — but that is a limit of the GitHub surfaces,
not of Jules: the docs describe task chat in the Jules web UI, where feedback
can make it replan ([docs](https://jules.google/docs/running-tasks/)), which
was not tested here. Steer from the chat, or close the task and file a sharper
issue.

**What the task reported about itself.** Asked in the issue to answer in the PR
description and label each claim, the 2026-08-23 task reported: submodules
arrive uninitialized in the task VM, so the Initial Setup script is the only
place that can populate them; a root `AGENTS.md` must name
`.agents/skills/agents-summoning/SKILL.md` explicitly or the skill is never
read; and the documented fallback — Jules consulting `AGENTS.md` or `README.md`
for environment hints — builds an environment but cannot restore files that an
uninitialized submodule left absent before the workspace is parsed. Treat these
as *reported*, not measured. The same answer labelled as its own observations
several findings that had been handed to it in the issue body, and claimed
`AGENTS.md` is read "at the repo root or subdirectories" where the docs
describe the root only. An earlier run, given no findings to copy, correctly
marked the nesting question unverified. Ask an agent what it observed *before*
telling it what you found — otherwise you get your own notes back wearing its
authority.
