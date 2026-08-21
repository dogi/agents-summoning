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
| **Jules** | connect the repo in the [Jules app](https://jules.google). Mentions do **not** summon it on either surface — silent on a PR and on an issue in the same test (2026-08-21); the `jules` label on an issue is the only reliable summon, and it means "implement this issue" | apply the `jules` label to an issue you actually want implemented | silence |
| **Claude Code** | sessions via [claude.ai/code](https://claude.ai/code) with the GitHub connector granted this repo; managed `@claude review` needs enabling **and funding** at claude.ai/admin-settings/claude-code | a session can read and push; for managed review check the admin page before summoning (Law 7) | silence, or a "review skipped" reply at the spend limit |
| **Dependabot** | commit `.github/dependabot.yml` | scheduled PRs appear on the configured cadence | no PRs, ever |

## After connecting

The first successful summon per agent is worth a dated line in `NOTES.md` —
what fired, how fast, under what identity — so the next silence can be compared
against a known-good baseline instead of guesswork.
