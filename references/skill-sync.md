# The Skill Sync (one repo, every agent)

Each shared skill (`merge-prepping`, `kotlin-importing`, `branch-overtaking`,
`agents-summoning` itself) is maintained **once** in its own repo, and Claude
Code, OpenHands, and Copilot all load it from that single source of truth:

- **OpenHands** auto-loads from `.agents/skills/<name>/SKILL.md` — submodules
  bootstrapped by `.openhands/setup.sh` before skill discovery runs.
- **Claude Code** loads the plugins through the marketplaces declared in the
  target repo's `.claude/settings.json`.
- **Copilot** picks the skills up via its instruction file.

One revision caveat: OpenHands and Copilot see the **pinned submodule commit**,
while Claude Code's marketplace fetch tracks the skill repo's **main tip** —
the two match only while the pin is current, so bump the submodule after every
skill-repo merge:

```bash
git submodule update --remote --checkout -- .agents/skills/<name>
```

then commit the gitlink — `--checkout` forces the mode the pin needs even where
the target repo configures `merge`/`rebase`: the submodule sits on a
detached HEAD. And the load is best-effort on fresh offline sessions:
`.openhands/setup.sh` is deliberately non-fatal, so an uninitialized submodule
means that skill simply doesn't load that session — run
`git submodule update --init --recursive` once connectivity is back.

## Why this works

- **Claude Code** reads `.claude/settings.json` → fetches plugin marketplaces
  from GitHub → follows internal symlinks to find `SKILL.md`.
- **OpenHands** reads `.agents/skills/<name>/SKILL.md` → auto-loads on every
  session. It does **not** read `.claude/settings.json` or fetch marketplaces.
- A **git submodule** at `.agents/skills/<name>/` makes the files physically
  present after `git submodule update --init`, on any machine or VM.
- **Internal symlinks** (inside the skill repo) normally resolve on every
  checkout, since target and link travel together. If one doesn't, check the
  stored target with `readlink -n <link> | od -c` (`-n` matters: without it,
  `readlink` appends its own newline and every target looks tainted) — a
  trailing `\n` (from
  generating the link via echo/printf instead of `ln -s`) makes it point at a
  filename ending in an invisible newline. Recreate with `ln -s`. Field-tested
  2026-08-09 in the sister repo `dogi/kotlin-importing` (not a file here): its
  `kotlin-importing.py` plugin symlink shipped with exactly this defect while
  the adjacent `SKILL.md` link was clean.

## Skill-authoring traps (field-tested)

Both were caught live by OpenHands verifying a skill in the target environment
(2026-08-09), after discovery had silently dropped the skill:

- A too-generic frontmatter `name:` (it was `title`) gets the skill dropped by
  discovery. Use a distinctive gerund (`prepping`, `importing`, `overtaking`,
  `summoning`).
- An unquoted `description:` containing a colon (`scope:` in that case) is
  invalid YAML and also drops the skill silently. Quote the description.
